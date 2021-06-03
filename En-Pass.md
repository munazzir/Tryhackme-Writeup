---
title: "EnPass - TryHackMe"
date: "2021-02-28"
categories: 
  - "tryhackme"
tags: 
  - "403bypass"
  - "crontab"
  - "linux"
---

Write-up for Tryhackme room [En-Pass](https://tryhackme.com/room/enpass).

Difficulty Level - Medium

Nmap scan on machine IP

```
nmap -A -oN initial $IP
```

![](/Images/enpass/Screenshot-2021-02-28-154838.png)

website hosted on port **8001**, lets do more enumeration on that running gobuster on website.

```
gobuster dir -u http://$IP:8000 -w /path/to/wordlist -x php,txt,html -t 40
```

![](/Images/enpass/Screenshot-2021-02-28-155532.png)

when i further look into "**/zip**" directory, i keep getting word "sadman" looks like it rabbit hole.

keep directory brute-force the web directory you eventually find this endpoint "**/web/resources/infoseek/configure/key**" which have SSH key.

download the key, it is encrypted i use john to decrypt but no luck.

so we move to **reg.php** which we saw earlier.

![](/Images/enpass/Screenshot-2021-02-28-160735.png)

looks like it need some data input I'm not developer don't have much understanding about javascript but when we read through code can see, we cannot enter **letters** or **numbers** input data SUM should be "**9**" input should be separate by comma "**,**". apart from these information i needed some help to understand.

![](/Images/enpass/Screenshot-2021-02-28-161934.png)

after entering above string we get password, it can be the passphrase for the SSH key.

![](/Images/enpass/Screenshot-2021-02-28-162404.png)

okay, now we have SSH private key, so i try to login with user "**sadman**" but it failed, so i moved on to **403.php**.

if we can bypass 403 error we might get something from it so i used [403FUZZER](https://github.com/intrudir/403fuzzer).

it have some good features open burp-suit and send all fuzzer traffic through burp proxy so we can see process in details.

```
python3 403fuzzer.py -u http://$IP:8001/403.php -hc 404,403 -p http://localhost:8080
```

![](/Images/enpass/Screenshot-2021-02-28-164114.png)

now we have a possible name and SSH private key, we can try to login with that.

![](/Images/enpass/Screenshot-2021-02-28-164453.png)

## Privilege Escalation

when we do basic enumeration we can see python script in "**/opt"** directory which run by root.

![](/Images/enpass/Screenshot-2021-02-28-165352.png)

when we look for the file in **/tmp** we can't find it. so run [PSPY](https://github.com/DominicBreuker/pspy), find hidden proccess.

![](/Images/enpass/Screenshot-2021-02-28-165953.png)

it execute file and delete it, so we can create file with same name and set SUID to "/bin/bash".

first create file and add our payload.

```
!!python/object/apply:os.system ["chmod 4777 /bin/bash"]
```

save this file as any name, then we need to create loop which will replace original "file.yml" with our payload.

![](/Images/enpass/Screenshot-2021-02-28-170624.png)

wait some time or till you get error then exit.

```
bash -p 
```

above command give root access since we set SUID to bash.

![](/Images/enpass/Screenshot-2021-02-28-170954.png)

END.

Happy Hacking!!!
