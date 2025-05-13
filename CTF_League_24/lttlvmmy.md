# Write up for Cookies with a Fork

## Overview

For this challenge we are doing some reverse engineering. We have a VM implementation of the LC-3 ISA. The
ISA specification can be found here: https://www.jmeiners.com/lc3-vm/supplies/lc3-isa.pdf


## Solution

### Flag 1

After decompiling the VM binary with Ghidra, we find that this implementation extends trap to include the trap

```0x26```

which opens and reads from a file with the filename string starting at ```R0```. This will be important later.

From the docker file, we find that there is a file ```flag.txt``` that is loaded to the same directory as the VM application.
From the ```hello_world``` sample program, we find that this VM had a weird endianness for strings where each pair of ascii hex has the 2 hex values switched.
For example, the string ```"ab"``` would be written as the hex ```62 61``` in the VM's code.

So we will need to use trap ```0x26``` to open the ```flag.txt``` file and then
trap ```0x25``` to halt the program. 
Thus we can manually write the machine code to:

* Set the origin of our program to ```0x3000```, which is the memory available for user programs
in the LC-3 ISA. 

* Load the effective address ```2``` to ```R0``` (so ```PC+2->R0```, the address of our ```flag.txt``` string).

* Run trap ```0x26``` to read the ```flag.txt``` file.

* Run trap ```0x25``` to halt the program.

* Store the flag.txt string in program memory.

A solution script implementing this is below

```Python
#!/usr/bin/env python3

from pwn import *
context.terminal = ['tmux', 'split']

p = remote("REDACTED", REDACTED)
# p = process("./vm")

print(p.recvline())
# Note endianness is messed up, each hex pair is flipped (abcd -> badc)
payload = bytes.fromhex(
    '30 00'  # .ORIG 0x3000;
    'E0 02'  # LEA R0, #2; (load effective address 2 to R0 (PC+2->R0))
    'F0 26'  # TRAP x26; (open file with string at R0 pointer)
    'F0 25'  # TRAP x25; (halt / returns 0 to stop program run by VM)
        # The file to open (note endianness flips hex pairs)
    '6C 66'  # 'f''l' (actually 'l''f', but endianess is weird);
    '67 61'  # 'a''g';
    '74 2E'  # '.' 't';
    '74 78'  # 'x''t';
    '00 00'  # terminating \0 to end string
)

# Send buffer size to be of sufficient space 
# (could be tighter with p.sendline(str(len(payoload)).encode()) )
p.sendline(b"300")
p.send(payload)

p.interactive()
```



