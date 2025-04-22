# Write up for Empire

## Overview

For this challenge we are doing some malware, reverse engineering, symlink injection, and bruteforce hash cracking. 
For this challenge the flag is the hacker's password enclosed in ```osu{...}```.

## Solution

### Flag 1

We are given the ransomware website's sourcecode in a password protected zip file. We can use ```zip2john``` and ```john``` to crack the zip file.
We do this with

```
./john-bleeding-jumbo/run/zip2john stolen_files.zip > stolen_files_hash.txt
./john-bleeding-jumbo/run/john stolen_files_hash.txt
```

From this, we get the password used to encrypt the zip files this hacker uses and the source code. We also see that their server uses Python's ```http.server``` 
which is vulnerable to file redirection attacks. Then we can create a zip symlink injection using this password to get the hacker's ```/etc/shadow``` file to
reverse hack the hacker and get their password.

First we create a symbolic link to /etc/shadow with

```ln -s /etc/shadow test```

Then we zip this with the hacker's password

```zip -y -P ZIP_PWD_ALREADY_FOUND test.zip test```

The ```-y``` ensures it keeps the symlink and doesn't zip our ```/etc/shadow```. 
This gives a zip file we can upload to the ransomware site "pay" (which for this ctf doesn't mean much) to have it decrypted, 
and get the hacker's ```/etc/shadow``` file. Note we could've done this on the original encrypted sourcecode file to easily get the source code, 
but we needed the password that encrypted it anyways for our attack.

copying this to a file entitled shadow.txt, we can run

```
./john-bleeding-jumbo/run/john shadow.txt
```

to get the hackers password, which is the flag.


