# Write up for Saloon.

## Overview

This challenge has two flags. You interact with a program running on a server, where bytes are sent back and forth. You must convince the robots in
the saloon that you are also a robot. We will be using pwntools to communicate with the server.

## Solutions

A complete solution script is provided at the end.

### Flag 1

First, send the bytestring ```b"y"``` to enter the saloon.
Then you will be send 100 numbers you need to multiply together. Instead of manually calculating each one, we can create a script to parse the 
expression and send the result.
```
for _ in range(100):
    m = p.recvline().split(b'=')[0].strip()
    m = eval(m)
    p.sendline(bytes(str(m), 'utf-8'))
```
Next you will be sent the bytes for an image. You can directly write those bytes to a file, and you will see the first flag.
```
with open("./img.jpg", "wb") as f:
    f.write(i)
```

### Flag 2

After you send the first flag, then you will get 100 hex strings you need to convert to a base 10 integer and send back.
We can again make a script to automatically do this.
```
for _ in range(100):
    a = p.recvline().split(b" ")
    p.sendline(bytes(str(int(a[0].decode("utf-8"), 16)), 'utf-8'))
```
After this you will get a one time pad key and message that you xor together to get the second flag.
```
key = p.recvline().strip()
p.recvline()
mes = p.recvline().strip()
print(bytes(a ^ b for a, b in zip(key, mes)))
```

### Solution Script

```
from pwn import *
p = remote("REMOTE_ADDRESS", PORT)

# Enter saloon
p.sendline(b"y")

# Calculate 100 expressions
p.recvuntil(b"math\n")
for _ in range(100):
    m = p.recvline().split(b'=')[0].strip()
    m = eval(m)
    p.sendline(bytes(str(m), 'utf-8'))

# Write image to file    
p.recvuntil(b"here you go!\n")
i = p.recvuntil(b"What's")
i = i[:-8]
with open("./img.jpg", "wb") as f:
    f.write(i)
p.recvline()
p.sendline(b"osu{FLAG1}")

# Convert 100 hex string to decimal integers
p.recvuntil(b"robot bar\n")
for _ in range(100):
    a = p.recvline().split(b" ")
    p.sendline(bytes(str(int(a[0].decode("utf-8"), 16)), 'utf-8'))

# Decode one time pad
p.recvuntil(b" key:\n")
key = p.recvline().strip()
p.recvline()
mes = p.recvline().strip()
print(bytes(a ^ b for a, b in zip(key, mes)))
p.interactive()
```
