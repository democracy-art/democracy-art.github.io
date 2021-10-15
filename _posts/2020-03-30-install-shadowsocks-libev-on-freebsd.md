--- 
layout: post
title: Install shadowsocks-libev on FreeBSD
subtitle:
date: 2020-03-30
author: D
header-img:
catalog: true
tags: [FreeBSD,shadowsocks,shadowsocks-libev]
---

# Install

Shadowsocks-libev is available in FreeBSD Ports Collection. You can install it in either way, `pkg` or `ports`.

#### pkg(recommended)

```
pkg install shadowsocks-libev
```
# Configuration
### Edit config.json
```
#ee /usr/local/etc/shadowsocks-libev/config.json
```
```
{
    "server":"your_server_ip",
    "mode":"tcp_and_udp",
    "server_port":your_server_port,
    "local_port":1080,
    "password":"your_server_password",
    "timeout":300,
    "method":"your_crypt_method"
}
```


### Enable shadowsocks-libev
```
# ee /etc/rc.conf
```
Add the following code to it.
```
shadowsocks_libev_enable="YES"
```
### Run as server
Start the Shadowsocks server:
```
service shadowsocks_libev start
```
Stop shadowsocks-libev
```
service shadowsocks_libev stop
```
### Run as client
By default, shadowsocks-libev is running as a server in FreeBSD. If you would like to start shadowsocks-libev in client mode,
you need to do as follows.
```
# ee `/usr/local/etc/rc.d/shadowsocks_libev`
```
Replace `ss-server` with `ss-local`.
```
# command="/usr/local/bin/ss-server"
command="/usr/local/bin/ss-local"
```
Start shadowsocks-libev
```
service shadowsocks_libev start
```
Stop shadowsocks-libev
```
service shadowsocks_libev stop
```
