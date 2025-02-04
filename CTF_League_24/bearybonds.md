# Write up for bearybonds

## Overview

For this challenge we are doing some side channel timing accounts. There is a password that we are trying to figure out, 
where the validation takes longer with for each correct sequential character. 

## Solution

### Flag 1

For this, we make a script that goes through each number 0-9, appends it to our current found pin, 
find the addition that takes the longest, then goes to the next character. From log information given
by the program, we find that the first character is "1". Next we use the below script to find the pin,
which is 123457801337.
```
#!/usr/bin/python

from pwn import *
import time
context.terminal = ['tmux', 'split']

# p = remote("1314", 1314)

def timepin(attempt):
    p = remote("1314", 1314)
    t1 = time.time()
    p.sendline(attempt)
    p.recvall()
    t2 = time.time()
    # p.interactive()
    p.close()
    return t2-t1



pin = b'1'
for _ in range(11):
    attempts = []
    for i in range(10):
        avg = []
        for j in range(1):
            temptime = timepin(pin + str(i).encode())
            avg += [temptime]
        attempts += [sum(avg)/len(avg)]
    print(attempts)
    pin += bytes(str(attempts.index(max(attempts))),'utf-8')
    print(pin)
```

When we input the pin, we are given back the flag encrypted in a one time pad. But it also tells us what time the program was run at.
We can use much of the leaked code, and this information to decrypt the flag, as it encrypts using srand, which is deterministic from the
time the program is run at (more specifically srand(null) is). The below script decrypts the one time pad from a program run 
at the time encoded as "1738639657".
```
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <time.h>

#define FLAG_LENGTH 16  // Length of encrypted flag given excluding "osu{}"

// Functions adapted from leaked.c
void generate_otp(char *otp, size_t length) {
    printf("%ld:LOG: generating secure key\n", time(NULL));
    srand(1738639657);  // Fixed seed ensures same OTP is generated
    for (int i = 0; i < length; i++) {
        otp[i] = (char)(rand() % 26) + 'a';
    }
    otp[length] = '\0';  // Null-terminate the string
}

void decrypt_bond(char *otp, char *ciphertext, size_t length) {
    printf("Decrypted text: osu{");
    for (int i = 0; i < length; i++) {
        printf("%c", ((ciphertext[i] - 'a') - (otp[i] - 'a') + 26) % 26 + 'a');
    }
    printf("}\n");
}

int main(void) {
    char otp[FLAG_LENGTH + 1];  // Set space for OTP
    char ciphertext[FLAG_LENGTH + 1] = "heuuohivnnqliqos";  // Encrypted flag given from server

    generate_otp(otp, FLAG_LENGTH);
    printf("Generated OTP: %s\n", otp);
    printf("Ciphertext: %s\n", ciphertext);
    decrypt_bond(otp, ciphertext, FLAG_LENGTH);

    return 0;
}
```
Which prints the flag!


