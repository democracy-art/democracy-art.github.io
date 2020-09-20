--- 
layout: post
title: 在FreeBSD12.0上用Let's Encrypt 申請SSL安全加固Nginx 
subtitle:
date: 2020-09-20
author: D
header-img:
catalog: true
tags: [freebsd, nginx, let's encrypt, ssl]
---
# 1.安裝Certbot
首先，获取端口树的压缩快照
```
# portsnap fetch
```
该命令可能需要几分钟的时间才能完成。 完成后，提取快照
```
# portsnap extract
```
此命令可能还需要一段时间才能完成。 完成后，导航至端口树内的py-certbot目录
```
# cd /usr/ports/security/py-certbot
```
然后使用make命令下载并编译Certbot源代码
```
# make install clean
```
接下来，导航到端口树中的py-certbot-nginx目录
```
# cd /usr/ports/security/py-certbot-nginx
```
从此目录再次运行make命令。 这将为Certbot安装nginx插件，我们将使用该插件获取SSL证书
```
# make install clean
```
在安装此插件的过程中，您将看到几个蓝色的对话框，如下所示
![]()
