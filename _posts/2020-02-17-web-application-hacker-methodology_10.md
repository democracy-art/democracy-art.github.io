---
layout:     post
title:      Web应用程序黑客的方法论(10) --Test for Shared Hosting Vulnerabilities
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

![Test for Shared Hosting Vulnerabilities](/img/test-for-shared-hosting-vulnerabilities.png)

# 10.1 共享基础架构中的测试隔离(Test Segregation in Shared Infrastructures)

10.1.1<br>
如果应用程序托管在共享基础结构中，请检查为共享环境的客户提供的访问机制，以更新和管理其内容和功能。请考虑以下问题:<br>
- 远程访问设施是否使用安全协议和经过适当加固的基础架构？
- 客户可以访问他们合法不需要访问的文件，数据和其他资源吗？
- 客户能否在托管环境中获得交互式shell并执行任意命令？

10.1.2<br>
如果使用专有应用程序来允许客户配置和自定义共享环境，请考虑以该应用程序为目标，以此来破坏环境本身和其中运行的单个应用程序。

10.1.3<br>
如果您可以在一个应用程序中实现命令执行，SQL注入或任意文件访问，请仔细研究这是否提供了将攻击升级到其他应用程序的任何方法。

# 10.2 测试ASP托管的应用程序之间的隔离(Test Segregation Between ASP-Hosted Applications)

10.2.1<br>
如果应用程序属于由共享和自定义组件的组合组成的ASP托管服务，请标识任何共享组件，例如日志记录机制，管理功能和数据库代码组件。 尝试利用它们来破坏应用程序的共享部分，从而攻击其他单个应用程序。

10.2.2<br>
如果在任何类型的共享环境中使用通用数据库，请使用数据库扫描工具（如NGSSquirrel）对数据库配置，补丁程序级别，表结构和权限进行全面审核。 数据库安全模型内的任何缺陷都可以提供一种将攻击从一个应用程序升级到另一个应用程序的方法。<br>


[Web应用程序黑客的方法论(9)](https://dm116.github.io/2020/02/17/web-application-hacker-methodology_9/)<br>
[Web应用程序黑客的方法论(11)](https://dm116.github.io/2020/02/17/web-application-hacker-methodology_11/)<br>


