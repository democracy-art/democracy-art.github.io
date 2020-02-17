---
layout:     post
title:      Web应用程序黑客的方法论(12) --Miscellaneous Checks
subtitle:
date:       2020-02-17
author:     D
header-img: 
catalog: true
mermaid: true
tags:
    - web hacking
---

Web应用程序黑客的方法论来自书本 *The Web Application Hacker's Handbook* <br>

![Miscellaneous Checks](/img/miscellaneous-checks.png)

# 12.1 检查基于DOM的攻击(Check for DOM-Based Attacks)

12.1.1<br>
对从应用程序收到的每段JavaScript进行简短的代码审查。 识别可以通过使用特制URL将恶意数据引入相关页面的DOM触发的任何XSS或重定向漏洞。 包括HTML页面（静态和动态生成）中包含的所有独立JavaScript文件和脚本。

12.1.2<br>
标识以下API的所有用法，这些API可用于访问可通过特制URL控制的DOM数据:<br>
```
document.location
document.URL
document.URLUnencoded
document.referrer
window.location
```

12.1.3<br>
通过代码跟踪相关数据，以识别将执行哪些操作。 如果将数据（或其操作形式）传递给以下API之一，则该应用程序可能容易受到XSS的攻击:<br>
```
document.write()
document.writeln()
document.body.innerHtml
eval()
window.execScript()
window.setInterval()
window.setTimeout()
```

12.1.4<br>
如果将数据传递到以下APIs之一，则该应用程序可能容易受到重定向攻击:<br>
```
document.location
document.URL
document.open()
window.location.href
window.navigate()
window.open()
```

# 12.2 检查本地隐私漏洞(Check for Local Privacy Vulnerabilities)

12.2.1<br>
查看由拦截代理创建的日志，以识别测试期间从应用程序收到的所有Set-Cookie指令。 如果其中任何一个包含一个带有未来日期的`expires`属性，则该cookie将由用户的浏览器存储到该日期为止。 查看所有持久性Cookie的内容以获取敏感数据。

12.2.2<br>
如果设置了包含任何敏感数据的永久cookie，则本地攻击者可能能够捕获此数据。 即使数据被加密，捕获数据的攻击者也可以将Cookie重新提交给应用程序，并获得对此允许的任何数据或功能的访问权限。

12.2.3<br>
如果通过HTTP访问了包含敏感数据的任何应用程序页面，请在服务器的响应中查找所有缓存指令。 如果以下任何指令不存在（在HTTP标头中或在HTML元标记中），则相关的页面可能被一个或多个浏览器缓存:<br>
```
Expires: 0
Cache-control: no-cache
Pragma: no-cache
```

12.2.4<br>
标识应用程序中通过URL参数传输敏感数据的所有实例。 如果存在任何情况，请检查浏览器历史记录以确认此数据已存储在此处。

12.2.5<br>
对于所有用于捕获用户敏感数据的表单（例如信用卡详细信息），请查看表单的HTML源。 如果未在表单标签或单个输入字段的标签中设置属性autocomplete = off，则输入的数据将存储在支持自动完成功能的浏览器中，前提是用户未禁用此功能。

12.2.6<br>
检查特定于技术的本地存储。

- 12.2.6.1 使用Firefox的BetterPrivacy插件检查Flash本地对象。
- 12.2.6.2 检查以下目录中的任何Silverlight隔离存储:
```
C:\Users\{username}\AppData\LocalLow\Microsoft\Silverlight\
```
- 12.2.6.3 检查是否使用了HTML5本地存储.

# 12.3 检查弱SSL密码(Check for Weak SSL Ciphers)

12.3.1<br>
如果应用程序使用SSL进行任何通信，请使用工具`THCSSLCheck`列出支持的密码和协议。

12.3.2<br>
如果支持任何弱的或过时的密码和协议，则位置适当的攻击者可能能够执行攻击以降级或解密应用程序用户的SSL通信，从而访问其敏感数据。

12.3.3<br>
某些Web服务器会发布某些受支持的弱密码和协议，但如果客户端请求，则拒绝使用它们来完成握手。 当您使用THCSSLCheck工具时，这可能导致误报。 您可以使用Opera浏览器尝试使用指定的弱协议进行完整的握手，以确认这些握手是否可以真正用于访问应用程序。

# 12.4 检查同源策略配置(Check Same-Origin Policy Confi guration)

12.4.1<br>
检查`/crossdomain.xml`文件。如果应用程序允许无限制访问(通过指定`<allow-access-from domain =" *" />`),则来自任何其他站点的Flash对象都可以基于应用程序用户的会话执行双向交互。这将允许任何其他域检索所有数据，并执行任何用户操作。

12.4.2<br>
检查`/clientaccesspolicy.xml`文件。 与Flash相似，如果`<cross-domain-access>`配置太宽松，其他站点可以与被评估站点进行双向交互。

12.4.3<br>
通过添加指定不同域的Origin标头并检查返回的所有Access-Control标头，使用XMLHttpRequest测试应用程序对跨域请求的处理。 允许从任何域或指定的其他域进行双向访问的安全隐患与Flash跨域策略所描述的隐患相同。<br>


[Web应用程序黑客的方法论(11)](https://dm116.github.io/2020/02/17/web-application-hacker-methodology_11/)<br>
[Web应用程序黑客的方法论(13)](https://dm116.github.io/2020/02/17/web-application-hacker-methodology_13/)<br>




