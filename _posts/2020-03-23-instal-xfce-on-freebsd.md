---
layout: post
title: FreeBSD install xfce
subtitle:
date: 2020-03-23
author: D
header-img:
catalog: true
tags: [xfce,FreeBSD]
---
# 1.安装依赖及xfce
```
# pkg install -y xorg
# pkg install -y xfce
# pkg install -y slim
```
# 2.使能xfce
```
# ee /etc/rc.conf
```
添加内容如下:
```
moused_enable="YES"
dbus_enable="YES"
slim_enable="YES"
```
root用户(这一步 **可选**，在handbook上没有,这样就不能用root用户直接界面登录,不知道是不推荐还是忘了，所以这一步可选)
```
# echo ". /usr/local/etc/xdg/xfce4/xinitrc" > ~/.xinitrc
```
普通用户
```
$ echo ". /usr/local/etc/xdg/xfce4/xinitrc" > ~/.xinitrc
```
# 3.重启
```
# reboot
```

参考:<br>
[FreeBSD Handbook 5.3. Installing Xorg](https://www.freebsd.org/doc/en_US.ISO8859-1/books/handbook/x-install.html)<br>
[FreeBSD Handbook 5.7.3. Xfce](https://www.freebsd.org/doc/en_US.ISO8859-1/books/handbook/x11-wm.html)<br>

