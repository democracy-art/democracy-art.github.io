--- 
layout: post
title: Attacking Session Management 
subtitle:
date: 2020-06-05
author: D
header-img:
catalog: true
tags: [web hacking, attacking session management]
---

# 1.The Need for State

In most cases, applications use HTTP cookies as the transmission mechanism for
passing these session tokens between server and client. The server's first 
response to new client contains an HTTP header like the following:
```
Set-Cookie: ASP.NET_SessionId=mza2ji454s04cwbgwb2ttj55
```
and subsequent requests from the client contain this header:
```
Cookie: ASP.NET_SessionId=mza2ji454s04cwbgwb2ttj55
```
The vulnerabilities that exist in session management mechanisms largely fall into two
categories:
- Weaknesses in the generation of session tokens
- Weaknesses in the handling of session tokens throughout their life cycle

Alternatives to Sessions
- HTTP authentication(basic, digest, NTLM)
- Sessionless state mechanisms

# 2.Weaknesses in Token Generation

**NOTE** There are numerous locations where an application's security depends on the
unpredictability of tokens it generates. Here are some examples:
- Password recovery tokens sent to the user's registered email address
- Tokens placed in hidden form fields to prevent cross-site request forgery attacks
- Tokens used to give one-time access to protected resources
- Persisten tokens used in **remember me** functions
- Tokens allowing customers of a shopping application that does not use authentication
to retrieve the current status of an existing order

**2.1 Meaningful Tokens**

For example, the following token may initially appear to be a long random string:
```
757365723d6461663b6170703d61646d696e3b646174653d30312f31322f3131
```
However, on closer inspection, you can see that it contains only hexadecimal 
characters. Guessing that the string may actually be a hex encoding of a string of 
ASCII characters, you can run it through a decoder to reveal the following:
```
user=daf;app=admin;date=10/09/11
```

Here are some components that may be encountered within structured tokens:
- The account username
- The numeric identifier that the application uses to distinguish between accounts
- The user's first and last names
- The user's e-mail address
- The user's group or role within the application
- A date/time stamp
- An incrementing or predictable number
- The client IP address

This can be deliberate measure to obfuscate their content, or it can simply ensure 
safe transport of binary data via HTTP Encoding schemes that are commonly encountered
include `XOR` `Base64` and `hexadecimal` representation using ASCII characters.

**HACK STEPS**
- 1.Try changing the token's value on **byte** at a time(or even on **bit** at a time)
and resubmitting the modified token to the application to determine whether it is
accepted.
- 2.Log in as serveral different users at different times, and record the tokens 
received from the server. If self-registration is available and you can choose your 
username, log in with a series of similar usernames containing small variations 
between them, such as `a`,`aa`, `aaa`, `aaaa`, `aaab`,`aaac`,`aaba`, and so on.
- 3.Analyze the tokens for any correlations that appear to  be related to the
username and other user-controllable data.
- 4.Analyze the tokens for any detectable encoding or obfuscation.
- 5.If any meaning can be reverse-engineered from the sample of session tokens,
consider whether you have sufficient information to attempt to guess the tokens
recently issued to other application users.

**2.2 Predictable Tokens**
The types of potential variations you might encounter here are open-ended,but the 
authors's experience in the field indicates that predictable session tokens
commonly arise from three different sources:
- Concealed sequences
- Time dependency
- Weak random number generation

**2.3 Encrypted Tokens**
- ECB Ciphers
- CBC Ciphers

# 3.Weaknesses in Session Token Handling
**3.1 Disclosure of Tokens on the network**
**3.2 Disclosure of Tokens in Logs**
**3.3 Vulnerable Mapping of Tokens to Sessions**
**3.4 Vulnerable Session Termination**
**3.5 Client Exposure to Token Hijacking**
**3.6 Liberal Cookie Scope**

# 4.Securing Session Management

# 5.summary
