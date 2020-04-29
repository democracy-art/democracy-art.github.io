--- 
layout: post
title: [!] Unable to synchronize module index. (ConnectionError). 
subtitle:
date: 2020-04-29
author: D
header-img:
catalog: true
tags: [SecTools,recon-ng]
---

解决方案对我自己的情况适用，**不**一定对所有的情况适用.
1.把动态ip改为静态ip
```
sudo vim /etc/network/interfaces.d/eth0
```
内容如下:
```
auto lo
iface lo inet loopback
auto eth0
iface eth0 inet static # static IP 
address 192.168.1.8
netmask 255.255.255.0
gateway 192.168.1.1
```
2.把DNS的nameserver从`192.168.1.1`改为:
```
nameserver 114.114.114.114
nameserver 8.8.4.4
```
