---
layout:     post
title:      Kali2019上安装shadowsocks-libev客户端
subtitle:   
date:       2020-01-07
author:     D
header-img: 
catalog: true
tags:
    - shadowsocks
---

更新apt
```
sudo apt-get update
```
安装 shadowsocks-libev
```
sudo apt-get install shadowsocks-libev
``` 
安装完成后,配置config.json
```
sudo vi  /etc/shadowsocks-libev/config.json
```
配置如下:
```
{
    "server":"server_ip",
    "mode":"tcp_and_udp",
    "server_port":8888,
    "local_port":1080,
    "password":"server_password",
    "timeout":60,
    "method":"aes-256-gcm"
}
```
`server_ip`, `server_port` , `password` 和 `method` 必须和服务端的保持`一致`.
启动shadowsocks-libev客户端
```
sudo ss-local -c /etc/shadowsocks-libev/config.json 
```
设置开机启动 shadowsocks-libev客户端
```
sudo cp /lib/systemd/system/shadowsocks-libev-local@.service  /lib/systemd/system/shadowsocks-libev-local.service
```
```
sudo vi /lib/systemd/system/shadowsocks-libev-local.service
```
修改配置如下:
```
#ExecStart=/usr/bin/ss-local -c /etc/shadowsocks-libev/%i.json
ExecStart=/usr/bin/ss-local -c /etc/shadowsocks-libev/config.json 
```
使能 shadowsocks-libev.service
```
sudo systemctl enable shadowsocks-libev-local.service
```
启动shadowsocks-libev客户端
```
sudo systemctl start shadowsocks-libev-local.service
```
查看是否自动启动shadowsocks-libev
```
sudo systemctl status shadowsocks-libev-local.service
```
>Active: active (running)  正常启动<br>
Active: inactive (dead)    没有启动


配置firefox浏览器:
Open-menu --> Preferences --> General --> Network Settings-->点击 Settings --> Manual proxy configuration
- HTTP Proxy: 127.0.0.1  Port: 1080
- [x] Use this proxy server for all protocols

点击 `OK` 按钮

