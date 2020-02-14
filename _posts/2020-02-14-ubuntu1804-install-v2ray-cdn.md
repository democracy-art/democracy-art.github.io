---
layout:     post
title:      Ubuntu18.04安装v2ray(用CDN避免被墙)
subtitle:
date:       2020-02-14
author:     D
header-img: 
catalog: true
mermaid: true
tags:
    - v2ray
---

# 1. v2ray

## 1.1 配置Linux环境

1.1.1 更新 apt
```
sudo apt update && sudo apt -y upgrade
```

1.1.2 设定时区(v2ray 必须将时间误差控制在 `90` 秒以內)
```
sudo timedatectl set-timezone Asia/Shanghai
```

## 1.2 升级权限
```
sudo su
```

## 1.3 安装v2ray
安装curl:<br>
```
sudo apt install curl
```
apt-get 可用的情况下，此脚本会自动安装 unzip 和 daemon。这两个组件是安装 V2Ray 的必要组件。<br>
如果你使用的系统不支持 yum 或 apt-get，请自行安装 unzip 和 daemon.<br>
```
bash <(curl -L -s https://install.direct/go.sh)
```


1.3.1 脚本详细情况(可选)<br>
此脚本会自动安装以下文件:<br>

- `/usr/bin/v2ray/v2ray`: v2ray程序
- `/usr/bin/v2ray/v2ctl`: V2Ray 工具
- `/etc/v2ray/config.json`: 配置文件
- `/usr/bin/v2ray/geoip.dat`: IP 数据文件
- `/usr/bin/v2ray/geosite.dat`: 域名数据文件

此脚本会配置自动运行脚本。自动运行脚本会在系统重启之后，自动运行 V2Ray.<br>
目前自动运行脚本只支持带有 `Systemd` 的系统，以及 `Debian / Ubuntu` 全系列。<br>

运行脚本位于系统的以下位置：<br>
- `/etc/systemd/system/v2ray.service`:Systemd
- `/etc/init.d/v2ray`:SysV

1.3.2 go.sh 参数(可选)<br>
- `-p` 或 `--proxy`: 使用代理服务器来下载 V2Ray 的文件，格式与 curl 接受的参数一致，比如 `"socks5://127.0.0.1:1080"` 或 `"http://127.0.0.1:3128"`
- `-f` 或 `--force`: 强制安装。在默认情况下，如果当前系统中已有最新版本的 V2Ray，go.sh 会在检测之后就退出。如果需要强制重装一遍，则需要指定该参数。
- `--version`: 指定需要安装的版本，比如 `"v1.13"`。默认值为最新版本。
- `--local`: 使用一个本地文件进行安装。如果你已经下载了某个版本的 V2Ray，则可通过这个参数指定一个文件路径来进行安装。

示列:<br>
- 使用地址为 127.0.0.1:1080 的 SOCKS 代理下载并安装最新版本：`./go.sh -p socks5://127.0.0.1:1080`
- 安装本地的 v1.13 版本：`./go.sh --version v1.13 --local /path/to/v2ray.zip`


## 1.4 编辑配置文件

**服务端**:<br>
```
sudo vi /etc/v2ray/config.json
```

```
{
  "inbounds": [
    {
      "port": 10086,// 服务器监听端口，必须和客户端的一样
      "protocol": "vmess",
      "settings": {
        "clients": [
          {
            "id": "b831381d-6324-4d53-ad4f-8cda48b30811"
          }
        ]
      }
    }
  ],
  "outbounds": [
    {
      "protocol": "freedom",
      "settings": {}
    }
  ]
}
```

**客户端**:<br>
```
{
  "inbounds": [
    {
      "port": 1080,// SOCKS 代理端口，在浏览器中需配置代理并指向这个端口
      "listen": "127.0.0.1",
      "protocol": "socks",
      "settings": {
        "udp": true
      }
    }
  ],
  "outbounds": [
    {
      "protocol": "vmess",
      "settings": {
        "vnext": [
          {
            "address": "server",// 服务器地址，请修改为你自己的服务器 ip 或域名
            "port": 10086,// 服务器端口
            "users": [
              {
                "id": "b831381d-6324-4d53-ad4f-8cda48b30811"
              }
            ]
          }
        ]
      }
    },
    {
      "protocol": "freedom",
      "tag": "direct",
      "settings": {}
    }
  ],
  "routing": {
    "domainStrategy": "IPOnDemand",
    "rules": [
      {
        "type": "field",
        "ip": [
          "geoip:private"
        ],
        "outboundTag": "direct"
      }
    ]
  }
}
```

## 1.5 运行 v2ray
设置开机启动且启动服务:<br>
```
sudo systemctl enable v2ray && sudo systemctl start v2ray 
```

## 1.6 设置防火墙
请根据情况调整端口号:<br>
```
sudo apt-get install ufw
sudo ufw enable
sudo ufw allow 10086/tcp
```

**到这里为止,v2ray已经可以用了,如果还要加上 CDN 请往下看.** 


