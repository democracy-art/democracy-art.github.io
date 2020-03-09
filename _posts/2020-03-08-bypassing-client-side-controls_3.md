---
layout:     post
title:      Bypassing Client-Side Controls(3) - Capturing User Data: Browser Extensions
subtitle:   
date:       2020-03-08
author:     D
header-img: 
catalog: true
mermaid: true
tags:
    - [web hacking]
---

参考:*The Web Application Hacker's Handbook* 

# 1.捕获用户数据：浏览器扩展(Capturing User Data: Browser Extensions)

除HTML表单外，捕获，验证和提交用户数据的另一种主要方法是使用在浏览器扩展（例如Java或Flash）中运行的客户端组件。在Web应用程序中首次使用时，浏览器扩展通常用于执行简单且通常是修饰性的任务。现在，公司越来越多地使用浏览器扩展来创建功能齐全的客户端组件。它们在浏览器中跨多个客户端平台运行，并提供反馈，灵活性和对桌面应用程序的处理。副作用是，出于速度和用户体验的原因，以前可能在服务器上进行过的处理任务可能会被分配给客户端。在某些情况下，例如在线交易应用程序，速度至关重要，以至于许多关键应用程序逻辑发生在客户端。应用程序设计可能会故意牺牲安全性以提高速度，这可能是因为错误地认为交易者是受信任的用户，或者浏览器扩展包含了自己的防御措施。回顾第2章和本章前面各节中讨论的核心安全问题，我们知道保护客户端组件的业务逻辑的概念是不可能的。

浏览器扩展可以多种方式捕获数据-通过输入表单，在某些情况下还可以通过与客户端操作系统的文件系统或注册表进行交互。 他们可以在将捕获的数据提交到服务器之前执行任意复杂的验证和操作。 此外，由于其内部工作不如HTML表单和JavaScript透明，因此开发人员更有可能认为自己执行的验证无法被规避。 因此，浏览器扩展通常是发现Web应用程序中漏洞的有效目标。

在客户端应用控件的浏览器扩展的经典示例是娱乐场组件。 鉴于我们已经观察到客户端控件的易错本质，使用在潜在攻击者的计算机上本地运行的浏览器扩展来实现在线赌博应用程序的想法很有趣。 如果游戏的任何方面是在客户端而不是在服务器中控制的，则攻击者可以精确地操纵游戏以提高赔率，更改规则或更改提交给服务器的分数。 在这种情况下，可能会发生几种攻击：
- 1.可以信任客户端组件以维持游戏状态。 在这种情况下，本地篡改游戏状态将为攻击者提供游戏优势。
- 2.攻击者可能绕过客户端控制，并执行旨在使自己在游戏中占优势的非法行为。
- 3.攻击者可能会找到一个隐藏的函数，参数或资源，该函数，参数或资源在被调用时会允许对服务器端资源的非法访问。
- 4.如果游戏涉及任何同龄人或自家玩家，则客户端组件可能会接收和处理有关其他玩家的信息（如果已知），这些信息可能会被用于攻击者的利益。

## 1.1 通用浏览器扩展技术(Common Browser Extension Technologies)
您最可能会遇到的浏览器扩展技术是Java applet，Flash和Silverlight。 由于这些产品争相实现相似的目标，因此它们在体系结构中具有与安全相关的相似属性：
- 1.它们被编译为中间字节码.
- 2.它们在提供执行沙箱环境的虚拟机中执行。
- 3.他们可以使用采用序列化的远程处理框架通过HTTP传输复杂的数据结构或对象。

**Java**<br>
Java小程序在Java虚拟机（JVM）中运行，并且受Java安全策略应用的沙箱的约束。 由于Java自网络历史悠久以来就已经存在，并且由于其核心概念相对保持不变，因此可以使用大量知识和工具来攻击和防御Java小程序，如本章稍后所述。

**Flash**<br>
Flash对象在Flash虚拟机中运行，并且与Java小程序一样，是从主机沙箱中筛选出来的。 Flash曾经被广泛用作传递动画内容的一种方法，但现在已经发展了。 使用ActionScript的较新版本，现在可以肯定地认为Flash可以交付功能完善的桌面应用程序。 Flash中最近的一项重要更改是ActionScript 3及其具有Action Message Format（AMF）序列化的远程处理功能。

**Silverlight**<br>
Silverlight是Microsoft替代Flash的替代产品。 它的设计目标与实现类似桌面的丰富应用程序一样，允许Web应用程序在沙盒环境中在浏览器内提供缩小的.NET体验。 从技术上讲，Silverlight应用程序可以使用从C＃到Python的任何.NET兼容语言进行开发，尽管C＃是迄今为止最常见的语言。

## 1.2 浏览器扩展的方法(Approaches to Browser Extensions)

在定位使用浏览器扩展组件的应用程序时，需要采用两种广泛的技术。

首先，您可以拦截和修改组件发出的请求以及从服务器收到的响应。 在许多情况下，这是开始测试组件的最快，最简单的方法，但是您可能会遇到一些限制。 可以对发送的数据进行混淆或加密，或者可以使用特定于所使用技术的方案来对其进行序列化。 通过仅查看由组件生成的流量，您可能会忽略某些关键功能或业务逻辑，而这些功能或业务逻辑只有通过分析组件本身才能发现。 此外，您可能会遇到以正常方式使用拦截代理的障碍； 但是，通常可以通过一些仔细的配置来规避这些问题，如本章稍后所述。

其次，您可以直接将组件本身作为目标，并尝试反编译其字节码以查看原始源，或者使用调试器与组件动态交互。 这种方法的优势在于，如果彻底完成，您将确定组件支持或引用的所有功能。 它还允许您修改在请求中提交给服务器的关键数据，而不管传输过程中使用的任何混淆或加密机制如何。 这种方法的缺点是它可能很耗时，并且可能需要详细了解组件中使用的技术和编程语言。

在许多情况下，将这两种技术结合使用是合适的。 以下各节将详细介绍每一个。

## 1.3 拦截来自浏览器扩展的流量(Intercepting Traffic from Browser Extensions)

如果您的浏览器已经配置为使用拦截代理，并且应用程序使用浏览器扩展程序加载了客户端组件，则您可能会看到来自该组件的请求通过您的代理。 在某些情况下，您无需执行任何其他操作即可开始测试相关功能，因为您可以按照通常的方式拦截和修改组件的请求。

在绕过在浏览器扩展中实现的客户端输入验证的上下文中，如果组件将验证的数据透明地提交给服务器，则可以使用拦截代理以与针对HTML表单数据所述相同的方式来修改此数据。例如，支持身份验证机制的浏览器扩展可能会捕获用户凭据，对其进行一些验证，然后将这些值作为请求内的纯文本参数提交给服务器。 无需对组件本身进行任何分析或攻击，就可以轻松地绕过验证。

在其他情况下，您可能会遇到各种障碍，使测试变得困难，如以下各节所述。

### 1.3.1 处理序列化数据(Handling Serialized Data)

应用程序可以在HTTP请求中传输数据或对象之前先序列化它们。 尽管仅通过检查原始序列化数据就可以解密某些基于字符串的数据，但是通常您需要对序列化数据进行解压缩，才能完全理解它们。 而且，如果您想修改数据以干扰应用程序的处理，则首先需要解压缩序列化的内容，根据需要对其进行编辑，然后正确地对其进行重新序列化。 简单地编辑原始序列化数据几乎可以肯定会破坏格式，并在应用程序处理消息时引起解析错误。

每种浏览器扩展技术都有其自己的用于序列化HTTP消息中的数据的方案。 因此，通常，您可以根据正在使用的客户端组件的类型来推断序列化格式，但是在任何情况下，仔细检查相关的HTTP消息，该格式通常都很明显。

### 1.3.2 Java序列化(Java Serialization)

Java语言包含对对象序列化的本机支持，并且Java applet可以使用它在客户端和服务器应用程序组件之间发送序列化的数据结构。 通常可以识别包含序列化Java对象的消息，因为它们具有以下Content-Type标头:
```
Content-Type: application/x-java-serialized-object
```
使用代理拦截了原始序列化数据之后，您可以使用Java本身对它进行反序列化，以访问其包含的原始数据项。

`DSer`是Burp Suite的便捷插件，它提供了一个框架，用于查看和处理在Burp中被截获的序列化Java对象。 该工具将拦截对象中的原始数据转换为XML格式，以便于编辑。 修改相关数据后，DSer然后重新序列化对象并相应地更新HTTP请求。下载[DSer](http://blog.andlabs.org/2010/09/re-visiting-java-de-serialization-it.html)

### 1.3.3 Flash序列化(Flash Serialization)

Flash使用其自己的序列化格式，该格式可用于在服务器和客户端组件之间传输复杂的数据结构。 通常可以通过以下Content-Type标头来标识操作消息格式（AMF):
```
Content-Type: application/x-amf
```
Burp本机支持AMF格式。 当它识别出包含序列化AMF数据的HTTP请求或响应时，它将对内容进行打包，并以树形形式呈现以供查看和编辑，如图5-5所示。修改结构中的相关原始数据项后，Burp会重新序列化消息，然后可以将其转发到要处理的服务器或客户端。

![figure5-5](/img/web_hacking/twahh/figure5-5.jpg)


### 1.3.4 Silverlight序列化(Silverlight Serialization)

Silverlight应用程序可以利用.NET平台中内置的Windows Communication Foundation（WCF）远程处理框架。 使用WCF的Silverlight客户端组件通常采用Microsoft的.NET SOAP二进制格式（NBFS），可以通过以下Content-Type标头进行标识:
```
Content-Type: application/soap+msbin1
```
Burp Proxy可以使用一个插件，该插件会自动反序列化NBFS编码的数据，然后再将其显示在Burp的拦截窗口中。 查看或编辑解码后的数据之后，该插件会对数据进行重新编码，然后再转发给服务器或客户端进行处理。

Burp的WCF二进制SOAP插件是由Brian Holyfield制作的，可以在这里下载：[WCF](www.gdssecurity.com/l/b/2009/11/19/wcf-binary-soap-plug-in-for-burp/)

### 1.3.5 拦截来自浏览器扩展的流量的障碍(Obstacles to Intercepting Traffic from Browser Extensions)

如果已将浏览器设置为使用拦截代理，则可能会发现浏览器扩展组件发出的请求没有被代理拦截或失败。 此问题通常是由于组件处理HTTP代理或SSL（或两者）存在问题。 通常，可以通过对您的工具进行仔细配置来进行处理。

第一个问题是客户端组件可能不支持您在浏览器中指定的代理配置或计算机的设置。 这是因为组件可能会在浏览器本身或扩展框架提供的API之外发出自己的HTTP请求。 如果发生这种情况，您仍然可以拦截组件的请求。 您需要修改计算机的主机文件以实现拦截并配置代理，以支持隐形代理和自动重定向到正确的目标主机。 有关如何执行此操作的更多详细信息，请参见第20章。

第二个问题是客户端组件可能不接受拦截代理提供的SSL证书。 如果您的代理使用的是通用的自签名证书，并且您已将浏览器配置为接受该证书，则浏览器扩展组件可能会拒绝该证书。 这可能是因为浏览器扩展程序没有为临时受信任的证书获取浏览器的配置，或者可能是因为组件本身以编程方式要求不应接受不受信任的证书。 在任何一种情况下，您都可以通过配置代理使用主CA证书（用于为您访问的每个站点签署有效的每台主机证书）并在计算机的受信任证书存储中安装CA证书来避免此问题。 。 有关如何执行此操作的更多详细信息，请参见第20章。

在极少数情况下，您可能会发现客户端组件正在使用HTTP以外的协议进行通信，而HTTP根本无法使用拦截代理进行处理。 在这些情况下，您仍然可以通过使用网络嗅探器或功能挂钩工具来查看和修改受影响的交通。 一个示例是Echo Mirage，它可以注入到进程中并拦截对套接字API的调用，从而使您可以在通过网络发送数据之前查看和修改数据。 可以从以下URL下载Echo Mirage：
```
www.bindshell.net/tools/echomirage
```
**HACK STEPS**
- 1.确保您的代理正确拦截了来自浏览器扩展的所有流量。 如有必要，使用嗅探器识别未正确代理的任何流量。
- 2.如果客户端组件使用标准的序列化方案，请确保您具有解压缩和修改它所必需的工具。 如果组件使用专有的编码或加密机制，则需要反编译或调试组件以对其进行全面测试。
- 3.查看来自服务器的，触发关键客户端逻辑的响应。 通常，及时拦截和修改服务器响应可以使您“解锁”客户端GUI，从而很容易显示然后执行复杂或多阶段的特权操作。
- 4.如果应用程序执行了不应该信任客户端组件执行的任何关键逻辑或事件（例如，在赌博应用程序中抓牌或掷骰子），请查找关键逻辑的执行与与服务器的通信之间的任何关联。 如果客户端不与服务器通信以确定事件的结果，则该应用程序肯定很容易受到攻击。

## 1.4 反编译浏览器扩展(Decompiling Browser Extensions)

到目前为止，攻击浏览器扩展组件的最彻底的方法是对对象进行反编译，对源代码进行完整的检查，并在必要时修改代码以更改对象的行为，然后重新编译。 如前所述，浏览器扩展被编译为字节码。 字节码是高级平台独立的二进制表示形式，可以由相关解释器（例如Java虚拟机或Flash Player）执行，并且每种浏览器扩展技术都使用其自己的字节码格式。 结果，该应用程序可以在解释器本身可以运行的任何平台上运行。

字节码表示的高级性质意味着，从理论上讲，总有可能将字节码反编译为类似于原始源代码的内容。 但是，可以采用各种防御技术来导致反编译器失败，或者输出很难遵循和解释的反编译代码。

受这些混淆防御的约束，通常，反编译字节码是理解和攻击浏览器扩展组件的首选途径。 这使您可以查看业务逻辑，评估客户端应用程序的全部功能，并以有针对性的方式修改其行为。

### 1.4.1 Downloading the Bytecode

第一步是下载可执行字节码供您开始使用。 通常，字节码是从运行浏览器扩展的应用程序页面的HTML源代码中指定的URL加载到单个文件中的。 Java小程序通常使用`<applet>`标记加载，而其他组件通常使用`<object>`标记加载。 例如：
```
<applet code=”CheckQuantity.class” codebase=”/scripts”
id=”CheckQuantityApplet”>
</applet>
```
在某些情况下，加载字节码的URL可能不太直接，因为组件可能是使用由不同浏览器扩展框架提供的各种包装脚本加载的。 标识字节码URL的另一种方法是在浏览器加载浏览器扩展后查看代理历史记录。 如果采用这种方法，则需要意识到两个潜在的障碍：
- 一些代理工具将过滤器应用于代理历史记录，以从您通常不感兴趣的视图项（例如图像和样式表文件）中隐藏。如果找不到请求浏览器扩展字节码的请求，则应修改代理历史记录显示 筛选，以便所有项目都可见。
- 与其他静态资源（例如图像）相比，浏览器通常更积极地为扩展组件缓存下载的字节码。 如果您的浏览器已经加载了组件的字节码，即使对使用该组件的页面进行了完全刷新，也可能不会导致浏览器再次请求该组件。 在这种情况下，您可能需要完全清除浏览器的缓存，关闭浏览器的每个实例，然后启动新的浏览器会话以强制浏览器再次请求字节码。

确定了浏览器扩展程序字节码的URL后，通常只需将该URL粘贴到浏览器的地址栏中即可。 然后，浏览器提示您将字节码文件保存在本地文件系统中。

**TIP**<br>
如果您在Burp代理历史记录中标识了对字节码的请求，并且服务器的响应中包含完整的字节码（而不是对早期缓存副本的引用），则可以将字节码直接从Burp中保存到文件中。 最可靠的方法是在响应查看器中选择“标题”选项卡，右键单击包含响应正文的下部窗格，然后从上下文菜单中选择“复制到文件”。

### 1.4.2 Decompiling the Bytecode

字节码通常以单文件包的形式分发，可能需要将其拆包以获得单个字节码文件以反编译为源代码。

Java applet通常打包为`.jar`（Java存档）文件，Silverlight对象打包为`.xap`文件。 这两种文件类型均使用`zip`存档格式，因此您可以轻松地将其解压缩，方法是使用`.zip`扩展名重命名文件，然后使用任何`zip`阅读器将其解压缩为包含的单个文件。 Java字节码包含在`.class`文件中，而Silverlight字节码包含在`.dll`文件中。 打开相关文件包装后， 您需要反编译这些文件以获得源代码。

Flash对象打包为`.swf`文件，在使用反编译器之前不需要进行任何拆包。

要执行实际的字节码反编译，您需要使用某些特定的工具，具体取决于所使用的浏览器扩展技术的类型，如以下各节所述。

#### 1.4.2.1 Java Tools
可以使用称为Jad（Java反编译器）的工具将Java字节码反编译为Java源代码，该工具可从以下站点获得：
```
www.varaneckas.com/jad
```
#### 1.4.2.2 Flash Tools

Flash字节码可以反编译为ActionScript源代码。 通常更有效的另一种方法是将字节码反汇编成易于阅读的形式，而无需实际将其完全反编译为源代码。 要反编译和反汇编Flash，可以使用以下工具：
- Flasm — `www.nowrap.de/flasm`
- Flare — `www.nowrap.de/flare`
- SWFScan — `www.hp.com/go/swfscan` (this works for Actionscript 2 and 3)

#### 1.4.2.3 Silverlight Tools

可以使用称为.NET Reflector的工具将Silverlight字节码反编译为源代码，该工具可从以下网站获得：
```
www.red-gate.com/products/dotnet-development/reflector/
```

### 1.4.3 Working on the Source Code

获得了组件或类似组件的源代码后，您可以采取各种方法来对其进行攻击。 第一步通常是查看源代码，以了解组件的工作方式以及其包含或引用的功能。 以下是一些要寻找的物品：
- 在客户端发生的输入验证或其他与安全性有关的逻辑和事件
- 在将用户提供的数据发送到服务器之前，使用混淆或加密例程将其包装
- “隐藏”客户端功能在用户界面中不可见，但您可以通过修改组件来解锁
- 引用您以前未通过应用程序映射识别的服务器端功能

通常，查看源代码会发现要修改或操作以标识潜在安全漏洞的组件中的一些有趣功能。 这可能包括删除客户端输入验证，向服务器提交非标准数据，处理客户端状态或事件，或直接调用组件中存在的功能。

您可以按照以下各节中所述的几种方式修改组件的行为。

### 1.4.4 Recompiling and Executing Within the Browser

您可以修改反编译的源代码以更改组件的行为，将其重新编译为字节码，然后在浏览器中执行修改后的组件。 当您需要操纵关键的客户端事件（例如游戏应用程序中的骰子滚动）时，通常首选此方法。

要执行重新编译，您需要使用与所用技术相关的开发人员工具：
- 对于Java，请使用JDK中的javac程序重新编译修改后的源代码。
- 对于Flash，您可以使用flask来组合修改后的字节码，也可以使用Adobe的Flash开发工作室之一来重新编译修改后的ActionScript源代码。
- 对于Silverlight，使用Visual Studio编译修改后的源代码。

将源代码重新编译为一个或多个字节码文件后，如果使用的技术需要，则可能需要重新打包可分发文件。 对于Java和Silverlight，将修改后的字节码文件替换为解压缩的存档中的文件，使用zip实用程序重新打包，然后根据需要将扩展名更改回`.jar`或`.xap`。

最后一步是将修改后的组件加载到浏览器中，以便您所做的更改可以在您正在测试的应用程序中生效。 您可以通过多种方式实现此目的：
- 如果您可以在浏览器的磁盘缓存中找到包含原始可执行文件的物理文件，则可以将其替换为修改后的版本，然后重新启动浏览器。 如果您的浏览器没有为每个缓存的资源使用不同的单个文件，或者仅在内存中实现了浏览器扩展组件的缓存，则此方法可能会很困难。
- 使用拦截代理，您可以修改加载组件的页面的源代码，并指定其他URL，指向您控制的本地文件系统或Web服务器。 这种方法通常很难，因为更改加载组件的域可能会违反浏览器的原始策略，并且可能需要重新配置浏览器或其他方法来削弱此策略。
- 您可以使浏览器从原始服务器上重新加载组件（如前面的“下载字节码”一节中所述），使用代理拦截包含可执行文件的响应，并用修改后的版本替换消息正文。 在Burp Proxy中，可以使用“从文件粘贴”上下文菜单选项来实现此目的。 这种方法通常最容易并且最不可能遇到前面描述的问题。

### 1.4.5 Recompiling and Executing Outside the Browser

在某些情况下，执行组件时无需修改组件的行为。 例如，某些浏览器扩展组件会验证用户提供的输入，然后在将结果发送到服务器之前对其进行混淆或加密。 在这种情况下，您可以修改组件以对任意未经验证的输入执行所需的混淆或加密，并仅在本地输出结果。 然后，当原始组件提交经过验证的输入时，您可以使用代理来拦截相关请求，并且可以将其替换为修改后的组件输出的值。

要进行此攻击，您需要将旨在在相关浏览器扩展中运行的原始可执行文件更改为可以在命令行上运行的独立程序。 完成的方式取决于所使用的编程语言。 例如，在Java中，您仅需要实现main方法。 “ Java Applets：有效示例”部分提供了有关如何执行此操作的示例。

### 1.4.6 Manipulating the Original Component Using JavaScript

在某些情况下，没有必要修改组件的字节码。 相反，您可以通过修改与组件进行交互的HTML页面中的JavaScript来实现您的目标。

回顾了组件的源代码之后，您可以确定可以直接从JavaScript调用的所有公共方法，以及这些方法的参数的处理方式。 通常，可用的方法比在应用程序页面中调用的方法更多，并且您可能还会发现有关这些方法的目的和参数处理的更多信息。

例如，组件可以公开一种可以调用以启用或禁用部分可见用户界面的方法。 使用拦截代理，您可以编辑加载组件的HTML页面，并修改或添加一些JavaScript来解锁隐藏的界面部分。

**HACK STEPS**
- 1.使用上述技术下载组件的字节码，解压缩并将其反编译为源代码。
- 2.查看相关的源代码以了解正在执行的处理。
- 3.如果该组件包含可以操纵以实现您的目标的任何公共方法，则拦截与该组件交互的HTML响应，并添加一些JavaScript以使用您的输入来调用适当的方法。
- 4.如果没有，请修改组件的源代码以实现您的目标，然后在您的浏览器中或作为独立程序重新编译并执行它。
- 5.如果使用该组件向服务器提交混淆或加密的数据，请使用该组件的修改版本向服务器提交各种适当混淆的攻击字符串，以探测漏洞，就像对其他任何参数一样。


### 1.4.7 Coping with Bytecode Obfuscation

由于可以很容易地将字节码反编译以恢复其源代码，因此已经开发了各种技术来混淆字节码本身。 应用这些技术会导致字节码更难反编译，或者反编译为误导性或无效的源代码，这些源代码可能很难理解，而且如果不付出大量精力就无法重新编译。 例如，考虑以下混淆的Java源代码：
```
package myapp.interface;
import myapp.class.public;
import myapp.throw.throw;
import if.if.if.if.else;
import java.awt.event.KeyEvent;
public class double extends public implements strict
{
	public double(j j1)
	{
		_mthif();
		_fldif = j1;
	}
	private void _mthif(ActionEvent actionevent)
	{
		_mthif(((KeyEvent) (null)));
		switch(_fldif._mthnew()._fldif)
		{
		case 0:
			_fldfloat.setEnabled(false);
			_fldboolean.setEnabled(false);
			_fldinstanceof.setEnabled(false);
			_fldint.setEnabled(false);
			break;
...
```
常用的混淆技术如下：
- 有意义的类，方法和成员变量名将替换为无意义的表达式，例如a，b和c。 这迫使反编译代码的读者通过研究如何使用来确定每个项目的目的。 这可能使得在跟踪源代码的同时很难跟踪不同的项目。
- 更进一步，一些混淆器将项目名称替换为该语言保留的关键字，例如new和int。 尽管从技术上讲这使字节码变得非法，但是大多数虚拟机（VM）都可以容忍非法代码，并且可以正常执行。 但是，即使反编译器可以处理非法字节码，生成的源代码也比刚刚描述的可读性差。 更重要的是，如果不进行大量修改以一致地重命名非法命名的项目，就无法重新编译源。
- 许多混淆器从字节码中剥离了不必要的调试和元信息，包括源文件名和行号（这使堆栈跟踪信息少），局部变量名（使调试受挫）和内部类信息（使反射无法正常工作） ）。
- 可能添加了冗余代码，这些代码以明显的方式创建和处理各种数据，但这些代码与应用程序功能实际使用的实际数据无关。
- 通过使用跳转指令，可以以复杂的方式修改通过代码执行的路径，从而在读取反编译源时很难辨别执行的逻辑顺序。
- 可能会引入非法的编程结构，例如不可达的语句和缺少返回语句的代码路径。 大多数虚拟机可以字节码形式容忍这些现象，但是如果不纠正非法代码，则无法重新编译反编译的源。

**HACK STEPS**<br>
应对字节码混淆的有效策略取决于所使用的技术和分析源的目的。 这里有一些建议：
- 1.您可以在不完全了解源代码的情况下查看公共方法的组件。 很明显，可以从JavaScript调用哪些方法，以及它们的签名是什么，从而使您能够通过传入各种输入来测试方法的行为。
- 2.如果将类，方法和成员变量名替换为无意义的表达式（但不是编程语言保留的特殊词），则可以使用许多IDE中内置的重构功能来帮助自己理解代码。 通过研究项目的使用方式，您可以开始为它们分配有意义的名称。 如果您在IDE中使用重命名工具，它将为您完成大量工作，在整个代码库中跟踪该项的使用并将其重命名为任何地方。
- 3.实际上，您可以通过第二次通过混淆器运行混淆后的字节码并选择合适的选项来消除大量混淆。Java有用的混淆器是Jode。 它可以删除由另一个混淆器添加的冗余代码路径，并通过为项目分配全局唯一名称来促进理解混淆名称的过程。

### 1.4.8 Java Applets: A Worked Example

现在，我们将通过查看在Java小程序内执行输入验证的购物应用程序来考虑反编译浏览器扩展的简短示例。<br>
在此示例中，提交用户请求的订单数量的表单如下所示：
```
<form method=”post” action=”Shop.aspx?prod=2” onsubmit=”return
validateForm(this)”>
<input type=”hidden” name=”obfpad”
value=”klGSB8X9x0WFv9KGqilePdqaxHIsU5RnojwPdBRgZuiXSB3TgkupaFigj
UQm8CIP5HJxpidrPOuQPw63ogZ2vbyiOevPrkxFiuUxA8Gn30o1ep2Lax6IyuyEU
D9SmG7c”>
<script>
function validateForm(theForm)
{
	var obfquantity =
	document.CheckQuantityApplet.doCheck(
	theForm.quantity.value, theForm.obfpad.value);
	if (obfquantity == undefined)
	{
		alert(‘Please enter a valid quantity.’);
		return false;
	}
	theForm.quantity.value = obfquantity;
	return true;
}
</script>
<applet code=”CheckQuantity.class” codebase=”/scripts” width=”0”
height=”0”
id=”CheckQuantityApplet”></applet>
Product: Samsung Multiverse <br/>
Price: 399 <br/>
Quantity: <input type=”text” name=”quantity”> (Maximum quantity is 50)
<br/>
<input type=”submit” value=”Buy”>
</form>
```
提交的表单数量为2时，将发出以下请求：
```
POST /shop/154/Shop.aspx?prod=2 HTTP/1.1
Host: mdsec.net
Content-Type: application/x-www-form-urlencoded
Content-Length: 77

obfpad=klGSB8X9x0WFv9KGqilePdqaxHIsU5RnojwPdBRgZuiXSB3TgkupaFigjUQm8CIP5
HJxpidrPOuQ
Pw63ogZ2vbyiOevPrkxFiuUxA8Gn30o1ep2Lax6IyuyEUD9SmG7c&quantity=4b282c510f
776a405f465
877090058575f445b536545401e4268475e105b2d15055c5d5204161000
```
从HTML代码中可以看到，提交表单后，验证脚本会将用户提供的数量以及`obfpad`参数的值传递给Java小程序`CheckQuantity`。 该小程序显然执行了必要的输入验证，并将该数量的混淆版本返回到脚本，然后将该数量提交给服务器。

由于服务器端应用程序将我们的订单确认为两个单位，因此很明显，数量参数以某种方式包含我们所请求的值。 但是，如果我们在不了解混淆算法的情况下尝试修改此参数，则攻击将失败，大概是因为服务器无法正确解压缩我们的混淆值。

在这种情况下，我们可以使用已经描述的方法来反编译Java小程序并了解其功能。 首先，我们需要从HTML页面的applet标签中指定的URL下载applet的字节码：
```
/scripts/CheckQuantity.class
```
由于可执行文件未打包为`.jar`文件，因此无需解压缩该文件，我们可以直接在下载的`.class`文件上运行`Jad`：
```
C:\tmp>jad CheckQuantity.class
Parsing CheckQuantity.class...The class file version is 50.0 (only 45.3,
46.0 and 47.0 are supported)
Generating CheckQuantity.jad
Couldn’t fully decompile method doCheck
Couldn’t resolve all exception handlers in method doCheck
```
Jad将反编译的源代码输出为`.jad`文件，我们可以在任何文本编辑器中查看：
```
// Decompiled by Jad v1.5.8f. Copyright 2001 Pavel Kouznetsov.
// Jad home page: http://www.kpdus.com/jad.html
// Decompiler options: packimports(3)
// Source File Name: CheckQuantity.java
import java.applet.Applet;
public class CheckQuantity extends Applet
{
	public CheckQuantity()
	{
	}
	public String doCheck(String s, String s1)
	{
		int i = 0;
		i = Integer.parseInt(s);
		if(i <= 0 || i > 50)
			return null;
		break MISSING_BLOCK_LABEL_26;
		Exception exception;
		exception;
		return null;
		String s2 = (new StringBuilder()).append(“rand=”).append
(Math.random()).append(“&q=”).append(Integer.toString(i)).append
(“&checked=true”).toString();
		StringBuilder stringbuilder = new StringBuilder();
		for(int j = 0; j < s2.length(); j++)
		{
			String s3 = (new StringBuilder()).append(‘0’).append
(Integer.toHexString((byte)s1.charAt((j * 19 + 7) % s1.length()) ^
s2.charAt(j))).toString();
			int k = s3.length();
			if(k > 2)
				s3 = s3.substring(k - 2, k);
			stringbuilder.append(s3);
		}
		return stringbuilder.toString();
	}
}
```
从反编译源中可以看到，Jad已经完成了合理的反编译工作，并且applet的源代码很简单。 当使用用户提供的数量和应用程序提供的obfpad参数调用doCheck方法时，小程序首先会验证该数量是有效数字，并且介于1到50之间。如果是，它将构建一个名称/值对字符串 使用URL查询字符串格式，其中包括已验证的数量。 最后，它通过使用应用程序提供的obfpad字符串对字符执行XOR操作来混淆该字符串。 这是在数据中添加一些表面混淆的相当简单且常见的方法，以防止琐碎的篡改。

我们描述了当您对浏览器扩展组件进行反编译和分析的源代码时可以采用的各种方法。 在这种情况下，颠覆applet的最简单方法如下：
- 1.修改doCheck方法以删除输入验证，从而允许您提供任意字符串作为数量。
- 2.添加一个main方法，使您可以从命令行执行修改后的组件。 此方法仅调用修改后的doCheck方法，并将混淆的结果打印到控制台。
进行这些更改后，修改后的源代码如下：
```
public class CheckQuantity
{
	public static void main(String[] a)
	{
		System.out.println(doCheck(“999”,
“klGSB8X9x0WFv9KGqilePdqaxHIsU5RnojwPdBRgZuiXSB3TgkupaFigjUQm8CIP5HJxpi
drPOuQPw63ogZ2vbyiOevPrkxFiuUxA8Gn30o1ep2Lax6IyuyEUD9 SmG7c”));
	}
	public static String doCheck(String s, String s1)
	{
		String s2 = (new StringBuilder()).append(“rand=”).append
(Math.random()).append(“&q=”).append(s).append
(“&checked=true”).toString();
		StringBuilder stringbuilder = new StringBuilder();
		for(int j = 0; j < s2.length(); j++)
		{
			String s3 = (new StringBuilder()).append(‘0’).append
(Integer.toHexString((byte)s1.charAt((j * 19 + 7) % s1.length()) ^
s2.charAt(j))).toString();
			int k = s3.length();
			if(k > 2)
				s3 = s3.substring(k - 2, k);
			stringbuilder.append(s3);
		}
		return stringbuilder.toString();
	}
}
```
修改后的组件的此版本为999的任意数量提供有效的混淆字符串。请注意，您可以在此处使用非数字输入，从而可以针对各种基于输入的漏洞探查应用程序。
**TIP**
>Jad程序使用.jad扩展名保存其反编译的源代码。 但是，如果要修改和重新编译源代码，则需要使用.java扩展名重命名每个源文件。

剩下的就是使用Java SDK附带的javac编译器重新编译源代码，然后从命令行执行该组件：
```
C:\tmp>javac CheckQuantity.java
C:\tmp>java CheckQuantity
4b282c510f776a455d425a7808015c555f42585460464d1e42684c414a152b1e0b5a520a
145911171609
```
我们修改后的组件现在已经对我们任意数量的999执行了必要的混淆处理。要将攻击发送到服务器，我们只需要使用有效输入以正常方式提交订单，使用我们的代理拦截结果请求，并 用我们修改后的组件提供的数量代替模糊数量。 请注意，如果每次加载订单时应用程序都发行了一个新的混淆板，则需要确保将提交回服务器的混淆板与用来混淆也要提交的数量的混淆板匹配。

## 1.5 附加调试器(Attaching a Debugger)

反编译是理解和破坏浏览器扩展的最完整方法。 但是，在包含成千上万行代码的大型复杂组件中，在执行过程中观察组件，将方法和类与界面中的关键操作相关联几乎总是快得多。 这种方法还避免了解释和重新编译混淆后的字节码时可能出现的困难。 通常，实现特定目标就像执行关键功能并更改其行为以规避组件内实现的控制一样简单。

由于调试器在字节码级别工作，因此可以轻松地用于控制和理解执行流程。 特别是，如果可以通过反编译获得源代码，则可以在特定的代码行上设置断点，从而可以通过对执行过程中所采取的代码路径的实际观察来支持通过反编译获得的理解。

尽管对于所有浏览器扩展技术而言，高效的调试器尚未完全成熟，但Java小程序已很好地支持调试。 到目前为止，最好的资源是JavaSnoop，它是Java调试器，可以集成Jad来反编译源代码，通过应用程序跟踪变量，并在方法上设置断点以查看和修改参数。 图5-6显示了JavaSnoop用于直接挂钩到运行在浏览器中的Java applet。 图5-7显示了JavaSnoop用于篡改方法的返回值。

**NOTE**<br>
>最好在加载目标小程序之前运行JavaSnoop。 JavaSnoop关闭Java安全策略设置的限制，以便它可以在目标上运行。 在Windows中，它通过向系统上的所有Java程序授予所有权限来做到这一点，因此请确保JavaSnoop干净关闭并在完成工作后恢复权限。

调试Java的另一种工具是JSwat，它是高度可配置的。 在包含许多类文件的大型项目中，有时最好对关键类文件进行反编译，修改和重新编译，然后使用JSwat将其热交换到正在运行的应用程序中。 要使用JSwat，您需要使用JDK中包含的appletviewer工具启动一个applet，然后将JSwat连接到它。 例如，您可以使用以下命令：
```
appletviewer -J-Xdebug -J-Djava.compiler=NONE -J-
Xrunjdwp:transport=dt_socket,
server=y,suspend=n,address=5000 appletpage.htm
```
在处理Silverlight对象时，可以使用Silverlight Spy工具监视运行时组件的执行情况。 这可以极大地帮助将相关代码路径与用户界面内发生的事件相关联。 可从以下URL获得Silverlight Spy：
```
http://firstfloorsoftware.com/SilverlightSpy/
```
## 1.6 本机客户端组件(Native Client Components)
某些应用程序需要在用户计算机内执行无法从基于浏览器的VM沙箱内部执行的操作。 在客户端安全控制方面，以下是此功能的一些示例：
- 1.验证用户具有最新的病毒扫描程序
- 2.验证代理设置和其他公司配置是否有效
- 3.与智能卡读卡器集成

通常，这些类型的操作需要使用本机代码组件，这些组件将本地应用程序功能与Web应用程序功能集成在一起。 本机客户端组件通常通过ActiveX控件交付。 这些是在浏览器沙箱外部运行的自定义浏览器扩展。

本地客户端组件可能比其他浏览器扩展更难解密，因为没有等效的中间字节码。 但是，即使需要其他工具集，绕过客户端控件的原则仍然适用。 以下是用于此任务的一些流行工具的示例：
- OllyDbg是Windows调试器，可用于单步执行本机可执行代码，设置断点以及将修补程序应用于磁盘上或运行时上的可执行文件。
- IDA Pro是一个反汇编程序，可以在各种平台上从本机可执行代码生成人类可读的汇编代码。

尽管完整的描述不在本书的讨论范围内，但是如果您想了解有关本机代码组件的反向工程和相关主题的更多信息，以下是一些有用的资源：
- *Reversing: Secrets of Reverse Engineering* by Eldad Eilam
- *Hacker Disassembling Uncovered* by Kris Kaspersky
- *The Art of Software Security Assessment* by Mark Dowd, John McDonald,and Justin Schuh
- *Fuzzing for Software Security Testing and Quality Assurance (Artech House Information Security and Privacy)* by Ari Takanen, Jared DeMott, and Charlie Miller
- *TheIDAProBook:TheUnoffi cialGuidetotheWorld’sMostPopularDisassembler* by Chris Eagle
- `www.acm.uiuc.edu/sigmil/RevEng`
- `www.uninformed.org/?v=1&a=7`

[Chapter 5 Bypassing Client-Side Controls(2) - Capturing User Data:HTML Forms](https://dm116.github.io/2020/03/08/bypassing-client-side-controls_2/)<br>
[Chapter 5 Bypassing Client-Side Controls(4) - Handling Client-Side Data Securely](https://dm116.github.io/2020/03/08/bypassing-client-side-controls_4/)<br>
