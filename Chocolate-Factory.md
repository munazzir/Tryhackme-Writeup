---
title: "Chocolate Factory - TryHackMe"
date: "2021-01-24"
categories: 
  - "tryhackme"
---

Write-up for Tryhackme room name [Chocolate Factory](https://tryhackme.com/room/chocolatefactory).

Difficulty Level - Easy

Start with Nmap scan as usual.

```
nmap -A -oN initial $IP
```

![](/Images/chocolatefactory/Screenshot-2021-01-24-233614.png)

FTP login allow anonymous login, so we can log in and check for available files.

![](/Images/chocolatefactory/Screenshot-2021-01-24-233817.png)

download the "**gum\_room.jpg**" and extract the data inside the image.

![](/Images/chocolatefactory/Screenshot-2021-01-24-234052.png)

as name mention content is base64 encoded. to decode this command

```
cat b64.txt | base64 -d 
```

looks like it is "**/etc/passwd**" file.

![](/Images/chocolatefactory/Screenshot-2021-01-24-234306.png)

using john we can easily crack the hash

```
john --wordlist=/opt/rockyou.txt hash.txt
```

![](/Images/chocolatefactory/Screenshot-2021-01-24-235246-1.png)

when we try to login SSH with these credential it won't work, we need to login in web site using this credential.

when we login to website it let enter commands, using that we can easily get a reverse shell.

![](/Images/chocolatefactory/Screenshot-2021-01-24-235815.png)

when you login you can see couple of files, the important one is **key\_rev\_key**. type Stings command to see available strings.

```
strings key_rev_key
```

![](/Images/chocolatefactory/Screenshot-2021-01-25-000317.png)

In charlie home directory we will find SSH private and public key, copy private key to local PC and login using SSH.

![](/Images/chocolatefactory/Screenshot-2021-01-25-000639.png)

## Privilege Escalation

initial information gather with available tools

![](/Images/chocolatefactory/Screenshot-2021-01-25-000915.png)

using VI we can easily escalate you can view methods in here [GTFO](https://gtfobins.github.io/gtfobins/vi/#sudo).

```
sudo vi -c ':!/bin/sh' /dev/null
```

![](/Images/chocolatefactory/Screenshot-2021-01-25-001247.png)

## EXTRA

for root flag you need to run root.py and enter the Key.

![](/Images/chocolatefactory/Screenshot-2021-01-25-001506.png)

Happy Hacking!!!
