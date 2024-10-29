# Write up for revpaperscissors

## Overview

For this challenge we are doing some reverse engineering. We have two executable files for the two flags. 
The files run a program that plays rock paper scissors against you. You must beat the program at rock paper scissors to get the flag.

## Solution

### Flag 1

For flag one, decompiling the file with Ghidra we see the program has hard coded options. 
Specifically it will do rock, scissors, paper, paper, and then scissors. Using this we can simply choose the options to beat those options.
Doing paper, rock, scissors, scissors, rock we will win and get the flag.

### Flag 2

Decompiling the second file with ghidra, we find that this time the program chooses its decision based on a function that uses the name you input. 
While you could reverse engineer the function, because it is a deterministic function, it is easier to brute force it. 
As long as you input the same name, the program will choose the same options. Thus by using the name "aaaaaaaaaa" the series of actions: 
scissors, paper, rock, paper, paper, scissors, scissors, scissors, paper, and then rock will get you the flag.

