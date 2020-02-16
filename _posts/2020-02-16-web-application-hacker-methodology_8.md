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

![Test for Function-Specific Input Vulnerabilities](/img/Testing for functionality-specific input vulnerabilities.png)

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



[Web应用程序黑客的方法论(7)](https://dm116.github.io/2020/02/14/web-application-hacker-methodology_7/)<br>
[Web应用程序黑客的方法论(9)](https://dm116.github.io/2020/02/17/web-application-hacker-methodology_9/)<br>


