---
layout: post
title: "Signed"
date: 2026-01-28 14:15:00 +0700
tags: redteam Machine-Medium Windows
categories: jekyll update
--- 

# Recon 

---

Đầu tiên, mình thực recon qua **nmap** thu được kết quả như sau: 

```nmap 
# Nmap 7.95 scan initiated Thu Oct 23 11:32:29 2025 as: /usr/lib/nmap/nmap -sC -sV -Pn -T4 -p- --stats-every=5s -oN Signed 10.10.11.90
Nmap scan report for 10.10.11.90 (10.10.11.90)
Host is up (0.35s latency).
Not shown: 65534 filtered tcp ports (no-response)
PORT     STATE SERVICE  VERSION
1433/tcp open  ms-sql-s Microsoft SQL Server 2022 16.00.1000.00; RTM
| ms-sql-ntlm-info: 
|   10.10.11.90:1433: 
|     Target_Name: SIGNED
|     NetBIOS_Domain_Name: SIGNED
|     NetBIOS_Computer_Name: DC01
|     DNS_Domain_Name: SIGNED.HTB
|     DNS_Computer_Name: DC01.SIGNED.HTB
|     DNS_Tree_Name: SIGNED.HTB
|_    Product_Version: 10.0.17763
|_ssl-date: 2025-10-23T06:16:58+00:00; +1s from scanner time.
| ssl-cert: Subject: commonName=SSL_Self_Signed_Fallback
| Not valid before: 2025-10-23T03:54:16
|_Not valid after:  2055-10-23T03:54:16
| ms-sql-info: 
|   10.10.11.90:1433: 
|     Version: 
|       name: Microsoft SQL Server 2022 RTM
|       number: 16.00.1000.00
|       Product: Microsoft SQL Server 2022
|       Service pack level: RTM
|       Post-SP patches applied: false
|_    TCP port: 1433

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Thu Oct 23 13:16:58 2025 -- 1 IP address (1 host up) scanned in 6268.31 seconds
```

Từ đây mình thu được: 
* 1443/tcp - **MSSQL**

Như vậy mình ghi vào trong `/etc/hosts`

![image](/assets/images/HTB/season9/Signed/image1.png)

Kèm theo 1 credential được cung cấp là: `scott:Sm230#C5NatH`

# Khai thác MSSQL 

---

Từ đây mình khai thác service này với credential được cung cấp. 

![image](/assets/images/HTB/season9/Signed/image2.png)

Tiếp tục tìm hiểu các trong service này có gì.... 

![image](/assets/images/HTB/season9/Signed/image3.png)

![image](/assets/images/HTB/season9/Signed/image4.png)

Từ đây mình thấy có user là `dbo` có **RoleName** là `db_owner`.

Như vậy nếu mà chiếm được account này thì hoàn có quyền **admin**

Nhưng đầu tiên trên user hiện tại có được dùng `xp_cmdshell` ko?

![image](/assets/images/HTB/season9/Signed/image5.png)

Thì user hiện tại ko có quyền. 

Đến đây thì tại sao tui phải mún có được `xp_cmdshell`

- `xp_cmdshell` thực thi lệnh OS từ SQL. Nếu mà mình bật được `xp_cmdshell` với 1 user có quyền cao hơn, thì mình hoàn toàn có thể thực thi code hoàn toàn hợp lí. 

Và thường dùng `xp_dirtree` để capture **NTLM Hashes** để có thể chiếm được account Administrator. 

```SQL 
SELECT OBJECT_ID('master..xp_dirtree') AS objid; 
SELECT HAS_PERMS_BY_NAME('master..xp_dirtree','OBJECT','EXECUTE');
``` 

```SQL
SELECT HAS_PERMS_BY_NAME('master..xp_dirtree','OBJECT','EXECUTE') AS can_execute_xp_dirtree
```

![image](/assets/images/HTB/season9/Signed/image6.png)

Và nếu nó trả về `1` từ có là user hiện tại có quyền dùng `xp_dirtree`

## Capture NTLM Hash qua Responder 

![image](/assets/images/HTB/season9/Signed/image7.png)

Từ trong **MSSQL** gửi về host attacker. 

![image](/assets/images/HTB/season9/Signed/image8.png)

Quay lại host attacker thì mình thu được như sau: 

![image](/assets/images/HTB/season9/Signed/image9.png)

Như vậy mình thu được hash rồi, mình cần đem hash này đem crack thử.

![image](/assets/images/HTB/season9/Signed/image10.png)

Từ đây mình tìm được passowrd là `purPLE9795!@`

Tiếp theo mình hoàn toàn có thể login vào bằng account này. 

![image](/assets/images/HTB/season9/Signed/image11.png)

Sau đó, mình thử kiểm tra xem account này có role gì ko. 

```SQL
SELECT r.name AS role, m.name AS member 
FROM sys.server_principals r 
JOIN sys.server_role_members rm ON r.principal_id = rm.role_principal_id 
JOIN sys.server_principals m ON rm.member_principal_id = m.principal_id 
WHERE r.name = 'sysadmin';
```

Đầu tiên, mình cần tìm hiểu trong `sys.server_principals` chứa cái gì? 

![image](/assets/images/HTB/season9/Signed/image12.png)

Như vậy trong `sys.server_principals` thì nó chứa tất cả tất cả các tài khoản và vai trò trong server. 

Và trong `sys.server_role_members` chứa gì? 

![image](/assets/images/HTB/season9/Signed/image13.png)

Như vậy thì khi đó thì sẽ lấy được **role** của **sysadmin** sẽ có các member nào. 

![image](/assets/images/HTB/season9/Signed/image14.png)

Ở đây với `sysadmin` server role thì mình có các members như sau: 
- **sa**
- **SIGNED\IT**
- **NT SERVICE\SQLWriter**
- **NT SERVICE\MSSQLSERVER**
- **NT SERVICE\SQLVERAGENT**

Với các cấu hình này thì tất cả các tài khoản này thì đều có đặc đủ đặc quyền quản trị trên phiên bản SQL Server. 

## SILVER TICKET

Như vậy nhìn thấy rõ ràng thì đây có thể khai thác theo hướng **Silver Ticket**

**Silver Ticket** là một kiểu tấn công sau khi đã chiếm (post-exploit) --> attacker giả mạo ticket service Kerberos. 

### Bước 1 - Lấy domain SID và SIDs cho groups/users 

Mình sẽ lấy domain name hiện tại 

![image](/assets/images/HTB/season9/Signed/image15.png)

Tiếp theo mình sẽ lấy SID cho **IT** và **MSSQLSVC**. 

![image](/assets/images/HTB/season9/Signed/image16.png)

**SID** là mã định dạnh cho groups/users.

Tóm tắt lại mình thu được: 
- Khi mình tìm kiếm **Domain** thì mình thu được Domain dưới dạng **NetBIOS domain name** 
- Khi mình tìm SID chuyển sang dạng hex thì thu được `0x010500`, thay vì ở định dạng mà con người đọc được `S-1-5-21...`

Và để convert từ SID dạng hex này sang format có thể đọc được `S-1-...`, mình cần bỏ prefix `0x` và cấu trúc của SID như sau: 
- **Revision - 1 byte - phiên bản của SID (thường là 1)**
- **Subauthority count - 1 byte - số lượng trường SubAuthority** 
- **Identifier authority - 6 byte - tổ chức cấp phát SID**
- **Little-endian 4-byte subauthorities - Các giá trị SubAuthority tạo nên phần cuối của SID**

Sắp xếp cấu trúc được form như sau: 
`S-<revision>-<identifier_authority>-<subauthority1>-...-<subauthorityN>` 

Như vậy mình có thể viết 1 script để phân tích như sau: 

```python 
import struct 

def parse_sid(hex_str):
    
    # Bỏ '0x' nếu có
    if hex_str.startswith('0x'):
        hex_str = hex_str[2:]
    
    # Chuyển chuỗi hex sang bytes 
    data = bytes.fromhex(hex_str)

    # Lấy revision và sub-authority count
    revision = data[0]
    subauth_count = data[1]
    
    # Đọc Identifier Authority (6 bytes big endian)
    id_auth = int.from_bytes(data[2:8], byteorder='big')
    
    # Đọc từng sub-authority (4 byte little endian)
    subs = [struct.unpack("<I", data[8+i*4:12+i*4])[0] for i in range(subauth_count)]
    
    # Tạo chuỗi SID đọc được 
    sid_str = f"S-{revision}-{id_auth}-" + "-".join(str(s) for s in subs)
    return sid_str 
    
hex_sid = "0x0105000000000005150000005b7bb0f398aa2245ad4a1ca451040000"
print(parse_sid(hex_sid))
``` 

Thu được kết quả một chuỗi có thể đọc được. 

![image](/assets/images/HTB/season9/Signed/image17.png)

Tiếp tục, mình parse của `SIGNED\mssqlsvc`

![image](/assets/images/HTB/season9/Signed/image18.png)

Như vậy thì thu được: 
- **Doamin SID**: S-1-5-21-4088429403-1159899800-2753317549
- **IT Group RID**: 1105 
- **mssqlsvc RID**: 1103 

### Bước 2 - Tính toán NTLM hash (ntlmhash) 

Thì trong mô hình Kerberos nó sẽ dùng password của 1 user để băm ra tạo thành 1 khóa bí mật để thực hiện các bước xác thực trong Authentication Server. 

![image](/assets/images/HTB/season9/Signed/image19.png)

Cho nên mình cần tính toán như sau: 

![image](/assets/images/HTB/season9/Signed/image20.png)

Hoặc mình chạy trên terminal như sau: 

![image](/assets/images/HTB/season9/Signed/image21.png)

### Bước 3 - Ép buộc tạo ra 1 Silver Ticket để có thể tiếp cận service 

![image](/assets/images/HTB/season9/Signed/image22.png)

Sau đó mình theo ticket này vào để tiến hành khai thác. 

```bash
export KRB5CCNAME=mssqlsvc.ccache 
```

![image](/assets/images/HTB/season9/Signed/image23.png)

Từ đây, mình có thể bật `enable_xp_cmdshell`

![image](/assets/images/HTB/season9/Signed/image24.png)

### Upload reverse shell - tương tác qua Metasploit

Sau khi bật `xp_cmdshell`, thì mình hoàn toàn có thể tương tác với command system. 

Rồi mình tiếp tục tìm kiếm trên folder của user `mssqlsvc` có gì 

![image](/assets/images/HTB/season9/Signed/image25.png)

![image](/assets/images/HTB/season9/Signed/image26.png)

![image](/assets/images/HTB/season9/Signed/image27.png)

Sau khi tìm kiếm thì mình sẽ upload 1 reverver shell như sau: 

![image](/assets/images/HTB/season9/Signed/image28.png)

Sau khi upload thành công 

![image](/assets/images/HTB/season9/Signed/image29.png)

Tiếp theo mình khởi động metatsploit và setup để có thể có kết nối. 

![image](/assets/images/HTB/season9/Signed/image30.png)

Đồng thời trên target cho khởi chạy file payload đã upload bằng 

```
xp_cmdshell "%TEMP%/rev.exe"
```

![image](/assets/images/HTB/season9/Signed/image31.png)

Như vậy hoàn thành kết nối. 

![image](/assets/images/HTB/season9/Signed/image32.png)

### Upload reverse shell - tương tác qua Netcat 

Đầu tiên thì mình cấn có file **nc.exe** trên máy tấn công. 

Sau đó, cũng thực hiện upload lên. 

![image](/assets/images/HTB/season9/Signed/image33.png)

Hoặc dùng 

![image](/assets/images/HTB/season9/Signed/image34.png)

Từ đây mình mở kết nối trên target. 

![image](/assets/images/HTB/season9/Signed/image35.png)

Trên máy attack thì mở kết nối để nhận 

![image](/assets/images/HTB/season9/Signed/image36.png)

## Leo thang đặc quyền để lấy Full Control  

Sau khi có được kết nối thì mình hoàn toàn có thể upload 1 file phổ biến để xác định thêm thông tin là **winPEASx64.exe**

![image](/assets/images/HTB/season9/Signed/image37.png)

Và để leo thang đặc quyền mình có thể biết đến là **Golden Ticket**

Nhưng có hai key group trong Active Directory: 
- **Domain Admins - RID** `512`
- **Enterprise Admins - RID** `519`

Đây là những group được coi là có quyền `admin`, như vậy nếu mình có ticket này thì hoàn toàn có thể truy cập thông tin nhạy cảm. 

Hiện tại thì mình chưa có quyền admin.

![image](/assets/images/HTB/season9/Signed/image38.png)

Mình cũng thực hiện lấy SID như ở trên. 

![image](/assets/images/HTB/season9/Signed/image39.png)

Nhưng mình cũng phải chuyển sang dạng đọc được. Và mình đã có revershell cho nên mình đọc bằng PowerShell 

![image](/assets/images/HTB/season9/Signed/image40.png)

Mình lặp lại các bước tạo ticket như sau: 

![image](/assets/images/HTB/season9/Signed/image41.png)

Mình tiếp tục `export KRB5CCNAME=mssqlsvc.ccache`

Sau đó, login lại bằng cái vé mới đó. 

![image](/assets/images/HTB/season9/Signed/image42.png)

Mình cần enable advanced configure options trong SQL Server. 

![image](/assets/images/HTB/season9/Signed/image43.png)

Tiếp theo mình cần bật `Ad Hoc Distributed Queries` cho phép các truy vấn `ad hoc` đến các nguồn dữ liệu bên ngoài. 

![image](/assets/images/HTB/season9/Signed/image44.png)

Dựa vào đây thì mình hoàn toàn có thể lấy nội dung qua lệnh PowerShell. 

```SQL
SELECT * FROM OPENROWSET(BULK 'C:\Users\Administrator\AppData\Roaming\Microsoft\Windows\PowerShell\PSReadLine\ConsoleHost_history.txt',SINGLE_CLOB) AS x;
``` 

![image](/assets/images/HTB/season9/Signed/image45.png)

```SQL
SELECT * FROM OPENROWSET(BULK 'C:\Users\Administrator\Desktop\root.txt',SINGLE_CLOB) AS x;
```

![image](/assets/images/HTB/season9/Signed/image46.png)

![image](/assets/images/HTB/season9/Signed/image47.png)

### Thông tin tìm được 

Ở đây, mình đã tìm ra 1 credentials admin
- **Administrator:Th1s889Rabb!t**

Như vậy mình đã chạy lệnh Admin trên tài khoản `mssqlsvc`, nhưng như vậy có thể bị lộ nếu mình ko chạy trên chính tài khoản Admin. 

Do đó, mình upload 1 tool tên là `RunasCs.exe` để có thể tiến hành chạy bằng credential mình mới lấy được. 

![image](/assets/images/HTB/season9/Signed/image48.png)

Sau khi upload thì dựa vào reverse shell lúc đầu mình thực hiện chạy. 

![image](/assets/images/HTB/season9/Signed/image49.png)

Quay lại chỗ netcat lắng nghe port 9999. 

![image](/assets/images/HTB/season9/Signed/image50.png)

![image](/assets/images/HTB/season9/Signed/image51.png)
