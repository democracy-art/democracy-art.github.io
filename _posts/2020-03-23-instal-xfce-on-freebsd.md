---
layout: post
title: FreeBSD install xfce
subtitle:
date: 2020-03-23
author: D
header-img:
catalog: true
tags: [FreeBSD]
---

# 1.Prepare
```
pkg install -y xorg
```
```
pkg install -y slim
```
# 2.Install
```
pkg install -y xfce
```
# 3.Configure
### 3.1 add content to file `rc.conf`
```
vi /etc/rc.conf
```
Add the following:
```
moused_enable="YES"
dbus_enable="YES"
hald_enable="YES"
slim_enable="YES"
```
### 3.2 create file `.xinitrc`
```
cd ~
vi .xinitrc
```
Add the following:
```
exec xfce4-session
```
# 4.Reboot
```
reboot
```
