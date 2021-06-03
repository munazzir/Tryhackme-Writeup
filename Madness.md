---
title: "Madness - TryHackMe"
date: "2020-12-13"
categories: 
  - "tryhackme"
---

This is a write-up for tryhackme room name UltraTech [https://tryhackme.com/room/madness](https://tryhackme.com/room/madness)

Deploy machine and start scanning also check all the provided resources.

![](/Images/madness/a.png)

In main page you can see the image, download run exiftool and steghide by doing that we can find hidden "**password.txt"** file. cat the file and get the password.

okay now we have password just by looking around, we didn't even initiate nmap scan on the target. lets go to scan

![](/Images/madness/Screenshot-2020-12-13-205802.png)

NMAP SCAN

nmap -A -oN initial 10.10.109.130

![](/Images/madness/Screenshot-2020-12-13-210400.png)

it seem only 2 ports are running, a website is running in port 80 let's take a look

in apache default web page we can notice some unusual thing, a image

![](/Images/madness/Screenshot-2020-12-13-210841.png)

you can clearly see image name "**thm.jpg**" we can download it

```
wget http://10.10.109.130/thm.jpg
```

when we try to open file we cannot because its file type is "**DATA**" , so we need to use hexeditor and change the signature value, you can find some of signature value here [https://en.wikipedia.org/wiki/List\_of\_file\_signatures](https://en.wikipedia.org/wiki/List_of_file_signatures)

final result after editing

![](/Images/madness/Screenshot-2020-12-13-211514.png)

after changing signature when we open the image we can find this

![](/Images/madness/Screenshot-2020-12-13-211730.png)

okay now we found the hidden directory when we go to that directory there is web page.

![](/Images/madness/Screenshot-2020-12-13-212131.png)

this page requesting some kind of secret when we look into source we can notice it looking for number between "0-99" we can enter number manually or can use burp or we can automate with python. I'm not python expert but after some time searching, after lots of mistake i made working python script.

```
#!/usr/bin python3
import requests

c = 1
url = "http://10.10.109.130/th1s_1s_h1dd3n?secret=" 



for i in range (0, 100):
        payload = (url) + str(c) 
        c += 1

        r= requests.post(payload)
        if str("That is wrong!") in str(r.text):
                continue
        else:
                print(r.text)
                break


```

after we running this script we get the result which is:

![](/Images/madness/Screenshot-2020-12-13-213803.png)

i try to SSH log in with details we collected but it failed, when i try steghide "**thm.jpg**" image earlier it given a error because incorrect password, so we can try password we found in hidden directory to extract file from "thm.jpg" .

![](/Images/madness/Screenshot-2020-12-13-220739.png)

we get "**hidden.txt**" file, when we "cat" file we get this details also

![](/Images/madness/Screenshot-2020-12-13-221030-1.png)

username is encoded with ROT13 we can easily decode it.

now we have username and password now we can login with SSH

find a method to escalate privilege run this command

![](/Images/madness/Screenshot-2020-12-13-222905.png)

screen looks suspicious so google dork for some exploit will find this [https://www.exploit-db.com/exploits/41154](https://www.exploit-db.com/exploits/41154) , all we need to do is upload is to /tmp folder of target machine and execute.

![](/Images/madness/Screenshot-2020-12-13-225708.png)

Finally Root.

Happy Hacking.
