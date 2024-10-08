# Write up for Elevator.

## Overview

This challenge works like an intro to the linux command line. You navigate through multiple users with different levels of permissions, 
utilizing a variety of command line tools as you do so.

## Solution

This challenge is split up into 8 levels (0-7). Each level will follow a similar workflow. For each level we'll start with ```ls``` to see 
our current directory, then ```cat README``` to get a hint for the current level.
You start by SSHing into the challenge. This puts you at level 0.

### Level 0
Once logged in we see we're logged in as a user titled level_0_-----@-------.
From here we run
```~ls```
The output is
```README  creds_level1```
Next we run
```~cat README```
The output is
```Have you seen my cat? Once you find credentials, log in with su --login usernamehere```
(From hereon we will put commands and outputs in the same box, with ~ denoting a commnd)
Next we checkout the file structure
```
~cd creds_level1/
~ls
creds_level1.txt
```
From here we checkout what's in creds_level1.txt
```
~cat creds_level1.txt
next user: level1_-----
password: -----
```
Ah ha, we've found the credentials for level 1!
Now as the README told us to, we log in with
```
~su --login level1_-----
password: -----
```

### Level 1
Now we're into logged in to level 1.
```
~ls
README  r5v73zgq  vf9g559u
~cat README
99% of gamblers give up right before they find creds
hint: find? grep? linux gives u tons of tools to run around directories like you're a little monkey and they're trees
```
Going into either of the directories shows a bunch of other directories, whic hwe don't want to look through.
Thus, we use the format of the previous credentials and grep to find the credentials.
In order to learn more about grep, I ran ```man grep```.
```
~grep -r -l "password" .
./vf9g559u/wpjqpl40/cvooihcw/y1gx9zt7/h6ra1hus/vhzlzi4f/creds_level2.txt
~cat ./vf9g559u/wpjqpl40/cvooihcw/y1gx9zt7/h6ra1hus/vhzlzi4f/creds_level2.txt
next user: level2_------
password: -----
```
The -r option makes it search recursively through the directory and subdirectories.
The -l option makes it output the filename of the file it finds the search query in.
The . at the end starts the search at the current directory.
From there we login in to level 2.

### Level 2

```
~ls
README  creds_level3.txt
~cat README
rocky raccoon checked into his room...
hint: grep.
```
Using cat on creds_level3.txt outputs a large file I'd rather not look through, so we'll use grep again.
```
~grep -r -h "user" .
password: -------
~grep -r -h "user" .
next user: level3_------
```
The -h option makes grep output the line that it finds the search query on.
From here we can login to level 3.

### Level 3

```
~ls
README  creds_level4.sh
~cat README
Can you get creds_level4.sh to run?
```
From here we cat the file to see what's going on inside
```
~cat creds_level4.sh
#!/bin/bash
e="U2FsdGVkX18z5RushwHZtfn75hVSFdJoW/v45hplpvplgMrWhTEl6DHtASRVLZmi
CJqEhy5lAk0ZLdWAd3PnDw=="
k=$(echo 'Z29pbmd0aGVsb25nd2F5' | base64 --decode)
echo "=====creds:====="
echo "$e" | openssl enc -d -aes-256-cbc -a -k "$k"
echo ""
echo "================"
```
Seeing nothing dangerous, we run:
```
~chmod +x ./creds_level4.sh
~./creds_level4.sh
=====creds:=====
*** WARNING : deprecated key derivation used.
Using -iter or -pbkdf2 would be better.
next user: level4_------ password: ------
```
To turn the shell script into an executable file, and then we run the file.
From there, we log in to level 4.

### Level 4

```
~ls
README
~cat README
Have you seen my file?
hint: ls has a lot of options :)
```
From the hint, it's likely there's hidden files, so we use ```ls -a``` to see all files and directories.
```
~ls -a
.  ..  .bash_logout  .bashrc  .hidden_creds_level5  .profile  README
~cd ./.hidden_creds_level5/
~ls -a
.  ..  .creds_level5.txt
~cat ./.creds_level5.txt
next user: level5_-----
password: ------
```
From here, we login to level 5.

### Level 5

```
~ls
README  'look in here'
~cat README
filename is weird!
can u read it anyway?
```
From here we can see the folder is named weirdly, but by copying it exactly, we can still navigate it like normal.
```
~cd ./'look in here'/
~ls
'creds in here!'
~cat ./'creds in here!'
next user: level6_-----
password: ------
```
From here we can login to level 6

### Level 6

As soon as we log in to level 6 we find ourselves in vim.
We can use ```:qa``` to quit vim while saving (even though there weren't any modifications)
From there we can run our normal workflow.
```
~ls
creds_level7.txt
~cat creds_level7.txt
next user: level7_------
password: ------
```
From here we can log in to level 7.

### Level 7

```
~ls
README
~cat README
What's in /flag.txt?
```
From here we simply run: 
```
~cat /flag.txt
OSU{--------------}
```
and we get the flag!













