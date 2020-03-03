---
layout:     post
title:      Web应用程序技术 - Chapter 3(2) - Web Functionality and Encoding Schemes
subtitle:   
date:       2020-03-03
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

# 1. Web Functionality(网络功能)

除了用于在客户端和服务器之间发送消息的核心通信协议之外，Web应用程序还采用了多种技术来交付其功能。 任何功能正常的应用程序都可以在其服务器和客户端组件中采用多种不同的技术。 在对Web应用程序发起严重攻击之前，您需要基本了解其功能的实现方式，所用技术的设计方式以及其弱点可能位于何处。

## 1.1 Server-Side Functionality(服务器端功能)

当用户浏览器请求动态资源时，它通常不会简单地请求该资源的副本。通常，它还会在提交请求时提交各种参数。正是这些参数使服务器端应用程序能够生成针对单个用户的内容。HTTP请求可以通过四种主要方式向应用程序发送参数:
- 在URL查询字符串中
- 在REST样式URL的文件路径中
- 在HTTP cookie
- 在使用POST方法的请求body中

与一般的计算机软件一样，Web应用程序在服务器端采用多种技术来提供其功能:
- 脚本语言如:`PHP`,`VBScript`,`Perl`等
- Web应用程序平台如:`ASP.NET`,`Java`等
- Web服务器如:`Apache`,`IIS`,`Netscape Enterprise`等
- 数据库如:`MS-SQL`,`Oracle`,`MySQL`等
- 其他后端组件如:文件系统,基于SOAP的web服务(`SOAP`:是基于`XML`的简易协议,可使应用程序在HTTP之上进行信息交换)

### 1.1.1 Java平台
基于Java的Web应用程序的描述通常使用许多可能令人困惑的术语，您可能需要注意这些术语:
- `EJB`:(Enterprise Java Bean)是一个相对较重的软件组件，它在应用程序内封装了特定业务功能的逻辑.EJB旨在解决应用程序开发人员必须解决的各种技术难题，例如事务完整性.
- `POJO`:(Plain Old Java Object)是普通的Java对象，与诸如EJB之类的特殊对象不同.POJO通常用于表示用户定义的对象，这些对象比EJB和其他框架中使用的对象更简单，更轻便.
- `Java Servlet`:是驻留在应用程序服务器上的对象，它从客户端接收HTTP请求并返回HTTP响应。 Servlet实现可以使用许多接口来促进有用应用程序的开发。
- `Java Web container`:是为基于Java的Web应用程序提供运行时环境的平台或引擎。 Java Web容器的示例是`Apache Tomcat`，`BEA WebLogic`和`JBoss`。

下面是一些通常用于关键应用程序功能的组件示例:
- `Authentication`(身份认证): JAAS,ACEGI
- `Presentation layer`(表示层):SiteMesh, Tapestry
- `Database object relational mapping`(数据库对象关系映射):Hibernate
- `Logging`(登录):Log4J

如果您可以确定要攻击的应用程序中使用了哪些开源软件包，则可以下载它们并执行**代码检查**或**安装**以进行试验。 其中任何一个漏洞都可能被利用来破坏更广泛的应用程序。

### 1.1.2 ASP.NET
ASP.NET是Microsoft的Web应用程序框架，是Java平台的直接竞争对手。 ASP.NET比它的同类产品年轻几年，但已大举进入Java领域。ASP.NET使用的是微软的`.NET`框架.

### 1.1.3 PHP
使用PHP开发了许多开源应用程序和组件。 其中许多提供了通用应用程序功能的现成解决方案，通常将这些功能集成到更广泛的定制应用程序中:
- `Bulletin boards`(布告栏)- PHPBB,PHP-Nuke
- `Administratiive front ends`(管理前端) - PHPMyAdmin
- `Web mail`(网页邮件) - SquirrelMail,IlohaMail
- `Photo galleries`(图片管理功能) - Gallery
- `Shopping carts`(购物车) - osCommerce,ECW-Shop
- `Wikis`(维基百科) - MediaWiki,WakkaWikki

### 1.1.4 Ruby on Rails
Rails 1.0于2005年发布，它非常强调模型-视图-控制器体系结构。Rails的一个关键优势是，它可以以极快的速度创建完全流畅的数据驱动应用程序。如果开发人员遵循Rails编码风格和命名约定，Rails可以自动生成用于数据库内容的模型、用于修改它的控制器操作和用于应用程序用户的默认视图。

[Ruby安全漏洞](http://www.ruby-lang.org/en/security/)

### 1.1.4 SQL
结构化查询语言(SQL)用于访问关系数据库中的数据，如Oracle、MS-SQL server和MySQL。

### 1.1.5 XML
可扩展标记语言（XML）是一种用于以机器可读形式编码数据的规范。 像任何标记语言一样，XML格式将文档分为内容（即数据）和标记（即对数据进行注释）。

XML是可扩展的，因为它允许任意的标记和属性名称。XML文档通常包含文档类型定义(DTD)，它定义了文档中使用的标记和属性以及它们的组合方式。

### 1.1.6 Web Services
虽然本书涵盖了web应用程序黑客攻击，但其中描述的许多漏洞同样适用于web服务。实际上，许多应用程序本质上是一组后端web服务的GUI前端。

Web服务使用简单对象访问协议(Simple Object Access Protocol, SOAP)交换数据。`SOAP`通常使用`HTTP`协议来传输消息，并使用`XML`格式表示数据。

一个典型的SOAP请求如下:
```
POST /doTransfer.asp HTTP/1.0
Host: mdsec-mgr.int.mdsec.net
Content-Type: application/soap+xml; charset=utf-8
Content-Length: 891
<?xml version=”1.0”?>
<soap:Envelope xmlns:soap=”http://www.w3.org/2001/12/soap-envelope”>
	<soap:Body>
		<pre:Add xmlns:pre=http://target/lists soap:encodingStyle=“http://www.w3.org/2001/12/soap-encoding”>
			<Account>
				<FromAccount>18281008</FromAccount>
				<Amount>1430</Amount>
				<ClearedFunds>False</ClearedFunds>
				<ToAccount>08447656</ToAccount>
			</Account>
		</pre:Add>
	</soap:Body>
</soap:Envelope>
```
## 1.2 Client-Side Functionality(客户端功能)

为了让服务器端应用程序接收用户输入和操作并将结果显示给用户，它需要提供一个客户端用户界面。因为所有web应用程序都是通过web浏览器访问的，所以这些接口都共享一个共同的核心技术。然而，这些都是以各种不同的方式构建的，并且应用程序利用客户端技术的方式在近年来继续快速发展。

### 1.2.1 HTML
HTML是一种基于标记的语言，用于描述在浏览器中呈现的文档结构.

XHTML是基于XML的HTML的发展，它比旧版本的HTML具有更严格的规范。XHTML的部分动机是需要转向更严格的HTML标记标准，以避免在浏览器必须容忍不太严格的HTML格式时可能出现的各种折衷和安全问题。

### 1.2.2 Hyperlinks(超链接)
一个超链接如下:
```
<a href=”?redir=/updates/update29.html”>What’s happening?</a>
```
当用户单击此链接时，浏览器发出以下请求:
```
GET /news/8/?redir=/updates/update29.html HTTP/1.1
Host: mdsec.net
...
```
### 1.2.3 Forms(表单)
HTML表单是允许用户通过浏览器输入任意输入的常用机制。典型的形式如下：
```
<form action=”/secure/login.php?app=quotations” method=”post”>
username: <input type=”text” name=”username”><br>
password: <input type=”password” name=”password”>
<input type=”hidden” name=”redir” value=”/secure/home.php”>
<input type=”submit” name=”submit” value=”log in”>
</form>
```
当用户在表单中输入值并单击Submit按钮时，浏览器发出如下请求:
```
POST /secure/login.php?app=quotations HTTP/1.1
Host: wahh-app.com
Content-Type: application/x-www-form-urlencoded
Content-Length: 39
Cookie: SESS=GTnrpx2ss2tSWSnhXJGyG0LJ47MXRsjcFM6Bd

username=daf&password=foo&redir=/secure/home.php&submit=log+in
```
在这个请求中，有几个要点反映了如何使用请求的不同方面来控制服务器端处理:
- 因为HTML `form`标记包含一个指定`POST`方法的属性，所以浏览器使用这个方法提交表单，并将表单中的数据放到请求消息的`正文`中。
- 除了用户输入的两项数据之外，表单还包含一个隐藏参数(`redir`)和一个提交参数(`submit`)。这两者都是在请求中提交的，服务器端应用程序可以使用它们来控制逻辑。
- 表单提交的目标URL包含一个预设参数(`app`)，如前所示的超链接示例所示。此参数可用于控制服务器端处理。
- 该请求包含一个cookie参数(`SESS`)，它在服务器的早期响应中被发送到浏览器。此参数可用于控制服务器端处理。

前面的请求包含一个头，它指定消息体中的内容类型是`x-www-form-urlencoded`。这意味着参数在消息体中以`名称/值`对的形式表示，其方式与在URL查询字符串中相同。在提交表单数据时，您可能要使用的另一种内容类型是多部分/表单数据。应用程序可以请求浏览器使用多部分编码，方法是在表单标记的enctype属性中指定这一点。使用这种形式的编码，请求中的`Content-Type`头还指定了一个`随机字符串`，该字符串用作请求体中包含的参数的`分隔符`。例如，如果表单指定了多部分编码，那么结果请求将如下所示:
```
POST /secure/login.php?app=quotations HTTP/1.1
Host: wahh-app.com
Content-Type: multipart/form-data; boundary=------------7d71385d0a1a
Content-Length: 369
Cookie: SESS=GTnrpx2ss2tSWSnhXJGyG0LJ47MXRsjcFM6Bd
------------7d71385d0a1a
Content-Disposition: form-data; name=”username”
daf
------------7d71385d0a1a
Content-Disposition: form-data; name=”password”
foo
------------7d71385d0a1a
Content-Disposition: form-data; name=”redir”
/secure/home.php
------------7d71385d0a1a
Content-Disposition: form-data; name=”submit”
log in
------------7d71385d0a1a--
```

### 1.2.4 CSS
层叠样式表(Cascading Style Sheets, CSS)是一种用于描述用标记语言编写的文档的表示形式的语言。

在web应用程序安全的早期，CSS在很大程度上被忽视，并且被认为没有安全含义。今天，CSS本身作为安全漏洞的来源和为其他类型的漏洞提供有效漏洞的手段(有关更多信息，请参阅第12章和第13章)越来越重要。

### 1.2.5 JavaScript
JavaScript是一种相对简单但功能强大的编程语言，可以很容易地使用它来扩展web接口，这是仅使用HTML无法做到的。它通常用于执行以下任务:
- 在将用户输入的数据提交给服务器之前对其进行验证，以避免在数据包含错误时出现不必要的请求
- 在将用户输入的数据提交给服务器之前对其进行验证，以避免在数据包含错误时出现不必要的请求
- 在浏览器中查询和更新文档对象模型(DOM)以控制浏览器的行为(稍后将描述浏览器DOM)

### 1.2.6 VBScript
VBScript是仅在Internet Explorer浏览器中支持的JavaScript的替代方案。它基于Visual Basic建模，并允许与浏览器DOM进行交互。但是一般来说，它的功能和开发程度都不如JavaScript。

由于VBScript的浏览器特性，它在当今的web应用程序中很少被使用。从安全的角度来看，它的主要兴趣在于作为一种为漏洞提供漏洞利用的方法，比如在使用JavaScript不可行的情况下，跨站点脚本攻击(参见第12章)。

### 1.2.7 DOM(Document Object Model)
文档对象模型(Document Object Model, DOM)是HTML文档的抽象表示，可以通过它的API查询和操作它。

DOM允许客户端脚本通过它们的id访问各个HTML元素，并以编程方式遍历元素的结构。诸如当前URL和cookie之类的数据也可以读取和更新。DOM还包括一个事件模型，允许代码连接事件，如表单提交、通过链接导航和击键。

浏览器DOM的操作是基于ajax的应用程序中使用的关键技术，如下一节所述。

### 1.2.8 Ajax
Ajax是一组用于客户端创建用户界面的编程技术，旨在模拟传统桌面应用程序的流畅交互和动态行为。

从历史上看，Ajax的使用为web应用程序引入了一些新的漏洞类型。更广泛地说，它还通过在服务器端和客户端引入更多潜在的攻击目标，增加了典型应用程序的攻击面。当攻击者为其他漏洞设计更有效的漏洞利用时，也可以使用Ajax技术。更多细节见第12章和第13章。

### 1.2.9 JSON
JSON(JavaScript Object Notation`):是一种简单的数据传输格式，可用于序列化任意数据。例子:
```
{
	“name”: “Mike Kemp”,
	“id”: “8041148671”,
	“email”: “fkwitt@layerone.com”
}
```
例如，当用户更新联系人的详细信息时，可以使用以下请求将新信息传递给服务器:
```
POST /contacts HTTP/1.0
Content-Type: application/x-www-form-urlencoded
Content-Length: 89

Contact={“name”:”Mike Kemp”,”id”:”8041148671”,”email”:”pikey@
clappymonkey.com”}
&submit=update
```
### 1.2.10 SOP(Same-Origin Policy)
同源策略(SOP)是浏览器中实现的一种关键机制，旨在防止来自不同来源的内容相互干扰。基本上，从一个网站收到的内容可以阅读和修改从同一网站收到的其他内容，但不允许访问从其他网站收到的内容。

在实践中，将这一概念应用于不同web特性和技术的细节会导致各种复杂性和折衷。下面是需要注意的同源策略的一些关键特性:
- 驻留在一个域中的页面可能导致向另一个域发出任意请求(例如，通过提交表单或加载图像)。但是它本身不能处理从该请求返回的数据。
- 驻留在一个域中的页面可以从另一个域中加载脚本并在自己的上下文中执行。这是因为脚本被假定包含代码，而不是数据，所以跨域访问不应该导致任何敏感信息的泄露。
- 驻留在一个域中的页面不能读取或修改属于另一个域的cookie或其他DOM数据。

### 1.2.11 HTML5
HTML5是HTML标准的重大更新。HTML5目前仍在开发中，仅在浏览器中部分实现。
从安全的角度来看，HTML5主要有以下原因:
- 它引入了各种新的标签、属性和api，可以通过杠杆化来交付跨站点脚本和其他攻击，如第12章所述。
- 它修改了核心Ajax技术XMLHttpRequest，以便在某些情况下支持双向跨域交互。这可能导致新的跨域攻击，如第13章所述。
- 它引入了客户端数据存储的新机制，这可能会导致用户隐私问题，以及新的攻击类型，如第13章中描述的客户端SQL注入。

### 1.2.12 Web2.0
这个时髦的词在最近几年变得很流行，它是一个相当松散和模糊的名称，用于描述web应用程序中的一系列相关趋势，包括以下内容:
- 大量使用Ajax执行异步、后台请求
- 使用各种技术增加跨域集成
- 在客户端使用新技术，包括XML、JSON和Flex
- 支持用户生成内容、信息共享和交互的突出功能

### 1.2.13 Browser Extension Technologies(浏览器扩展技术)

- Java applets
- ActiveX controls
- Flash objects
- Silverlight objects

### 1.2.14 State and Sessions(状态和会话)
到目前为止所描述的技术使web应用程序的服务器和客户机组件能够以多种方式交换和处理数据。然而，为了完善大多数有用的功能，应用程序需要跨多个请求跟踪每个用户与应用程序的交互状态。例如，购物应用程序允许用户浏览产品目录、向购物车添加商品、查看和更新购物车内容、继续结帐并提供个人和付款细节。

# 2. Encoding Schemes(编码方案)
Web应用程序对其数据采用几种不同的编码方案。HTTP协议和HTML语言都是基于文本的，并且已经设计了不同的编码方案来确保这些机制能够安全地处理不寻常的字符和二进制数据。当您攻击web应用程序时，您将经常需要使用相关的方案对数据进行编码，以确保按照您希望的方式进行处理。

## 2.1 URL Encoding
任何字符的`URL Encoding`编码形式都是后面跟以十六进制表示的字符的两位数ASCII码的`%`前缀:
- `%3` - `=`
- `%25` - `%`
- `%20` - Space(空格)
- `%0a` - New line(换行)
- `%00` - Null byte(空字节)

要注意的另一种编码是`+`字符，它表示URL Encoding的`空格`（除了空格的％20表示之外）。

>为了攻击web应用程序，您应该在将下列字符作为数据插入HTTP请求时对它们进行URL编码

```
space % ? & = ; + #
```

## 2.2 Unicode Encoding
16位Unicode Encoding的工作方式与URL编码相似。对于通过HTTP传输，字符的16位Unicode编码形式是`%u`前缀后面跟以十六进制表示的字符的Unicode编码点:
- `%u2215` - `/`
- `%u00e9` - `é`

`UTF-8`是一种可变长度编码标准，它使用一个或多个字节来表示每个字符。对于HTTP上的传输，多字节字符的UTF-8编码形式只是使用十六进制表示的每个字节并在其前面加上`%`前缀:
- `%c2%a9` - `©`
- `%e2%89%a0` - (不等号,我这里显示出来)

为了攻击web应用程序，Unicode编码主要是人们感兴趣的，因为它有时会被用来`破坏`输入验证机制。如果输入筛选器阻止某些恶意表达式，但是随后处理输入的组件理解Unicode编码，那么可以使用各种标准的和格式不正确的Unicode编码`绕过`筛选器。

## 2.3 HTML Encoding
HTML编码用于表示有问题的字符，以便将它们安全地合并到HTML文档中。各种字符在HTML中作为元字符具有特殊的意义，它们被用来改变文档的`结构`，而不是其内容。为了安全地将这些字符用作文档内容的一部分，有必要对它们进行HTML编码。

HTML编码定义了大量的HTML实体来表示特定的文字字符:
- `&quot;` - `"`
- `&apos;` - `'`
- `&amp;` - `&`
- `&lt;` - `<`
- `&gt` - `>`

此外，任何字符都可以使用十进制形式的ASCII码进行html编码:
- `&#34;` - `"`
- `&#39;` - `'`

或者使用十六进制形式的ASCII码(以x为前缀):
- `&#x22;` - `"`
- `&#x27;` - `'`

当您攻击web应用程序时，您对HTML编码的主要兴趣可能是探测`跨站点脚本`漏洞。如果应用程序在其响应中返回未经修改的用户输入，那么它可能是脆弱的，而如果危险字符是html编码的，那么它可能是安全的

## 2.4 Base64 Encoding
Base64编码允许只使用可打印的ASCII字符来安全地表示任何二进制数据。它通常用于对电子邮件`附件`进行编码，以便通过`SMTP`进行安全传输。它还用于在基本`HTTP身份验证`中对用户凭据进行编码。

## 2.5 Hex Encoding
十六进制编码的数据通常很容易被发现.

## 2.6 Remoting and Serialization Frameworks(远程和序列化框架)
近年来，已经出现了各种用于创建用户界面的框架，其中客户端代码可以远程访问服务器端的各种编程api。这使开发人员能够部分地从web应用程序的分布式本质中抽象出来，并以一种更接近传统桌面应用程序范例的方式编写代码。

这些类型的远程和序列化框架的示例包括以下内容:
- Flew and AMF
- Silverlight and WCF
- Java serialized objects

[Web应用程序技术 - Chapter 3(1) - HTTP](https://dm116.github.io/2020/03/02/web-application-technologies)
