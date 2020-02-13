---
layout:     post
title:      Web应用程序黑客的方法论(3)--Test Client-Side Controls
subtitle:   
date:       2020-02-09
author:     D
header-img: 
catalog: true
mermaid: true
tags:
    - web hacking
---

Web应用程序黑客的方法论来自书本 *The Web Application Hacker's Handbook*


![Testing client-side controls](/img/testing-client-side-controls.png)

# 测试客户端控件(Test Client-Side Controls)

## 3.1 测试通过客户端传输的数据(Test Transmission of Data Via the Client)

3.1.1<br>
定位应用程序中的所有实例，其中隐藏的表单字段、cookie和URL参数显然用于通过客户机传输数据。

3.1.2<br>
根据项目出现的上下文及其名称和值,尝试确定项目在应用程序逻辑中的用途.

3.1.3<br>
以与应用程序功能中的角色相关的方式修改项的值。确定应用程序是否处理字段中提交的任意值，以及是否可以利用这一事实干扰应用程序的逻辑或破坏任何安全控制。

3.1.4<br>
如果应用程序通过客户端传输不透明的数据，则可以通过各种方式进行攻击.如果项目是模糊的，您可能能够破译模糊算法，从而提交不透明项目中的任意数据.即使它是安全加密的，您也可以在其他上下文中重播该项目，以干扰应用程序的逻辑.有关这些攻击和其他攻击的详细信息，请参阅Chapter 5.

3.1.5<br>
如果应用程序使用`ASP.NET` `ViewState`，测试以确定是否可以篡改它或它是否包含任何敏感信息.注意`ViewStatep能在不同的应用程序页面上使用不同的方式.<br>

- 3.1.5.1 使用Burp套件中的ViewState分析器来确认是否启用了EnableViewStateMac选项，这意味着ViewState的内容不能修改.
- 3.1.5.2 检查已解码的 ViewState 以识别其中包含的任何敏感数据.
- 3.1.5.3 修改已解码的参数值之一reencode并提交ViewState.如果应用程序接受修改后的值，您应该将ViewState作为输入通道，将任意数据引入应用程序处理.对它包含的数据执行与对任何其他请求参数执行相同的测试.

## 3.2 测试用户输入的客户端控件(Test Client-Side Controls Over User Input)

3.2.1<br>
在将用户输入提交到服务器之前，确定客户端控件(如长度限制和JavaScript检查)用于验证用户输入的情况.可以很容易地绕过这些控件，因为可以向服务器发送任意请求.比如:
```
<form action=”order.asp” onsubmit=”return Validate(this)”>
<input maxlength=”3” name=”quantity”>
...
```
3.2.2<br>
通过提交通常会被客户端控件阻止的输入来依次测试每个受影响的输入字段，以验证这些输入是否复制到服务器上.

3.2.3<br>
能绕过客户端验证并不一定代表任何漏洞.然而,您应该仔细检查正在执行的验证.确认应用程序是否依赖于客户端控件来保护自己免受格式错误的输入.还确认是否存在任何可利用的条件，可以由这样的输入触发.

3.2.4<br>
检查每个HTML表单，以识别任何禁用的元素，比如灰色的提交按钮.比如:<br>
```
<input disabled="true" name="product">
```
如果您找到了，请将它们连同表单的其他参数一起提交给服务器.查看该参数是否对服务器的处理有任何影响,您可以在攻击中利用这些影响.或者，使用自动代理规则自动启用禁用字段，如Burp代理的`HTML Modifi cation`规则.

## 3.3 测试浏览器扩展组件(Test Browser Extension Components)

### 3.3.1 理解客户端应用程序的操作(Understand the Client Application's Operation)

3.3.1.1<br>
为审查中的客户端技术设置一个本地拦截代理，并监视客户端和服务器之间的所有流量.如果数据是序列化的，请使用`反序列化`工具，如Burp的内置`AMF`支持或Java的`DSer Burp`插件.

3.3.1.2<br>
单步调试客户端中提供的功能.确定任何潜在的敏感或强大的功能,使用拦截代理中的标准工具来replay密钥请求或修改服务器响应.

### 3.3.2 对客户端进行反编译(Decompile the Client)

3.3.2.1<br>
识别应用程序使用的任何小程序.查找通过拦截代理请求的下列任何文件类型.
- `.class`, `.jar`: `Java`
- `.swf`:`Flash`
- `.xap`:`Silverlight`

您还可以在应用程序页面的HTML源代码中查找applet标记.比如:<br>
```
<applet code="input.class" id="TheApplet" codebase="/scripts/"></
applet>
```

3.3.2.2<br>
检查调用HTML中对applet方法的所有调用,并确定从applet返回的数据是否提交到服务器.如果该数据是不透明的(也就是说,是模糊的或加密的),为了修改它,您可能需要对applet进行反编译以获得其源代码.

3.3.2.3<br>
通过在浏览器中输入URL下载applet字节码，并在本地保存文件.字节码文件的名称在applet标记的代码属性中指定.如果存在,文件将位于codebase属性中指定的目录中.否则,它将位于与applet标记所在的页面相同的目录中.

3.3.2.4<br>
使用合适的工具将字节码反编译为源代码.比如:<br>

```
C:\>jad.exe input.class
Parsing input.class... Generating input.jad
```

下面是一些适合反编译不同浏览器扩展组件的工具:<br>

- Java -- Jad
- Flash -- SWFScan, Flasm/Flare
- Silverlight -- .NET Reflector
<br>
如果applet被打包到JAR、XAP或SWF文件中，则可以使用标准的存档阅读器(如WinRar或WinZip)将其解压.<br>

3.3.2.5<br>
检查相关的源代码(从返回不透明数据的方法的实现开始),以了解正在执行什么处理.

3.3.2.6<br>
确定applet是否包含可用于对任意输入执行相关模糊处理的任何公共方法.applet是否包含可用于对任意输入执行相关模糊处理的任何公共方法.

3.3.2.7<br>
如果没有,修改applet的源代码,使它执行的任何验证无效,或者允许您混淆任意输入.然后可以使用供应商提供的编译工具将源代码重新编译为其原始文件格式.

### 3.3.3 附加一个调试器(Attach a Debugger)

3.3.3.1<br>
对于大型客户端应用程序，通常很难在不遇到大量错误的情况下对整个应用程序进行反编译、修改和重新打包.对于这些应用程序，通常将运行时调试器附加到进程上更快.JavaSnoop在Java中做得很好.Silverlight Spy是一个免费的工具，允许在运行时监视Silverlight客户端.

3.3.3.2<br>
定位应用程序用于驱动安全相关业务逻辑的关键函数和值，并在调用目标函数时放置断点.根据需要修改参数或返回值以实现安全绕过.

### 3.3.4 测试ActiveX控件(Test ActiveX controls)

3.3.4.1<br>
识别应用程序使用的任何ActiveX控件.查找通过拦截代理请求的任何`.cab`文件类型,或者在应用程序页面的HTML源代码中查找对象标记.比如:<br>

```
<OBJECT
    classid="CLSID:4F878398-E58A-11D3-BEE9-00C04FA0D6BA"
    codebase="https://wahh app.com/scripts/input.cab"
    id="TheAxControl">
</OBJECT>
```

3.3.4.2<br>
通过将调试器附加到进程并直接修改正在处理的数据或更改程序的执行路径，通常可以破坏在ActiveX控件中执行的任何输入验证.有关这种攻击的详细信息，请参阅Chapter 5.

3.3.4.3<br>
通常可以根据ActiveX控件导出的不同方法的名称和传递给它们的参数来猜测它们的用途.用`COMRaider`工具列举出该控件输出的方法.测试是否可以操纵其中任何一个来影响控件的行为并击败它实现的任何验证测试.

3.3.4.4<br>
如果控件的目的是收集或验证关于客户端计算机的某些信息，则使用`Filemon`和`Regmon`工具监视控件收集的信息.通常可以在系统注册表和文件系统中创建适当的项来修复控件使用的输入,从而影响其行为.

3.3.4.5<br>
测试任何ActiveX控件的漏洞,这些漏洞可能被用来攻击应用程序的其他用户.您可以修改用于调用控件的HTML，以便将任意数据传递给它的方法并监视结果.寻找名称听起来很危险的方法，比如`LaunchExe`您还可以用COMRaider对ActiveX控件执行一些基本的模糊测试，以识别缓冲区溢出等缺陷.


[Web应用程序黑客的方法论(2)](https://dm116.github.io/2020/02/09/web-application-hacker-methodology_2/)<br>
[Web应用程序黑客的方法论(4)](https://dm116.github.io/2020/02/11/web-application-hacker-methodology_4/)
