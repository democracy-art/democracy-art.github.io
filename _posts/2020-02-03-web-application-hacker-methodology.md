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

Web应用程序黑客的方法论来自书本 *The Web Application Hacker's Handbook*

![methodology](/img/web_application_hacker_methodology.PNG)

请记住，HTTP请求的不同部分中有几个字符具有特殊的含义。当您在请求中修改数据时，<br>
您应该对这些字符进行url编码，以确保它们按照您希望的方式进行解释.

|ASCII|URL Encode|用途|
|-|-|-|
|&|%26|分隔URL查询字符串和消息正文中的`参数`|
|=|%3d|分隔URL查询字符串和消息体中的每个`参数`的名称和`值`|
