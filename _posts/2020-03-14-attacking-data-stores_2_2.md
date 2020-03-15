---
layout:     post
title:      Chapter 9 Attacking Data Stores(2)-Injecting into SQL(2)
subtitle:   SQL注入(2)
date:       2020-03-14
author:     D
header-img: 
catalog: true
mermaid: true
tags: [web hacking]
---

参考:**The Web Application Hackers Handbook** Chapter 9

# 1.指纹识别数据库(Fingerprinting the Database)(这里是唯一确定的意思，不是真的指纹识别)

到目前为止，所描述的大多数技术对所有常见的数据库平台都有效，并且通过对语法的较小调整可以解决任何差异。 但是，随着我们开始研究更高级的利用技术，平台之间的差异变得越来越重要，您将越来越需要知道您要处理哪种类型的后端数据库。

您已经了解了如何提取主要数据库类型的版本字符串。 即使由于某种原因无法完成此操作，通常也可以使用其他方法对数据库进行指纹打印。 最可靠的方法之一是数据库连接字符串的不同方法。 在控制字符串数据项的查询中，可以在一个请求中提供特定值，然后测试不同的串联方法来生成该字符串。 当获得相同的结果时，您可能已经确定了
正在使用的数据库。 以下示例说明如何在常见的数据库类型上构造字符串服务：
- **Oracle:** 'serv'||'ices'
- **MS-SQL:** 'serv'+'ices'
- **MySQL:** 'serv' 'ices'(注意空格)

如果要注入数字数据，则可以使用以下攻击字符串对数据库进行指纹识别。 这些项目中的每一项在目标数据库上的计算结果均为0，并在其他数据库上生成错误：
- **Oracle:** BITAND(1,1)-BITAND(1,1)
- **MS-SQL:** @@PACK_RECEIVED-@@PACK_RECEIVED
- **MySQL:** CONNECTION_ID()-CONNECTION_ID()

**NOTE**
MS-SQL和Sybase数据库具有共同的起源，因此它们在表结构，全局变量和存储过程方面具有许多相似之处。 在实践中，后面各节中描述的大多数针对MS-SQL的攻击技术都将以相同的方式针对Sybase。

指纹打印数据库时的另一个兴趣点是MySQL如何处理某些类型的内联注释。 如果注释以感叹号开头，后跟数据库版本字符串，则注释的内容将解释为实际SQL，前提是实际数据库的版本等于或晚于该字符串。 否则，内容将被忽略并视为注释。 程序员可以像使用C中的预处理器指令一样使用此功能，从而使他们能够编写不同的代码，这些代码将在使用的数据库版本中有条件地进行处理。 攻击者还可以使用此功能来指纹打印数据库的确切版本。 例如，如果使用的MySQL版本大于或等于`3.23.02`，则注入以下字符串将导致SELECT语句的WHERE子句为false：
```
/*!32302 and 1=0*/
```

# 2.UNION运算符(The UNION Operator)

SQL中使用UNION运算符将两个或多个SELECT语句的结果合并为一个结果集。 当Web应用程序包含SELECT语句中发生的SQL注入漏洞时，您通常可以使用UNION运算符执行第二个完全独立的查询，并将其结果与第一个结果结合在一起。 如果查询结果返回到浏览器，则可以使用此技术轻松地从数据库中提取任意数据。 所有主要的DBMS产品都支持UNION。 在直接返回查询结果的情况下，这是从数据库检索任意信息的最快方法。

回顾使用户能够根据作者，书名，出版者和其他条件搜索书籍的应用程序。 搜索由Wiley出版的书籍会使应用程序执行以下查询：
```
SELECT author,title,year FROM books WHERE publisher = 'Wiley'
```
假设此查询返回以下结果集：

|AUTHOR|TITLE|YEAR|
|-|-|-|
|Litchfield|The Database Hacker’s Handbook|2005|
|Anley|The Shellcoder’s Handbook|2007|

前面已经看到了攻击者如何向搜索功能提供精心设计的输入，以颠覆查询的`WHERE`子句，从而返回数据库中所有的书。 更为有趣的攻击方法是使用`UNION`运算符注入第二个`SELECT`查询，并将其结果附加到第一个查询中。 第二个查询可以从另一个数据库表中提取数据。 例如，输入搜索词：
```
Wiley' UNION SELECT username,password,uid FROM users--
```
使应用程序执行以下查询：
```
SELECT author,title,year FROM books WHERE publisher = 'Wiley'
UNION SELECT username,password,uid FROM users--'
```
这将返回原始搜索的结果，然后返回users表的内容：

|AUTHOR|TITLE|YEAR|
|-|-|-|
|Litchfield|The Database Hacker’s Handbook|2005|
|Anley|The Shellcoder’s Handbook|2007|
|admin|r00tr0x|0|
|cliff|Reboot|1|

**NOTE**
当使用UNION运算符组合两个或多个SELECT查询的结果时，组合结果集的列名与第一个SELECT查询返回的列名相同。 如上表所示，用户名显示在`author`列中，密码显示在`title`列中。 这意味着，当应用程序处理修改后的查询的结果时，它无法检测到返回的数据是否源自其他表。

这个简单的示例演示了在SQL注入攻击中使用UNION运算符的潜在强大功能。 但是，在以这种方式利用它之前，需要考虑两个重要的条件：
- 使用UNION运算符合并两个查询的结果时，两个结果集必须具有相同的结构。 换句话说，它们必须包含相同数量的列，它们具有相同或兼容的数据类型，并且以相同的顺序出现。
- 要注入第二个查询以返回有趣的结果，攻击者需要知道他要定位的数据库表的名称以及相关列的名称。

让我们更深入地了解这些前提条件。 假设攻击者尝试注入第二个查询，该查询返回错误的列数。 他提供了以下输入：
```
Wiley' UNION SELECT username,password FROM users--
```
原始查询返回三列，而注入的查询仅返回两列。 因此，数据库返回以下错误：
```
ORA-01789: query block has incorrect number of result columns
```
假设攻击者尝试注入第二个查询，这些查询的列具有不兼容的数据类型。 他提供了以下输入：
```
Wiley' UNION SELECT uid,username,password FROM users--
```
这将导致数据库尝试将第二个查询（包含字符串数据）中的password列与第一个查询（包含数字数据）中的year列组合在一起。 由于无法将字符串数据转换为数字数据，因此会导致错误：
```
ORA-01790: expression must have same datatype as corresponding expression
```

**NOTE**
此处显示的错误消息是针对Oracle的。 其他数据库的等效消息在后面的“SQL Syntax and Error Reference.”中列出。

在许多实际情况下，显示的数据库错误消息被应用程序捕获，并且不会返回给用户的浏览器。 因此，可能会发现在尝试发现第一个查询的结构时，您只能进行纯粹的猜测。 然而，这种情况并非如此。 三个要点意味着您的任务通常很容易：
- 为了使注入的查询能够与第一个查询合并，严格地讲，不必包含相同的数据类型。 相反，它们必须兼容。 换句话说，第二个查询中的每个数据类型必须与第一个查询中的相应类型相同或可以隐式转换为它。 您已经看到数据库隐式将数字值转换为字符串值。 实际上，值`NULL`可以转换为任何数据类型。 因此，如果您不知道特定字段的数据类型，则可以简单地为该字段`SELECT NULL`。
- 如果应用程序捕获数据库错误消息，则可以轻松确定是否执行了注入的查询。 如果是这样，则将其他结果添加到应用程序从其原始查询返回的结果中。 这使您能够系统地工作，直到发现需要注入的查询的结构。
- 在大多数情况下，只需在原始查询中标识一个具有字符串数据类型的字段即可实现目标。 这足以让您注入任意查询，这些查询返回基于字符串的数据并检索结果，从而使您能够系统地从数据库中提取任何所需的数据。

**HACK STEPS**<br>
您的第一个任务是发现应用程序正在执行的原始查询返回的列数。 您可以通过两种方式执行此操作：
- 1.您可以利用以下事实：可以将NULL转换为任何数据类型，以系统地注入具有不同列数的查询，直到执行注入的查询为止。 例如：
```
' UNION SELECT NULL--
' UNION SELECT NULL, NULL--
' UNION SELECT NULL, NULL, NULL--
```
执行查询时，您已确定所需的列数。 如果应用程序未返回数据库错误消息，您仍然可以判断注入的查询何时成功。 将返回另一行数据，其中包含单词NULL或空字符串。 请注意，注入的行可能仅包含空表单元格，因此当呈现为HTML时可能很难看到。 因此，最好在执行此攻击时查看原始响应。
- 2.确定了所需的列数之后，下一个任务是发现具有字符串数据类型的列，以便您可以使用它从数据库中提取任意数据。 您可以像以前一样通过注入包含`NULLs`的查询来完成此操作，然后系统地将每个`NULL`替换为`a`。 例如，如果您知道查询必须返回三列，则可以注入以下内容：
```
' UNION SELECT 'a', NULL, NULL--
' UNION SELECT NULL, 'a', NULL--
' UNION SELECT NULL, NULL, 'a'--
```
执行查询时，您会看到包含值`a`的另一行数据。 然后，您可以使用相关列从数据库中提取数据。

**NOTE**
在Oracle数据库中，每个`SELECT`语句都必须包含`FROM`属性，因此，无论列数如何，注入`UNION SELECT NULL`都会产生错误。 您可以通过从全局可访问的表`DUAL`中进行选择来满足此要求。 例如：
```
' UNION SELECT NULL FROM DUAL--
```
确定注入查询中所需的列数并找到具有字符串数据类型的列后，就可以提取任意数据了。 一个简单的概念验证测试是提取数据库的版本字符串，这可以在任何DBMS上完成。 例如，如果有三列，并且第一列可以获取字符串数据，则可以通过在MS-SQL和MySQL上注入以下查询来提取数据库版本：
```
' UNION SELECT @@version,NULL,NULL--
```
注入以下查询可以在Oracle上获得相同的结果：
```
' UNION SELECT banner,NULL,NULL FROM v$version--
```
在易受攻击的书籍搜索应用程序的示例中，我们可以使用以下字符串作为搜索项来检索Oracle数据库的版本：

|AUTHOR TITLE YEAR|
|-|
|CORE 9.2.0.1.0 Production|
|NLSRTL Version 9.2.0.1.0 - Production|
|Oracle9i Enterprise Edition Release 9.2.0.1.0 - Production|
|PL/SQL Release 9.2.0.1.0 - Production|
|TNS for 32-bit Windows: Version 9.2.0.1.0 - Production|

当然，即使数据库的版本字符串可能很有趣，并且可能使您能够使用所使用的特定软件来研究漏洞，但在大多数情况下，您会对从数据库中提取实际数据更感兴趣。 为此，您通常需要解决前面描述的第二条附加条件。 也就是说，您需要知道要定位的数据库表的名称及其相关列的名称。

# 3.提取有用的数据(Extracting Useful Data)

要从数据库中提取有用的数据，通常您需要知道包含要访问的数据的表和列的名称。 主要企业DBMS包含大量数据库元数据，您可以查询这些数据库元数据以发现数据库中每个表和列的名称。 在每种情况下，提取有用数据的方法都是相同的。 但是，细节在不同的数据库平台上有所不同。

[Chapter 9 Attacking Data Stores(2)-Injecting into SQL(1)](https://dm116.github.io/2020/03/14/attacking-data-stores_2_1/)<br>
[Chapter 9 Attacking Data Stores(2)-Injecting into SQL(3)](https://dm116.github.io/2020/03/15/attacking-data-stores_2_3/)<br>
