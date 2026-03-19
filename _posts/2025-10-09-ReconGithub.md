---
layout: post
title: Recon Github
date: 2025-10-09 8:13:00 +0700
tags: redteam Recon 
categories: jekyll update
---

# Recon Github

> Mục tiêu 1: tìm đến đường dẫn một trang blog cũ của **`CJ23Group`**
> 

### Google

---

Đầu tiên ta thực hiện search google về từ khóa **`CJ23Group`** + **`github`** 

![image.png](/assets/images/redteam/Recon/github/image.png)

Mình vào thử trong trang xem thử có gì ?

![image.png](/assets/images/redteam/Recon/github/image%201.png)

Tại đây thì mình chỉ thấy có một chức năng **`CONTACT US`** 

![image.png](/assets/images/redteam/Recon/github/image%202.png)

### Github Search

---

Có vẻ là không có thông tin gì → nên mình dùng **`Github Search`** 

![image.png](/assets/images/redteam/Recon/github/image%203.png)

Mình tìm ra repo có liên quan 😎 → nhảy vào xem thử

![image.png](/assets/images/redteam/Recon/github/image%204.png)

Có vẻ đây đúng là repo dành cho web blog trên. 

Khi tìm ra github thì mình nên vào đọc commit.

![image.png](/assets/images/redteam/Recon/github/image%205.png)

Đọc qua các commit thì mình tìm dược đoạn như sau: 

![image.png](/assets/images/redteam/Recon/github/image%206.png)

Thì đây có vẻ đường link cũ của blog này 

![image.png](/assets/images/redteam/Recon/github/image%207.png)

**`CBJS{http://blog.cj23group.com/innovation-projects-and-blog/}`**

> Mục tiêu 2: Tìm Github cá nhân của nhân viên thuộc tổ chức **`CJ23Group`**
> 

Thì trong commit lúc thì quan sát xem ai đã commit 

![image.png](/assets/images/redteam/Recon/github/image%208.png)

Đó là **`ak4txuk1`** → cho nên mình tìm về profile user này trên github **`https://github.com/ak4txuk1`**

![image.png](/assets/images/redteam/Recon/github/image%209.png)

**`CBJS{https://github.com/ak4txuk1}`**

> Mục tiêu 3: tìm username/password trong repo của nhân viên
> 

Từ profile này thì mình thấy có một repo 😎

Mình tiến hành search trong repo này thôi 😀 

![image.png](/assets/images/redteam/Recon/github/image%2010.png)

> Mục tiêu 4:  Xác định thời gian anh nhân viên ở Lab 1.2 đã bất cẩn commit credential lên repo **`php-todo-list`**.
> 

Giờ mình xác định thời gian đã commit credential lên github. 

Nó nằm trong **`todo.class.php`** 

![image.png](/assets/images/redteam/Recon/github/image%2011.png)

Nhưng sau khi đi vào phần commit thì quá nhiều commit 😣 Cho nên mình sẽ dùng 1 tool 😎 → **`gitleaks`** 

![image.png](/assets/images/redteam/Recon/github/image%2012.png)

Sau đó mình đọc **`report.json`** 

Mình phát hiện ra khá nhiều thứ thú vị 😎 

![image.png](/assets/images/redteam/Recon/github/image%2013.png)

`CBJS{2023-09-14}`
