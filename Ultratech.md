---
title: "UltraTech - TryHackMe"
date: "2020-12-11"
categories: 
  - "tryhackme"
---

This is a write-up for tryhackme room name UltraTech: https://tryhackme.com/room/ultratech1

Deploy the machine and start basic scan, usually i do two scan; reason is if room have only common ports no need to waste time by scanning all ports.

1. Initial scan for all common ports
2. all port scan

**nmap -A -oN initial 10.10.160.79**

![](/Images/ultratech/Screenshot-2020-12-11-123855-1.png)

save output always, that is best practice with result you can go through again. when reading questions we can see there are some uncommon ports are ups. for that we need to run all port scan. if you want you can start from 10000 ports.

**nmap -A -p- -oN allport 10.10.160.79 -T4**

![](/Images/ultratech/Screenshot-2020-12-11-143558.png)

when we visit the port 8081 nothing there just some Node.js file running, keep it in mid everything we find in enumeration can be useful later

in uncommon port 31331 website is hosted, if there is website you can find directories also next thing you should do is run gobuster or dirbuster

Result will be like this,

![](/Images/ultratech/Screenshot-2020-12-11-144812.png)

like mentioned earlier every enumeration result are valuable. go through all directories.

In **/js** directory you can find **api.js**

![](/Images/ultratech/Screenshot-2020-12-11-145153.png)

go through **api.js** we can find some useful code which are,

![](/Images/ultratech/Screenshot-2020-12-11-145435.png)

with port mention can guess it related which we saw in port 8081

\`http://${getAPIURL()}/ping?ip=${window.location.hostname}\`

looking at above function we can guess it will be useful for future enumeration.

go back to port 8081 craft URL using function we found. it would like this.

**http://10.10.160.79:8081/ping?ip=`` `ls` ``**

Result:

![](/Images/ultratech/Screenshot-2020-12-11-150434.png)

so listed the file inside that directory and notice some file called "utech.db.sqlite" since there only one we can use command "`` `cat *``\`" to print the file.

cat result would give two username and two hashes, two users are r00t and admin. you can use [https://crackstation.net/](https://crackstation.net/) to crack the hashes

we can try login in port 8081 but it is a dead end

![](/Images/ultratech/Screenshot-2020-12-11-151836.png)

"/auth" directory we found when using gobuster on port 8081

this is dead-end but we know r00t is valid user we can try login with SSH

![](/Images/ultratech/Screenshot-2020-12-11-152208.png)

The r00t was part of docker group this maybe help to get privilege Escalation

[https://gtfobins.github.io/](https://gtfobins.github.io/) is great website for Linux privilege escalation,

search for docker and we find some method to get root shell.

```
sudo docker run -v /:/mnt --rm -it alpine chroot /mnt sh
```

while running this command we get error because "alpine" there is no image on that name in the system so need to modify the command like below

```
docker run -v /:/mnt --rm -it bash chroot /mnt s
```

when we run, it take some times but eventually we get root shell

![](/Images/ultratech/Screenshot-2020-12-11-153043.png)
