---
layout:     post
title:      Web应用程序黑客的方法论
subtitle:   
date:       2020-02-03
author:     D
header-img: 
catalog: true
mermaid: true
tags:
    - methodology
---

# Web黑客方法论框架

Web应用程序黑客的方法论来自书本 *The Web Application Hacker's Handbook*

一般步骤如下:

![methodology](/img/web_application_hacker_methodology.PNG)

请记住，HTTP请求的不同部分中有几个字符具有特殊的含义。当您在请求中修改数据时，<br>
您应该对这些字符进行url编码，以确保它们按照您希望的方式进行解释.

|ASCII|URL Encode|用途|
|-|-|-|
|&|%26|分隔URL查询字符串和消息正文中的`参数`|
|=|%3d|分隔URL查询字符串和消息体中的每个`参数`的名称和`值`|
|?|%3f|标记URL查询字符串的`开头`|
|空格|`%20` or `+`|请求的第一行中标记URL的结束,并可以在cookie头中指示cookie值的结束|
|+|%2b||
|;|%3b|在Cookie标头中`分隔`单独的Cookie|
|#|%23|在URL中标记`片段`标识符|
|%|%25|在URL编码方案中用作前缀|
|null|%00||
|换行|%0a||

# 1.映射应用程序的内容(Map the Application's Content)
|链接内容|其他内容|非标准的访问方法|
|-|-|-|
|1.1 探索可见的内容|1.3 发现隐藏的内容|1.5 标识特殊函数|
|1.2 查阅公共资源|1.4 发现默认的内容|1.6 调试参数|
## 1.1 探索可见的内容(Explore Visible Content)



