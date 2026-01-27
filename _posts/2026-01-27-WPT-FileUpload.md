---
layout: post
title:  "File Upload"
date:   2026-01-27 10:37:00 +0700
tags: WPT
categories: jekyll update
---

# File Upload

## 🧪 Level 1

---

![image.png](/assets/images/WPT/File-Upload/image.png)

Đầu tiên ta thực hiện upload 1 file **`.txt`** xem nó xử lý như thế nào? 

![image.png](/assets/images/WPT/File-Upload/image%201.png)

Mình truy cập vào nơi nó lưu file đã upload. 

![image.png](/assets/images/WPT/File-Upload/image%202.png)

Như vậy nó thực hiện in ra nội dung file mình đã upload. 

Để biết nó xử lý như thế nào thì mình nên đọc source của level 1 này. 

![image.png](/assets/images/WPT/File-Upload/image%203.png)

Nhìn vào thì thấy các vấn đề quan trọng.

- nơi lưu tại **`/upload/<session_id>/`**
- Không chặn các file thực thi như **`.php`**

Như vậy ta thực hiện upload 1 file **`.php`** với nội dung in ra **`phpinfo();`** 

![image.png](/assets/images/WPT/File-Upload/image%204.png)

Minh thực hiện truy cập. 

![image.png](/assets/images/WPT/File-Upload/image%205.png)

Đến đây thì mình thực hiện upload 1 file shell. 

![image.png](/assets/images/WPT/File-Upload/image%206.png)

Sau khi truy cập vào file shell.

![image.png](/assets/images/WPT/File-Upload/image%207.png)

## 🧪 Level 2

---

![image.png](/assets/images/WPT/File-Upload/image%208.png)

Tương tự như mình cũng đọc source của level 2 này. 

![image.png](/assets/images/WPT/File-Upload/image%209.png)

Level 2 này đã chặn mình upload file **`.php`** bằng cách kiểm tra

```php
**$extension = explode(".", $filename)[1];
if ($extension === "php") {
   die("Hack detected");
}**
```

Nó dùng hàm **`explode`** để phân tích dấu **`.`** sau đó lấy index **`[1]`** là đuôi file. Nhưng thực tế thì đâu có chỉ đuôi file **`.php`** mới được xử lí code **`php`**. 

Quan sát config trên apache server. 

![image.png](/assets/images/WPT/File-Upload/image%2010.png)

Như vậy ngoài file **`.php`** thì còn có **`.phar`** , **`.phtml`** cũng có khả năng thực hiện code **`php`**

Mình thực hiện upload file **`.phar`** lên server với nội dung là tương tự như level 1.

![image.png](/assets/images/WPT/File-Upload/image%2011.png)

Sau đó mình thực hiện truy cập vào.

![image.png](/assets/images/WPT/File-Upload/image%2012.png)

## 🧪 Level 3

---

![image.png](/assets/images/WPT/File-Upload/image%2013.png)

Đến level 3 cách ngăn chặn đã thay đổi.

```php
**$extension = end(explode(".", $filename));
if ($extension === "php") {
    die("Hack detected");
}**
```

Cách kiểm chuyển sang dùng thêm hàm **`end()`** thì nó cho phép lấy cuối cùng, lấy cuối cùng mà tận cùng là **`php`** thì nó sẽ chặn. 

Nhưng vẫn có cách bypass → bằng cách mình chỉ cần đổi đuôi file cuối cùng không phải **`php`** là được.

![image.png](/assets/images/WPT/File-Upload/image%2014.png)

Sau đó mình truy cập vào shell đã cắm.

![image.png](/assets/images/WPT/File-Upload/image%2015.png)

## 🧪 Level 4

---

![image.png](/assets/images/WPT/File-Upload/image%2016.png)

Với level này thì nó đã chặn các đuôi file là **`.php`** , **`.phtml`**  và **`.phar`** và nó đã thực hiện blacklist.

Nhưng việc thực hiện blacklist rất dễ bỏ xót các file. Và đặc biệt là file **`.htaccess`** 

Nhưng để đảm bảo thì mình tìm đọc config của server có cho phép file **`.htaccess`** thực hiện hay không.

![image.png](/assets/images/WPT/File-Upload/image%2017.png)

Nhìn thấy thì server cho phép → mình thực hiện upload file **`.htaccess`** để server thực hiện cho phép đuôi file khác thực hiện code backend.

![image.png](/assets/images/WPT/File-Upload/image%2018.png)

Sau đó mình up shell với đuôi file **`.txt`** 

![image.png](/assets/images/WPT/File-Upload/image%2019.png)

Giờ thì mình tiến hành truy cập. 

![image.png](/assets/images/WPT/File-Upload/image%2020.png)

## 🧪 Level 5

---

![image.png](/assets/images/WPT/File-Upload/image%2021.png)

Đối với level 5 này thì cho phép mình upload file hình ảnh. Nhưng nếu khác các **`mime_type`** trong mảng cho phép thì nó sẽ bị detect.

Như vậy nó chỉ kiểm tra `Content-Type` có nằm trong danh sách cho phép hay không.

Như vậy nếu mình upload 1 file giả dạng phần **`Content-Type`** nhưng code **`php`** thì sao. 

![image.png](/assets/images/WPT/File-Upload/image%2022.png)

Sau đó mình truy cập vào file này.

![image.png](/assets/images/WPT/File-Upload/image%2023.png)

## 🧪 Level 6

---

![image.png](/assets/images/WPT/File-Upload/image%2024.png)

Đối với level 6 này thì cho phép mình upload file hình ảnh. Nhưng nếu khác các **`mime_type`** trong mảng cho phép thì nó sẽ bị detect và nội dung bên trong file.

Đối với các file hình ảnh này để server có thể nhận biết thì dựa vào các magic bytes → đối với **`JPEG`** , **`PNG`** và **`GIF`** luôn có các magic bytes riêng biệt.

Như vậy nếu mình upload 1 file có magic bytes giả dạng nhưng code **`php`** thì sao. 

![image.png](/assets/images/WPT/File-Upload/image%2025.png)

Sau đó mình truy cập vào khi mình mới upload.

![image.png](/assets/images/WPT/File-Upload/image%2026.png)

### ✨ Cách phòng chống file upload

Upload file an toàn cần:

- Dùng **whitelist extension**
- Chặn **path traversal trong filename**
- **Đổi tên file** khi lưu
- **Validate xong mới lưu thật**
- **Dùng framework chuẩn**, tránh code tay