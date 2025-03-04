# Write up for Mobhit

## Overview

For this challenge we are doing some obfuscated reverse engineering. We have an obsufacted (and defanged) malware file, a transmission, 
and a transmission timestamp. 

## Solution

### Flag 1

First, we want to decode the obsufacted malware file. To do so, we can replace all of the ```exec``` statements with ```print``` statements.
This outputs the code one layer decrypted. Doing that again gets a file of mostly code. We can run print on the obsfucated functions to find that 
```ad(ac(REDACTED))``` becomes a list of integers. Repeating this methodology, we find that ```af``` and ```ag``` are used as to create a encoding mapping
from one character to another. Then we find that ```ai-aq``` are the import statements:
```Python
#ai = "<module 'shutil' from '/usr/lib64/python3.13/shutil.py'>"
#aj = "<module 'hashlib' from '/usr/lib64/python3.13/hashlib.py'>"
#ak = "<module 'struct' from '/usr/lib64/python3.13/struct.py'>"
#al = "<module 'base64' from '/usr/lib64/python3.13/base64.py'>"
#am = "<module 'random' from '/usr/lib64/python3.13/random.py'>"
#an = "<module 'socket' from '/usr/lib64/python3.13/socket.py'>"
#ao = "<module 'time' (built-in)>"
#ap = "<module 'hmac' from '/usr/lib64/python3.13/hmac.py'>"
#aq = "<class 'cryptography.fernet.Fernet'>"
```
we will use these later for Flag 2. Then, we can print ```aw``` to get an encoded string. Messing around with Cyber Chef, we find that it can be decoded
with a ROT13 decoding and then a base 32 decoding. This can also be found by looking at the ```af-ag``` translation and realizing 
it rotates the alphabet by 13 characters. This gets us the first flag.


### Flag 2

For the second flag, we have a encrypted string and a time stamp. This is likely encoded as a Time-based One-Time Password.
We can edit the ```ar``` function and replace ```ao.time()``` with the ```REDACTED.REDACTED``` floating point representation 
of the time in the Unix time format from the info.txt file associated with the challenge. Then we can edit the ```au``` lambda function
from ```.encrypt()``` to ```.decrypt()``` to decrypt the transmission using the timestamp. Finally running 
```Python
print(au("REDACTED_TRANSMISSION"))
```
we get the second flag.

