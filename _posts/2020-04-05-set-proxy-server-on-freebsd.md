--- 
layout: post
title: Set proxy server on FreeBSD 12.1
subtitle:
date: 2020-04-05
author: D
header-img:
catalog: true
tags: [FreeBSD,proxy]
---

Inside a controlled network, it is alittle harder to use FreeBSD.<br>
The simple things become hard, such as install software by `pkg`.<br>
So that is why I set a local proxy server on FreeBSD in China.<br>

For `csh` or `tcsh`, set proxy in `/etc/csh.cshrc`:
```
setenv HTTP_PROXY socks5://127.0.0.1:1080
setenv HTTPS_PROXY socks5://127.0.0.1:1080
```
For `sh`, set proxy in `/etc/profile`:
```
export HTTP_PROXY socks5://127.0.0.1:1080
export HTTPS_PROXY socks5://127.0.0.1:1080
```
References:
[Using FreeBSD inside a controlled network - A required HTTP Proxy and No FTP](https://www.rhyous.com/2012/04/13/using-freebsd-inside-a-controlled-network-a-required-http-proxy-and-no-ftp/)<br>
[Configure proxy](https://nanxiao.gitbooks.io/freebsd-101-hacks/content/posts/configure-proxy.html)
