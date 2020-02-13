---
layout:     post
title:      Web应用程序黑客的方法论(7) --Test for Input-Based Vulnerabilities
subtitle:
date:       2020-02-14
author:     D
header-img: 
catalog: true
mermaid: true
tags:
    - web hacking
---

Web应用程序黑客的方法论来自书本 *The Web Application Hacker's Handbook* <br>

漏洞的许多重要类别是由用户的意外输入触发的，并且可以出现在应用程序中的任何位置。<br>
探测应用程序中这些漏洞的有效方法是使用一组攻击字符串来模糊每个参数到每个请求。<br>

![Testing access controls](/img/test-for-input-based-vulnerabilities.png)

# 7.1 模糊测试所有请求参数(Fuzz All Request Parameters)

7.1.1<br>
查看您的应用程序映射练习的结果，并识别提交服务器端应用程序处理的参数的每个不同的客户端请求。相关参数包括URL查询字符串中的项目，请求正文中的参数和HTTP cookie。还应包括观察到的对应用程序行为有影响的任何其他用户输入项，例如Referer或User-Agent标头。

7.1.2<br>
要模糊化参数，可以使用自己的脚本或现成的模糊化工具。例如，要使用Burp Intruder，请将每个请求依次加载到工具中。一种简单的方法是在Burp Proxy中拦截请求，然后选择Send to Intruder操作，或右键单击Burp Proxy历史记录中的项目，然后选择此选项。使用此选项可使用请求的内容以及正确的目标主机和端口来配置Burp Intruder。 它还会自动将所有请求参数的值标记为有效负载位置，以准备进行模糊测试。

7.1.3<br>
使用有效负载选项卡，配置一组合适的攻击有效负载，以探测应用程序中的漏洞。您可以手动输入有效负载，从文件中加载它们，或选择预设的有效负载列表之一。对应用程序内的每个请求参数进行模糊处理通常需要发出大量请求并检查结果是否存在异常。如果攻击字符串集太大，可能会适得其反，并产生大量的输出供您查看。因此，明智的方法是针对一系列常见漏洞，这些漏洞通常可以在对特定制作的输入的异常响应中容易地检测到，并且通常在应用程序中的任何位置而不是特定功能类型中显现出来。这是一组合适的有效载荷，可用于测试一些常见类别的漏洞：<br>

**SQL Injection** <br>

```
'
'--
'; waitfor delay '0:30:0'--
1; waitfor delay '0:30:0'--
```

**XSS and Header Injection** <br>

```
xsstest
"><script>alert('xss')</script>
```



<br>

[Web应用程序黑客的方法论(6)](https://dm116.github.io/2020/02/13/web-application-hacker-methodology_6/)<br>
[Web应用程序黑客的方法论(8)](https://dm116.github.io/2020/02/14/web-application-hacker-methodology_8/)<br>


