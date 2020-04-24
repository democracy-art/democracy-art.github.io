--- 
layout: post
title: Download, Install and Active CN Windows10 Enterprise LTSC 2019
subtitle:
date: 2020-04-24
author: D
header-img:
catalog: true
tags: [windows]
---

# Download
Download URL:
```
ed2k://|file|cn_windows_10_enterprise_ltsc_2019_x64_dvd_2efc9ac2.iso|4027760640|4B1C7640C3A280F205A0BCFFF65472FC|/
```
SHA1:
```
37CD47E5B0E28ACE85672DC6731B58FE7539F84B
```

# Active 
`Search` text input `cmd` open with administrator rights. Copy as following and pressEnter.
```
slmgr -ipk M7XTQ-FN8P6-TTKYV-9D4CC-J462D
```
Set available KMS server.
```
slmgr -skms kms.03k.org
```
Last step active.
```
slmgr -ato
```
Check.
```
slmgr -dlv
```
**Note:** Active only for half a year, so if you use this method to active windows10 you need to do again after half a year.



References:<br>
[微软最新Windows 10官方正式版ISO镜像v1809原版下载大全（中文/英文/日文/韩文）...](https://blog.csdn.net/weixin_34174422/article/details/92475921?ops_request_misc=%257B%2522request%255Fid%2522%253A%2522158771965519724835814115%2522%252C%2522scm%2522%253A%252220140713.130102334..%2522%257D&request_id=158771965519724835814115&biz_id=0&utm_source=distribute.pc_search_result.none-task-blog-2~all~sobaiduend~default-4)<br>
[[图文] Windows 10 LTSC 2019 正式版轻松激活教程](https://www.landiannews.com/archives/51131.html)<br>
