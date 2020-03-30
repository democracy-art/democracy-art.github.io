--- 
layout: post
title: Debian 9 turns on Google BBR
subtitle:
date: 2020-03-30
author: D
header-img:
catalog: true
tags: [BBR,linux_basic]
---

The network is bumpy, especially during late peak hours.<br>
We will inevitably experience slow speeds when accessing foreign servers.<br>
If our server is a Linux system, fortunately, the kernel version of the Debian 9 system is 4.9.x.<br>
The kernel comes with BBR congestion algorithm developed by Google, we can easily start BBR to accelerate access.

Debian 9 turns on Google BBR congestion control algorithm:
```
vi /etc/sysctl.conf
```
Add contents as following to `sysctl.conf`:
```
net.core.default_qdisc=fq
net.ipv4.tcp_congestion_control=bbr
```
reload `sysctl`:
```
sysctl -p
```
If there is no error message in the sysctl you should reboot Debian 9 to make it work.
```
reboot
```
After reboot Enter command:
```
lsmod |grep bbr
```
It show something like as following means it worked.
```
root@localhost:~# lsmod |grep bbr
tcp_bbr                16384  21
```

