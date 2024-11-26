# Write up for Yikes

## Overview

For this challenge we are trying to find and decrypt files that were encrypted by a randsomware attack. 
We are given a zip file with a text file in it and a hidden python file that contains the randsomware attack.

## Solution

### Flag 1

Looking in the text file we find the website: 
https://yikes.ctf-league.osusec.org/?id=3

Looking in the attack file, we find the subdirectories the attack is sending data to, which leads us to:
https://yikes.ctf-league.osusec.org/files/3 (where the 3 is the id associated to Timmy).

This gives us Timmy's encrypted files. We also find:
http://yikes.ctf-league.osusec.org/victim/3. 

This gives us Timmy's hostname, which is the first flag, and the date of the attack.

### Flag 2

Looking back at the attack, we find that the secret key that the data is encrypted with is determined by a random number 
in range (1, 1333337), the victim's hostname, and the date of the attack. From this, we can break the encryption by trying all the numbers 
between 1 and 1333337. To know if we decrypted the file we check if the first so many bytes contain 
the expected file signature, or magick bytes. To do this, we create a script:

```
from pathlib import Path
from hashlib import md5
from random import randint
import urllib.parse

hostname = "osu{loose_lips_sink--oh_wait_wrong_metaphor}"
time = "2024-11-22"

def decryptfile(filepath, magic):
    worked = False

    # These byte don't matter, they are just used to create a blank bytearray of proper length
    ans = bytearray(b'\x66\x74\x79\x70\x69\x66\x74\x79\x70\x69\x66\x74\x79\x70\x69\x66\x74\x79\x70\x69')
    with open(filepath, "rb") as f:
        contents = bytearray(f.read())

    for randomThing in range(1333337):
        secretkey = md5((hostname + str(randomThing+1) + time).encode()).digest()
        for i in range(20):
            ans[i] = contents[i] ^ secretkey[i % len(secretkey)]
        if bytearray(magic) in ans:
            print(secretkey)
            worked = True
            break

    if not worked:
        print("failed")
        return
    
    with open(filepath, "rb") as f:
        contents =bytearray(f.read())
    for i in range(len(contents)):
        contents[i] ^= secretkey[i % len(secretkey)]
    dec_path = filepath + ".pdf"
    with open(dec_path, "wb") as f:
        f.write(contents)

    return dec_path

mp4_magic = b'\x66\x74\x79\x70\x69\x73\x6F\x6D'
pdf_magic = b'\x25\x50\x44\x46\x2D'
txt_magic = b'\xEF\xBB\xBF'
print(decryptfile("your-filepath/hoemwork.pdf.ohio", pdf_magic))
```
From here, we get the second flag in the decrypted pdf.

### Flag 3

Using the secret key that decrypted the homework pdf, we use the second part of the script to also decrypt the chat logs.
In here we find:

```
xX_LITTLE_TIMMY_Xx: u free?
xX_jimjam_Xx: yah! already on fortnite no cap
xX_LITTLE_TIMMY_Xx: yay
=======================
xX_LITTLE_TIMMY_Xx: cringe my mom's making me get off, hoemwork or smth
xX_jimjam_Xx: it's so joever
xX_LITTLE_TIMMY_Xx: ya
xX_LITTLE_TIMMY_Xx: cya tomorrow probably..
xX_jimjam_Xx: stay sigma
=======================
xX_LITTLE_TIMMY_Xx: WANT FREE ROBUX? I USED THIS TOOL AND GOT 139487 INSTANTLY
xX_LITTLE_TIMMY_Xx: <attached free_robux.exe>
xX_jimjam_Xx: yo that's so cool!
xX_jimjam_Xx: wait timmy your aura seems off is that u?
xX_jimjam_Xx: timmy?
```

This gives us the username of Timmy's friend: xX_jimjam_Xx.
Manually going through the victim IDs, we find xX_jimjam_Xx's info at:
http://yikes.ctf-league.osusec.org/victim/17

We download their encrypted data and find that it's an encrypted mp4.
We can use the same decryption script as above, but inputtting the mp4 
magick bytes to decrypt the mp4.

Here we get the final flag

One thing to note is that the script above originally only directly checked the first 
(length of magick bytes) bytes of the encrypted file, but was later modified to check for 
a substring in a longer header, which works for both the second and third flag.




