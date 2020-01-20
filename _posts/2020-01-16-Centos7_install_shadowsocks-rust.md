---
layout:     post
title:      Centos7安装shadowsocks-rust
subtitle:   
date:       2020-01-16
author:     D
header-img: 
catalog: true
mermaid: true
tags:
    - shadowsocks
---
# 依赖安装
### 安装Rust
#### 安装runstup
1. 运行如下脚本:
```
curl https://sh.rustup.rs -sSf | sh
```

>1) Proceed with installation (default)
2) Customize installation
3) Cancel installation

2. 输入默认项1, 完成余下的安装

#### 导入环境变量
运行source命令导入变量
```
source $HOME/.cargo/env
```
查看rust版本
```
rustup --version
```

### 安装wget
```
yum install wget
```

# 安装shadowsocks-rust
wget下载最新的release版本
版本地址:
```
https://github.com/shadowsocks/shadowsocks-rust/releases
```
下载现在最新版本：
```
wget https://github.com/shadowsocks/shadowsocks-rust/releases/download/v1.8.7/shadowsocks-v1.8.7-stable.x86_64-unknown-linux-musl.tar.xz
```
解压到 `/usr/local/bin` 目录
```
tar atf shadowsocks-v1.8.7-stable.x86_64-unknown-linux-musl.tar.xz -C /usr/local/bin
```

# 配置shadowsocks-rust
### 配置config.json
以root用户创建目录/etc/shadowsocks-rust，编辑/etc/shadowsocks-rust/config.json：
```
{
    "servers": [
        {
            "address": "127.0.0.1",
            "port": 1080,
            "password": "hello-world",
            "method": "aes-256-cfb"
            "timeout": 300
        },
        {
            "address": "127.0.0.1",
            "port": 1081,
            "password": "hello-kitty",
            "method": "aes-256-cfb"
        }
    ],
    "local_port": 8388,
    "local_address": "127.0.0.1"
}
```
配置文件请根据自己的需求修改.

### 配置shadowsocks-rust服务
以root用户创建/etc/systemd/system/shadowsocks-rust.service:
```
[Unit]
Description=Shadowsocks-Rust Custom Client Service.
Documentation=sslocal -h
After=network.target

[Service]
Type=simple
CapabilityBoundingSet=CAP_NET_BIND_SERVICE
AmbientCapabilities=CAP_NET_BIND_SERVICE
User=nobody
Group=nogroup
ExecStart=/usr/local/bin/sslocal --log-without-time -c /etc/shadowsocks-rust/config.json

[Install]
WantedBy=multi-user.target
```

注册并启动服务：

```
chown -R root:nogroup /etc/shadowsocks-rust
chmod -R g-w,o-rwx /etc/shadowsocks-rust
systemctl daemon-reload
systemctl enable shadowsocks-rust
systemctl start shadowsocks-rust
```

**注意:**
- 为降低sslocal进程权限，以nobody用户和nogroup组运行它。
- 为防止密码泄漏，/etc/shadowsocks-rust/config.json仅root用户或nogroup组可读。
- 限制sslocal进程仅能监听socket.


