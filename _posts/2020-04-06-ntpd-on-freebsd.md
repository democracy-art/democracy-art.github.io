--- 
layout: post
title: NTP on FreeBSD 12.1
subtitle:
date: 2020-04-06
author: D
header-img:
catalog: true
tags: [NTP,FreeBSD]
---

Add the following to `/etc/ntp.conf`
```
pool ntp2.aliyun.com iburst
pool 1.tw.pool.ntp.org iburst
pool 0.freebsd.pool.ntp.org iburst
server ntp1.aliyun.com iburst
```
Add the following to `/etc/rc.conf`
```
ntpd_enable="YES"
ntpd_sync_on_start="YES"
```
Set `ntpd_enable="YES"` to start ntpd at boot time. Once `ntpd_enable=YES` has been added to `/etc/rc.conf`, ntpd can be started immediately without rebooting the system by typing:
```
service ntpd start
```
Set `ntpd_sync_on_start="YES"` to allow ntpd to step the clock any amount, one time at startup. 


Reference:<br>
[Clock Synchronization with NTP](https://www.freebsd.org/doc/handbook/network-ntp.html)
