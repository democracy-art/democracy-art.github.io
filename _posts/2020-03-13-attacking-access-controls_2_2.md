---
layout:		post
title:		Chapter 8 Attacking Access Controls(2)-Attacking Access Controls(2)
subtitle:	
date:		2020-03-13
author:		D
header-img:
catalog:	true
mermaid:	true
tags: [web hacking]:
---

参考: *The Web Application Hacker's Handbook* Chapter 8

# 1.测试对方法的直接访问(Testing Direct Access to Methods)

在应用程序使用直接访问服务器端API方法的请求的情况下，通常使用已描述的方法来识别这些方法中的任何访问控制弱点。 但是，您还应该测试是否存在可能未得到适当保护的其他API。
例如，可以使用以下请求来调用servlet：
```
POST /svc HTTP/1.1
Accept-Encoding: gzip, deflate
Host: wahh-app
Content-Length: 37

servlet=com.ibm.ws.webcontainer.httpsession.IBMTrackerDebug
```
由于这是一个众所周知的servlet，也许您可以访问其他servlet来执行未经授权的操作。

**HACK STEPS**
- 1.标识遵循Java命名约定的任何参数（例如，获取，设置，添加，更新，是或后面带有大写字母的单词），或明确指定包结构（例如，`com.companyname.xxx.yyy.ClassName`）。 记录所有可以找到的引用方法。
- 2.寻找一种列出可用接口或方法的方法。 检查您的代理历史记录，以查看它是否已作为应用程序正常通信的一部分被调用。 如果不是，请尝试使用观察到的命名约定进行猜测。
- 3.咨询诸如搜索引擎和论坛站点之类的公共资源，以确定其他可能可以访问的方法。
- 4.使用第4章中介绍的技术来猜测其他方法名称。
- 5.尝试访问使用各种用户帐户类型收集的所有方法，包括未经身份验证的访问。
- 6.如果您不知道某些方法所需的参数数目或类型，请查找不太可能采用参数的方法，例如`listInterfaces`和`getAllUsersInRoles`。

# 2.测试对静态资源的控制(Testing Controls Over Static Resources)

如果最终通过URL直接访问应用程序保护的静态资源到资源文件本身，则应该测试未经授权的用户是否可以直接直接请求这些URL。

**HACK STEPS**
- 1.逐步完成获取受保护的静态资源访问的正常过程，以获取最终通过其检索到的URL的示例。
- 2.使用其他用户上下文（例如，特权较低的用户或尚未进行必购的帐户），尝试使用您标识的URL直接访问资源。
- 3.如果此攻击成功，请尝试了解用于受保护的静态文件的命名方案。 如果可能，构造一个自动攻击程序，以搜寻可能有用或可能包含敏感数据的内容（请参阅第14章）。

# 3.测试对HTTP方法的限制(Testing Restrictions on HTTP Methods)

尽管可能尚无法通过一种简便的方法来检测应用程序的访问控制是否通过HTTP方法使用了平台级控制，但是您可以采取一些简单的步骤来识别任何漏洞。

**HACK STEPS**
- 1.使用高特权帐户，识别一些执行敏感操作的特权请求，例如添加新用户或更改用户的安全角色。
- 2.如果这些请求不受任何反CSRF令牌或类似功能的保护（请参阅第13章），请使用高特权帐户来确定如果修改了HTTP方法，则应用程序是否仍执行所请求的操作。 测试以下HTTP方法：
	- POST
	- GET
	- HEAD
	- An arbitrary invalid HTTP method
- 3.如果应用程序使用与原始方法不同的HTTP方法接受任何请求，请使用具有较低特权的帐户，使用已描述的标准方法测试对这些请求的访问控制。


[Chapter 8 Attacking Access Controls(2)-Attacking Access Controls(1)](https://dm116.github.io/2020/03/13/attacking-access-controls_2_1/)<br>
[Chapter 8 Attacking Access Controls(3)-Securing Access Controls](https://dm116.github.io/2020/03/13/attacking-access-controls_3/)<br>
