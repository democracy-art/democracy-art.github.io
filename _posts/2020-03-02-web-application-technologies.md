---
layout:     post
title:      Web应用程序技术 - Chapter 3(1) - HTTP
subtitle:   
date:       2020-03-02
author:     D
header-img: 
catalog: true
mermaid: true
tags:
    - [web hacking]
---

参考:*The Web Application Hacker's Handbook* 
Chapter 3 Web Application Technologies

关于Web应用程序技术查询建议去[W3C](http://w3.org).

# 1. HTTP协议(The HTTP Protocol)

**HTTP:**超文本传输协议(Hyper Text Transfer Protocol).<br>
HTTP使用基于消息的模型(message-based model)发送信息,该协议本质上是无连接的.<br>
无连接:虽然HTTP使用TCP作为底层传输的机制,但是每一次请求和响应都是自主传输,<br>
可能会用不同的TCP连接.

## 1.1 HTTP Requests(HTTP请求)
所有HTTP消息(请求和响应)都由一个或多个header组成，每个header位于单独的行上，后面是强制的空白行，后面是可选的消息正文。典型的HTTP请求如下所示:
```
GET /auth/488/YourDetails.ashx?uid=129 HTTP/1.1
Accept: application/x-ms-application, image/jpeg, application/xaml+xml,
image/gif, image/pjpeg, application/x-ms-xbap, application/x-shockwave-
flash, */*
Referer: https://mdsec.net/auth/488/Home.ashx
Accept-Language: en-GB
User-Agent: Mozilla/4.0 (compatible; MSIE 8.0; Windows NT 6.1; WOW64;
Trident/4.0; SLCC2; .NET CLR 2.0.50727; .NET CLR 3.5.30729; .NET CLR
3.0.30729; .NET4.0C; InfoPath.3; .NET4.0E; FDM; .NET CLR 1.1.4322)
Accept-Encoding: gzip, deflate
Host: mdsec.net
Connection: Keep-Alive
Cookie: SessionId=5B70C71F3FD4968935CDB6682E545476
```

每个HTTP请求的第一行由三个条目组成，用空格分隔:<br>
- 表示HTTP方法的动词
- 请求的URL
- 使用的HTTP的版本.常用版本是`1.0`和`1.1`,大多数浏览器默认使用`1.1`,区别是`1.1`中`Host`是必需的.

其他有意思的 HTTP Header:
- `Referer`:用于指示发出请求的URL(例如:因为用户单击了页面上的链接).注意,这个头在最初的HTTP规范中拼错了,应该是`referrer`才对,但从那以后这个拼错的版本就一直保着.
- `User-Agent`:用于提供有关生成请求的浏览器或其他客户机软件的信息.注意，由于历史原因，大多数浏览器都包含Mozilla前缀。这是最初占主导地位的Netscape浏览器使用的用户代理字符串，其他浏览器希望向网站断言它们与此标准兼容.
- `Host`:指定在被访问的完整URL中出现的主机名。当多个网站托管在同一台服务器上时，这是必要的，因为在请求的第一行发送的URL通常不包含主机名。
- `Cookie`:用于提交服务器已发送给客户机的附加参数.

## 1.2 HTTP Responses(HTTP响应)
一个典型HTTP响应如下:
```
HTTP/1.1 200 OK
Date: Tue, 19 Apr 2011 09:23:32 GMT
Server: Microsoft-IIS/6.0
X-Powered-By: ASP.NET
Set-Cookie: tracking=tI8rk7joMx44S2Uu85nSWc
X-AspNet-Version: 2.0.50727
Cache-Control: no-cache
Pragma: no-cache
Expires: Thu, 01 Jan 1970 00:00:00 GMT
Content-Type: text/html; charset=utf-8
Content-Length: 1067
<!DOCTYPE html PUBLIC “-//W3C//DTD XHTML 1.0 Transitional//EN” “http://
www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd”><html xmlns=”http://
www.w3.org/1999/xhtml” ><head><title>Your details</title>
...
```
每个HTTP响应的第一行由三个条目组成，用空格分隔:<br>
- 正在使用的HTTP版本
- 指示请求结果的数字状态码
- 进一步描述响应状态的文本推理短语。它可以有任何值，当前浏览器不会将其用于任何目的

以下是该回复中其他一些有趣的地方:
- `Server`:指示正在使用的web服务器软件，有时还包含其他细节，如已安装的模块和服务器操作系统。所载的资料可能正确，也可能不正确。
- `Set-Cookie`:向浏览器发送另一个cookie;这将在后续请求的Cookie头中提交回此服务器。
- `Pragma`:指示浏览器不要将响应存储在缓存中
- `Expires`:指示响应内容在过去过期，因此不应该缓存。这些指令通常在返回动态内容时发出，以确保浏览器在后续情况下获得该内容的新版本。
- `Content-Type`:表示此消息的包含的文档类型(本例中是:HTML文档).
- `Content-Length`:标头表示消息体的长度(以`字节`为单位).

## 1.3 HTTP Methods
- `GET`:用于获取资源.
- `POST`:用于执行操作.
- `HEAD`:获得报文首部.HEAD方法和GET方法一样，只是不返回报文的主体部分，用于确认`URI`的`有效性`及资源`更新`的日期时间等。
- `TRACE`:追踪路径.客户端可以对请求消息的传输路径进行追踪，TRACE方法是让Web服务器端将之前的请求通信还给客户端的方法.
- `OPTIONS`:询问支持的方法.OPTIONS方法用来查询针对请求URI指定资源支持的方法（客户端询问服务器可以提交哪些请求方法）
- `PUT`:传输文件.尝试使用请求体中包含的内容将指定的资源上载到服务器。如果启用了此方法，则可以利用它来攻击应用程序，例如通过上载任意脚本并在服务器上执行它.

## 1.4 URLs
URL:Uniform Resource Locator(统一资源定位符),大多数URLs格式如下:
```
protocol://hostname[:port]/[path/]file[?param=value]
```
## 1.5 REST
`REST`:具象状态传输(Representational State Transfer)是一种分布式系统的体系结构，其中`请求`和`响应`包含系统资源当前状态的表示。万维网中使用的核心技术，包括HTTP协议和URL格式，都符合REST架构风格。
## 1.6 HTTP Header
### 1.6.1 General Headers
- `Connection`:是一种分布式系统的体系结构，其中请求和响应包含系统资源当前状态的表示。万维网中使用的核心技术，包括HTTP协议和url格式，都符合REST架构风格。
- `Content-Encoding`:指定对消息体中包含的内容使用何种编码，例如`gzip`，一些应用程序使用它来压缩响应以实现更快的传输。
- `Content-Length`:指定消息体的长度(以字节为单位)
- `Content-Type`:指定消息体中包含的内容类型，如用于HTML文档的`text/html`。
- `Transfer-Encoding`:指定在消息体上执行的任何编码，以促进其通过HTTP传输。当使用分块编码时，通常使用它来指定分块编码。

### 1.6.2 Request Headers
- `Accept`:告诉服务器客户机愿意接受什么类型的内容，如图像类型、办公文档格式等。
- `Accept-Encoding`:告诉服务器客户机愿意接受哪种类型的内容编码。
- `Authorization`:向服务器提交内置HTTP身份验证类型之一的凭据。
- `Cookie`:将cookie提交到服务器之前发出的服务器。
- `Host`:指定在被请求的完整URL中出现的主机名
- `If-Modified-Since`:指定浏览器最后一次收到请求资源的时间。如果从那时起资源没有改变，服务器可以指示客户端使用它的缓存副本，使用带有状态码`304`的响应。
- `If-None-Match`:指定实体标记，它是表示消息正文内容的标识符。浏览器提交实体标记，该标记是服务器在最后一次接收到请求的资源时发出的。服务器可以使用实体标记来确定浏览器是否可以使用其缓存的资源副本。
- `Origin`:在跨域Ajax请求中使用，以指示发出请求的域
- `Referer`:指定当前请求起源的URL。
- `User-Agent`:提供有关生成请求的浏览器或其他客户端软件的信息。

### 1.6.3 Response Headers
- `Access-Control-Allow-Origin`:指示是否可以通过跨域Ajax请求检索资源
- `Cache-Control`:向浏览器传递缓存指令(例如，`no-cache`)。
- `Etag`:指定实体标记。在将来的请求中，客户端可以在If-None-Match标头中提交这个标识符，以通知服务器浏览器当前在其缓存中保存的资源的版本。
- `Expires`:告诉浏览器消息体的内容有效多长时间。在此之前，浏览器可以使用此资源的缓存副本。
- `Location`:用于重定向响应(状态码从3开始的响应)，以指定重定向的目标。
- `Pragma`:向浏览器传递缓存指令(例如，`no-cache`)。
- `Server`:提供有关正在使用的web服务器软件的信息。
- `Set-Cookie`:向浏览器发出cookie，然后在后续请求中将其提交回服务器。
- `WWW-Authenticate`:在具有`401`状态码的响应中使用，以提供服务器支持的身份验证类型的详细信息。
- `X-Frame-Options`:指示是否以及如何在浏览器框架中加载当前响应(see Chapter 13)。

## 1.7 Cookies
除了cookie的实际值之外，Set-Cookie头还可以包含以下任意可选属性，可用于控制浏览器如何处理cookie:
- `expires`:设置cookie有效之前的日期。这将导致浏览器将cookie保存到持久存储中，并在后续浏览器会话中重用它，`直到`达到`过期`日期。如果未设置此属性，则cookie仅在`当前`浏览器会话中使用。
- `domain`:指定cookie对其有效的域。它必须是接收cookie的相同的域或父域。
- `path`:指定cookie有效的URL路径。
- `secure`:如果设置了此属性，cookie将仅在HTTPS请求中提交。
- `HttpOnly`:如果设置了此属性，则**无**法通过客户端JavaScript直接访问cookie。

## 1.8 Status Codes
每个HTTP响应消息的第一行中必须包含一个状态码，以确定请求的结果。根据代码的第一个数字，状态码分为五组：
- `1xx`:信息
- `2xx`:请求成功
- `3xx`:客户端被重定向到另一个资源
- `4xx`:该请求包含某种类型的错误
- `5xx`:服务器在执行请求时遇到错误

以下是您在攻击web应用程序时最有可能遇到的状态代码，以及与它们相关的常见原因短语:
- `100 Continue`:在某些情况下，当客户端提交包含主体的请求时发送。响应表明已经接收到请求头，客户机应该继续发送正文。当请求完成时，服务器返回第二个响应。
- `200 OK`:指示请求已成功，且响应主体包含请求的结果。
- `201 Created`:在响应`PUT`请求时返回，以指示请求已成功。
- `301 Moved Permanently`:将浏览器永久重定向到一个不同的URL，该URL在Location头文件中指定。将来客户机应该使用新的URL，而不是原来的URL。
- `302 Found`:将浏览器临时重定向到一个不同的URL，该URL在Location头文件中指定。在随后的请求中，客户机应该恢复到原始URL。
- `304 Not Modified`:指示浏览器使用所请求资源的缓存副本。服务器使用`If-Modified-Since`和`If-None-Match`请求标头来确定客户端是否拥有资源的最新版本。
- `400 Bad Request`:指示客户端提交了无效的HTTP请求。当您以某些无效的方式修改请求时，例如通过在URL中放置空格字符，您可能会遇到这种情况。
- `401 Unauthorized`:指示在授予请求之前，服务器需要HTTP身份验证。`WWW-Authenticate`头包含支持的认证类型的详细信息。
- `403 Forbidden`:指示不允许任何人访问请求的资源，无论身份验证如何。
- `404 Not Found`:指示所请求的资源不存在。
- `405 Method Not Allowed`:指示不支持指定的URL中使用的方法。例如，如果您尝试在不支持PUT方法的地方使用它，则可能会收到此状态代码。
- `413 Request Entity Too Large`:如果您正在探测本机代码中的缓冲区溢出漏洞，并因此提交了长串数据，这表明您的请求主体太大，服务器无法处理。
- `414 Request URI Too Long`:类似于`413`响应。它表示请求中使用的URL太大，服务器无法处理。
- `500 Internal Server Error`:指示服务器在执行请求时遇到错误。对于指示错误性质的任何细节，您应该仔细检查服务器响应的全部内容。
- `503 Service Unavailable`:通常指示，虽然web服务器本身正在运行并可以响应请求，但是通过服务器访问的应用程序没有响应。您应该验证这是否是您所执行的任何操作的结果。

## 1.9 HTTPS

HTTP协议使用普通的TCP作为它的传输机制，它是不加密的，因此可以被网络上适当位置的攻击者截获。HTTPS本质上与HTTP是相同的应用层协议，但它是通过安全传输机制安全套接字层(SSL)隧道传输的。这保护了通过网络传输的数据的隐私和完整性，减少了非侵入性截取攻击的可能性。无论是否使用SSL进行传输，HTTP请求和响应的功能都是完全相同的。

>SSL已被传输层安全性(TLS)严格取代，但后者通常仍使用旧的名称。

## 1.10 HTTP Proxies

在攻击web应用程序时，工具包中最有用的一项是一种专门的代理服务器，它位于浏览器和目标网站之间，允许您拦截和修改所有请求和响应，甚至包括使用HTTPS的请求和响应。

## 1.11 HTTP Authentication
HTTP协议包含自己的使用各种身份验证方案对用户进行身份验证的机制，包括以下内容:
- `Basic`:是一种简单的身份验证机制，它在每个消息的请求标头中以`base64`编码的字符串的形式发送用户凭证
- `NTLM`:是一种质询-响应机制，使用的是Windows NTLM协议的一个版本。
- `Digest`:是一种质询响应机制，它使用带有用户凭据的随机数的MD5校验和。


