--- 
layout: post
title: Set NTP Service on Kali Linux
subtitle:
date: 2020-04-01
author: D
header-img:
catalog: true
tags: [NTP,linux_basic]
---

Kali Linux version: Kali2019/2020

NTP means **Network Time Protocol**.

1. Install NTP service
```
apt install ntp
```
2. Check ntp
```
dpkg --get-selections ntp
```
3. Modify `ntp.conf`
```
vi /etc/ntp.conf
```
4. Find key word 
```
pool 0.debian.pool.ntp.org iburst
pool 1.debian.pool.ntp.org iburst
pool 2.debian.pool.ntp.org iburst
pool 3.debian.pool.ntp.org iburst
```
Instead of the 4 lines pools with your own **NTP Server**.
For example:
```
pool ntp.api.bz iburst  # Shanghai server
pool asia.pool.ntp.org iburst # Taiwang server
pool time.nist.gov iburst # Can't to connect in China Because GFW of China 
pool time.windows.com iburst # Can't to connect in China Because GFW of China
```
5. Enable NTP on Boot
```
systemctl enable ntp.service
```
6. Restart NTP service
```
systemctl restart ntp.service
```
Check it.
```
systemctl status ntp.service
```
Or 
```
ntpq -pn
```
