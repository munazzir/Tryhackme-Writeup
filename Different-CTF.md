DIFFERENT CTF
=============


Starting with initial nmap scan for available services.

`nmap --min-rate 300 -Pn -A -oN initial adana.thm `

```
Nmap scan report for adana.thm (10.10.102.68)
Host is up (0.22s latency).
Not shown: 998 closed ports
PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 3.0.3
80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
|_http-generator: WordPress 5.6
|_http-server-header: Apache/2.4.29 (Ubuntu)
|_http-title: Hello World &#8211; Just another WordPress site
Service Info: OS: Unix
```
# Web Flag

look like wordpress site it's better to run `wpscan` on it, wpscan return some user name.

![nmap](/Images/different_ctf/nmap.png)

I try to bruteforce **FTP** not working so we might need this user later, moving on bruteforce directory in website.

![gobuster](/Images/different_ctf/gobuster.png)

**/announcements** is new not usual wordpress directory when we visit we find **wordlist** and **image**.

So i try to check hidden note in image, using `Stegcracker` and given wordlist and got the hidden note.

![steghide](/Images/different_ctf/steghide.png)

hidden text file include `FTP-LOGIN` creds with is encoded in **base64** 

```
FTP-LOGIN
USER: hakanftp
PASS: 12***********
```

after login we can see it is like web root directory.uploaded webshell but no use, it is look like copy of web.
so dowload the config file and check for any creds.

In config file we can find `phpmyadmin` **username** and **password**

![phpmyadmin](/Images/different_ctf/phpmyadmin.png)

after login i try to get shell but failed then i notice other database is running.

![phpmyadmin](/Images/different_ctf/phpmyadmin1.png)

a subdomain is running to see the subdomain url goto

`phpmyadmin1 ----> wp_option ------> site_url` 

now add it to **/etc/hosts** and visit the page, it look like same website but when i looked around,i found earlier uploaded webshells in here.
so executed the shell and got my reverse shell back.

![shell](/Images/different_ctf/shell.png)

# User Flag

Searched everything but nothing to escalate privilege, when we crack **steghide password** and **FTP password** had some similarity, so there might be chance to user password also in same format.

I created wordlist using `mentalist` with prefix of **123adana** and the wordlist we found in web.

To crack the password i used [sucrack](https://github.com/hemp3l/sucrack).

Upload **sucrak** binary and **wordlist** to remote machine and crack the password.

![User-FLag](/Images/different_ctf/user.png) 

Switched to user `hakanbey` lets find method escalate privilege

# Root Flag

`find / -user root -perm /4000 2>/dev/null`

![Escalation](/Images/different_ctf/escalation.png)

**/usr/bin/binary** not ordinary binary so lets poke it.

![ltrace](/Images/different_ctf/ltrace.png)

it have the string expecting when it executed, with `ltrace` we can execute it and get result.

![root](/Images/different_ctf/root.png)

it give us image and hint to use **Cyberchef** and **Hexeditor**.

open image on hexeditor hint say start with **00000020** (hint)

copy block of code into Cyberchef

```
00000000  FF D8 FF E0  00 10 4A 46   49 46 00 01  01 01 00 60                          
00000010  00 60 00 00  FF E1 00 78   45 78 69 66  00 00 4D 4D           
00000020  FE E9 9D 3D  79 18 5F FC   82 6D DF 1C  69 AC C2 75           
00000030  00 00 00 56  03 01 00 05   00 00 00 01  00 00 00 68              
00000040  03 03 00 01  00 00 00 01   00 00 00 00  51 10 00 01                              
00000050  00 00 00 01  01 00 00 00   51 11 00 04  00 00 00 01 
```

decode `From Hex` ---> `To Base85`

then you get password for root.

![Root-Flag](/Images/different_ctf/rootflag.png)

# Flags

+ THM{343a7e2064a1d992c01ee201c346edff}

+ THM{8ba9d7715fe726332b7fc9bd00e67127}

+ THM{c5a9d3e4147a13cbd1ca24b014466a6c}
