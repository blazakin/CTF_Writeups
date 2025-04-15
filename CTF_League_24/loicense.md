# Write up for Loicense

## Overview

For this challenge we are doing some reverse engineering. There is an executable that requires a "license" before it gives you the flag, 
we want to reverse engineer their validation function to get a valid license.

## Solution

### Flag 1

Using Ghidra to decompile the binary, we find the "validate" function which takes a buffer of up to 4096 bytes from main, sums the data in the buffer and
tries to execute the data in the buffer at an offeset of the sum of the data in the buffer. It then checks if the return value from that execution equals 
the sum of the buffer casted to a char.

First, note that chars will overflow 256 to 0. So we can have our assembled machine code return 0 and have our buffer sum to a multiple of 256.
We can write and assemble some simple assemply that will just call the syscall exit with status code 0, such as ```mov eax, 1; mov edx, 0x0; int 0x80```.
Then we can pad our payload at the back to sum to a multiple of 256 and pad the front with the sum of our payload in zero bytes (which won't impact the 
sum of our payload). Thereby creating a payload that when executing the machnine code offset by the sum of the payload returns the same value as the sum
of the payload cast to a char. This payload gets us the flag. The code below is a solution script our team made.
```Python
#!/usr/bin/python

from pwn import *
context.terminal = ['tmux', 'split']

p = remote("REDACTED", REDACTED)
p.recvline()

code = asm(f'mov eax, 1; mov edx, 0x0; int 0x80')
code_sum = sum(code)
end_padding = 256 - (code_sum % 256)
payload = code + p64(end_padding)

p.send((b'\0' * sum(payload)) + payload)

p.interactive()
```
