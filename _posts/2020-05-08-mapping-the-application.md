--- 
layout: post
title: Mapping the Application
subtitle:
date: 2020-05-08
author: D
header-img:
catalog: true
tags: [web hacking, mapping the application]
---

# 1 Enumerating content and functionality

**Manual browsing**

Walk through the application starting from the main initial page, following every link, and navigating through all multistage functions.(such as user registration or password resetting).

## 1.1 Web Spidering
- Burp Suite

**TIP:** interesting file: **robots.txt**

Tools' automated approach to content enumeration has some significant limitations.

## 1.2 User-Directed Spidering
hacker + browser + burp -- Walk though every link you can find. 

## 1.3 Discoverinig Hidden Content
- Functionality that has been implemented for testing or debugging purposes.
- Functionality to differenct categories of users (for example, anonymous users,
authenticated regular users, and administrators).

**1.Brute-Force Techniques**

- Burp Intruder + SecLists
- DirBuster (project from OWASP)

**2.Inference from Published Content**

For example: we have already identified a page called `ForgotPassword` in the `/auth`
directory, we can search for similary named items, such as the following:
`http://eis/auth/ResetPassword`

**3.Use of Public Information**
- Search engines(Google,Yahoo,MSN,DuckDuckGo,Baidu,Bing)
- Web archives(WayBack Machine)

## 1.4 Application Pages Versus Functional Paths
It isn't REST-type URL. Example as following:
```
POST /bank.jsp HTTP/1.1
Host: wahh-bank.com
Content-Length: 106

servlet=TransferFunds&method=confirmTransfer&fromAccount=10372918&toAccount=
3910852&amount=291.23&Submit=Ok
```
## 1.5 Discovering Hidden Parameters
Example: if the parameter `debug=true` is added to the query string of any URL.

Parameter names (`debug`, `test`, `hide`, `source`, etc) and common values (`true`, `yes`, `on`, `1`, etc).

For `POST` requests, insert the added parameter to both the URL query string and the message body.

Choose functions where it is most likely that developer have implemented debug logic, such as login,
search, and file uploading and downloading.

# 2 Analyzing the applicatioin

## 2.1 Identifying Entry Points for User Input

## 2.2 Identifying Server-Side Technologies

## 2.3 Identifying Server-Side Functionality

## 2.4 Mapping the Attack Surface
