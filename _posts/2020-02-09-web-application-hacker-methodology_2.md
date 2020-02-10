---
layout:     post
title:      Web应用程序黑客的方法论(2)
subtitle:   
date:       2020-02-09
author:     D
header-img: 
catalog: true
mermaid: true
tags:
    - web hacking
---

Web应用程序黑客的方法论来自书本 *The Web Application Hacker's Handbook*

# 分析应用程序(analyze the application)

![analyzing the application](/img/analyzing-the-application.png)

## 2.1 识别功能(Identify Functionality)

2.1.1<br>
识别为应用程序创建的核心功能，以及每个功能在按预期使用时要执行的操作。

2.1.2<br>
确定应用程序使用的核心安全机制及其工作方式。特别是，要了解处理`身份验证`、`会话管理`和`访问控制`的密钥机制，以及支持这些机制的功能，如`用户注册`和`帐户恢复`。

2.1.3<br>
识别所有更`外围`的功能和行为，如使用`重定向`、`站点外链接`、`错误消息`、`管理`和`日志`记录功能.

2.1.4<br>
识别与应用程序中其他地方使用的`标准GUI`外观、`参数命名`或`导航机制`不同的任何功能，并将其挑出来进行深入测试.

## 2.2 识别数据入口点(Identify Data Entry Points)

2.2.1<br>
识别用于将用户输入引入应用程序处理的所有不同`入口`点，包括`URLs`、query string parameters、`POST`数据、`cookies`和应用程序处理的其他`HTTP headers`.

2.2.2<br>
检查应用程序使用的任何自定义数据传输或编码机制，如非标准查询字符串格式。了解提交的数据是否封装了参数名称和值，或者是否使用了替代的表示方法。

2.2.3<br>
识别`user-controllable`或其他`third-party`数据被引入应用程序处理的任何`out-of-band`通道.一个示例是处理并呈现通过SMTP接收到的消息的web邮件应用程序。

## 2.3 识别使用的技术(Identify the Technologies Used)

2.3.1<br>
识别客户端使用的不同技术，如forms、scripts、cookie、Java applet、ActiveX控件和Flash对象.

2.3.2<br>
尽可能确定服务器端正在使用哪些技术，包括脚本语言、应用程序平台以及与后端组件(如数据库和电子邮件系统)的交互。

2.3.3<br>
检查应用程序响应中返回的`HTTP Server header`，并检查自定义HTTP标头或HTML源代码注释中包含的任何其他软件标识符。注意，在某些情况下，应用程序的不同区域由不同的后端组件处理，因此可能接收到不同的标识。

2.3.4<br>
运行`Httprint`工具来识别 web server.

2.3.5<br>
检查content-mapping习的结果，以确定任何看起来有趣的文件扩展名、目录或其他URL子序列，它们可能提供关于服务器上使用的技术的线索。检查任何会话tokens和发出的其他cookies的names。使用谷歌搜索与这些项相关的技术.

2.3.6<br>
识别可能属于第三方代码组件的任何有趣的脚本名称和查询字符串参数.使用`inurl: qualifier`在谷歌上搜索这些，以找到使用相同脚本和参数的其他应用程序，因此可能使用相同的第三方组件.对这些站点进行非侵入性检查，因为这可能会发现在您要攻击的应用程序上没有显式链接的其他内容和功能.

## 2.4 映射攻击面(Map the Attack Surface)

2.4.1<br>
尝试确定服务器端应用程序可能的内部结构和功能，以及它在幕后用于交付从客户端角度可见的行为的机制。 例如，检索客户订单的函数可能与数据库交互.

2.4.2<br>
对于每个功能项，确定经常与之相关的常见漏洞的类型。例如,`file upload`函数可能容易受到路径遍历的攻击，用户间消息传递可能容易受到`XSS`的攻击，而`Contact Us`函数可能容易受到`SMTP injection`的攻击。

2.4.3<br>
制定一个攻击计划，优先考虑最有趣的功能和与之相关的最严重的潜在漏洞.使用您的计划来指导您在此方法的每个剩余领域中所投入的时间和精力.

[Web应用程序黑客的方法论(1)](https://dm116.github.io/2020/02/03/web-application-hacker-methodology/)<br>
[Web应用程序黑客的方法论(3)](https://dm116.github.io/2020/02/09/web-application-hacker-methodology_3/)
