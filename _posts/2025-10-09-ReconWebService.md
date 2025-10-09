---
layout: post
title: "Recon Web Service"
time: 2025-10-09 10:11:00 +0700
tags: redteam Recon
---

# Recon Web Service 

![image](https://hackmd.io/_uploads/SJgA3tE6eg.png)

Mình truy cập vào **http://blog.cj23group.com/**

![image](https://hackmd.io/_uploads/H1Qw6KE6ee.png)

Quan sát các phân tích thì web dùng **joomla**, chạy trên webserver là **Nginx** và host là **ubuntu** với server backend dùng **PHP 5.6.26**

![image](https://hackmd.io/_uploads/H1DtaFETex.png)

Thường thì sẽ có các file xml để có thể tóm tắt các thông tin vì nhìn chung **joomla** cũng là 1 CMS. 

Sau khi scan thì mình tìm thấy 1 đường dẫn như sau: 

![image](https://hackmd.io/_uploads/S1eqX5NTxl.png)

![image](https://hackmd.io/_uploads/BkRjXc4pxe.png)

![image](https://hackmd.io/_uploads/r1q6Q546le.png)

Mình tiếp tục đến scan plugins trên này. 

![image](https://hackmd.io/_uploads/Hkoa4c46xl.png)

Và để xác định được version của plugins thì mình đã tìm thấy 1 bài post để xác định version plugins. 

- https://joomla.stackexchange.com/questions/24882/how-to-find-the-version-of-extensions-used-on-a-joomla-website-without-access-to 

![image](https://hackmd.io/_uploads/SJ4rHqE6gg.png)

![image](https://hackmd.io/_uploads/BkXIB946el.png)

Giờ thì mình sẽ thực hiện trên host **http://cj23group.com**

![image](https://hackmd.io/_uploads/BkYJDc46ex.png)

Quan sát thì có chức năng **Contact us** là tương tác được. 

![image](https://hackmd.io/_uploads/rkMfD5N6xl.png)

Khi tương tác thì nó gọi tới **dev-api.cj23group.com**

![image](https://hackmd.io/_uploads/BJ7iwqN6lg.png)

Truy cập vào host này.

![image](https://hackmd.io/_uploads/HygTY5Vaxg.png)

Cho nên mình tiếp tục scan về host này. 

![image](https://hackmd.io/_uploads/H11Av5E6gl.png)

![image](https://hackmd.io/_uploads/r1Wg554pxx.png)

Ở mục default thì có các mục 

![image](https://hackmd.io/_uploads/r1f4694alg.png)

Và mình tập trung vào **GET /forms** thì mình muốn đọc thông tin. Mà để đọc được thì phải cần có xác thực quyền qua 1 token. 

Ở đây thì khi quan sát nhìn thấy ở hai chỗ. 

Một là lưu trong file js tại official website.

![image](https://hackmd.io/_uploads/B1DsT9Naxg.png)

Hai là lưu khi gửi form 

![image](https://hackmd.io/_uploads/HJByA5E6xg.png)

Cho nên mình truyền vào token để tương tác endpoint. 

![image](https://hackmd.io/_uploads/SkagC5E6gg.png)

![image](https://hackmd.io/_uploads/Hyi6bsETee.png)

Tiếp tục scan trên host **dev-api.cj23group.com**

![image](https://hackmd.io/_uploads/BkLWGsEpel.png)

Mình dùng `git-dumper` để lấy source. Sau đó mình tìm kiếm trong source. 

![image](https://hackmd.io/_uploads/H1E-Xo4peg.png)

Sau đó mình login vào 

![image](https://hackmd.io/_uploads/B1iDmo4aeg.png)

![image](https://hackmd.io/_uploads/S1OlEsVaxe.png)

Quan sát thì thấy có file `docker-compose.yml`

![image](https://hackmd.io/_uploads/HJ3X4oV6ge.png)

Cho nên mình thử chạy docker rồi exec vào trong. 

Sau đó mình đọc được file .env 

![image](https://hackmd.io/_uploads/B1eSvs46ge.png)

Từ đó mình truy cập host này và login vào. 

![image](https://hackmd.io/_uploads/SkfvDoNael.png)
