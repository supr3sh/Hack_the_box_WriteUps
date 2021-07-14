# BLOCKY
## Difficulty: Easy
## OS: Linux

Nmap Scan: Open Ports: 21, 22, 80.

Started gobuster scan on web browser and found that webpage was made using wordpress.

Then I started a wpscan to enumerate users and plugins.

Got 1 user:

```
notch
```

In gobuster scan, I got phpmyadmin and a plugins folder.

In plugins folder, there were two jar files.

Downloaded both and unzipped and decompiled them with jad.

There were mysql credentials for root user.

```
sqlHost = "localhost";
sqlUser = "root";
sqlPass = "8YsqfCTnvxAUeduzjNSXe22";
```

Used these creds to login to phpmyadmin and found phpass hash for notch user.

But I was not able to crack it.

Then I used the same password for notch to login to ssh and succeeded.

Got user.txt

We are able to run any command as root.

Got root.txt
