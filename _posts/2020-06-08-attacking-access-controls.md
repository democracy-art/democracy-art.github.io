--- 
layout: post
title: Attacking Access Controls
subtitle:
date: 2020-06-08
author: D
header-img:
catalog: true
tags: [web hacking, attacking access controls]
---

# 1.Common Vulnerabilities

Access controls can be divided into three broad categories:`vertical`,`horizontal`, and
 `context-dependent`.
- Vertical privilege escalation
- Horizontal privilege escalation
- Business logic exploitation

**1.1 Completely Unprotected Functionality**
In many cases of broken access controls, sensitive functionlity and resources can be
accessed by anyone who knows the relevant URL. For example,with many applications,
anyone who visits a specific URL can make full use of its administrative functions:
```
https://wahh-app.com/admin/
```
Sometimes,the URL that grants access to powerful functions may be less easy to guess
and may even be quite cryptic:
```
https://wahh-app.com/menus/secure/ff457/DoAdminMenu2.jsp
```
UI JavaScript as following:
```
var isAdmin = false;
...
if (isAdmin) {
	adminMenu.addItem("/menus/secure/ff457/DoAdminMenu2.jsp
", "create a new user");
}
```

**1.1.1 Direct Access to Methods**

Outside of this situation, some instances of direct access to methods can be
identified where URLs or parameters use the standard Java naming conventions,
such as `getBalance` and `isExpired`.

The following example shows the `getCurrentUserRoles` method being invoked from
within the interface `securityCheck`:
```
http://wahh-app.com/public/securityCheck/getCurrentUserRoles
```
In this example,in addition to testing the access controls over the 
`getCurrentUserRoles` method, you should  check for the existence of other similarly
named methods such as `getAllUserRoles`,`getAllRoles`,`getAllUsers`,and 
`getCurrentUserPermissions`.

**1.2 Identifier-Based Functions**
**1.3 Multistage Functions**
**1.4 Static Files**
**1.5 Platform Misconfiguration**
**1.6 Insecure Access Control Methods**

1.6.1 Parameter-Based Access Control 

In some versions of this model,the application determines a user's role or 
access level at the time of login and from this point onward transmits this
information via the client in a `hidden form field`,`cookie`,or preset 
`query string parameter`. For example:
```
https://wahh-app.com/login/home.jsp?admin=true
```

1.6.2 Referer-Based Access Control

In other unsafe access control models,the application uses the HTTP `Referer`
header as the basis for making access control decisions.

1.6.3 Location-Based Access Control

Many businesses have a regulatory or business requirement to restrict access to 
resources depending on the user's geographic location. Location-based access
controls are relatively easy for an attacker to circumvent. Here are some 
common methods of bypassing them:
- Using a web proxy that is based in the required location
- Using a VPN that terminates in the required location
- Using a mobile device that supports data roaming
- Direct manipulation of client-side mechanism for geolocation

# 2.Attacking Access Controls

Before starting to probe the application to detect any actual access control 
vulnerabilities,you should take a moment to review the results of your application
`mapping exercise`. You need to understand what the application's actual 
requirements are in terms of access control, and therefore where it will probably
be most fruitful to `focus` your attention.

**2.1 Testing with Different User Accounts**

**HACK STEPS**
- 2.Review the contents of Burp's site map to ensure that you have identified all the
functionality you want to test. The use the context menu to select the "compare site maps" feature.

**2.2 Testing Multistage Processes**

**2.3 Testing with Limited Access**

**HACK STEPS**
- 2.try adding parameters such as `admin=true` to the URL query string and the body of
`POST` requests.
- 3.Test whether the application uses the `Referer` header as the basis for making
access control decisions.
- 4.Review all client-side HTML and scripts to find references to hidden functionality
or functionality that can be manipulated on the client side, such as script-based user
interfaces. Also, decompile all browser extension components to discover an references
to server-side functionality.

**2.4 Testing Direct Access to Methods**
**2.5 Testing Controls Over Static Resources**
**2.6 Testing Restrictions on HTTP Methods**

# 3.Securing Access Controls
**3.1 A Multilayered Privilege Model**
