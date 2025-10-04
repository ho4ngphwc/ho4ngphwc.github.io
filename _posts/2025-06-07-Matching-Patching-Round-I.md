---
layout: post
title:  "Game Matching Patching Round 1"
date:   2025-06-07 16:37:00 +0700
tags: CBJS 
categories: jekyll update
---

# GAME MATCHING PATCHING ROUND 1  
Các bug được thể hiện dưới đây: 
![patch](https://hackmd.io/_uploads/B1mz2tWNxl.png)
Các patching được thể hiện dưới đây: 
![patch_1](https://hackmd.io/_uploads/r1-EnY-Egl.png)

## Chọn các patching theo bug để đảm bảo an toàn cho code
Đáp theo team_4 chọn như sau: A5 B4 C2 

## Sau đó được cung cấp link các đội
![image](https://hackmd.io/_uploads/r1Zf6tb4ex.png)

## FLAG 3 (SSRF) -> Nằm trong file `flag.php` tại thư mục gốc của webserver

![image](https://hackmd.io/_uploads/r1FlNqbNel.png)

Mình cần encode URL hai lần --> localhost 

## FLAG 2 (Path_Traversal) -> Nằm trong tập tin `/FLAG_PATH_TRAVERSAL`

Quan sát source sau, nó dựa trên bug kết hợp patch. 

![image](https://hackmd.io/_uploads/HJfRN9-Ell.png)

Mình thấy biến trong Class `Scroll` có hàm `__toString()` gọi tới hàm `file_get_contents()`.

Như vậy nếu biến `secret` chứa dấu đường dẫn thì nó có thể thực hiện Path Traversal rồi. 

Nhưng mình phải làm sao để nó gọi tới `__toString` và phải vượt qua được `validate()`

![image](https://hackmd.io/_uploads/SyA9H5-Vee.png)

Mình chạy serialize thì được như sau: 

![image](https://hackmd.io/_uploads/HJATSqbVxl.png)

Nhưng sau đó, mình phải chỉnh lại `s:44` thành `s:31`. Vì sau khi bị filter thì nó mất đi ký tự thì phải chỉnh lại độ dài của String. 

![image](https://hackmd.io/_uploads/HkiT8cZ4ee.png)

## FLAG 1 (SQL Injection) -> Nằm trong một bảng bí mật của DB 

Source như sau:
![image](https://hackmd.io/_uploads/ryh7wqbNxe.png)

Đọc qua thì thấy mắc lỗi SQLi tại tham số `name`, nhưng mình đã bị chặn hầu hết các từ khóa để có thể khai thác tên bảng. 

Và mình có thể khai thác được tên db bằng `extractvalue()`

![image](https://hackmd.io/_uploads/SkChO9-Vlx.png)

Cho nên patch này hoàn toàn có thể chặn được SQLi. 



