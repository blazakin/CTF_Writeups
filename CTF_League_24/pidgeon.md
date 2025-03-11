# Write up for Pidgeon

## Overview

For this challenge we're doing some cryptography. There's a new protocol over messenger pidgeon. Unfortunately, 
we overheard just a little bit too much information.

## Solution

### Flag 1

For this challenge, we have the code, so we can see our group is modulus $n = 323509627153465998556191522930492845687$, with a generator $g=5$.
Then, in the challenge we are given public values $A$ and $B$, which change each time the program is run, so they must be dynamically read in.
Then, they allow us to send a message which will be XORed with key 2, denoted $K_2$ and sent back to us. By sending all zeros, we can 
get key 2. Next, as between key 1, 2, and 3 they just iterate the private $a$ and $b$, we can use some algebra to find key 1 and 3 from 
$n, g, A, B$, and $K_2$.

Specifically, as $K_{2} =$ $g^{ (a+1)(b+1) } \mod n = g^{ab}g^ag^bg \mod n = k_1 A B g \mod n$,
we find that $K_1 = \frac{K_2}{A B g} \mod n$.

Similarly, we find that $K_3 = K_1 A^2 B^2 g^3 \mod n$

With these two keys, we can decrypt the first and second part of the flag. Below is a solution script. Most of it is parsing input.

```Python
#!/usr/bin/env python3

from pwn import *
from Crypto.Cipher import AES
import sys
import ast

context.terminal = ['tmux', 'splitw', '-h']

p = remote("REDACTED", REDACTED)
# p = process("./chal.py")

# size of cyclic group / modulus of group
n = 323509627153465998556191522930492845687
# generator of group
g = 5

# Get A
p.recvuntil("Quoth the pigeon, \n")
A = int(p.recvline().strip())
print("A")
print(A)

# Get B
p.sendline()
p.sendline()
p.recvuntil("Qůirm leans over and says to the pigeon:\n")
B = int(p.recvline().strip())
print("B")
print(B)

# Get encrypted part 1 of the flag 
p.sendline()
p.recvuntil("carrying a message: \n")
f1_enc = ast.literal_eval(p.recvline().strip().decode('ascii'))
print("f1_enc")
print(f1_enc)

# Get K2
# Send just zeros so that K2 will be sent back to us
sent_letter = b"\x00"*16
p.send(sent_letter)
p.recvuntil("you can barely hear the phrase\n")
K2 = int.from_bytes(ast.literal_eval(p.recvline().strip().decode('ascii')), 'big')
print("K2")
print(K2)

# Get encrypted part 2 of the flag 
p.sendline()
p.recvuntil("(continue)")
f2_enc = ast.literal_eval(p.recvline().strip().decode('ascii'))
print("f2_enc")
print(f2_enc)

# Calculate K1 and K3
K1 = ((K2*pow((A*B*g), -1, n)) % n)
K3 = ((K1*pow(A,2,n)*pow(B,2,n)*pow(g,4,n)) % n)

# Convert keys to proper AES keys
def to_aes_key(key_i):
    result = key_i.to_bytes(n.bit_length() // 8, "big")
    return result[-16:]

# Decrypt flags
cipher1 = AES.new(to_aes_key(K1), AES.MODE_ECB)
cipher3 = AES.new(to_aes_key(K3), AES.MODE_ECB)
print(cipher1.decrypt(f1_enc))
print(cipher3.decrypt(f2_enc))

p.interactive()
```

 
