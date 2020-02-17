---
layout:     post
title:      Web应用程序黑客的方法论(13) --Follow Up Any Information Leakage
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

# 跟进任何信息泄漏
13.1<br>
在对目标应用程序进行的所有探测中，请监视其对错误消息的响应，这些错误消息可能包含有关错误原因，使用的技术以及应用程序的内部结构和功能的有用信息。

13.2<br>
如果收到任何异常错误消息，请使用标准搜索引擎进行调查。 您可以使用各种高级搜索功能来缩小搜索范围.例如:<br>
```
"unable to retrieve" filetype:php
```
13.3<br>
查看搜索结果，查找有关错误消息的任何讨论以及出现相同消息的任何其他网站。 其他应用程序可能会在更详细的上下文中产生相同的消息，使您可以更好地了解是哪种情况导致了错误。 使用搜索引擎缓存来检索错误消息的示例，这些示例不再出现在实时应用程序中。

13.4<br>
使用Google代码搜索来查找可能导致特定错误消息的所有公开代码。 搜索错误消息的片段，这些片段可能被硬编码到应用程序的源代码中。 如果已知，您还可以使用各种高级搜索功能来指定代码语言和其他详细信息。 例如:<br>
```
unable\ to\ retrieve lang:php package:mail
```

13.5<br>
如果您收到包含堆栈跟踪的错误消息，其中包含库和第三方代码组件的名称，请在两种类型的搜索引擎上搜索这些名称。<br>

test comment.<br>

[Web应用程序黑客的方法论(12)](https://dm116.github.io/2020/02/17/web-application-hacker-methodology_12/)<br>


