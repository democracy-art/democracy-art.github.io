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

1.5 Opaque Data
1.6 The ASP.NET ViewState

# 2. Capturing User Data: HTML Forms
# 3. Capturing User Data: Browser Extensions
# 4. Handling Client-Side Data Securely
# 5. Summary
