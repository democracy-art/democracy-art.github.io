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

参考:[Project V](v2ray.com)

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
## 1.7 v2ray客户端

- [v2rayN](https://github.com/2dust/v2rayN)是一个基于 V2Ray 内核的 Windows 客户端。
- Pepi 是一个兼容 V2Ray 的 iOS 应用，它可以创建基于 VMess 的 VPN 连接，并与 V2Ray 服务器通信。
- [BifrostV](https://play.google.com/store/apps/details?id=com.github.dawndiy.bifrostv)是一个基于 V2Ray 内核的 Android 应用，它支持 VMess、Shadowsocks、Socks 协议。
- [V2RayNG](https://github.com/2dust/v2rayNG)是一个基于 V2Ray 内核的 Android 应用，它可以创建基于 VMess 的 VPN 连接。

**到这里为止,v2ray已经可以用了,如果还要加上 CDN 请往下看.** 

# 2.注册域名

申请域名注册可以去 `freenom` 网站(有`免费试用1年`的域名).<br>
注册 freenom 账号,然后注册域名.步骤如下:<br>
```
登录freenom主页
->Services标签
->Register a New Domain
-> 在输入框(显示Find your new Domain的输入框)中输入域名(自己喜欢的随意几个字符) 
->点击Check Availability(查看域名是否可用)
->如果域名可用它会出现 .tk, .ml, .ga, .cf, .gq 结尾的5个免费的域名在最上面5行,选一个喜欢的点击 Get it now 
->它会跳转到另一个页面,在 Period 下面,点击下拉框 选择 12 Months@FREE(12个月免费) 
->点击 Continue(继续)
->Review & Checkout页面 输入一个有效的邮箱用来接收信息 
->Your Details 页面,填写信息随便填就好,只要能通过,只要上一步的邮箱是有效的就行,填完同意协议,点击 Complete Order 
->完成后注意查收邮件,它会通知你的域名是否生效,或者登录freenom来到主页面,点击 Services标签
->My Domains 
->看到刚注册的域名 Status是 ACTIVE 说明注册成功
```

# 3.设置DNS

freenom申请的域名默认使用freenom的DNS解析,现在要将其改为 Cloudflare 的 DNS.<br>


## 3.1 获取Cloudflare的DNS

注册 Cloudflare 账号,登录 Cloudflare.进行如下步骤:<br>
```
登录 Cloudflare主页
->点击 +Add site 标签
->输入你在freenom注册的域名
->来到另一个页面 显示We're querying your DNS records 点击 Next
->来到另一个页面 Select a Plan 选择 FREE $0/month 点击 Confire Plan
-> Add more DNS records for xxx.kt 
(这里假设你申请的域名为 xxx.tk ) 假设你的域名是 xxx.tk 下面都以它为例子 
添加记录如下: A www xxx.tk AutomaticTTL 
点击 Add Record A @ xxx.tk AutomaticTTL 点击 Add Record
-> 点击刚添加的两条记录的橘黄小云朵的,点击之后它会变为灰色
-> 点击”Continue”继续
-> 然后会来的一个页面显示你当前域名使用的DNS服务器和Cloudflare要你使用的它的DNS服务器 
复制且保存 Replace with Cloudflare’s nameservers:
下面的两条记录 Nameserver 1 和 Nameserver 2 一般形式是:
xxx.ns.cloudflare.com
yyy.ns.cloudflare.com
->复制保存好后 点击 Done,check nameservers
```

## 3.2 设置解析域名的DNS

3.2.1 将原来freenom提供的DNS服务器替换为Cloudflare的DNS服务器<br>
```
登录freenom主页
->Services标签
->My Domains
->Manage Domain
->Management Tools
->Nameservers 
->Use custom nameservers (enter below)粘贴你保存的两个cloudflare的DNS地址即,
Nameserver 1: xxx.ns.cloudflare.com 
Nameserver 2: yyy.ns.cloudflare.com 然后点击 Change Nameservers 
->这样之后它会在24小时之内帮你跟新好DNS服务器,一般用不到24小时,大概10分钟左右可以了
之后你登录 Cloudflare 就可以看到 DNS 更新了为 xxx.ns.cloudflare.com 和 yyy.ns.cloudflare.com 
同时它也会发送邮件跟你说.
```

3.2.2 设置SSL/TLS:<br>
点击域名 xxx.tk -> SSL/TLS -> Flexible <br>
**当nginx设置好且申请了证书后,把Flexible设置为Full或Full strict**

3.2.3 设置缓存加速:<br>
Cloudflare主页,标签 Caching -> Browser Cache TTL -> 1 year <br>

## 3.3 查看DNS更新是否成功
登录cloudflare主页 你会看到你的域名 xxx.tk 下面的 ACTIVE 打勾 说明成功了

# 4.安装 Nginx

安装:
```
sudo apt install nginx
```
设置开机启动且启动Nginx服务:
```
sudo systemctl start nginx.service
sudo systemctl enable nginx.service
```
# 5.创建伪站点

5.1 创建伪站点目录:
```
rm -rf /home/wwwroot && mkdir -p /home/wwwroot && cd /home/wwwroot
```
5.2 安装git:
```
sudo apt install git
```
5.3 获取伪站点:
```
git clone https://github.com/dm116/sCalc.git
```
5.4 返回 `/root` 目录
```
cd /root
```
# 6.获取HTTPS证书

6.1 acme.sh 实现了 acme 协议, 可以从 letsencrypt 生成免费的证书:
```
curl  https://get.acme.sh | sh
```
6.2 使生效:
```
source ~/.bashrc
```
6.3 Key和Email<br>
```
登录Cloudflare主页
->最右上角小用户头像
->MyProfile
->API Tokens
->API Keys里Global API Key的View
->输入Cloudflarez密码和验证码
->复制Key
```
假设得到的Key为:
```
9m15DA8R!rkmHN4MHdDjb772F$#aGR$1YXZ1M
```
则:
```
export CF_Key="9m15DA8R!rkmHN4MHdDjb772F$#aGR$1YXZ1M"
```
邮箱就是注册Cloudflare的邮箱
```
export CF_Email="xyz@test.com"
```
6.4 利用脚本`acme.sh`配置<br>
```
~/.acme.sh/acme.sh --issue --dns dns_cf -d xxx.tk -d *.xxx.tk -k ec-256
~/.acme.sh/acme.sh --installcert -d xxx.tk -d *.xxx.tk --fullchainpath /etc/v2ray/xxx.tk.crt --keypath /etc/v2ray/xxx.tk.key --ecc
```

6.5 设置定期自动更新
```
~/.acme.sh/acme.sh --upgrade --auto-upgrade
```
# 7.再配置v2ray
```
sudo vi /etc/v2ray/config.json
```
配置如下:
```
{
  "inbounds": [
    {
      "port": 10086,//端口保持跟nginx的default.conf里面端口一致
      "listen": "127.0.0.1",
      "tag": "vmess-in",
      "protocol": "vmess",
      "settings": {
        "clients": [
          {
            "id": "0e658508-bbf1-4655-995f-2c00543ce3d4",
            "alterId": 64
          }
        ]
      },
      "streamSettings": {
        "network": "ws",
        "wsSettings": {
          "path": "/yf321" //保持跟nginx的default.conf里面location一致
        }
      }
    }
  ],
  "outbounds": [
    {
      "protocol": "freedom",
      "settings": {},
      "tag": "direct"
    },
    {
      "protocol": "blackhole",
      "settings": {},
      "tag": "blocked"
    }
  ],
  "routing": {
    "domainStrategy": "AsIs",
    "rules": [
      {
        "type": "field",
        "inboundTag": [
          "vmess-in"
        ],
        "outboundTag": "direct"
      }
    ]
  }
}
```

# 8 Nginx配置文件
```
sudo vi /etc/nginx/conf.d/default.conf
```
配置如下:
```
server {
 listen 443 ssl;
 ssl on;
 ssl_certificate       /etc/v2ray/xxx.tk.crt;
 ssl_certificate_key   /etc/v2ray/xxx.tk.key;
 ssl_protocols         TLSv1 TLSv1.1 TLSv1.2 TLSv1.3; 
 ssl_ciphers           HIGH:!aNULL:!MD5;
 ssl_prefer_server_ciphers on;
 ssl_session_cache shared:SSL:10m;
 ssl_session_timeout 10m;
 server_name www.xxx.tk;
 index index.html index.htm;
 root  /home/wwwroot/sCalc;
 error_page 400 = /400.html ;
 location /yf321  #保持跟v2ray的config.json里面path一致
 { 
     proxy_redirect off;
     proxy_pass http://127.0.0.1:10086; #保持跟v2ray的config.json里面port一致
     proxy_http_version 1.1;
     proxy_set_header Upgrade $http_upgrade;
     proxy_set_header Connection "upgrade";
     proxy_set_header Host $http_host;
 }
 add_header Strict-Transport-Security "max-age=31536000; includeSubDomains" always;
}
# 配置 80 重定向 443 强制 SSL
server {
 listen 80;
 server_name xxx.tk;
 return 301 https://xxx.tk$request_uri;
}
```
# 9.TLS优化

9.1 TLS开启OCSP
```
openssl s_client -connect xxx.tk:443 -status -tlsextdebug < /dev/null 2>&1 | grep -i "OCSP response"
```
9.2 开启TCP fastopen
```
echo 3 > /proc/sys/net/ipv4/tcp_fastopen
```
9.3 使能优化<br>
重启Nginx和v2ray:
```
sudo systemctl restart nginx
sudo systemctl restart v2ray
```
# 10.设置代理和TLS
Proxy:
```
登录Cloudflare
->点击域名 xxx.tk 
->DNS 
->点击两条记录的灰色小云朵,变为橘黄
```
TLS:
```
登录Cloudflare
->点击域名 xxx.tk 
->SSL/TLS 
->Full strict
```
# 11.v2rayN Windows客户端
配置如下:

- 地址(address):`www.xxx.tk` //这里写你申请的域名
- 端口(port):`443`
- 用户(ID): `0e658508-bbf1-4655-995f-2c00543ce3d4` //客户端生成,拷贝到服务端,保存两边一致
- 额外ID(alterId):`64` //不能大于服务器上面的alterId的值
- 加密方式(security):`auto`
- 传输协议(network):`ws`
- 别名(remarks): 随便填都行
- 伪装域名(host):不需要填
- 路径(path):`/yf321` //与服务器保持一致
- 底层传输安全:`tls`
- allowInsecure:`false`

# 12.v2rayy速度优化

12.1 可用全局代理<br>

12.2 设置Cloudflare防火墙参考:[websocket+tls+nginx+cdn断流严重 #1742](https://github.com/v2ray/v2ray-core/issues/1742)<br>
 
Firewall -> Firewall Rules -> Create a Firewall rule:<br>
- Field:`URI Path`, Operator:`equals`, Value:`/yf321`(与v2ray服务的Path一致)
- Firewall rules里面添加一条Action为Allow规则, Field选IP(填你自己的IP),**只适合固定IP**
- 在Tools里面的IP Access Rules添加一条你自己的IP选Whitelist,**只适合固定IP**
- 还可以直接把settings里面的Security Level调成Essentially Off,这样会有**被墙**的风险,不建议


