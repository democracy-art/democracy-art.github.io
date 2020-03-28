
---
layout: post
title: `server certificate verification failed. CAfile: none CRLfile: none`
subtitle:
date: 2020-03-27
author: D
header-img:
catalog: true
tags: [linux_basic, git]
---

When you use command `git pull` error message as following:
```
server certificate verification failed. CAfile: none CRLfile: none
```

Solution:
```
git config --global http.sslverify false
git config --global https.sslverify false
```
