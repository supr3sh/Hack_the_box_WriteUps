# Netmon
## IP: 10.10.10.152
## OS: Windows

Nmap Scan: 

```
Open 10.10.10.152:21
Open 10.10.10.152:80
Open 10.10.10.152:135
Open 10.10.10.152:139
Open 10.10.10.152:445
```

## User.txt

Anonymous login was enabled. I logged in as anonymous and we were in C directory.
I went to Users/Public and there was user.txt

```
dd58ce67b49e15105e88096c8d9255a5
```

## Root.txt

In the web browser, I saw it was PRTG network monitor webpage and it was asking to login.
I searched online and found that configuration files of PRTG is saved in C:\Users\All Users\Application Data\Paessler\PRTG Network Monitor.

I went there and there were 3 configuration files.

```
'PRTG Configuration.dat'  
'PRTG Configuration.old'  
'PRTG Configuration.old.bak'
```

Downloaded all of them.
I also found out that default username in PRTG webpage is prtgadmin.

So I searched for prtgadmin in all of these files and got the password.

```
prtgadmin:PrTg@dmin2018
```

But it didn't worked. Then I changed the password from 2018 to 2019 and it worked.

Since I had crendentials now, I searched online for any exploit.
I found that there is authenticated RCE. I used metasploit for this purpose.

Started metasploit and searched for prtg.

Then I used the only exploit and set the required options and ran the exploit.

I got shell as system.

```
3018977fb944bf1878f75b879fba67cc
```
