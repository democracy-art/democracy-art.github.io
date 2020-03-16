---
layout:     post
title:      Chapter 10 Attacking Back-End Components(4)-Injecting into Back-end HTTP  Requests
subtitle:   注入后端HTTP请求
date:       2020-03-16
author:     D
header-img: 
catalog: true
mermaid: true
tags: [web hacking]
---

参考:**The Web Application Hackers Handbook** Chapter 10

上一节描述了某些应用程序如何将用户提供的数据合并到对用户无法直接访问的服务的后端SOAP请求中。 更一般而言，应用程序可以将用户输入嵌入到任何类型的后端HTTP请求中，包括那些将参数作为常规名称/值对传输的应用程序。 这种行为通常容易受到攻击，因为应用程序经常有效地代理用户提供的URL或参数。 对该功能的攻击可以分为以下几类：
- **服务器端HTTP重定向**攻击使攻击者可以指定前端应用程序服务器随后请求的任意资源或URL。

- **HTTP参数注入**攻击使攻击者可以将任意参数注入到应用程序服务器发出的后端HTTP请求中。 如果攻击者注入了后端请求中已经存在的参数，则可以使用HTTP参数污染（HPP）攻击来覆盖服务器指定的原始参数值。

# 1.服务器端HTTP重定向(Server-side HTTP Redirection)

当应用程序接受用户可控制的输入并将其合并到使用后端HTTP请求检索的URL中时，服务器端重定向漏洞就会出现。 用户提供的输入可以包括检索到的整个URL，或者应用程序可以对其进行一些处理，例如添加标准后缀。

后端HTTP请求可能针对公共Internet上的域，也可能针对用户无法直接访问的内部服务器。 所请求的内容可能是应用程序功能的核心，例如到支付网关的接口。 或者可能更外围，例如从第三方提取的静态内容。 该技术通常用于将多个不同的内部和外部应用程序组件编织到一个单独的前端应用程序中，该前端应用程序代表这些其他系统处理访问控制和会话管理。 如果攻击者可以控制后端HTTP请求中使用的IP地址或主机名，则可以使应用程序服务器连接到任意资源，有时还可以检索后端响应的内容。

考虑以下前端请求示例，其中`loc`参数用于指定客户端要使用哪个CSS文件版本：
```
POST /account/home HTTP/1.1
Content-Type: application/x-www-form-urlencoded
Host: wahh-blogs.net
Content-Length: 65

view=default&loc=online.wahh-blogs.net/css/wahh.css
```
如果在loc参数中未指定URL的验证，则攻击者可以指定一个任意的主机名来代替`online.wahh-blogs.net`。 该应用程序检索指定的资源，从而使攻击者可以使用该应用程序作为潜在敏感后端服务的代理。 在以下示例中，攻击者使应用程序连接到后端SSH服务：
```
POST /account/home HTTP/1.1
Content-Type: application/x-www-form-urlencoded
Host: blogs.mdsec.net
Content-Length: 65

view=default&loc=192.168.0.1:22
```
应用程序的响应包括来自所请求的SSH服务的标语：
```
HTTP/1.1 200 OK
Connection: close

SSH-2.0-OpenSSH_4.2Protocol mismatch.
```
攻击者可以利用服务器端HTTP重定向错误来有效地利用易受攻击的应用程序作为开放的HTTP代理来执行各种进一步的攻击：
- 攻击者可能能够使用代理来攻击Internet上的第三方系统。 对于目标来说，恶意流量似乎来自运行有漏洞的应用程序的服务器。
- 攻击者可能能够使用代理连接到组织内部网络上的任意主机，从而达到无法直接从Internet访问的目标。
- 攻击者可能能够使用代理连接回应用程序服务器本身上运行的其他服务，从而规避防火墙的限制，并可能利用信任关系来绕过身份验证。
- 最后，通过使应用程序在响应中包含攻击者控制的内容，代理功能可用于传递攻击（例如跨站点脚本）（有关更多详细信息，请参见第12章）。

**HACK STEPS**
- 1.标识任何看起来包含主机名，IP地址或完整URL的请求参数。
- 2.对于每个参数，修改其值以指定替代资源，类似于请求的资源，然后查看该资源是否出现在服务器的响应中。
- 3.尝试指定一个以您控制的Internet上的服务器为目标的URL，并监视该服务器是否有来自正在测试的应用程序的传入连接。
- 4.如果没有收到传入连接，请监视应用程序响应所花费的时间。 如果存在延迟，则由于出站连接的网络限制，应用程序的后端请求可能会超时。
- 5.如果成功使用该功能连接到任意URL，请尝试执行以下攻击：
	- a.确定是否可以指定端口号。 例如，您可以提供`http://mdattacker.net:22`。
	- b.如果成功，请尝试使用Burp Intruder之类的工具对内部网络进行端口扫描，以依次连接到一系列IP地址和端口（请参阅第14章）。
	- c.尝试连接到应用程序服务器的环回地址上的其他服务。
	- d.尝试将您控制的网页加载到应用程序的响应中，以进行跨站点脚本攻击。

**NOTE**
某些服务器端重定向API，例如ASP.NET中的`Server.Transfer()`和`Server.Execute()`，仅允许重定向到同一主机上的相对URL。 将用户提供的输入传递给这些方法之一的功能仍可能被利用，以利用信任关系并访问受平台级身份验证保护的服务器上的资源。

# 2.HTTP参数注入(HTTP Parameter Injection)

当用户提供的参数用作后端HTTP请求中的参数时，会出现HTTP参数注入（HPI）。 考虑以前容易受到SOAP注入攻击的银行转帐功能的以下变化：
```
POST /bank/48/Default.aspx HTTP/1.0
Host: mdsec.net
Content-Length: 65

FromAccount=18281008&Amount=1430&ToAccount=08447656&Submit=Submit
```
该前端请求是从用户的浏览器发送的，它使应用程序向银行基础架构内的另一台Web服务器发出另一个后端HTTP请求。 在此后端请求中，应用程序从前端请求中复制一些参数值：
```
POST /doTransfer.asp HTTP/1.0
Host: mdsec-mgr.int.mdsec.net
Content-Length: 44
fromacc=18281008&amount=1430&toacc=08447656
```
该请求使后端服务器检查已清除的资金是否可用于执行转帐，如果可以，则执行该转帐。 但是，前端服务器可以选择提供以下参数，以指定已清算资金可用，从而绕过支票：
```
clearedfunds=true
```
如果攻击者知道此行为，则他可以尝试执行HPI攻击，将`clearedfunds`参数注入到后端请求中。 为此，他将必需的参数添加到现有参数值的末尾，并对URL编码`＆`和`=`，这两个字符用于分隔名称和值：
```
POST /bank/48/Default.aspx HTTP/1.0
Host: mdsec.net
Content-Length: 96

FromAccount=18281008&Amount=1430&ToAccount=08447656%26clearedfunds%3dtru
e&Submit=Submit
```
当应用程序服务器处理此请求时，它将以常规方式对参数值进行URL解码。 因此，前端应用程序接收到的ToAccount参数的值如下：
```
08447656&clearedfunds=true
```
如果前端应用程序未验证此值，并将其未经过滤处理传递给后端请求，则会发出以下后端请求，该请求将成功绕过对已清资金的检查：
```
POST /doTransfer.asp HTTP/1.0
Host: mdsec-mgr.int.mdsec.net
Content-Length: 62

fromacc=18281008&amount=1430&toacc=08447656&clearedfunds=true
```

**NOTE**
与SOAP注入不同，将任意意外参数注入后端请求不太可能引起任何类型的错误。 因此，成功的攻击通常需要确切地了解所使用的后端参数。 尽管在黑盒环境中可能很难确定这一点，但如果应用程序使用可以获取和研究其代码的任何第三方组件，则可能会很简单。

## 2.1 HTTP参数污染(HTTP Parameter Pollution)

HPP是一种在各种情况下都会出现的攻击技术（有关其他示例，请参阅第12和13章），并且通常适用于HPI攻击的情况。

当请求包含多个具有相同名称的参数时，HTTP规范不提供有关Web服务器应如何行为的准则。 实际上，不同的Web服务器的行为方式不同。 以下是一些常见的行为：
- 使用参数的第一个实例。
- 使用参数的最后一个实例。
- 连接参数值，可能在它们之间添加分隔符。
- 构造一个包含所有提供的值的数组。

在前面的HPI示例中，攻击者可以向后端请求添加新参数。 实际上，实际上，攻击者可以向其注入的请求中已经包含一个参数，该参数具有他所针对的名称。 在这种情况下，攻击者可以使用HPI条件注入相同参数的第二个实例。 然后，产生的应用程序行为取决于后端HTTP服务器如何处理重复的参数。 攻击者可能可以使用HPP技术将原始参数的值替换为其注入的参数的值。

例如，如果原始的后端请求如下：
```
POST /doTransfer.asp HTTP/1.0
Host: mdsec-mgr.int.mdsec.net
Content-Length: 62

fromacc=18281008&amount=1430&clearedfunds=false&toacc=08447656
```
并且后端服务器使用任何重复参数的第一个实例，攻击者可以将攻击放入前端请求的FromAccount参数中：
```
POST /bank/52/Default.aspx HTTP/1.0
Host: mdsec.net
Content-Length: 96

FromAccount=18281008%26clearedfunds%3dtrue&Amount=1430&ToAccount=0844765
6&Submit=Submit
```
相反，在此示例中，如果后端服务器使用任何重复参数的最后一个实例，则攻击者可以将攻击放入前端请求中的ToAccount参数中。

HPP攻击的结果在很大程度上取决于目标应用程序服务器如何处理多次出现的相同参数以及后端请求内的精确插入点。 如果两种技术需要处理同一个HTTP请求，则将产生严重后果。 Web应用程序防火墙或反向代理可以处理请求并将其传递给Web应用程序，Web应用程序可能会继续丢弃变量，甚至从请求的先前完全不同的部分中构建字符串！

可以在这里找到涵盖常见应用程序服务器的不同行为的好论文：
```
www.owasp.org/images/b/ba/AppsecEU09_CarettoniDiPaola_v0.8.pdf
```

## 2.2 攻击URL翻译(Attacks Against URL Translation)

许多服务器在到达时重写请求的URL，以将它们映射到应用程序中的相关后端功能上。 除了常规的URL重写之外，此行为还可能出现在REST样式参数，自定义导航包装和其他URL转换方法的上下文中。 此行为涉及的处理类型可能容易受到HPI和HPP攻击。

为了简化并帮助导航，某些应用程序将参数值放置在URL的文件路径内，而不是查询字符串内。 这通常可以通过一些简单的规则来实现，以转换URL并将其转发到真实的目的地。 Apache中的以下`mod_rewrite`规则用于处理对用户配置文件的公共访问：
```
RewriteCond %{THE_REQUEST} ^[A-Z]{3,9}\ /pub/user/[^\&]*\ HTTP/
RewriteRule ^pub/user/([^/\.]+)$ /inc/user_mgr.php?mode=view&name=$1
```
该规则接受美学上令人愉悦的要求，例如：
```
/pub/user/marcus
```
并将其转换为对用户管理页面`user_mgr.php`中包含的`view`功能的后端请求。 它将`marcus`参数移动到查询字符串中，并添加`mode=view`参数：
```
/inc/user_mgr.php?mode=view&name=marcus
```
在这种情况下，有可能使用HPI攻击将第二`mode`参数注入重写的URL。 例如，如果攻击者要求这样做：
```
/pub/user/marcus%26mode=edit
```
URL解码的值嵌入在重写的URL中，如下所示：
```
/inc/user_mgr.php?mode=view&name=marcus&mode=edit
```
如针对HPP攻击所述，此攻击的成功取决于服务器如何处理现在重复的参数。 在PHP平台上，`mode`参数被视为具有值`edit`，因此攻击成功。

**HACK STEPS**
- 1.依次定位每个请求参数，并尝试使用各种语法附加新的注入参数：
	- `%26foo%3dbar` — URL-encoded `&foo=bar`
	- `%3bfoo%3dbar` — URL-encoded `;foo=bar`
	- `%2526foo%253dbar` — Double URL-encoded `&foo=bar`
- 2.标识应用程序的行为就像原始参数未修改一样。 （这仅适用于通常在修改后会导致应用程序响应有所不同的参数。）
- 3.上一步中确定的每个实例都有机会注入参数。 尝试在请求中的各个点插入已知参数，以查看其是否可以覆盖或修改现有参数。 例如：
```
FromAccount=18281008%26Amount%3d4444&Amount=1430&ToAccount=08447656
```
- 4.如果这导致新值覆盖现有值，请确定是否可以通过注入后端服务器读取的值来绕过任何前端验证。
- 5.如第4章中针对应用程序映射和内容发现所述，将注入的已知参数替换为其他参数名称。
- 6.测试应用程序对请求中同一参数的多次提交的容忍度。 在其他参数之前和之后，以及在请求内的不同位置（在查询字符串，cookie和消息正文中）提交冗余值。


[Chapter 10 Attacking Back-End Components(3)-Injecting into XML Interpreters](https://dm116.github.io/2020/03/16/attacking-back-end-components_3/)<br>
[Chapter 10 Attacking Back-End Components(5)-Injecting into Mail Services](https://dm116.github.io/2020/03/16/attacking-back-end-components_5/)<br>
