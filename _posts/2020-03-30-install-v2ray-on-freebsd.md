--- 
layout: post
title: Install v2ray on FreeBSD
subtitle:
date: 2020-03-30
author: D
header-img:
catalog: true
tags: [FreeBSD,v2ray]
---

# Install
```
pkg install v2ray
```
# Configuration
Set v2ray system startup. Add content as following below to `/etc/rc.conf`
```
v2ray_enable="YES"
```
### set v2ray's config.json of client.
```
vim /usr/local/etc/v2ray/config.json
```
`config.json` of client.
```

```
### set v2ray's config.json of server.

# run v2ray
```
service v2ray start
```
But it didn't work well on FreeBSD now.
