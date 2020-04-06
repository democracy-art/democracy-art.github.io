---
layout: post
title: Install v2ray in binary packages
subtitle:
date: 2020-03-27
author: D
header-img:
catalog: true
tags: [v2ray]
---

下载: [v2ray-core](https://github.com/v2ray/v2ray-core/releases)

以下文件,放到相应位置,如果没有目录便创建：
- `/usr/bin/v2ray/v2ray`：V2Ray 程序；
- `/usr/bin/v2ray/v2ctl`：V2Ray 工具；
- `/etc/v2ray/config.json`：配置文件；
- `/usr/bin/v2ray/geoip.dat`：IP 数据文件
- `/usr/bin/v2ray/geosite.dat`：域名数据文件

命令:
```
mkdir -p  /usr/bin/v2ray/
mkdir -p /etc/v2ray/

cp v2ray /usr/bin/v2ray/
cp v2ctl /usr/bin/v2ray/
cp geoip.dat /usr/bin/v2ray/
cp geosite.dat /usr/bin/v2ray/

cp config.json /etc/v2ray/
```
运行脚本:
```
vim /etc/systemd/system/v2ray.service
```
v2ray.service内容如下:
```
[Unit]
Description=V2Ray Service
After=network.target
Wants=network.target

[Service]
# This service runs as root. You may consider to run it as another user for security concerns.
# By uncommenting the following two lines, this service will run as user v2ray/v2ray.
# More discussion at https://github.com/v2ray/v2ray-core/issues/1011
# User=v2ray
# Group=v2ray
Type=simple
PIDFile=/run/v2ray.pid
ExecStart=/usr/bin/v2ray/v2ray -config /etc/v2ray/config.json
Restart=on-failure
# Don't restart in the case of configuration error
RestartPreventExitStatus=23

[Install]
WantedBy=multi-user.target
```
使能并重启v2ray:
```
systemctl enable v2ray.service
systemctl restart v2ray
```
