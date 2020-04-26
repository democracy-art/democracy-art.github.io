--- 
layout: post
title: Set Static IP on FreeBSD 
subtitle:
date: 2020-04-26
author: D
header-img:
catalog: true
tags: [FreeBSD]
---

Add as following to `/etc/rc.conf`.
```
ifconfig_em0="inet 192.168.1.100 netmask 255.255.255.0"
defaultrouter="192.168.1.1"
```
Replace `192.168.1.100` with your own ip address.

And reboot.
```
reboot
```

Reference:
[Setting up static IP address](https://forums.freebsd.org/threads/setting-up-static-ip-address.46025/)

