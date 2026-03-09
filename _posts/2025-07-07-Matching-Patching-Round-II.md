---
layout: post
title:  "Game Matching Patching Round 2"
date:   2025-07-07 12:00:00 +0700
tags: CBJS 
categories: jekyll update
---

# GAME MATCHING PATCHING ROUND 2 
Các bug được thể hiện dưới đây: 

![image](/assets/images/WPT/matching-patching-round2/image1.png)

Các matching được thể hiện dưới đâu: 

![image](/assets/images/WPT/matching-patching-round2/image2.png)

## Chọn các patching theo bug để đảm bảo an toàn cho code
Đáp theo team_4 chọn như sau: A12 B10 C6 

## Sau đó được cung cấp link các đội
![image](/assets/images/WPT/matching-patching-round2/image3.png)

## FLAG 1 (SQL Injection) -> Nằm trong một bảng bí mật của DB 

![image](/assets/images/WPT/matching-patching-round2/image4.png)

Nếu team nào chọn patch này để chặn thì hoàn toàn bị khai thác SQLi. 

Mấu chốt nằm ở câu lệnh `INSERT INTO`

```bash=mysql
INSERT INTO candidates(name, village, status) VALUES ('$name', 'Konoha', 'Tham gia')
```

Và biến `$name` hoàn toàn do người dùng kiểm soát. Và nếu truyền payload vào để đóng chuỗi thì nó như sau: 

![image](/assets/images/WPT/matching-patching-round2/image5.png)

Lúc đó câu lệnh query sẽ là: 

![image](/assets/images/WPT/matching-patching-round2/image6.png)

![image](/assets/images/WPT/matching-patching-round2/image7.png)

Như vậy thì mình hoàn toàn triển khai thêm payload để lấy tên bảng và tên cột. Rồi trích xuất thông tin từ cột và bảng đáng nghi. 

![image](/assets/images/WPT/matching-patching-round2/image8.png)
Nhưng mình bị chặn từ `jutsu` cho nên mình thay bằng `JUTSU`

![image](/assets/images/WPT/matching-patching-round2/image9.png)

## FLAG 3 (SSRF) -> Nằm trong tập tin `flag.php` tại thư mục gốc của webserver 

![image](/assets/images/WPT/matching-patching-round2/image10.png)

Source code cho thấy rằng chặn khá ổn. 

Nhưng có 1 kỹ thuật là khi thêm 1 dấu chấm ở cuối thì vẫn được hiểu đúng. 

Nghĩa là `localhost.` vẫn được hiểu là `localhost`

![image](/assets/images/WPT/matching-patching-round2/image11.png)

Và ở team_4 cũng chọn như sau:

![image](/assets/images/WPT/matching-patching-round2/image12.png)

Nhưng patch này thì hoàn toàn ko thể chặn `SSRF` --> hoàn toàn khai thác dễ dàng. 

![image](/assets/images/WPT/matching-patching-round2/image13.png)

Mà nó còn dẫn đến lỗi đọc file do ko chặn các từ khóa quan trọng. 

## FLAG 2 (PATH_TRAVERSAL) -> Nằm tại tập tin (`/FLAG_PATH_TRAVERSAL`)

![image](/assets/images/WPT/matching-patching-round2/image14.png)

Trong Source code này hoàn ko thể bypass đoạn regex đã chặn nghiêm ngặt. 

Nhưng nó có đoạn `unserialize_callback_func` nghĩa là nếu unserialize 1 class ko định nghĩa ở ngoài thì nó sẽ gọi tới hàm `loadMissing`.

Nhưng mình cần cung cấp đúng tên class trong hàm `loadMissing` thì hoàn toàn có flag.

Lợi dụng vào lỗ hổng SSRF của team_4 thì có thể đọc file từ đó có thể biết được tên class đã bị che đó. 

![image](/assets/images/WPT/matching-patching-round2/image15.png)

Mình biết thêm rằng có 1 folder `validations/10.php` như vậy mình sẽ đọc nó.

![image](/assets/images/WPT/matching-patching-round2/image16.png)

Như vậy, mình đã biết được class bị che là `System`

Cuối cùng, mình truyền payload đã serialize như sau: 

![image](/assets/images/WPT/matching-patching-round2/image17.png)

Và payload mình phải bọc bên trong 1 mảng để có thể bypass regex chặn `/^O:\d+:.*/s`

Nhưng ở team_3 cũng chọn patch như sau: 

![image](/assets/images/WPT/matching-patching-round2/image18.png)

Như vậy thì mình có thể bypass bằng payload này: 

![image](/assets/images/WPT/matching-patching-round2/image19.png)

Sau đó, mình thay đổi `../../../../FLAG_PATH_TRAVERSAL` thành hexdemical vì trong php hoàn toàn có thể hiểu được. 

![image](/assets/images/WPT/matching-patching-round2/image20.png)

Payload lúc truyền nó sẽ như sau: 
![image](/assets/images/WPT/matching-patching-round2/image21.png)
