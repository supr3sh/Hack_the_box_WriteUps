# CAP <br>
## IP 10.10.10.245 <br>
## Difficulty Easy

Added the IP in /etc/hosts with cap.htb.
Scanned the IP with rustscan and got 3 open ports: 21, 22, 80.
Started a gobuster scan on the IP and also started manual enumeration on the website.
It was already logged in as Nathan user.

In the webiste, in the /data/ directory, I saw some packet capture file and it was in /1.
I found something fishy here. Started a burpsuite intruder attack on the /1 parameter and found that there are 2 more pages /0 and /3.

I opened the 0.pcap in wireshark and found the password for the Nathan user.

```
Nathan:Buck3tH4TF0RM3!
```

Logged into ssh using these credentials and got user.txt

```
005a89726f40707eca7ceb244b283cb4
```

My first thought was since the name of machine was cap, it might have some capability vulnerability.

```
getcap -r / 2>/dev/null
```

This showed that python3.8 has setuid capability set.

To exploit this:

```
/usr/bin/python3.8 -c 'import os; os.setuid(0); os.system("/bin/bash")'
```

Got root shell and root.txt

```
d707599a6d4346be8c1b4983fd1ccbe9
```
