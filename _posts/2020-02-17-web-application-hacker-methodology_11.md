---
layout:     post
title:      Web应用程序黑客的方法论(11) --Test for Shared Hosting Vulnerabilities
subtitle:
date:       2020-02-17
author:     D
header-img: 
catalog: true
mermaid: true
tags:
    - web hacking
---

Web应用程序黑客的方法论来自书本 *The Web Application Hacker's Handbook* <br>

![Test for Application Server Vulnerabilities](/img/test-for-application-server-vulnerabilities.png)

# 11.1 测试默认凭据(Test for Default Credentials)

11.1.1<br>
查看应用程序映射练习的结果，以识别Web服务器和其他正在使用的技术，其中可能包含可访问的管理界面。

11.1.2<br>
对Web服务器执行端口扫描，以识别在与主要目标应用程序不同的端口上运行的所有管理接口。

11.1.3<br>
对于任何已标识的接口，请查阅制造商的文档和常见的默认密码列表，以获取默认凭据。

11.1.4<br>
如果默认凭据不起作用，请使用第4节中列出的步骤尝试猜测有效的凭据。

11.1.5<br>
如果您有权访问管理界面，请查看可用功能并确定是否可以使用该功能进一步破坏主机并攻击主应用程序。

# 11.2 测试默认内容(Test for Default Content)

11.2.1<br>
查看Nikto扫描的结果（步骤1.4.1），以识别服务器上可能存在的但不是应用程序组成部分的任何默认内容。

11.2.2<br>
使用搜索引擎和其他资源（例如www.exploit-db.com和www.osvdb.org）来标识您知道正在使用的技术中包含的默认内容和功能。 如果可行，请对它们进行本地安装，并检查它们是否具有可以在攻击中利用的任何默认功能。

11.2.3<br>
检查默认内容，以了解您可以用来攻击服务器或应用程序的任何功能或漏洞。

# 11.3 测试危险的HTTP方法(Test for Dangerous HTTP Methods)

11.3.1<br>
使用`OPTIONS`方法列出服务器状态可用的HTTP方法。 请注意，可以在不同的目录中启用不同的方法。 您可以在`Paros`中执行漏洞扫描以执行此检查。

11.3.2<br>
手动尝试每种报告的方法，以确认是否可以实际使用。

11.3.3<br>
如果发现某些WebDAV方法已启用，请使用启用了WebDAV的客户端进行进一步调查，例如Microsoft FrontPage或Internet Explorer中的“作为Web文件夹打开”选项。

# 11.4 测试代理功能(Test for Proxy Functionality)

11.4.1<br>
同时使用`GET`和`CONNECT`请求，尝试将Web服务器用作代理以连接到Internet上的其他服务器并从中检索内容。

11.4.2<br>
使用`GET`和`CONNECT`请求，尝试连接到主机基础结构中的不同IP地址和端口。

11.4.3<br>
通过使用`GET`和`CONNECT`请求，通过将`127.0.0.1`指定为请求中的目标主机，尝试连接到Web服务器本身的通用端口号。

# 11.5 测试虚拟主机配置错误(Test for Virtual Hosting Misconfiguration)

11.5.1<br>
使用以下命令将GET请求提交到根目录:<br>
- 正确的`Host` header
- 伪 `Host` header
- 在 `Host` header 中的服务器IP地址
- 没有 `Host` header(仅使用HTTP/1.0)

11.5.2<br>
比较对这些请求的响应。 常见的结果是，在主机标头中使用服务器的IP地址时，将获得目录列表。 您可能还会发现可以访问其他默认内容。

11.5.3<br>
如果您观察到不同的行为，请使用产生不同结果的主机名重复第1节中描述的应用程序映射练习。 确保使用`-vhost`选项执行`Nikto`扫描，以识别在初始应用程序映射期间可能被忽略的任何默认内容。

# 11.6 测试Web服务器软件错误(Test for Web Server Software Bugs)

11.6.1<br>
运行`Nessus`和任何其他可用的类似扫描仪，以识别您正在攻击的Web服务器软件中的任何已知漏洞。

11.6.2<br>
查看诸如`Securityfocus`，`Bugtraq`和`Full Disclosure`之类的资源，以查找可能尚未在目标上修复的任何最近发现的漏洞的详细信息。

11.6.3<br>
如果该应用程序是由第三方开发的，请调查该应用程序是否带有自己的Web服务器（通常是开源服务器）。 如果是这样，请调查是否存在任何漏洞。 请注意，在这种情况下，服务器的标准标语可能已被修改。

11.6.4<br>
如果可能，请考虑对要攻击的软件执行本地安装，然后进行自己的测试以发现尚未发现或广泛传播的新漏洞。

# 11.7 测试Web应用程序防火墙(Test for Web Application Firewalling)

11.7.1<br>
向应用程序提交任意参数名称，并在值中添加明确的攻击有效负载，理想情况下，应用程序在响应中包含名称`and/or`值的某个位置。 如果应用程序阻止了攻击，则可能是由于外部防御。

11.7.2<br>
如果可以提交服务器响应中返回的变量，请提交一系列的模糊字符串和编码变体，以识别用户输入的应用程序防御行为。

11.7.3<br>
通过对应用程序内的变量执行相同的攻击来确认此行为。

11.7.4<br>
对于所有模糊测试字符串和请求，请使用标准签名数据库中不太可能存在的有效负载字符串。 尽管按照定义不可能提供这些示例，但请避免将`/etc/passwd`或`/windows/system32/config/sam`用作文件检索的有效负载。 还要避免在XSS攻击中使用诸如`<script>`之类的术语，并避免将`alert()`或`xss`用作XSS负载。

11.7.5<br>
如果特定请求被阻止，请尝试在不同的位置或上下文中提交相同的参数。 例如，在GET请求的URL中，POST请求的正文中以及POST请求的URL中提交相同的参数。

11.7.6<br>
在ASP.NET上，也尝试将参数提交为Cookie。 如果在查询字符串或消息正文中找不到参数`foo`，则API `Request.Params ["foo"]`将检索名为`foo`的cookie的值。

11.7.7<br>
查看第4章中介绍用户输入的所有其他方法，选择任何不受保护的方法。

11.7.8<br>
确定以非标准格式（例如序列化或编码）提交（或可以提交）用户输入的位置。 如果没有可用的攻击，请通过串联和/或将其跨多个变量构建攻击字符串。 （请注意，如果目标是ASP.NET，则可以使用HPP使用同一变量的多个规范来串联攻击。）<br>


[Web应用程序黑客的方法论(10)](https://dm116.github.io/2020/02/17/web-application-hacker-methodology_10/)<br>
[Web应用程序黑客的方法论(12)](https://dm116.github.io/2020/02/17/web-application-hacker-methodology_12/)<br>


