--- 
layout: post
title: Install shadowsocks-libev on FreeBSD
subtitle:
date: 2020-03-30
author: D
header-img:
catalog: true
tags: [FreeBSD,shadowsocks]
---

### Install

Shadowsocks-libev is available in FreeBSD Ports Collection. You can install it in either way, `pkg` or `ports`.

#### pkg(recommended)

```
pkg install shadowsocks-libev
```
#### ports

```
cd /usr/ports/net/shadowsocks-libev
make install
```
### Configuration

Edit your `config.json` file. Be default, it's located int `/usr/local/etc/shadowsocks-libev`.

To enable shadowsocks-libev, add the following rc variable to your `/etc/rc.conf` file:
```
shadowsocks_libev_enable="YES"
```
### Run as server
Start the Shadowsocks server:
```
service shadowsocks_libev start
```
### Run as client
By default, shadowsocks-libev is running as a server in FreeBSD.<br>
If you would like to start shadowsocks-libev in client mode,<br> 
you can modify the rc script(`/usr/local/etc/rc.d/shadowsocks_libev`) manually.
```
# modify the following line from "ss-server" to "ss-local"
command="/usr/local/bin/ss-local"
```
and then run as client:
```
service shadowsocks_libev start
```
### Stop shadowsocks-libev
```
service shadowsocks_libev stop
```
### Disable shadowsocks-libev 
To enable shadowsocks-libev, add the following rc variable to your `/etc/rc.conf` file:
```
shadowsocks_libev_enable="NO"
```

**NOTE:** that is simply a workaround, each time you upgrade the port your changes will be overwitten by the new version.
