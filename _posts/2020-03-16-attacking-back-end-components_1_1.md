---
layout:     post
title:      Chapter 10 Attacking Back-End Components(1)-Injecting OS Commands(1)
subtitle:   注入操作系统命令(1)
date:       2020-03-16
author:     D
header-img: 
catalog: true
mermaid: true
tags: [web hacking]
---

参考:**The Web Application Hackers Handbook** Chapter 10

大多数Web服务器平台已经发展到可以使用内置API来执行与服务器操作系统的任何实际交互的程度。 正确使用这些API，可使开发人员访问文件系统，与其他进程进行接口并以安全的方式进行网络通信。 但是，在许多情况下，开发人员选择使用更重的技术来直接向服务器发布操作系统命令。 由于它的功能强大且简单易用，因此它很有吸引力，通常可以为特定问题提供直接且实用的解决方案。 但是，如果应用程序将用户提供的输入传递给操作系统命令，则可能容易受到命令注入的攻击，从而使攻击者能够提交修改了开发人员打算执行的命令的精心设计的输入。

通常用于发出操作系统命令的功能（例如PHP中的`exec`和ASP中的`wscript.shell`）对可能执行的命令范围没有任何限制。 即使开发人员打算使用API来执行诸如列出目录内容之类的相对温和的任务，攻击者也可能会破坏它以编写任意文件或启动其他程序。 任何注入的命令通常都在Web服务器进程的安全上下文中运行，这对于攻击者破坏整个服务器通常是足够强大的。

在许多现成的和定制的Web应用程序中已经出现了这种命令注入漏洞。 它们在为企业服务器或防火墙，打印机和路由器等设备提供管理接口的应用程序中尤为普遍。 这些应用程序通常对操作系统交互有特殊要求，这导致开发人员使用包含用户提供的数据的直接命令。

# 1.示例1：通过Perl注入(Example 1: Injecting Via Perl)

考虑以下Perl CGI代码，它是用于服务器管理的Web应用程序的一部分。 此功能使管理员可以在服务器上指定目录并查看其磁盘使用情况的摘要：
```
#!/usr/bin/perl
use strict;
use CGI qw(:standard escapeHTML);
print header, start_html("");
print "<pre>";

my $command = "du -h --exclude php* /var/www/html";
$command= $command.param("dir");
$command=`$command`;
print "$command\n";

print end_html;
```
如预期使用时，此脚本仅将用户提供的用于参数的值附加到预设命令的末尾，执行该命令并显示结果，如图10-1所示。

![figure10-1](/img/web_hacking/twahh/figure10-1.jpg)

通过提供包含外壳元字符的精心设计的输入，可以以多种方式利用此功能。 这些字符对处理命令的解释器有特殊含义，可用于干扰开发人员要执行的命令。 例如，管道字符（|）用于将一个过程的输出重定向到另一个过程的输入，从而使多个命令可以链接在一起。 攻击者可以利用此行为来注入第二条命令并检索其输出，如图10-2所示。

在这里，原始du命令的输出已被重定向为`cat /etc/passwd`命令的输入。 该命令仅忽略输入，并执行其唯一的任务，即输出`passwd`文件的内容。

这样简单的攻击似乎是不可能的。 但是，正是这种类型的命令注入已在许多商业产品中找到。 例如，发现HP OpenView容易受到以下URL中命令注入漏洞的攻击：
```
https://target:3443/OvCgi/connectedNodes.ovpl?node=a| [your command] |
```

![figure10-2](/img/web_hacking/twahh/figure10-2.jpg)

# 2.示例2：通过ASP注入(Example 2: Injecting Via ASP)

考虑以下C＃代码，该代码是用于管理Web服务器的Web应用程序的一部分。 该功能允许管理员查看所请求目录的内容：
```
string dirName = "C:\\filestore\\" + Directory.Text;
ProcessStartInfo psInfo = new ProcessStartInfo("cmd", "/c dir " +
dirName);
...
Process proc = Process.Start(psInfo);
```
按预期使用时，此脚本会将用户提供的`Directory`参数的值插入到预设命令中，执行该命令并显示结果，如图10-3所示。

与易受攻击的Perl脚本一样，攻击者可以使用Shell元字符来干扰开发人员想要的预设命令，并注入自己的命令。 字符（`＆`）用于批处理多个命令。 提供包含`＆`字符和第二个命令的文件名，将导致该命令被执行并显示其结果，如图10-4所示。

![figure10-3](/img/web_hacking/twahh/figure10-3.jpg)

![figure10-4](/img/web_hacking/twahh/figure10-4.jpg)

# 3.通过动态执行注入(Injecting Through Dynamic Execution)

许多Web脚本语言都支持动态执行在运行时生成的代码。 此功能使开发人员可以创建应用程序，以响应各种数据和条件而动态修改其自己的代码。 如果将用户输入合并到动态执行的代码中，则攻击者可能能够提供经过精心设计的输入，这些输入突破了预期的数据上下文，并指定了在服务器上执行的命令，就像它们是由服务器编写的一样。 原始开发人员。 此时，攻击者的首要目标通常是注入运行OS命令的API。

PHP函数`eval`用于动态执行在运行时传递给该函数的代码。 考虑一个搜索功能，该功能使用户能够创建存储的搜索，然后将其动态生成为用户界面中的链接。 用户访问搜索功能时，将使用如下网址：
```
/search.php?storedsearch=\$mysearch%3dwahh
```
服务器端应用程序通过动态生成包含存储的搜索参数中指定的名称/值对的变量来实现此功能，在这种情况下，将创建一个值为`wahh`的`mysearch`变量
```
$storedsearch = $_GET['storedsearch'];
eval("$storedsearch;");
```
在这种情况下，您可以提交由`eval`函数动态执行的精心设计的输入，从而导致将任意PHP命令注入服务器端应用程序。 分号字符可用于在单个参数中批处理命令。 例如，要检索文件`/etc/password`的内容，可以使用`file_get_contents`或`system`命令：
```
/search.php?storedsearch=\$mysearch%3dwahh;%20echo%20file_get
_contents('/etc/passwd')
/search.php?storedsearch=\$mysearch%3dwahh;%20system('cat%20/etc/
passwd')
```

**NOTE**
Perl语言还包含一个`eval`函数，可以以相同的方式加以利用。 请注意，分号字符可能需要进行URL编码（如`%3b`），因为某些CGI脚本解析器将其解释为参数定界符。 在经典ASP中，`Execute()`扮演类似的角色。

[Chapter 9 Attacking Data Stores(5)-Injecting into LDAP](https://dm116.github.io/2020/03/16/attacking-data-stores_5/)<br>
[Chapter 10 Attacking Back-End Components(1)-Injecting OS Commands(2)](https://dm116.github.io/2020/03/16/attacking-back-end-components_1_2/)<br>
