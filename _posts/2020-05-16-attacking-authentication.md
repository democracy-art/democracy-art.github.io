--- 
layout: post
title: Attacking Authentication
subtitle:
date: 2020-05-16
author: D
header-img:
catalog: true
tags: [web hacking, authentication]
---

# 1.Authentication Technologies
- HTML forms-based authentication
- Mutifactor mechanisms, such as those combining passwords and physical tokens
- Client SSL certificates and / or smartcards
- HTTP basic and digest authentication
- Windows-integrated authentication using NTLM or Kerberos
- Authentication services
 
# 2.Design Flaws in Authentication Mechanisms

**2.1 Bad Passwords**
- Very short or blank
- Common dictionary words or names
- The same as the username
- Still set to a default value

**2.2 Brute-Forcible Login**

Here are the most popular real-world passwords: `password`, `website name`, `12345678`, `qwerty`, `abc123`, `111111`, `monkey`, `12345`, `letmein`.

**2.3 Verbose Failure Messages**

For Example:
```
Password is incorrect.
User in not recognised.
```
**NOTE:** Many authentication mechanism disclose usernames either implicitly or explicitly.
In a web mail account, the username is often the **e-mail address**, which is common 
knowledge by design. Many other sites expose usernames within the application without 
considering the advantage this grants to an attacker, or generate usernames in a way that
can be predicted(for exmaple, **user1842, user1843**, and so on).

**NOTE:** Subtle differences hidden with the HTML source, such as **comments** or 
**layout differences**.

The **timing difference** between the two responses.

**2.4 Vulnerable Transmission of Credentials**

**2.5 Password Change Functionality**

Many web applications' password change functions are accessible without authentication
and do the following:
- Provide a verbose error message indicating whether the requested username is valid.
- Allow unrestricted guesses of the "**existing password**" field.
- Check whether the "**new password**" and "**confirm new password**" fields have the
same value only after validating the existing password, thereby allowing an attack
to succeed in discovering the existing password noninvasively.

**2.6 Forgotten Password Functionality**

**TIP** The application may transmit the address via **hidden form field** or **cookie**.

**2.7 "Remember Me" Functionality**

- Some "**remember me**" functions are implemented using a simple persistent cookie, such as `RememberUser=daf`.
- Some "**remember me**" functions set a cookie that contains not the username but a kind of
persistent session identifier, such as `RememberUser=1328`.

**2.8 User Impersonation Functionality**

**2.9 Incomplete Validation of Credentials**

**2.10 Nonunique Usernames**

- If self-registration is possible, attempt to register the same username twice with differentpasswords.

**2.11 Predictable Usernames**

**2.12 Predictable Initial Passwords**

**2.13 Insecure Distribution of Credentials**

# 3.Implementation Flaws in Authentication

**3.1 Fail-Open Login Mechanisms**

If the call to `db.getUser()` throws an **exception** for some reason(for example, a null pointer
exception arising because the user's request did no contain a username or password parameter),
the **login succeeds**.
```
public Response checkLogin(Session session) {
	try {
		String uname = session.getParameter("username");
		String passwd = session.getParameter("password");
		User user = db.getUser(uname, passwd);
		if (user == null) {
			// invalid credentials
			session.setMessage("Login failed.");
			return doLogin(session);
		}
	}
	catch (Exception e) {}
	
	// valide user
	session.setMessage("Login successful.");
	return doMainMenu(session);
}
```

Repeat the login process numerous times, modifying pieces of the data submitted in unexpected ways. For example, for each request parameter or cookie sent by the client, do the following:
- Submit an empty string as the value.
- Remove the name/value pair altogether.
- Submit very long and very short values.
- Submit strings instead of numbers and vice versa.
- Submit the same item multiple times, with the same and different values.

**3.2 Defects in Multistage Login Mechanisms**

**3.3 Insecure Storage of Credentials**

# 4.Securing Authentication

**4.1 Use Strong Credentials**

**4.2 Handle Credentials Secretively**

**4.3 Validate Credentials Properly**

**4.4 Prevent Information Leakage**

**4.5 Prevent Brute-Force Attacks**

**4.6 Prevent Misuse of the Password Change Function**

**4.7 Prevent Misuse of the Account Recovery Function**

**4.8 Log, Monitor, and Notify**

# 5.Summary

