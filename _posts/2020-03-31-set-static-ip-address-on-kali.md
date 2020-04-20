--- 
layout: post
title: Set static IP address and DNS on Kali2019/2020
subtitle:
date: 2020-03-31
author: D
header-img:
catalog: true
tags: [linux_basic, static IP]
---

# Set static IP address
Your just need to edit **one** of `/etc/network/interfaces`, /etc/network/interfaces.d/eth0` 
```
vim /etc/network/interfaces
```
Or
```
touch /etc/network/interfaces.d/eth0
vim /etc/network/interfaces.d/eth0
```
content of `interfaces`
```
# This file describes the network interfaces available on your system
# and how to activate them. For more information, see interfaces(5).

source /etc/network/interfaces.d/*

# The loopback network interface
auto lo
iface lo inet loopback
auto eth0
iface eth0 inet static # static IP 
address 192.168.1.8
netmask 255.255.255.0
gateway 192.168.1.1
```
Or Content of `eth0`
```
auto lo
iface lo inet loopback
auto eth0
iface eth0 inet static # static IP 
address 192.168.1.8
netmask 255.255.255.0
gateway 192.168.1.1
```

# Set DNS
```
vim /etc/resolv.conf
```
content of resolv.conf
```
nameserver 119.29.29.29
nameserver 8.8.4.4
```
If you set `iface eth0 inet auto` your nameserver would be **overwitten** very time when the system reboot. 

# Enable the configuration
```
systemctl restart network-manager.service
systemctl restart network-online.target
systemctl restart networking.service
```
