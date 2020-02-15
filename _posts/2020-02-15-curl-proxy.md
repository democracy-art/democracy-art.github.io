---
layout:     post
title:      curl如何使用socks5做代理
subtitle:
date:       2020-02-15
author:     D
header-img: 
catalog: true
mermaid: true
tags:
    - linux_basic
---

参考:[How to Use Socks5 Proxy in Curl](https://blog.emacsos.com/use-socks5-proxy-in-curl.html)

如果你要使用本机的1080端口做代理那么命令如下:<br>
curl 版本 `>=`  `7.21.7`使用如下命令:
```
curl -x socks5h://localhost:1080 http://www.google.com/
```
curl 版本 `>=`  `7.18.0`使用如下命令:
```
curl --socks5-hostname localhost:1080 http://www.google.com/
```

比如需要下载 v2ray它本身下载命令如下:
```
bash <(curl -L -s https://install.direct/go.sh)
```
则把命令改为:
```
bash <(curl -L -s -x socks5h://localhost:1080 https://install.direct/go.sh)
```
