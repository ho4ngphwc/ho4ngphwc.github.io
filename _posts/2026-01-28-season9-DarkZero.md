# DarkZero 

## Recon 

```bash 
nmap -sC -sV -T4 -p- --stats-every=5s -oN DarkZero 10.10.11.89
```

Thu được kết quả scan

```b
# Nmap 7.95 scan initiated Sat Oct 18 09:22:04 2025 as: /usr/lib/nmap/nmap -Pn -T4 -sC -sV -p- --stats-every=5s -oN DarkZero 10.10.11.89
Nmap scan report for 10.10.11.89
Host is up (0.30s latency).
Not shown: 65512 filtered tcp ports (no-response)
PORT      STATE SERVICE       VERSION
53/tcp    open  domain        Simple DNS Plus
88/tcp    open  kerberos-sec  Microsoft Windows Kerberos (server time: 2025-10-18 09:30:30Z)
135/tcp   open  msrpc         Microsoft Windows RPC
139/tcp   open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp   open  ldap          Microsoft Windows Active Directory LDAP (Domain: darkzero.htb0., Site: Default-First-Site-Name)
|_ssl-date: TLS randomness does not represent time
| ssl-cert: Subject: commonName=DC01.darkzero.htb
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1:<unsupported>, DNS:DC01.darkzero.htb
| Not valid before: 2025-07-29T11:40:00
|_Not valid after:  2026-07-29T11:40:00
445/tcp   open  microsoft-ds?
464/tcp   open  kpasswd5?
593/tcp   open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp   open  ssl/ldap      Microsoft Windows Active Directory LDAP (Domain: darkzero.htb0., Site: Default-First-Site-Name)
|_ssl-date: TLS randomness does not represent time
| ssl-cert: Subject: commonName=DC01.darkzero.htb
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1:<unsupported>, DNS:DC01.darkzero.htb
| Not valid before: 2025-07-29T11:40:00
|_Not valid after:  2026-07-29T11:40:00
1433/tcp  open  ms-sql-s      Microsoft SQL Server 2022 16.00.1000.00; RTM
| ms-sql-ntlm-info: 
|   10.10.11.89:1433: 
|     Target_Name: darkzero
|     NetBIOS_Domain_Name: darkzero
|     NetBIOS_Computer_Name: DC01
|     DNS_Domain_Name: darkzero.htb
|     DNS_Computer_Name: DC01.darkzero.htb
|     DNS_Tree_Name: darkzero.htb
|_    Product_Version: 10.0.26100
| ms-sql-info: 
|   10.10.11.89:1433: 
|     Version: 
|       name: Microsoft SQL Server 2022 RTM
|       number: 16.00.1000.00
|       Product: Microsoft SQL Server 2022
|       Service pack level: RTM
|       Post-SP patches applied: false
|_    TCP port: 1433
|_ssl-date: 2025-10-18T09:32:11+00:00; +7h00m01s from scanner time.
| ssl-cert: Subject: commonName=SSL_Self_Signed_Fallback
| Not valid before: 2025-10-18T06:21:20
|_Not valid after:  2055-10-18T06:21:20
2179/tcp  open  vmrdp?
3268/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: darkzero.htb0., Site: Default-First-Site-Name)
| ssl-cert: Subject: commonName=DC01.darkzero.htb
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1:<unsupported>, DNS:DC01.darkzero.htb
| Not valid before: 2025-07-29T11:40:00
|_Not valid after:  2026-07-29T11:40:00
|_ssl-date: TLS randomness does not represent time
3269/tcp  open  ssl/ldap      Microsoft Windows Active Directory LDAP (Domain: darkzero.htb0., Site: Default-First-Site-Name)
|_ssl-date: TLS randomness does not represent time
| ssl-cert: Subject: commonName=DC01.darkzero.htb
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1:<unsupported>, DNS:DC01.darkzero.htb
| Not valid before: 2025-07-29T11:40:00
|_Not valid after:  2026-07-29T11:40:00
5985/tcp  open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-title: Not Found
9389/tcp  open  mc-nmf        .NET Message Framing
49664/tcp open  msrpc         Microsoft Windows RPC
49666/tcp open  msrpc         Microsoft Windows RPC
49690/tcp open  msrpc         Microsoft Windows RPC
49691/tcp open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
49917/tcp open  msrpc         Microsoft Windows RPC
49952/tcp open  msrpc         Microsoft Windows RPC
50001/tcp open  msrpc         Microsoft Windows RPC
64811/tcp open  msrpc         Microsoft Windows RPC
Service Info: Host: DC01; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
| smb2-security-mode: 
|   3:1:1: 
|_    Message signing enabled and required
| smb2-time: 
|   date: 2025-10-18T09:31:31
|_  start_date: N/A
|_clock-skew: mean: 7h00m00s, deviation: 0s, median: 7h00m00s

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Sat Oct 18 09:32:13 2025 -- 1 IP address (1 host up) scanned in 608.72 seconds
```

Ở đây thì mình quan tâm thu được: 
- **SMB - 445** 
- **MSSQL - 1433**

Và kèm theo thì mình được cung cấp 1 credential là: `john.w / RFulUtONCOL!`

## Khai thác SMB 

![image](https://hackmd.io/_uploads/r11iHS-Cgx.png)

Thì đây mình thấy có **ADMIN$ và C$**, thường thì nếu user ko phải admin thì sẽ ko vào được.

Còn **IPC$, NETLOGON và SYSVOL** thường thì trên Domain Controller sẽ có. 

![image](https://hackmd.io/_uploads/SJt6LSbAxl.png)

![image](https://hackmd.io/_uploads/Bk1bvSbClg.png)

Nhìn thấy có 4 user:
- **Administrator**
- **Guest**
- **krbgt**
- **john.w**

Mục tiêu thì mình cần lấy được user **Administrator**.

## Khai thác MSSQL 

![image](https://hackmd.io/_uploads/H17yFBWCgl.png)

Điều đầu tiên cần kiểm tra là có link tới server nào ko ? 

![image](https://hackmd.io/_uploads/rk2Vtrb0gl.png)

Kết quả thì có link qua **DC02.darkzero.ext**, và server hiện tại mình đang đứng là **dc01_sql_svc** được dùng làm Remote login. Như vậy thì coi như mình có thể thực hiện query trên **DC02**

![image](https://hackmd.io/_uploads/By_QnHZClg.png)

Khi đó mình dùng link tới **DC02** và thực hiện dùng `xp_cmdshell` thì đây là 1 dạng cho phép mình chạy lệnh hệ điều hành mà MSSQL cho phép. 

Và để chạy được thì mình trước tiên phải bật qua: `enable_xp_cmdshell` cho nên mình đã list ra được 2 quyền mà user hiện tại đáng có. 

Tiếp theo thì mình đi tìm xem trong các folder có gì ko ? và đáp án là hầu như đều trống. 

Nhưng mà từ đây thì mình có thể tải 1 tool lên là **winPEAS** và 1 payload dùng cho **reverse shell**

Thì lệnh để tạo 1 file payload chứa reverse shell 

![image](https://hackmd.io/_uploads/rkcxyLbRlg.png)

Sau khi tạo thì cần mở server trên host attack, rồi mình tải file payload thông qua shell MSSQL. 

![image](https://hackmd.io/_uploads/rkoJeLbClx.png)

Tiếp đó mình set trong metasploit để chuẩn bị kết nối. 

![image](https://hackmd.io/_uploads/ryfHMIZRgg.png)

Đồng thời thì chạy thực thi file revshell.exe tại DC02 qua DC01.

![image](https://hackmd.io/_uploads/S1ViG8-Rle.png)

![image](https://hackmd.io/_uploads/r1yCGIbCxl.png)

Đến đây thì mình list folder, và vào trong folder Administrator nhưng bị cấm. 

![image](https://hackmd.io/_uploads/SkYdQUbRlx.png)

Và đến đây, thì cũng hết ý để khai thác, cho nên mình cần đến tool **winPEAS** ra tay. 

![image](https://hackmd.io/_uploads/BkWrLRbRxg.png)

Sau khi chạy ra thì mình thu được version OS, cũng như nó là Datacenter. 

![image](https://hackmd.io/_uploads/Byrpv0WAxx.png)

Sau khi tìm hiểu thì mình tìm được CVE-2024-30088 

![image](https://hackmd.io/_uploads/H1VVHyzCxx.png)

- [Nist ](https://nvd.nist.gov/vuln/detail/cve-2024-30088)
- [Microsoft](https://msrc.microsoft.com/update-guide/vulnerability/CVE-2024-30088)

Thay vì phải tìm PoC để exploit thủ công thì mình dùng metasploit 

![image](https://hackmd.io/_uploads/ByBH8kzCgx.png)

Từ reverse shell lúc đầu thì mình Ctrl+Z để search cve-2024-30088, sau khi set up dùng payload cve thì mình cần set vào session đang chạy reverse shell cũ. 

![image](https://hackmd.io/_uploads/HyTDRJGReg.png)

Kết quả thì mình đã leo lên thành **nt authority\system** đây là tài khoản có quyền mạnh nhất trong windows. Sau đó thì khai thác tiếp trong folder của user.

![image](https://hackmd.io/_uploads/B1xLJlGRgl.png)

#### Cách khai thác mà ko dùng tới metasploit

Đầu tiên thì mình dùng `xp_cmdshell` để list ip trên **DC02**

![image](https://hackmd.io/_uploads/ryms1QG0ee.png)

Như vậy thì nó hoàn toàn nằm bên trong đây là ip internal. 

Sau đó, mình thiết lập **ligolo** để tiến hành pivoting. 
- [Ligolo](https://github.com/nicocha30/ligolo-ng/releases)

Mình cần chuẩn bị 1 file agent để upload lên windows và 1 proxy trên kali. 

![image](https://hackmd.io/_uploads/BJ3AemM0ex.png)

Sau đó thiết lập proxy dựa theo tài liệu này [Thiết lập Proxy](https://www.stationx.net/how-to-use-ligolo-ng/)

![image](https://hackmd.io/_uploads/SJBKW7MAle.png)

Trên server proxy đã nhận kết nối về. 

![image](https://hackmd.io/_uploads/ryMoWQMRlx.png)

Tiếp tục thiết lập tunnel - để lấy hash của Admin qua **psexec.py**

Nhưng đây mới là DC02, vậy còn DC01 thì chưa khai thác gì?

![image](https://hackmd.io/_uploads/ry3yllMAxl.png)

Nhưng mà cả hai server này lúc đầu đã link với nhau. Quay lại kết quả scan ban đầu thấy trên DC01 có service kerberos. 

Mà máy DC02 đã chiếm quyền, nếu mình chạy 1 tool bắt ticket trên DC02 và ép cho DC01 gửi xác thực thì lúc đó mình có thể lấy được ticket này. 

Đầu tiên thì mình cần upload tool **Rubeus.exe** lên DC02 

![image](https://hackmd.io/_uploads/H10hkzLCge.png)

Để upload như trên thì cần phải leo lên được Admin bằng cve lúc đầu. 

Sau khi, upload xong thì mình dùng **shell** để chạy. 

![image](https://hackmd.io/_uploads/S1deVMIAge.png)

Thì tool đã bắt được 7 ticket, sau đó mình truy cập vào DC01 để yêu cầu tới DC02. 

![image](https://hackmd.io/_uploads/rk2HVMLCxg.png)

Thì trên Rebeus đã bắt được. 

![image](https://hackmd.io/_uploads/Hyiv4zLAll.png)

Tiếp theo, mình cần lấy về bỏ vào trong 1 file là **ticket.bs64.kirbi**

Tiến hành decode nó **cat ticket.bs64.kirbi | base64 -d > ticket.kirbi**

Mình cần chạy 1 file để convert ticket này sang file cache bằng **python3 ticketConvert.py ticket.kirbi dc01_admin.ccache**

Sau khi conver thì chạy `export KRB5CCNAME=dc01_admin.ccache` rồi dùng klist để hiển thị ticket keberos này. 

![image](https://hackmd.io/_uploads/rkZRB78Rlg.png)

Sau đó, dùng tool `impacket-secretsdump` để dump hash, nhưng giờ bị như thế này. 

![image](https://hackmd.io/_uploads/Bks_UmI0eg.png)

Điều này có nghĩa liên quan đến **SPN** - Service Principal Name - tên duy nhất của dịch vụ trong Kerberos. 

Khi mình dùng **impacket-secretsdump** gửi tới thì nó sẽ trả SPN nào khớp với **dc01.darkzero.htb** rồi sẽ trả về. 

Cho nên mình sẽ dùng `sudo nptdate -u 10.10.11.89`, rồi mình chạy lại. 

![image](https://hackmd.io/_uploads/rJ-EbzQ1bx.png)

Sau khi có hash thì mình dùng `evil-winrm` để vào

![image](https://hackmd.io/_uploads/SJx3-Gmkbx.png)

Từ đây, mình có thể leo vào bằng tài khoản Admin, sau đó mình có thể đọc `root.txt`

![image](https://hackmd.io/_uploads/SyiRbzXkWx.png)

![image](https://hackmd.io/_uploads/H19eMGmyWx.png)



