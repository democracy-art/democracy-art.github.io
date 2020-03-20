---
layout:     post
title:      When ssh host key has been changed
subtitle:
date:       2020-03-20
author:     D
header-img: 
catalog: true
mermaid: true
tags:
    - [v2ray]
---

When ssh host key has been changed,what you should do?

You should generate a new key.

Example 1: Win10 powershell
```
ssh-keygen -f "C:\\Users\jun/.ssh/known_hosts"
```
`jun` is a user's account of computer.


