# Write up for super_memory

## Overview

For this challenge we are trying to find the flag is a simple CLI game, which we are given the C source code for.

## Solution

### Flag 1

Looking at the code for game 1 we find the we need to get to a spot containing "1". However, playing the game we find this spot impossible to get to.
Looking into the source code, we see the world size is an unsigned character. 
Thus by going to beyond the left border it will overflow and put us at the far right border.

In the game, by following the standard path, then once at the highest point, jumping to the left we can land on the clouds. 
We can follow the clouds to the left to get over the border on the left and go beyond the edge. This puts us at "-1" which is interpretted as "256" 
and puts us at the end of the level, where the character "1" is allowing us to get the flag.

### Flag 2

Looking at the code for game 2 we find that if we have exactly 1650549605 dollors it will let us through a gate, 
which will allow us to reach the spot with the flag. In addition we see that our inventory is vulnerable to overflowing into our money.
Thus by overflowing specifc characters we should be able to get to 1650549605 dollars.

Using the simple python code below, we can find what characters are needed to overflow the dollars to the correct amount.
```
key = 1650549605
print(key.to_bytes(4, 'little'))
```
This outputs get "ecab"; however, we can only put a-d into our inventory. Thus through trial and error we find
```
key = 1650549605 - 1
print(key.to_bytes(4, 'little'))
```
which outputs "dcab". Using this with the fact that the money items increment our dollars by one, we can get to 1650549605 dollars.

Thus by getting all of the keys like normal, but jumping over the dollar in front of the "b" key we can begin our overflow. 
By overflowing our inventory with the "dcab" keys in that order, it overwrites our dollars with 1650549604. 
Then when we pick up the last dollar we get 1650549605. Now we simply go to the gate and collect the flag.





