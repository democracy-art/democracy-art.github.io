---
layout:     post
title:      Web应用程序技术 - Chapter 4 - Mapping the Application
subtitle:   
date:       2020-03-03
author:     D
header-img: 
catalog: true
mermaid: true
tags:
    - [web hacking]
---

参考:*The Web Application Hacker's Handbook* 
Chapter 4 Mapping the Application


攻击应用程序的第一步是收集和检查有关该应用程序的一些关键信息，以更好地了解您所面临的挑战。

映射练习首先枚举应用程序的`内容`和`功能`，以了解应用程序的功能以及行为方式。 此功能的大部分易于识别，但其中某些功能可能是`隐藏`的，需要一定程度的猜测和运气才能发现。

主要任务是仔细检查其行为的各个方面，其核心`安全机制`以及所采`用的技术`（在`客户端`和`服务器`上）。 这将使您能够识别应用程序暴露的主要`攻击面`，从而可以确定最有趣的`区域`，应在这些区域中进行后续探测以发现可利用的漏洞。

随着应用程序变得越来越大，功能越来越多，`有效的映射`是一项宝贵的技能。 经验丰富的专家可以`快速`对功能的所有区域进行分类，查找与实例相对应的漏洞类别，同时将大量时间投入到测试其他特定领域，以发现`高风险`问题。

本章描述了在应用`程序映射`过程中需要遵循的实际`步骤`，可以用来`最大化`其`有效性`的各种技术和技巧，以及一些可以在此过程中帮助您的工具。

# 1. Enumerating Content and Functionality(枚举内容和功能)

在典型的应用程序中，大多数内容和功能都可以通过`手动浏览`来识别。基本的方法是`遍历`应用程序，从主初始页面开始，跟踪每个`链接`，并通过导航控制所有多级功能(如`用户注册`或`密码重置`)。如果应用程序包含`site map`，这可以为枚举内容提供一个有用的起点。

然而，要对列举的内容进行严格的检查，并获得所标识的所有内容的完整记录，您必须使用比简单浏览更高级的技术。

### 1.1 Web Spidering(网络搜索)
各种工具可以执行自动的网站搜索,这些工具的工作方式是:请求一个web页面,解析它以获得到其他内容的链接,请求这些链接,然后递归地继续操作,直到没有发现新的内容为止.
工具:Burp Suite, WebScarab, Zed Attack, CAT.

许多Web服务器在Web根目录中包含一个名为`robots.txt`的文件，该文件包含该站点**不**希望Web蜘蛛访问或搜索引擎建立索引的URL列表。 有时，robots.txt文件可能会对Web应用程序的安全起反作用。

### 1.2 User-Directed Spidering(用户控制的搜索)
### 1.3 Discovering Hidden Content(挖掘隐藏的内容)
### 1.4 Application Pages Versus Functional Paths(应用程序页面与功能路径)
### 1.5 Discovering Hidden Parameters(挖掘隐藏的参数)

# 2. Analyzing the Application(分析程序)
### 2.1 Identifying Entry Points for User Input(识别用户输入的入口点)
### 2.2 Identifying Server-Side Technologies(识别服务器端技术)
### 2.3 Identifying Server-Side Functionality(识别服务器端功能)
### 2.4 Mapping the Attack Surface(绘制攻击面)

[Web应用程序技术 - Chapter 3(2) - Web Functionality and Encoding Schemes](https://dm116.github.io/2020/03/03/web-application-technologies_2)
