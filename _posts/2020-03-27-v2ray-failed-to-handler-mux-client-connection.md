---
layout: post
title: failed to handler mux client connection
subtitle:
date: 2020-03-27
author: D
header-img:
catalog: true
tags: [v2ray]
---

**V2ray client(on Kali2019) connect to server(Debian9) Error Message as following:**
```
failed to handler mux client connection > v2ray.com/core/proxy/vmess/outbound: connection ends > v2ray.com/core/proxy/vmess/outbound: failed to read header > v2ray.com/core/proxy/vmess/encoding: failed to read response header > websocket: close 1000 (normal)
```

**Solution:**
On the client:<br>
1.Time synchronization with `ntp`:
```
apt-get install -y ntp
```
2.Restart service:
```
/etc/init.d/ntp restart
```

