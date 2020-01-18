---
layout:     post
title:      Centos7 yum源安装shadowsocks-libev(非最新版本)
subtitle:   
date:       2020-01-18
author:     D
header-img: 
catalog: true
mermaid: true
tags:
    - shadowsocks
---

### 1. 添加yum源
```
vi /etc/yum.repos.d/CentOS-Base.repo
```
添加内容如下:
```
[copr:copr.fedorainfracloud.org:librehat:shadowsocks]
name=Copr repo for shadowsocks owned by librehat
baseurl=https://copr-be.cloud.fedoraproject.org/results/librehat/shadowsocks/epel-7-$basearch/
type=rpm-md
skip_if_unavailable=True
gpgcheck=1
gpgkey=https://copr-be.cloud.fedoraproject.org/results/librehat/shadowsocks/pubkey.gpg
repo_gpgcheck=0
enabled=1
enabled_metadata=1
```

### 2. 安装
```
yum -y install shadowsocks-libev
```

### 3. 配置
```
vi /etc/shadowsocks-libev/config.json
```
根据自己需要修改配置
```
{
    "server":"192.168.1.86", 
    "server_port":10086,
    "local_port":1080,
    "password":"ab10086K",
    "timeout":60,
    "method":"chacha20-ietf-poly1305"
}
```
server:服务器地址
server_port:服务器端口
local_port:本地端口
password:密码
timeout:超时时间
method:加密方法

### 4. 设置开机启动
```
systemctl enable shadowsocks-libev
systemctl start shadowsocks-libev
systemctl status shadowsocks-libev
```
如果对配置有任何更改，只要重启下服务就行了，不用重启服务器，命令如下：
```
systemctl restart shadowsocks-libev
```

### 5. 防火墙设置
开启防火墙
```
systemctl start firewalld
```
查看状态
```
systemctl status firewalld
```
开机启用
```
systemctl enable firewalld
```
开启端口
```
firewall-cmd --permanent --add-port={PORT/tcp,PORT/udp}
```
其中 PORT/tcp 和 PORT/udp 中 `PORT` 要替换成自己想要开设的端口号，比如我要开设 tcp 的 80 端口
和 udp 的 443 端口，那么就设置为:
```
firewall-cmd --permanent --add-port={80/tcp, 443/udp}
```

```
firewall-cmd --reload
firewall-cmd --list-all
```

删除已开通端口
```
firewall-cmd --permanent --remove-port=80/tcp 
```


### 7. 客户端连接

在客户端上输入之前设置的IP、密码、等数据，连接服务器
