# Write up for Hulk Enterprises.

## Overview

This challenge has two flags. You interact with a program where you enter a username. Your goal is to buffer overflow the stack to get the flags.

## Solutions

### Flag 1

Recompiling the file with Ghidra, we see 3 main functions: main, menu, and get_flag. 
In the menu function, the program allows a user to input text to a 20 character text buffer. 
We can overflow the return address from the text input and change it from main to get_flag.

We need to fill the 20 characters and the 4 bytes for the EBP register.
However, looking at the assembly in Ghidra we see the buffer is actually offset by 24 bytes. 
Thus we need to fill 28 bytes, then put the address for the get_flag function.

The below script uses pwntools to send the bytes to buffer overflow the stack.
```
p.sendline(28*b"A" + p32(0x08049192))
```
After sending this we get the first flag.

### Flag 2

For the second flag, we do a similar thing. However, this time we want to call the menu function with the input ```0xcafed00d``` which gets it 
to output the second flag. Looking at the assembly, we can see the input is behind the return address with a 4 byte buffer. 

The below script uses pwntools to send the bytes to get the second flag.
```
p.sendline(28*b"A" + p32(0x08049234) + 4*b"a" + p32(0xcafed00d))
```
This gets us the second flag.







