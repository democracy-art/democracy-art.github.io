---
layout:     post
title:      vulnerabilities Broken Access Control
subtitle:   IDOR 介绍
date:       2020-01-09
author:     D
header-img: 
catalog: true
tags:
	- vulnerabilities
	- Broken Access Control
	- IDOR 
---

OWASP已经把IDOR归属为Broken Access control.

### 什么是IDOR？
IDOR 全称是Insecure Direct Object Reference(不安全的直接对象引用).

### Simple numeric IDOR
假设你登陆的帐号链接如下：
```javascript
https://www.abc.com/orders/id?=43976
```
你将它改为下面之后,可以查看甚至是修改
id=43975的人的信息;但是这个帐号并不属于你。
```javascript
https://www.abc.com/orders/id?=43975
```

### IDOR in POST
下面的POST请求你可能很容易猜到修改什么地方可能个产生IDOR
```javascript
POST /account/deleteaccnt HTTP/1.1
Host: acme.com
Connection: close
Content-Length: 22
Cache-Control: max-age=0
Origin: https://acme.com
Upgrade-Insecure-Requests: 1
Content-Type: application/x-www-form-urlencoded
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36
(KHTML, like Gecko) Chrome/67.0.3396.99 Safari/537.36
Accept:
text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,image/apng
,*/*;q=0.8
Accept-Encoding: gzip, deflate
Accept-Language: en-US,en;q=0.9
Cookie: JSESSIONID=3214536754363414df3142gf2341

acID=4321&action=Delete
```
没错它就是 `acID`.

### GUID based IDOR
你注册且登录 account 1 会有如下链接
```javascript
https://www.acme.com/changepw/id?=13d573
e8-5210-408a-aa77-6e2e9993d264
```
你注册且登录 account 2 会有如下链接
```javascript
https://www.acme.com/changepw/id?=cec4d0
ff-f133-4ffd-9ed9-3e0d0c5a3990
```
这种情况如果你想要形成`IDOR`要先找到另外一个用户的 `GUID`.
要枚举`guid`或**不可枚举的帐户ID**，请查找可能**返回此数据**的其他`端点`或`web服务`。
在您的**代理历史**中快速搜索您的ID应该是您首先检查的请求，并**尝试篡改**以获得其他ID(**有时这本身就是一个漏洞**)。


### GUID based IDOR (cont.)
**GUID 是微软对UUID这个标准的实现。UUID是由开放软件基金会（OSF）定义的。UUID还有其它各种实现，不止GUID一种.**

```javascript
GET /api/data/admin@acme.com HTTP/1.1
Host: acme.com
Connection: close
Content-Length: 22
Cache-Control: max-age=0
Origin: https://acme.com
Upgrade-Insecure-Requests: 1
Content-Type: application/json
Accept-Encoding: gzip, deflate
Accept-Language: en-US,en;q=0.9
```

```javascript
HTTP/1.1 200 OK
Accept-Ranges: bytes
Vary: Accept-Encoding
Content-Type: text/json; charset=UTF-8
<... SNIPPED ...>

{"accountdata":{"account":"admin@acme.com"},{"uuid":"cec4d0ff-f
133-4ffd-9ed9-3e0d0c5a3990"},{"name":"admin"},{"role":"admin"}}
```
很多时候，存在将用户`电子邮件`转换为`UUID的端点`，这些函数有时可以用于获取其他用户的GUID。
搜索引擎抓取和查找任何相关`移动应用程序`的功能也是如此。`移动API`通常返回详细的数据级别。
它还与真正验证UUID或ID是随机的有关。有时看起来复杂的ID只有`部分随机`，这使得它们很`容易迭代`。

### Hash based IDOR
IDOR函数值可以有多种形式。基于`字符串`、`散列`、`编码`等。本例是MD5散列。

```javascript
POST /account/updatepasswd HTTP/1.1
Host: acme.com
Connection: close
Content-Length: 22
Cache-Control: max-age=0
Origin: https://acme.com
Upgrade-Insecure-Requests: 1
Content-Type: application/x-www-form-urlencoded
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36
(KHTML, like Gecko) Chrome/67.0.3396.99 Safari/537.36
Accept:
text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,image/apng
,*/*;q=0.8
Accept-Encoding: gzip, deflate
Accept-Language: en-US,en;q=0.9
Cookie: JSESSIONID=3214536754363414df3142gf2341

userid=912134131a7b11f2dfee0b92bf6b0eed&action=updatepasswd
```

