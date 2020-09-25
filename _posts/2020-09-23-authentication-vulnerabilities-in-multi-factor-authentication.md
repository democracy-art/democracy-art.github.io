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
有時候2FA出現邏輯漏洞,當一個用戶完成了第一步登錄,網頁並沒有充分的去驗證是否是同一個用戶進行的第二步.
比如:用戶登錄的第一步如下:
```
POST /login-steps/first HTTP/1.1
Host: vulnerable-website.com
...
username=carlos&password=qwerty
```
在進行登錄的第二步之前,用戶被分配一個與帳號相關的cookie如下:
```
HTTP/1.1 200 OK
Set-Cookie: account=carlos
```
然後進行登錄的第二步:
```
GET /login-steps/second HTTP/1.1
Cookie: account=carlos
```
當提交驗證碼,請求使用cookie決定訪問哪一個賬戶:
```
POST /login-steps/second HTTP/1.1
Host: vulnerable-website.com
Cookie: account=carlos
...
verification-code=123456
```
 如果是這種情況，攻擊者可以登錄他們自己的帳號但是呢修改cookiel里面`account`的值,
 可以把它修改爲任意值當提交驗證碼的時候:
 ```
 POST /login-steps/second HTTP/1.1
 Host: vulerable-website.com
 Cookie: account=victim-user
 ...
 verification-code=123456
 ```
 這樣子是非常危險的，如果攻擊者可以暴力破解驗證碼,那麼他們可以登錄任何帳號僅僅需要
 知道受害者的用戶名,而不用知道受害者的密碼.<br>

 例子:<br>
 ```
 Your credentials: wiener:peter
 Victim's username: carlos
 ```

 - 1.運行Burp,登錄你自己的帳號研究網站的2FA驗證過程.注意到在`POST /login2`請求,
 參數`verify`用於決定哪個用戶將被訪問
 - 2.登出你的帳號
 - 3.發送`GET /login2`請求到Burp Repeate.把`verify`的值改爲受害者`carlos`,然後發送請求.
 這樣做是爲了確保受害者帳號`carlos`的臨時的2FA驗證碼會生成.
 - 4.去到登錄頁面且輸入你的用戶何密碼.然後提交一個無效的2FAY驗證碼.
 - 5.發送`POST /login2`請求到Burp Intruder或Turbo Intruder.
 - 6.在Burp Intruder或Turbo Intruder,設置`verify`的值爲`carlos`且添加一個payload到`mfa-code`
 的參數.然後暴力破解驗證碼.驗證碼的生成參考[bash shell編程產數組0000-9999並導入文件code
](https://dm116.github.io/2020/09/23/shell-generate-numbers-from-0000-to-9999/).
 - 7.加載含有`302`的response頁面到你的瀏覽器.
 - 8.點擊"My account"就完成該實驗了. 

# 3. 2FA bypass using a brute-force attack(暴力破解2FA验证码)
就像密碼一樣,網站需要採取措施來防止2FA的驗證碼被暴力破解.這個非常重要因爲<br>
驗證碼經常是4到6位的數字.沒有足夠的保護,暴力破解驗證碼就是小事一樁.<br>

一些網站嘗試去防止對驗證碼的暴力破解,通過把嘗試輸入驗證碼錯誤次數超過一定的數量<br>
的用戶登出.但是這個個防禦措施在現實中是無效的,因爲高級的攻擊者通過Burp Intruder<br>
創建`macros`或者用Turbo Intruder擴展，能夠將多個步驟的操作自動化.<br>

例子:<br>
假設你已經有了一個有效的受害者的用戶名和密碼.但是卻沒有該用戶的驗證碼.<br>
用戶的憑證: `carlos:montoya`<br>

Tip:你需要使用Burp macros結合Burp Intruder來完成這個實驗.更多的關於**macros**信息<br>
請參考[Burp Suite documentation](https://portswigger.net/burp/documentation/desktop/options/sessions).<br>
如果你精通Python可能更喜歡用Turbo Intruder.<br>

- 1.運行Burp,登錄carlos帳號且研究網站的2FA的驗證過程.注意到如果你輸入錯誤的驗證碼
**兩**次的話,你將會被log out(登出).在發送**每一個**request之前你都需要使用Burp session處理
相關的特性爲了**自動**的登錄回去.
- 2.去到Burp的"Project options">"Sessions".在"Session Handling Rules"panel,點擊<kbd>Add</kbd>.
"Session handling rule editor"對話框就會打開.
- 3.在對話,去到"Scop"標籤.在"URL Scope"下面選擇"Include all URLs"選項.
- 4.切回到"Details"標籤且在"Rule Actions",下面點擊<kbd>Add</kbd> > "Run a macro".
- 5.在"Select macro"下面點擊<kbd>Add</kbd>去打開"Macro Recorder".選擇3個requests:
```
GET /login
POST /login
GET /login2
```
然後,點擊<kbd>OK</kbd>.然後"Macro Editor"dialog opens.
- 6.點擊<kbd>Test macro</kbd>且檢查最後的response包含頁面里面內容是要你提供4位數的
安全碼.這確定了macro(宏)是正確的工作的.
- 7.一直點擊<kbd>OK</kbd>直到關掉各種對話框,直到回到Burp的主窗口.macro現在將會**自動**
地幫你以Carlos的身份登錄回去的在Burp Intruder每一次的發送的request中.
- 8.發送`POST /login2`request到Burp Intruder.
- 9.在Burp Intruder,添加一個payload position到`mfa-code`參數.
- 10.在"Payloads"標籤,選擇"Numbers"payload類型.輸入範圍0-9999且設置步長爲1.
設置 min/max 整數位數都爲4, 且max的分數位數爲0.這將會爲每一個可能的4位整數創建payload
且這4位整數不滿4位的，用0在前面補齊,比如99補齊爲0099.
- 11.在"Options"標籤下,在"Request Engine"下面,把threads設置爲1.
- 12.開始攻擊.最終,會有一個request將會返回302狀態碼.用右鍵點擊該回應且選擇
"Show response in browser".複製它的URL到你的瀏覽器且加載.
- 13.點擊"My account"完成該實驗.

[Vulnerabilities in multi-factor authentication](https://portswigger.net/web-security/authentication/multi-factor)
