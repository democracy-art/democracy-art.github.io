--- 
layout: post
title: Set static IP address and DNS on FreeBSD
subtitle:
date: 2020-03-29
author: D
header-img:
catalog: true
tags: [FreeBSD]
---

# 1.Set IP address and default router
```
# ee /etc/rc.conf
```
Add content as following into `rc.conf`. 
```
ifconfig_em0="inet 192.168.1.18 netmask 255.255.255.0"
defaultrouter="192.168.1.1"
```

# 2.Setting up DNS
```
# ee /etc/resolv.conf
```
add Content as following to resolv.conf
```
nameserver 8.8.8.8
nameserver 1.1.1.1
```

# 3. Restart network
```
# sh /etc/rc
# /etc/netstart
```
