---
layout: post
title: Recon Services
time: 2025-10-09 10:11:00 +0700
tags: redteam Recon
categories: jekyll update
---

# Recon File Sharing Service

## 🗃️ FTP

---

> Target 1: Thu thập thông tin về tập tin PDF được chia sẻ, bao gồm:
> 
> - **`pdf_author`**: Tác giả của tập tin PDF
> - **`credenti_username`**, **`credential_password`**: Credential được chia sẻ trong tập tin

Mình truy cập vào **`files.cj23group.com`** với port **`21`**

Đầu tiên với service FTP thì mình nên truy cập với tài khoản **`anonymous`** là **`anonymous:anonymous`** 

![image.png](/assets/images/redteam/Recon/services/image.png)

Tiếp theo mình cần check xem trong đây có file gì. 

![image.png](/assets/images/redteam/Recon/services/image%201.png)

Mình cứ tiếp tục tìm kiếm để thây file **`.pdf`** 

![image.png](/assets/images/redteam/Recon/services/image%202.png)

Sau đó thấy rồi thì lấy dìa thoai 😎 

![image.png](/assets/images/redteam/Recon/services/image%203.png)

Check tại máy local bằng tool kiểm tra metadata của file là **`exiftool`** 

![image.png](/assets/images/redteam/Recon/services/image%204.png)

Thông tin mình tìm được:

- author: **`tima.dev8888`**

Mình mở file trên máy → thu được thông tin 

![image.png](/assets/images/redteam/Recon/services/image%205.png)

- **`test:test`**

> Target 2: Thu thập thông tin về access token của bot Discord, bao gồm:
> 
> - **`bot_name`**: Tên bot (viết liền)
> - **`issued_at`**: Thời gian token này được tạo ra (dạng số nguyên)

Tiếp tục recon trên server ftp này. 

![image.png](/assets/images/redteam/Recon/services/image%206.png)

Mình tải xong thì check lệnh **`strings`** 

![image.png](/assets/images/redteam/Recon/services/image%207.png)

Mình thu được:

- Bot name: **`CBJS Staff`**
- JWT: **`eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJuYW1lIjoiQ0JKUyBTdGFmZiIsImlhdCI6MTY5NzE5MjY1MH0.jzMx2UQ_5249mW5-44FJfXBNr7Q6CaVc6Ka0Ay2BiW8`**

Sau đó mình decode JWT

![image.png](/assets/images/redteam/Recon/services/image%208.png)

**`CBJS{CBJSStaff-1697192650`**

> Target 3: Thu thập thông tin về tập tin zip được chia sẻ, bao gồm:
> 
> - **`zip_password`**: Mật khẩu giải nén file zip
> - **`framework`**: Tên framework được sử dụng trong **`requirements.txt`**

Lại mò tiếp trên ftp server. 

![image.png](/assets/images/redteam/Recon/services/image%209.png)

Sau khi lấy dìa thì ko giải nén được 😑 

![image.png](/assets/images/redteam/Recon/services/image%2010.png)

Mình đoán là nó có thể có mật khẩu rồi. 

Nếu mà như vậy thì mình dùng tool **`zip2john`** để xác định xem nếu có mật khẩu thì sẽ có hash.

![image.png](/assets/images/redteam/Recon/services/image%2011.png)

Cái hash khá bự nên mình cho nó vào 1 file. 

Sau đó mình cần crack cái hash này → **`john`**

![image.png](/assets/images/redteam/Recon/services/image%2012.png)

**`CBJS{letmein-flask}`**

### Tools

---

- exiftool
- strings
- john
- zip2john
- file

## 📂 SMB

---

Mình truy cập vào **`files.cj23group.com`** với port **`4445`**

> Target 1: Thu thập credential để kết nối đến database trong tập tin **`.ps1`** được chia sẻ
> 

Dựa vào credentials tìm thấy trong file ở trên ftp server thì mình truy cập vào SMB. 

![image.png](/assets/images/redteam/Recon/services/image%2013.png)

Mình thấy có share ổ **`Shared`** mình truy cập thử 😎 

![image.png](/assets/images/redteam/Recon/services/image%2014.png)

Sau đó mình đọc file mới lấy về. 

![image.png](/assets/images/redteam/Recon/services/mage%2015.png)

**`CBJS{dbadmin-dd4A5FMHk4s1D9VV-dbserver.cj23group.com}`** 

> Target 2: Thu thập credential được hardcode trong tập tin **`.exe`** được chia sẻ
> 

Tiếp tục tìm kiếm thui 😎 

![image.png](/assets/images/redteam/Recon/services/image%2016.png)

Sau đó mình check bằng **`strings`** 

![image.png](/assets/images/redteam/Recon/services/image%2017.png)

**`CBJS{SecurePass123`** 

> Mục tiêu 3:
Target: Thu thập master password của Keepass Database
> 

Ở trên thì mình đã lấy được file **`test_account.kdbx`** 

Mình check nó bằng **`strings`** lun 😎

![image.png](/assets/images/redteam/Recon/services/image%2018.png)

Lại có hash mật khẩu rồi :”> 

Nhưng quan sát thì đuôi file là **`.kdbx`** thì đây có thể là **`keepass`** 😎

Cho nên mình crack nó bằng **`keepass2john`** 

![image.png](/assets/images/redteam/Recon/services/image%2019.png)

**`CBJS{123321}`** 

### Tools

---

- strings
- keepass2john

## 🕸️ Service Web

---

> Target 1: Xác định tên và version của framework mà trang blog sử dụng
> 

Link: **`http://blog.cj23group.com`** 

Đầu tiên sau khi truy cập vào web thì vào dashboard. 

![image.png](/assets/images/redteam/Recon/services/image%2020.png)

Sau đó, mình có thể dùng tool để phân tích.

![image.png](/assets/images/redteam/Recon/services/image%2021.png)

Quan sát các phân tích thì web dùng **joomla**, chạy trên webserver là **Nginx** và host OS là **ubuntu** với server backend dùng **PHP 5.6.26**

Thì **joomla** là một **CMS** nên sẽ có các file xml để tóm tắt các thông tin.

Cho nên mình sẽ cho nó scan qua wordlist **CMS joomla.** Nhưng trước đó mình nên tìm các đường dẫn trên wordlist **common.txt.**

![image.png](/assets/images/redteam/Recon/services/image%2022.png)

Nhìn thấy có **administrator** là khả nghi nhất nên tiếp tục scan này.

![image.png](/assets/images/redteam/Recon/services/image%2023.png)

Truy cập thử thì có vẻ ko ra kết quả gì 

![image.png](/assets/images/redteam/Recon/services/image%2024.png)

Cho nên mình phải đổi là ko scan tại **administrator nữa mà scan ngay từ đầu lun**.

![image.png](/assets/images/redteam/Recon/services/image%2025.png)

Sau đó mình scan thêm thì được đường dẫn này.

![image.png](/assets/images/redteam/Recon/services/image%2026.png)

Từ đây mình truy cập qua đường dẫn này. 

![image.png](/assets/images/redteam/Recon/services/image%2027.png)

**`CBJS{joomla_3.6.2}`** 

> Target 2: Xác định version của plugin JCE đang được cài đặt trên trang blog
> 

Mình dùng tool **`ffuf`** cùng với wordlist **`joomla`** **CMS** → thu được như sau:

![image.png](/assets/images/redteam/Recon/services/image%2028.png)

Và để xác định được version của plugins thì mình đã tìm thấy 1 bài post để xác định version plugins.

- **https://joomla.stackexchange.com/questions/24882/how-to-find-the-version-of-extensions-used-on-a-joomla-website-without-access-to**

Làm theo bài blog trên thì mình thu được kết quả thoai 😎 

![image.png](/assets/images/redteam/Recon/services/image%2029.png)

**`CBJS{2.9.14}`** 

> Link: **`http://cj23group.com/`**
> 
> 
> Target 3: Tìm cách đọc tất cả tin nhắn mà khách hàng đã gửi cho **`CJ23Group`** thông qua contact form
> 

Sau khi vào web thì đây là giao diện

![image.png](/assets/images/redteam/Recon/services/image%2030.png)

Quan sát thì mình thấy nó chỉ có mỗi chức năng **Contact**

![image.png](/assets/images/redteam/Recon/services/image%2031.png)

Khi tương tác thì mình thấy nó gọi qua **`dev-api.cj23group.com`** 

![image.png](/assets/images/redteam/Recon/services/image%2032.png)

Nên mình sẽ mò qua **dev-api.cj23group.com** 

![image.png](/assets/images/redteam/Recon/services/image%2033.png)

Rồi mình lại `ffuf` nó để tìm manh mối

![image.png](/assets/images/redteam/Recon/services/image%2034.png)

Mình thấy có endpoint này khá uy tín nên mình truy cập thử 😎 

![image.png](/assets/images/redteam/Recon/services/image%2035.png)

Và mình tập trung vào **GET /forms** thì mình muốn đọc thông tin. Mà để đọc được thì phải cần có xác thực quyền qua 1 token.

Ở đây thì khi quan sát nhìn thấy ở hai chỗ.

- Một là lưu trong file js tại official website

![image.png](/assets/images/redteam/Recon/services/image%2036.png)

- Hai là lưu khi gửi form

![image.png](/assets/images/redteam/Recon/services/image%2037.png)

Cho nên mình sẽ tương tác nó qua endpoint **forms** 

![image.png](/assets/images/redteam/Recon/services/image%2038.png)

> Target 3: Tìm kiếm credential để login vào trang quản trị của website này
> 

Như vậy mình thực hiện fuzz trên web api này. 

![image.png](/assets/images/redteam/Recon/services/image%2039.png)

Mình dùng **`git-dumper`** để lấy source. Sau đó mình tìm kiếm trong source.

Khi có source thì mình tiến hành tìm kiếm. 

![image.png](/assets/images/redteam/Recon/services/image%2040.png)

Sau đó thì mình login vào trong dashboard.

![image.png](/assets/images/redteam/Recon/services/image%2041.png)

> Target 5: Tìm kiếm credential để login vào database của trang web
> 

Từ source trên thì mình cần quan sát file **`docker-compose.yml`** 

![image.png](/assets/images/redteam/Recon/services/image%2042.png)

Cho nên mình thử chạy docker rồi exec vào trong. Sau đó mình đọc được file **`.env`**

![image.png](/assets/images/redteam/Recon/services/image%2043.png)

Từ đó mình truy cập vào db host này và login vào.

![image.png](/assets/images/redteam/Recon/services/image%2044.png)