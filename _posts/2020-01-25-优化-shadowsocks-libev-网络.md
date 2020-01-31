---
layout:     post
title:      优化 shadowsocks-libev 网络
subtitle:   
date:       2020-01-31
author:     D
header-img: 
catalog: true
mermaid: true
tags:
    - shadowsocks
---

参考:[优化 shadowsocks-libev 网络](https://www.24kplus.com/linux/624.html)<br>

# 一、优化吞吐量
1、新建配置文件：<br>
```
sudo vi /etc/sysctl.d/local.conf
```
复制粘贴：
```
#max open files
fs.file-max = 51200
#max read buffer
net.core.rmem_max = 67108864
#max write buffer
net.core.wmem_max = 67108864
#default read buffer
net.core.rmem_default = 65536
#default write buffer
net.core.wmem_default = 65536
#max processor input queue
net.core.netdev_max_backlog = 4096
#max backlog
net.core.somaxconn = 4096
#resist SYN flood attacks
net.ipv4.tcp_syncookies = 1
#reuse timewait sockets when safe
net.ipv4.tcp_tw_reuse = 1
#turn off fast timewait sockets recycling
net.ipv4.tcp_tw_recycle = 0
#short FIN timeout
net.ipv4.tcp_fin_timeout = 30
#short keepalive time
net.ipv4.tcp_keepalive_time = 1200
#outbound port range
net.ipv4.ip_local_port_range = 10000 65000
#max SYN backlog
net.ipv4.tcp_max_syn_backlog = 4096
#max timewait sockets held by system simultaneously
net.ipv4.tcp_max_tw_buckets = 5000
#turn on TCP Fast Open on both client and server side
net.ipv4.tcp_fastopen = 3
#TCP receive buffer
net.ipv4.tcp_rmem = 4096 87380 67108864
#TCP write buffer
net.ipv4.tcp_wmem = 4096 65536 67108864
#turn on path MTU discovery
net.ipv4.tcp_mtu_probing = 1

net.ipv4.tcp_congestion_control = bbr
```
2、运行：<br>
```
sysctl --system
```
3、编辑配置文件limits.conf
```
sudo vi /etc/security/limits.conf
```
在文件结尾添加两行：
```
* soft nofile 51200
* hard nofile 51200
```
4、编辑 shadowsocks-libev 服务
```
# 服务配置文件名称视具体而定
sudo vi /etc/systemd/system/shadowsocks-libev.service
```
在[Service]之后加入 `ExecStartPre=/bin/sh -c ‘ulimit -n 51200’`
```
[Unit]
Description=Shadowsocks-libev Server
After=network.target

[Service]
Type=simple
# 服务配置可能有所不一样，视实际而定
# 在这里加入 ExecStartPre=/bin/sh -c 'ulimit -n 51200'
ExecStartPre=/bin/sh -c 'ulimit -n 51200'
ExecStart=/usr/local/bin/ss-server -c /etc/shadowsocks-libev/config.json -u
Restart=on-abort

[Install]
WantedBy=multi-user.target
```
5、重新加载 shadowsocks-libev 服务配置
```
sudo systemctl daemon-reload
```
6、重启 Shadowsocks-libev 服务
```
sudo systemctl restart shadowsocks-libev
```

# 二、开启TCP Fast Open

TCP Fast Open可以降低Shadowsocks服务器和客户端的延迟。<br>
实际上在上一步已经开启了TCP Fast Open，现在只需要在Shadowsocks配置中启用TCP Fast Open。<br>

1、编辑config.json：<br>
```
sudo vi /etc/shadowsocks-libev/config.json
```
将 fast_open 的值由 false 修改为 true
```
{
     "server":"0.0.0.0",
     "server_port":8388,
     "local_port":1080,
     "password":"mypassword",
     "timeout":600,
     "method":"aes-256-gcm",
     /*这里设置 fast_open：true，如果没有则加入*/
     "fast_open": true
 } 
```
2、重启 shadowsocks-libev 服务：
```
sudo systemctl restart shadowsocks-libev
```
# 三、开启 Google BBR
[Google BBR](https://www.24kplus.com/linux/150.html)
优化到此基本完成。
