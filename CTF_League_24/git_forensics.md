# Write up for git_forensics

## Overview

For this challenge, we're going through git logs to find insecurely stored credentials in a series of 7 levels of privilege escalation. 
Flag 1 is at level 5 and flag 2 is at level 7.

## Solution

### Level 0

Running ```git status``` we find the file containing the credentials has been deleted, so we use ```git restore``` to restore it.

### Level 1

Running ```git reflog``` we find the credentials directly stored in the commit message

### Level 2

Running ```git reflog``` again, we see there is a commit where the message says the credentials are stored. 
Using ```git checkout *commit_hash*``` we are able to go to that commit and get the creds from a text file.

### Level 3

Running ```git reflog``` we are able to see a commit that says "next time it won't be this easy", checking out this 
commit with ```git checkout *commit_hash*``` we are able to find the credentials in creds_level2.txt (yes it doesn't match).

### Level 4

Running ```git reflog``` we see a bunch of commits, by running ```git shortlog -sn --all```, we are able to see the two contributers: 
"imacybercriminal" and "xX_liltimmyT_Xx". Then by running ```git log --author=xX_liltimmyT_Xx``` we see the commit made by Timmy.
From there we can checkout that commit with ```git checkout *commit_hash*```. From there we can use ```git blame creds_level2.txt``` 
and find the real credentials commited by Timmy.

### Level 5

Here the flag is directly accessible by ```cat /flag5.txt``` and the next credentials are also accessible by running ```cat creds_level6.txt```.

### Level 6

Running ```git reflog``` we can find the commit in the branch they deleted. Then running ```git checkout *commit_hash*``` we can go to that commit, 
where the creds are availably in a text file.

### Level 7

Here the flag is directly accessible by ```cat /flag7.txt```.






