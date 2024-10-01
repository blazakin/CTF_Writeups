# Write up for Arcade.

## Overview

This challenge has two flags. You interact with the website: https://arcade.ctf-league.osusec.org/.
They also provide the server code at: https://chal.ctf-league.osusec.org/web/arcade/server.js.

Looking at the website, there are 2 games: breakout and raffle.

## Solutions

### Breakout

Looking at the backend code, it shows that it will give you the flag if your score is greater than or equal to 30. However, when you play the game, the ball moves exceptionally slowly. I have found 3 ways to make this easier.

#### Method 1:
Opening your browser's dev tools, enter:
```score = 30```
Once the game updates again, it will give you the flag

#### Method 2:
You can manually send the score through a query in the url.
Simply go to:
```https://arcade.ctf-league.osusec.org/breakout?score=30```

#### Method 3:
Opening your browser's dev tools, enter:
```
lives = 300
speed = 10
dx = 10
dy = 1
```
This increases your number of lives, and increases your speed to a reasonable speed. Allowing you to simply play breakout to get the flag.

### Raffle

Looking at the game you enter 5 numbers, then spin. If the randomly chosen 5 numbers align with your 5 numbers you get the flag. 
Looking at the code in the server and the html, it shows the 5 inputed numbers are entered under the variables field1, field2, ..., field5. Looking at the backend of the server, it shows that it is using the inputed fields in some comparisons. Thus by simply setting the inputs to ```true```, you should be able to get the flag. This can be done by inputing the 5 field values as true into a query.
```https://arcade.ctf-league.osusec.org/raffle?field1=true&field2=true&field3=true&field4=true&field5=true```

