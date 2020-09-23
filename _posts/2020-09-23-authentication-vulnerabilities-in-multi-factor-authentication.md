--- 
layout: post
title:  Vulerabilities in multi-factor authentication(多因素身份验证中的漏洞)
subtitle:
date: 2020-09-23
author: D
header-img:
catalog: true
tags: [multi-factor, authentication]
---

# 1. 2FA simple bypass(绕过两因素验证)
例子: 你已經有一個網站的有效用戶名和密碼,但是你卻沒有用戶的`2FA`驗證碼.<br>
Your credentials: `wiener:peter`<br>
Victim's credentials: `carlos:montoya`<br>
- 1.登錄你自己的帳號.你的2FA驗證碼會發送到你的email.點擊<kbd>Email client</kbd>按鈕查看你的郵件.
- 2.驗證完成，登錄後，去到你的"My account"頁面且記錄下該網址.
- 3.登出你的帳號
- 4.使用受害者的憑證登錄.
- 5.當提示要求輸入驗證碼的時候，手工修改URL導航到 `/my-account`.該驗證就會被繞過了.

# 2. 2FA broken logic(有缺陷的两因素验证逻辑) 
# 3. 2FA bypass using a brute-force attack(暴力破解2FA验证码)
