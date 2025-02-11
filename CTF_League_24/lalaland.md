# Write up for Lalaland

## Overview

For this challenge we are doing some crypto involving decryption and RSA.

## Solution

### Flag 1

For the first flag, we see that they will give use the flag encrypted as a hex string and then will decrypt other stuff 
for us using the same decryption method, but if it's the flag, they won't do it. As they decrpyt stuff multiple times per run,
we can simply put in the flag with the first hex character changed, then again with the last hex character changed and combine the two, 
leaving off the garbagee at the ends formed from our modifications to get the full flag. This works, 
because their encryption algorithm doesn't proerly hash the output space.

### Flag 2

For this program, we want to cause an exception in the program. We can do this by finding two primes p, q 
such that $\phi = (p-1)(q-1)$ is coprime with $e=65537$. We can do this by iterating through the 64 bit primes till we find one such that 
$(p-1)$ % $e = 0$. I used the simple python script below to find 9223372036859527241, which works.
```python
#!/usr/bin/python
import sympy.ntheory as nt

def GenPrime():
    p1 = nt.nextprime(2**63)
    while((p1-1) % 65537 != 0 and p1 < 2**64):
        p1 = nt.nextprime(p1 + 1)
    print(p1)

# GenPrime()
```
Then I can simply send that prime with another 64 bit prime, say 12620510808675587131, and ask it to try to decrypt something and it'll break and give 
us the flag. I used the script below to achieve this.
```python
#!/usr/bin/python

from pwn import *
context.terminal = ['tmux', 'split']

p = remote("REDACTED", REDACTED)

print(p.recvuntil(b"please"))
p.sendline(b"9223372036859527241")
p.sendline(b"12620510808675587131")
p.sendline(b"2")

p.interactive()
```
Then we get the second flag!
