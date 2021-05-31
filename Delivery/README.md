# ScriptKiddie
IP: 10.10.10.222		Difficulty: Easy
Scanned the IP with rustscan and got 22,80,8065 open.
80 and 8065 are HTTP.

## Web enumeration

Added IP to /etc/hosts.
Stated a gobuster scan on IP.
In the port 80 homepage, I say a link to helpdesk.delivery.htb, so added it to hosts file.
Then I went to the helpdesk page.
It was an ticket center where I can generate a ticket to login as per the homepage.
I generated a ticket by entering details.
As I created a ticket, it gave me a ticket number and an email with delivery.htb as hostname.

```
ticket number 3966336
email 3966336@delivery.htb
```

Now since I have a delivery.htb email, I can create an account in the 8065 port (mattermost).
Signup with this email and fill other details.

It will ask to verify email. Go to helpdesk again and click check status and enter the main email and your ticket number.
It will have an link to verify email.
Now that email is verified, signin to the account in mattermost.

Select the Internal channel.

In the chat, I got credentials of maildeliverer user.

```
maildeliverer:Youve_G0t_Mail! 
```

Now SSH into the machine with these creds.

## User.txt

user.txt was in the home directory of maildeliverer.

```
97403e55849ab0741b638aef67516afe
```

## Root.txt

I found out that mysql was running locally on this machine.
Now I have to find the credentials to login to the mysql.
I found the creds in /opt/mattermost/config/config.json.

```
mmuser:Crack_The_MM_Admin_PW
```

```
mysql -h localhost -u mmuser -p
```

Selected the mattermost database and dumped the Users table.

There was an entry for root with password hash.

```
$2a$10$VM6EeymRxJ29r8Wjkr8Dtev0O.1STWb4.4ScG.anuu7v0EFJwgjjO
```

But when I tried to crack the hash with john, I did not do it.
When I saw the hint again on the Internal room, I found that "PleaseSubscrbe!" must be in the password. And there was something about rules. Most probably hashcat rules.
I saw another writeup and got to know how to use hashcat rules. They used best64 rule to create a custom wordlist.

```
echo "PleaseSubscribe!" > pass.list

hashcat --stdout pass.list -r /usr/share/hashcat/rules/best64 > newList.txt
```

Now to crack the password, I again used john.

```
john rootHash -w=./newList.txt
```

Got the password.

```
PleaseSubscribe!21
```

Now ssh into machine as root.
root.txt is in root's home directory.

```
223726ec96aed416342576763a1ca0d6
```
