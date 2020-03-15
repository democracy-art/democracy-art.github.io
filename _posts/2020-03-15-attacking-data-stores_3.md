---
layout:     post
title:      Chapter 9 Attacking Data Stores(3)-Injecting into NoSQL
subtitle:   NoSQL注入
date:       2020-03-15
author:     D
header-img: 
catalog: true
mermaid: true
tags: [web hacking]
---

参考:**The Web Application Hackers Handbook** Chapter 9

# Injecting into NoSQL

NoSQL术语是指不同于标准关系数据库体系结构的各种数据存储。 NoSQL数据存储区使用键/值映射表示数据，并且不依赖诸如常规数据库表之类的固定模式。 键和值可以任意定义，并且值的格式通常与数据存储无关。 键/值存储的另一个特征是，值本身可以是数据结构，从而允许分层存储，这与数据库架构内部的数据结构不同。

NoSQL的倡导者声称这具有几个优点，主要是在处理非常大的数据集时，可以根据需要精确地优化数据存储的层次结构，以减少检索数据集的开销。 在这些情况下，常规数据库可能需要对表进行复杂的交叉引用才能代表应用程序检索信息。

从Web应用程序安全性的角度来看，关键考虑因素是应用程序如何查询数据，因为这确定了可能的注入形式。 在使用SQL注入的情况下，不同数据库产品之间的SQL语言大致相似。 相比之下，NoSQL是为不同种类的数据存储提供的名称，它们都有各自的行为。 它们并非全部使用一种查询语言。

以下是NoSQL数据存储使用的一些常见查询方法：
- 键/值查找
- XPath
- 编程语言，例如JavaScript

NoSQL是一项相对较新的技术，已迅速发展。 尚未像更成熟的技术（例如SQL）那样大规模部署它。 因此，对与NoSQL相关的漏洞的研究仍处于起步阶段。 此外，由于许多NoSQL实现都允许使用固有的简单方法来访问数据，因此有时会讨论有关注入NoSQL数据存储的示例。

几乎可以肯定的是，在当今和未来的Web应用程序中使用NoSQL数据存储的方式都将产生可利用的漏洞。 下一部分将描述一个这样的示例，该示例是从真实世界的应用程序派生的。

# Injecting into MongoDB

许多NoSQL数据库都利用现有的编程语言来提供灵活的可编程查询机制。 如果查询是使用字符串连接构建的，则攻击者可能会尝试破坏数据上下文并更改查询的语法。 考虑以下示例，该示例基于MongoDB数据存储中的用户记录执行登录：
```
$m = new Mongo();
$db = $m->cmsdb;
$collection = $db->user;
$js = "function() {
return this.username == '$username' & this.password == '$password'; }";
$obj = $collection->findOne(array('$where' => $js));
if (isset($obj["uid"]))
{
	$logged_in=1;
}
else
{
	$logged_in=0;
}
```
`$js`是一个JavaScript函数，其代码是动态构造的，并包括用户提供的用户名和密码。 攻击者可以通过提供用户名来绕过身份验证逻辑：
```
Marcus'//
```
以及任何密码。 产生的JavaScript函数如下所示：
```
function() { return this.username == 'Marcus'//' & this.password == 'aaa'; }
```
**NOTE**<br>
在JavaScript中，双斜杠（//）表示一行剩余的注释，因此该函数中的其余代码被注释掉了。
确保`$js`函数始终返回true而不使用注释的另一种方法是提供以下用户名：
```
a' || 1==1 || 'a'=='a
```
JavaScript这样解释各种运算符：
```
(this.username == 'a' || 1==1) || ('a'=='a' & this.password =='aaa');
```
由于第一析取条件始终为true（1始终等于1），因此这导致用户集合中的所有资源都匹配。


[Chapter 9 Attacking Data Stores(2)-Injecting into SQL(5)](https://dm116.github.io/2020/03/15/attacking-data-stores_2_5/)<br>
[Chapter 9 Attacking Data Stores(3)-Injecting into XPath(1)](https://dm116.github.io/2020/03/16/attacking-data-stores_4_1/)<br>
