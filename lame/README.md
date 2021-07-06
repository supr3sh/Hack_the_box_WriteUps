# Lame

IP: 10.10.10.3

Nmap scan:

```
Open 10.10.10.3:21
Open 10.10.10.3:22
Open 10.10.10.3:139
Open 10.10.10.3:445
Open 10.10.10.3:3632
```

I tried to search for any exploit for the versions of the services.
FTP: vsftpd 2.3.4
Samba: samba 3.0.20

I found backdoor RCE exploit for FTP version but it didn't worked.

And I also got a exploit for samba version.

```
'Username' map script' Command Execution (Metasploit)
```

Started metasploit and searched for exploit.

```
exploit/multi/samba/usermap_script
```

Set the options and exploit.
Got shell as root.

```
user.txt : /home/makis/user.txt : 5659af259289982a653f37a25bf82111
root.txt : /root/root.txt : f5d501c07e568e70910313ef2ac6378d
```
