---
layout:     post
title:      Chapter 5 Bypassing Client-Side Controls(2) - Capturing User Data:HTML Forms
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

# 1.捕获用户数据：HTML表单(Capturing User Data: HTML Forms)

应用程序使用客户端控件来限制客户端提交的数据的另一种主要方式发生于服务器最初未指定但在客户端计算机本身上收集的数据。

HTML表单是捕获用户输入并将其提交给服务器的最简单，最常见的方法。 使用此方法的最基本用途，用户可以将数据键入命名的文本字段，然后以名称/值对的形式提交给服务器。 但是，可以以其他方式使用表格。 它们可以对用户提供的数据施加限制或执行验证检查。 当应用程序使用这些客户端控件作为安全机制来防御恶意输入时，通常可以轻松地规避这些控件，从而使应用程序容易受到攻击。

## 1.1 长度限制(Length Limits)

请考虑原始HTML表单的以下变体，该变体在数量字段上施加最大长度为1：
```
<form method=”post” action=”Shop.aspx?prod=1”>
Product: iPhone 5 <br/>
Price: 449 <br/>
Quantity: <input type=”text” name=”quantity” maxlength=”1”> <br/>
<input type=”hidden” name=”price” value=”449”>
<input type=”submit” value=”Buy”>
</form>
```
在此，浏览器阻止用户在输入字段中输入多个字符，因此服务器端应用程序可以假定其接收到的数量参数小于10。但是，可以通过拦截来轻松地避免此限制。 包含表单提交的请求以输入任意值，或者通过拦截包含表单的响应以移除maxlength属性。

截取Responses:<br>
当您尝试拦截和修改服务器响应时，您可能会发现代理中显示的相关消息如下所示:<br>
```
HTTP/1.1 304 Not Modified
Date: Wed, 6 Jul 2011 22:40:20 GMT
Etag: “6c7-5fcc0900”
Expires: Thu, 7 Jul 2011 00:40:20 GMT
Cache-Control: max-age=7200
```
之所以出现此响应，是因为浏览器已经拥有它所请求资源的缓存副本。 当浏览器请求缓存的资源时，通常会向请求添加两个标头-`If-Modified-Since`和`If-None-Match`：
```
GET /scripts/validate.js HTTP/1.1
Host: wahh-app.com
If-Modified-Since: Sat, 7 Jul 2011 19:48:20 GMT
If-None-Match: “6c7-5fcc0900”
```
这些标头告诉服务器浏览器上次更新其缓存副本的时间。 服务器随资源提供的Etag字符串是服务器分配给每个可缓存资源的一种序列号。

每次修改资源时都会更新。 如果服务器拥有的资源版本比If-Modified-Since标头中指定的日期新，或者如果当前版本的Etag与If-None-Match标头中指定的版本匹配，则服务器将以 资源的最新版本。 否则，它将返回304响应（如此处所示），通知浏览器该资源尚未修改，并且浏览器应使用其缓存的副本。

发生这种情况时，您需要拦截和修改浏览器已缓存的资源，则可以拦截相关请求并删除If-Modified-Since和If-None-Match标头。 这将导致服务器以请求的资源的完整版本进行响应。 Burp代理包含一个选项，可从每个请求中剥离这些标头，从而覆盖浏览器发送的所有缓存信息。

**HACK STEPS**
- 1.查找包含maxlength属性的表单元素。 提交长于此长度但在其他方面格式正确的数据（例如，如果应用程序需要数字，则为数字）。
- 2.如果应用程序接受超长数据，则可以推断出客户端验证没有复制到服务器上。
- 3.根据应用程序对该参数执行的后续处理，您可能能够利用验证中的缺陷来利用其他漏洞，例如SQL注入，跨站点脚本或缓冲区溢出。

## 1.2 基于脚本的验证(Script-Based Validation)

内置在HTML表单中的输入验证机制非常简单，并且无法对许多输入进行相关验证。 例如，用户注册表单可能包含名称，电子邮件地址，电话号码和邮政编码的字段，所有这些字段均要求使用不同类型的输入。 因此，常见的是在脚本中实现自定义的客户端输入验证。 考虑原始示例的以下变体：
```
<form method=”post” action=”Shop.aspx?prod=2” onsubmit=”return
validateForm(this)”>
Product: Samsung Multiverse <br/>
Price: 399 <br/>
Quantity: <input type=”text” name=”quantity”> (Maximum quantity is 50)
<br/>
<input type=”submit” value=”Buy”>
</form>

<script>function validateForm(theForm)
{
	var isInteger = /^\d+$/;
	var valid = isInteger.test(quantity) &&
		quantity > 0 && quantity <= 50;
	if (!valid)
		alert(’Please enter a valid quantity’);
	return valid;
}
</script>
```
表单标记的`onsubmit`属性指示浏览器在用户单击“提交”按钮时执行`validateForm`函数，并且仅在此函数返回true时才提交表单。 这种机制使客户端逻辑可以拦截尝试的表单提交，对用户输入执行自定义的验证检查，以及决定是否接受该输入。 在前面的示例中，验证很简单； 它检查在字段中输入的数据是否为整数并且在1到50之间。

这种客户端控件通常很容易规避。 通常，在浏览器中禁用JavaScript就足够了。 如果这样做，将忽略onsubmit属性，并且无需任何自定义验证即可提交表单。

但是，如果禁用JavaScript依赖于其正常操作的客户端脚本（例如，构造用户界面的一部分），则可能会破坏应用程序。 一种更整洁的方法是在浏览器的输入字段中输入一个良性（已知良好）值，使用您的代理拦截经过验证的提交，然后将数据修改为所需的值。 这通常是击败基于JavaScript的验证的最简单，最优雅的方法。

另外，您可以拦截包含JavaScript验证例程的服务器响应，并修改脚本以抵消其影响-在上一示例中，通过将ValidateForm函数更改为在每种情况下均返回true来进行。

**HACK STEPS**
- 1.确定在提交表单之前使用客户端JavaScript进行输入验证的所有情况。
- 2.通过修改提交请求以注入无效数据或通过修改表单验证代码以中和数据，将数据提交到验证通常会阻止的服务器。
- 3.与长度限制一样，确定是否在服务器上复制了客户端控件，如果不是，则确定是否可以将其用于任何恶意目的。
- 4.请注意，如果在表单提交之前对多个输入字段进行了客户端验证，则需要使用无效数据分别测试每个字段，同时在所有其他字段中保留有效值。 如果您同时在多个字段中提交无效数据，则服务器在识别出第一个无效字段时可能会停止处理表单。 因此，您的测试不会到达应用程序中所有可能的代码路径。

**NOTE**
>验证用户输入的客户端JavaScript例程在Web应用程序中很常见，但不能得出结论，每个此类应用程序都容易受到攻击。 仅当未在服务器上复制客户端验证时才暴露应用程序，即使只有在绕过客户端验证的精心设计的输入可用于引起应用程序某些不良行为的情况下，应用程序才暴露。

>在大多数情况下，客户端对用户输入的验证会对应用程序的性能和用户体验质量产生有益的影响。 例如，填写一份详细的注册表格时，普通用户可能会犯各种错误，例如省略必填字段或错误地格式化其电话号码。 在没有客户端验证的情况下，纠正这些错误可能需要重新加载页面和往返消息到服务器。 在客户端实施基本的验证检查可以使用户的体验更加流畅，并减少服务器上的负载。

## 1.3 禁用元素(Disabled Elements)

如果将HTML表单上的元素标记为已禁用，则该元素会显示在屏幕上，但通常显示为灰色，并且无法以普通控件的方式进行编辑或使用。 此外，提交表单后，它不会发送到服务器。 例如，考虑以下形式:
```
<form method=”post” action=”Shop.aspx?prod=5”>
Product: Blackberry Rude <br/>
Price: <input type=”text” disabled=”true” name=”price” value=”299”>
<br/>
Quantity: <input type=”text” name=”quantity”> (Maximum quantity is 50)
<br/>
<input type=”submit” value=”Buy”>
</form>
```
这包括作为禁用文本字段的产品价格，并显示在屏幕上，如图5-4所示。

![figure5-4](/img/web_hacking/twahh/figure5-4.jpg)

提交此表单后，仅数量参数将发送到服务器。 但是，禁用字段的存在表明价格参数可能最初是由应用程序使用的，也许用于开发过程中的测试目的。 该参数将已提交到服务器，并且可能已由应用程序处理。 在这种情况下，您绝对应该测试服务器端应用程序是否仍在处理此参数。 如果确实如此，则寻求利用这一事实。

**HACK STEPS**
- 1.在每种形式的应用程序中查找禁用的元素。 只要找到一个，请尝试将其与表单的其他参数一起提交给服务器，以确定它是否有效。
- 2.通常，提交元素被标记为已禁用，以便在相关操作不可用时，上下文中的按钮显示为灰色。 您应始终尝试提交这些元素的名称，以确定应用程序是否在尝试执行请求的操作之前执行服务器端检查。
- 3.请注意，提交表单时，浏览器不包含禁用的表单元素。 因此，如果仅浏览应用程序的功能并监视浏览器发出的请求，就不会识别这些内容。 要识别禁用的元素，您需要监视服务器的响应或在浏览器中查看页面源。
- 4.您可以使用Burp Proxy中的HTML修改功能来自动重新启用应用程序中使用的所有禁用字段。


[Chapter 5 Bypassing Client-Side Controls(1) - Transmitting Data Via the Client](https://dm116.github.io/2020/03/08/bypassing-client-side-controls/)

[Chapter 5 Bypassing Client-Side Controls(3) - Capturing User Data: Browser Extensions](https://dm116.github.io/2020/03/08/bypassing-client-side-controls_3/)
