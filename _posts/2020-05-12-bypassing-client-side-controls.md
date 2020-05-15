--- 
layout: post
title: Bypassing Client-Side Controls
subtitle:
date: 2020-05-12
author: D
header-img:
catalog: true
tags: [web hacking, bypass client-side controls]
---

# 1. Transmitting Data Via the Client

**1.1 Hidden Form Fields**

- `hidden`
```
<input type="hidden" name="price" value="449">
```

**1.2 HTTP Cookies**

The customer has logged in to the application(server), she receives the following response:
```
HTTP/1.1 200 OK
Set-Cookie: DiscountAgreed=25
Content-Length: 1530
...
```
The customer can try to change the cookie. If the application trusts the value of the 
`DiscountAgreed` cookie when it is submitted back to the server, customer can obtain 
arbitrary discounts by modifying its value. For example:
```
POST /shop/92/Shop.aspx?prod=3 HTTP/1.1
Host: mdsec.net
Cookie: DiscountAgreed=40
Content-Length: 10

quantity=1
```
The fact that cookies normally can't be modified.

**1.3 URL Parameters**

**1.4 The Referer Header**

It is used to indicate the URL of the page from which the current request **originated**.

For example:
```
GET /auth/472/CreateUser.ashx HTTP/1.1
Host: mdsec.net
Referer: https://mdsec.net/auth/472/Admin.ashx
```
The application may use the `Referer` header to verify that this request originated from the
correct stage(`Admin.ashx`). If it did, the user can access the requested functionality.

**1.5 Opaque Data**

**1.6 The ASP.NET ViewState**

# 2. Capturing User Data: HTML Forms

**2.1 Length Limits**

Example as following:
```
Quantity: <input type="text" name="quantity" maxlength="1"> <br/>
```
Look for form elements containing a `maxlength` attribute.

**2.2 Script-Based Validation**

Example:
```
<form method="post" action="Shop.aspx?prod=2" onsubmit="return validateForm(this)">
Product: Samsung Multiverse <br/>
Price: 399 <br/>
Quantity: <input type="text" name="quantity"> (Maximum quantity is 50) <br/>
<input type="submit" value="Buy">
</form>

<script>function validateForm(theForm)
{
	var isInteger = /^\d+$/;
	var valid = isInteger.test(quantity) && quantity > 0 && quantity <= 50;
	if (!valid)
		alert('Please enter a valid quantity');
	return valid;
}
</script>
```
Client-side controls of this kind are usually easy to circumvent. For example:
- Disable JavaScript within the browser
- Intercept the validated submission with your proxy, and modify the data to your desired value.
- Intercept the server's response that contians the JavaScript validation routine and modify the script to neutralize its effect -- in the previous example, by changing the `ValidateForm` function to return the true in every case.

**2.3 Disabled Elements**

For example:
```
Price: <input type="text" disable="true" name="price" value="299"> <br/>
Quantity: <input type="text" name="quantity"> (Maximum quantity is 50)
```
In this situation, you should definitely test whether the server-side application 
still processes this parameter. If it does, seek to exploit this fact.

**NOTE:** The browsers do **NOT** include disabled form elements when forms
are submitted. Therefore, you will **not** identify these if you simply walk
through the application's functionality, monitoring the requests issued
by the browser. To identify disabled elements, you need to monitor the server's
**responses** or view the **page source** in your browser.

# 3. Capturing User Data: Browser Extensions

**3.1 Common Browser Extension Technologies**

**3.2 Approaches to Browser Extensions**

**3.3 Intercepting Traffic from Browser Extensions**

3.3.1 Handling Serialized Data

- Java serialization
```
Content-Type: application/x-java-serialized-object
```
`DSer` is a handy plug-in to Burp Suite that provides a framework for viewing and 
manipulating serialized Java objects that have been intercepted within Burp.

- Flash Serialization
```
Content-Type: application/x-amf
```

- Silverlight Serialization
```
Content-Type: application/soap+msbinl
```

3.3.2 Obstacles to Intercepting Traffic from Browser Extensions

**3.4 Decompiling Browser Extensions**

3.4.1 Downloading the Bytecode

In general, the bytecode is loaded in a single file from a URL specified within the HTML
source code for application pages that run the browser extension. Java applets generally
are loaded using the `<applet>` tag, and other components generally are loaded using the
`<object>` tag. For example:
```
<applet code="CheckQuantity.class" codebase="/scripts" id="CheckQuantityApplet">
</applet>
```

3.4.2 Decompiling the Bytecode

- Java applets normally are packaged as `.jar`, its bytecode is contained in `.class` file
- Silverlight objects are packaged as `.xap` files, its bytecode is contained in `.dll` file 
- Flash objects are packaged as `.swf` files and don't require any unpacking before you use a decompiler.

To perform the actual bytecode decompilation, you need to use some specific tools as following:

- Java Tools - Jad
- Flash Tools - Flasm, Flare, SWFScan
- Silverlight Tools - .NET Reflector

3.4.3 Working on the Source Code

Here are some items to look for:
- Input validation or other security-relevant logic and events that occur on the client sid
- Obfuscation or encryption routines being used to wrap user-supplied data before it is sent to the server
- Hidden clien-side functionality.
- References to server-side functionality that you have not previously identified via your application mapping

You can modify the component's behavior in several ways, as described in the following sections.
- Recompiling and Executing Within the Browser
	- For Java, use the `javac` program in the JDK to recompile your modified source code.
	- For Flash, you can use `flasm` to reassemble your modified bytecode...
	- For Silverlight, use  Visual Studio to recompile your modified source code.
- Recompiling and Executing Outside the Browser
- Manipulating the Original Component Using JavaScript

3.4.4 Coping with Bytecode Obfuscation

**3.5 Attaching a Debugger**

**3.6 Native Client Components**

Native client components may be significantly harder to decipher than other browser extension.
Here are some examples of popular tools used for this task:
- OllyDbg - A Windows debugger
- IDA Pro - A disassembler

The following are some useful resources if you want to know more about reverse engineering
of native code components and related topics:
- *Reversing: Secrets of Reverse Engineering* by Eldad Eilam
- *Hacker Disassembling Uncovered* by Kris Kaspersky
- *The Art of Software Security Assessment* by Mark Dowd, John McDonld, and Justin Schuh
- *Fuzzing for Software Security Testing and Quality Assurance(Artech House Information Security and Privacy)* by Ari Takanen, Jared DeMott, and Charlie Miller
- *The IDA Pro Book: The Unofficial Guide to the World's Most Popular Disassembler* by Chris Eagle
- www.acm.uiuc.edu/sigmil/RevEng
- www.uninformed.org/?v=1&a=7

# 4. Handling Client-Side Data Securely

**4.1 Transmitting Data Via the Client**
**4.2 Validating Client-Generated Data**
**4.3 Logging and Alerting**

# 5. Summary
