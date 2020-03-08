---
layout:     post
title:      Chapter 5 Bypassing Client-Side Controls(1) - Transmitting Data Via the Client
subtitle:   
date:       2020-03-08
author:     D
header-img: 
catalog: true
mermaid: true
tags:
    - [web hacking]
---

参考:*The Web Application Hacker's Handbook* 

# 1. 通过客户端传输数据(Transmitting Data Via the Client)

第1章描述了Web应用程序的`核心`安全问题是如何出现的，因为客户端可以提交`任意输入`。 尽管如此，尽管如此，仍有很大一部分Web应用程序依赖客户端执行的各种措施来控制它们提交给服务器的数据。 通常，这表示一个基本的安全漏洞：用户对客户端及其提交的数据具有完全控制权，并且可以绕过客户端实现的但不在服务器上复制的任何控件。

应用程序可能依靠客户端控件以两种主要方式限制用户输入。 首先，应用程序可以使用一种假定的机制通过客户端组件传输数据，该机制将阻止用户在以后读取数据时修改该数据。 其次，应用程序可以在客户端实施控制用户与他或她自己的客户端交互的措施，以限制功能and/or在提交用户输入之前围绕用户输入应用控件。 这可以使用HTML表单功能，客户端脚本或浏览器扩展技术来实现。

本章介绍了各种客户端控件的示例，并介绍了绕过它们的方法。

通常会看到应用程序以最终用户无法直接查看或修改的形式将数据传递给客户端，并期望此数据将在后续请求中发送回服务器。 通常，应用程序的开发人员只是假设使用的传输机制将确保通过客户端传输的数据不会在此过程中被修改。

由于从客户端提交到服务器的所有内容都在用户的控制范围之内，因此认为通过客户端传输的数据不会被修改的假设通常是错误的，并且经常使应用程序容易受到一种或多种攻击。

您可能会合理地想知道，如果服务器知道并指定了特定的数据项，那么应用程序将永远需要将此值传输给客户端然后再读回。 实际上，由于各种原因，以这种方式编写应用程序通常使开发人员更容易:
- 无需跟踪用户会话中的各种数据。 减少存储在服务器上的每次会话数据量也可以提高应用程序的性能。
- 如果应用程序部署在几台不同的服务器上，并且用户可能与多台服务器交互以执行多步操作，那么在可能处理同一用户请求的主机之间共享服务器端数据可能并不容易。 使用客户端传输数据可能是解决该问题的诱人方法。
- 如果应用程序使用服务器上的任何第三方组件（例如购物车），则很难或不可能进行修改，因此，通过客户端传输数据可能是集成这些组件的最简单方法。
- 在某些情况下，跟踪服务器上的新数据可能需要更新服务器核心API，从而触发成熟的正式变更管理过程和回归测试。 实施涉及客户端数据传输的更零碎的解决方案可以避免这种情况，从而可以满足紧迫的期限。

但是，以这种方式传输`敏感数据`通常是不安全的，并且已成为应用程序中无数漏洞的原因。

# 1.1 隐藏表单字段(Hidden Form  Fields)

隐藏的HTML表单字段是一种通用的机制，用于通过客户端以超表面的方式进行数据传输。 如果字段标记为隐藏，则该字段不会显示在屏幕上。 但是，字段的名称和值存储在表单中，并在用户提交表单时发送回应用程序。

此安全漏洞的经典示例是一个零售应用程序，该程序将产品的价格存储在隐藏字段中。 在Web应用程序的早期，此漏洞非常普遍，并且今天还没有消除。 图5-1显示了一种典型形式。

![figure5-1](/img/web_hacking/twahh/figure5-1.jpg)

该表格后面的代码如下：
```
<form method=”post” action=”Shop.aspx?prod=1”>
Product: iPhone Ultimate<br/>
Price: 449 <br/>
Quantity: <input type=”text” name=”quantity”> (Maximum quantity is 50)
<br/>
<input type=”hidden” name=”price” value=”449”>
<input type=”submit” value=”Buy”>
</form>
```
请注意，称为价格的字段字段被标记为隐藏.当用户提交表单时，该字段将发送到服务器:
```
POST /shop/28/Shop.aspx?prod=1 HTTP/1.1
Host: mdsec.net
Content-Type: application/x-www-form-urlencoded
Content-Length: 20

quantity=1&price=449
```
尽管`price`字段未显示在屏幕上，并且用户无法对其进行编辑，但这仅是因为该应用程序已指示浏览器隐藏该字段。 由于客户端发生的一切最终都在用户的控制范围内，因此可以绕开此限制来编辑价格。

一种实现方法是保存HTML页面的源代码，编辑该字段的值，将源代码重新加载到浏览器中，然后单击“购买”按钮。 但是，一种更简单，更优雅的方法是使用拦截代理来实时修改所需的数据。代理位于您的Web浏览器和目标应用程序之间。

一旦安装了拦截代理并进行了适当配置，您就可以捕获提交表单的请求，并将价格字段修改为任何值，如图5-2所示。

![figure5-2](/img/web_hacking/twahh/figure5-2.jpg)

如果您发现以这种方式易受攻击的应用程序，请查看是否可以提交负数作为价格。 在某些情况下，应用程序实际上已经使用`负价`格接受了交易。 攻击者会从他的信用卡以及所订购的物品中获得退款，这是双赢的情况，如果有的话。

# 1.2 HTTP Cookies

如果您发现以这种方式易受攻击的应用程序，请查看是否可以提交负数作为价格。 在某些情况下，应用程序实际上已经使用负价格接受了交易。 攻击者会从他的信用卡以及所订购的物品中获得退款，这是双赢的情况，如果有的话。

考虑上一个示例的以下变体。 客户登录到应用程序后，她收到以下响应:
```
HTTP/1.1 200 OK
Set-Cookie: DiscountAgreed=25
Content-Length: 1530
...
```
`DiscountAgreed` Cookie指出了依靠客户端控件（通常无法修改Cookie的事实）来保护通过客户端传输的数据的经典案例。 如果应用程序在将`DiscountAgreed` cookie提交回服务器时信任它的值，则客户可以通过修改其值来获得任意折扣。 例如:
```
POST /shop/92/Shop.aspx?prod=3 HTTP/1.1
Host: mdsec.net
Cookie: DiscountAgreed=25
Content-Length: 10

quantity=1
```

# 1.3 URL参数(URL Parameters)

应用程序经常使用预设的URL参数通过客户端传输数据。 例如，当用户浏览产品目录时，应用程序可能会向他提供指向URL的超链接，如下所示:
```
http://mdsec.net/shop/?prod=3&pricecode=32
```
当包含参数的URL显示在浏览器的位置栏中时，任何用户都可以轻松修改任何参数，而无需使用工具。 但是，在许多情况下，应用程序可能希望普通用户无法查看或修改URL参数:
- 使用包含参数的URL加载嵌入式图像的位置
- 包含参数的网址用于加载框架的内容
- 表单使用POST方法且其目标URL包含预设参数的地方
- 应用程序使用弹出窗口或其他技术隐藏浏览器位置栏的地方

当然，在任何这种情况下，都可以按照前面的讨论使用拦截代理来修改任何URL参数的值。

# 1.4 Referer头(The Referer Header)

浏览器在大多数HTTP请求中都包含Referer标头。 它用于指示当前请求所源自的页面的URL，或者是因为用户单击了超链接或提交了表单，或者是因为该页面引用了其他资源（例如图像）。 因此，可以利用它作为通过客户端传输数据的机制。 因为应用程序处理的URL在其控制范围内，所以开发人员可以假定Referer标头可用于可靠地确定哪个URL生成了特定请求。

例如，考虑一种机制，如果用户忘记了密码，该机制可使用户重置密码。 该应用程序要求用户按照以下顺序依次执行几个步骤，然后根据以下请求实际重置密码值:
```
GET /auth/472/CreateUser.ashx HTTP/1.1
Host: mdsec.net
Referer: https://mdsec.net/auth/472/Admin.ashx
```
应用程序可以使用Referer标头来验证此请求是否源自正确的阶段（`Admin.ashx`）。 如果确实如此，则用户可以访问请求的功能。

但是，由于用户控制了每个请求的各个方面，包括HTTP标头，因此可以通过直接进入`CreateUser.ashx`并使用拦截代理将Referer标头的值更改为应用程序所需的值来轻松地规避此控件。

根据w3.org标准，Referer标头严格是可选的。 因此，尽管大多数浏览器都实现了它，但使用它来控制应用程序功能应视为黑客。

**HACK STEPS**
- 1.在应用程序中的所有实例中，显然都使用`隐藏的表单`字段，`cookie`和`URL参数`来通过客户端传输数据。
- 2.根据项目出现的上下文和参数名称等线索，尝试确定或猜测项目在应用程序逻辑中扮演的角色。
- 3.以与其在应用程序中的目的相关的方式修改项目的值。确定应用程序是否处理参数中提交的任意值，以及是否使应用程序暴露于任何漏洞。

# 1.5 不透明的数据(Opaque Data)

有时，通过客户端传输的数据不是透明可理解的，因为它已经以某种方式进行了加密或混淆。 例如，您可能没有看到某个隐藏的值，而没有看到存储在隐藏字段中的产品价格:
```
<form method=”post” action=”Shop.aspx?prod=4”>
Product: Nokia Infinity <br/>
Price: 699 <br/>
Quantity: <input type=”text” name=”quantity”> (Maximum quantity is 50)
<br/>
<input type=”hidden” name=”price” value=”699”>
<input type=”hidden” name=”pricing_token”
value=”E76D213D291B8F216D694A34383150265C989229”>
<input type=”submit” value=”Buy”>
</form>
```
观察到这种情况时，您可以合理地推断出，在提交表单时，服务器端应用程序将检查不透明字符串的完整性，甚至对其进行解密或反模糊处理，以对其纯文本值进行一些处理。 这种进一步的处理可能容易受到任何类型的错误的影响。 但是，要对此进行探测和利用，首先需要适当包装有效负载。

通过客户端传输的不透明数据项通常是应用程序会话处理机制的一部分。 HTTP cookie中发送的会话令牌，隐藏字段中发送的anti-CSRF令牌以及用于访问应用程序资源的一次性URL令牌都是客户端篡改的潜在目标。 如第7章深入讨论的，对于这些类型的令牌有很多考虑。

**HACK STEPS**<br>
面对通过客户端传输的不透明数据，可能有几种攻击途径:
- 如果您知道不透明字符串后面的纯文本的值，则可以尝试解密所采用的混淆算法。
- 如第4章所述，应用程序可能包含其他功能，您可以在其中利用这些功能返回由您控制的一段纯文本产生的不透明字符串。 在这种情况下，您可能能够直接获取所需的字符串，以将任意有效负载传递给目标函数。
- 即使不透明字符串不可穿透，也有可能在其他情况下重播其值以达到恶意效果。 例如，以前显示的表单中的price_token参数可能包含产品价格的加密版本。尽管不可能以您选择的任意价格生成加密的等价物，但是您可以从其他`更便宜`的产品中复制加密的价格，然后将其提交。
- 如果所有其他方法均失败，则可以尝试攻击服务器端逻辑，该逻辑将通过提交不正确的字符串格式不正确的变体来解密或模糊不清字符串，例如，包含超长值，不同的字符集等。

# 1.6 The ASP.NET ViewState

通过客户端传输不透明数据的一种常见机制是`ASP.NET ViewState`。 这是默认情况下在所有`ASP.NET Web`应用程序中创建的`隐藏字段`。 它包含有关当前页面状态的`序列化`信息。 ASP.NET平台使用ViewState来增强服务器性能。 它使服务器能够在连续的请求中保留用户界面中的元素，而无需在服务器端维护所有相关的状态信息。 例如，服务器可以基于用户提交的参数来填充下拉列表。 当用户发出后续请求时，浏览器不会将列表的内容提交回服务器。 但是，浏览器确实提交了隐藏的ViewState字段，该字段包含列表的序列化形式。 服务器反序列化ViewState并重新创建再次显示给用户的相同列表。

除了ViewState的核心目的之外，开发人员还可以使用它来存储连续请求中的任意信息。 例如，应用程序可以如下将其保存在ViewState中，而不是将产品的价格保存在隐藏的表单字段中:
```
string price = getPrice(prodno);
ViewState.Add(“price”, price);
```
现在，返回给用户的表单如下所示：
```
<form method=”post” action=”Shop.aspx?prod=3”>
<input type=”hidden” name=”__VIEWSTATE” id=”__VIEWSTATE”
value=”/wEPDwULLTE1ODcxNjkwNjIPFgIeBXByaWNlBQMzOTlkZA==” />
Product: HTC Avalanche <br/>
Price: 399 <br/>
Quantity: <input type=”text” name=”quantity”> (Maximum quantity is 50)
<br/>
<input type=”submit” value=”Buy”>
</form>
```
用户提交表单时，其浏览器将发送以下内容：
```
POST /shop/76/Shop.aspx?prod=3 HTTP/1.1
Host: mdsec.net
Content-Type: application/x-www-form-urlencoded
Content-Length: 77

__VIEWSTATE=%2FwEPDwULLTE1ODcxNjkwNjIPFgIeBXByaWNlBQMzOTlkZA%3D%3D&
quantity=1
```
该请求显然不包含产品价格-仅包含订购的数量和不透明的ViewState参数。 随机更改该参数将导致错误消息，并且未处理购买。
ViewState参数实际上是一个Base64编码的字符串，可以很容易地解码以查看放置在此处的price参数：
```
3D FF 01 0F 0F 05 0B 2D 31 35 38 37 31 36 39 30 ; =ÿ.....-15871690
36 32 0F 16 02 1E 05 70 72 69 63 65 05 03 33 39 ; 62.....price..39
39 64 64                                        ; 9dd
```
**TIP**
>当您尝试解码看似是Base64编码的字符串时，一个常见的错误是在字符串中错误的位置开始解码。 由于Base64编码的工作原理，如果从错误的位置开始，则解码的字符串将包含乱码。 Base64是基于块的格式，其中每4字节的编码数据转换为3字节的解码数据。 因此，如果您尝试解码Base64字符串时没有发现任何有意义的内容，请尝试从四个相邻的偏移量开始到编码的字符串。

默认情况下，ASP.NET平台通过向其添加键控哈希来保护ViewState免受篡改（称为MAC保护）。 但是，某些应用程序会禁用此默认保护，这意味着您可以修改ViewState的值以确定它是否对应用程序的服务器端处理产生影响。

Burp Suite包含一个ViewState解析器，该解析器指示ViewState是否受MAC保护，如图5-3所示。 如果未受保护，则可以使用ViewState树下方的十六进制编辑器在Burp中编辑ViewState的内容。 当您将消息发送到服务器或客户端时，Burp将发送更新的ViewState，并且在本示例中，使您能够更改所购买商品的价格。

![figure5-3](/img/web_hacking/twahh/figure5-3.jpg)

**HACK STEPS**
- 1.如果要攻击ASP.NET应用程序，请验证是否为ViewState启用了`MAC保护`。 这由ViewState结构末尾存在20个字节的哈希表示，您可以使用Burp Suite中的ViewState解析器来确认是否存在此哈希。
- 2.即使ViewState受保护，也可以使用Burp对各种应用程序页面上的ViewState进行解码，以发现应用程序是否正在使用ViewState通过客户端传输任何敏感数据。
- 3.即使ViewState受保护，也可以使用Burp对各种应用程序页面上的ViewState进行解码，以发现应用程序是否正在使用ViewState通过客户端传输任何敏感数据。
- 4.如果可以修改ViewState而不会导致错误，则应查看ViewState中每个参数的功能，并查看应用程序是否使用它来存储任何自定义数据。 尝试提交精心设计的值作为每个参数以探查常见漏洞，就像通过客户端传输的任何其他数据项一样。
- 5.请注意，可以`逐页`启用或禁用MAC保护，因此可能有必要测试应用程序的每个重要页面是否存在ViewState黑客漏洞。 如果使用启用了被动扫描的Burp扫描仪，则Burp会自动报告使用ViewState且未启用MAC保护的所有页面。

[Chapter 5 Bypassing Client-Side Controls(2) - Capturing User Data: HTML Forms](https://dm116.github.io/2020/03/08/bypassing-client-side-controls_2/)
