# Write up for Swamp.

## Overview

This challenge has 1 flag. There is a website that makes a shrek meme given a text input. 
Our goal is to somehow get the flag off of the server.

### Flag 1

The text input is a red herring. Looking at the page source via inspect and the python source code,
we see the form has a hidden input that sends the current time. This input is vulnerable to exploition.
Inputting ```{{ 3*4 }}``` we see it responds with ```12``` in the network response of the image viewing page.
Thus we have code execution. But we still need to escape the context of jinja to have arbitary code execution.

Researching jinja exploits, we find the page

[https://github.com/dgtlmoon/changedetection.io/security/advisories/GHSA-4r7v-whpg-8rx3](https://github.com/dgtlmoon/changedetection.io/security/advisories/GHSA-4r7v-whpg-8rx3)

which has the injection template 

```{{ self.__init__.__globals__.__builtins__.__import__('os').popen('id').read() }}```.

Using inpect to put this in the date part of the input form, we see it sends the output
in the response of image viewing page. Then we can send

```{{ self.__init__.__globals__.__builtins__.__import__('os').popen('ls').read() }}```

to get all files in the server's directory. We see that ```flag.txt``` is one of the files.
Finally, sending

```{{ self.__init__.__globals__.__builtins__.__import__('os').popen('cat flag.txt').read() }}```

we get the flag.






