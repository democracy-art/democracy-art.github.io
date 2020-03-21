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
 apt update &&  apt -y upgrade
```

1.1.2 设定时区(v2ray 必须将时间误差控制在 `90` 秒以內)
```
 timedatectl set-timezone Asia/Shanghai
```

## 1.2 升级权限
```
 su
```

## 1.3 安装v2ray
安装curl:<br>
```
 apt install curl
```
apt-get 可用的情况下，此脚本会自动安装 unzip 和 daemon。这两个组件是安装 V2Ray 的必要组件。<br>
如果你使用的系统不支持 yum 或 apt-get，请自行安装 unzip 和 daemon.<br>
```
bash <(curl -L -s https://install.direct/go.sh)
```

## 1.4 运行 v2ray
设置开机启动且启动服务:<br>
```
 systemctl enable v2ray &&  systemctl start v2ray 
```

## 1.5 设置防火墙
请根据情况调整端口号:<br>
```
 apt-get install ufw
 ufw enable
 ufw allow 10086/tcp
```
## 1.6 v2ray客户端

- [v2rayN](https://github.com/2dust/v2rayN)是一个基于 V2Ray 内核的 Windows 客户端。
- [Other Client tools](http://v2ray.com/awesome/tools.html)

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
 apt install nginx
```
设置开机启动且启动Nginx服务:
```
 systemctl start nginx.service
 systemctl enable nginx.service
```
# 5.创建伪站点

5.1 创建伪站点目录:
```
rm -rf /home/wwwroot && mkdir -p /home/wwwroot && cd /home/wwwroot
```
5.2 安装git:
```
 apt install git
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

## 方法一
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
## 方法二
请参考:[How To Setup WordPress With Nginx And Cloudflare CDN / SSL On Ubuntu 16.04 | 18.04](https://websiteforstudents.com/how-to-setup-wordpress-with-nginx-and-cloudflare-cdn-ssl-on-ubuntu-16-04-18-04/)
# 7.再配置v2ray
```
 vi /etc/v2ray/config.json
```
配置如下:
```
{
  "inbound": {
    "port": 9000, //(此端口与nginx配置相关)
    "listen": "127.0.0.1",
    "protocol": "vmess",
    "settings": {
      "clients": [
        {
          "id": "eb950add-608e-409d-937f-e797324387093z", //你的UUID， 此ID需与客户端保持一致
          "level": 1,
          "alterId": 64 //此ID也需与客户端保持一致
        }
      ]
    },
   "streamSettings":{
      "network": "ws",
      "wsSettings": {
           "path": "/yf321" //与nginx配置相关
      }
   }
  },
  "outbound": {
    "protocol": "freedom",
    "settings": {}
  },
  "outboundDetour": [
    {
      "protocol": "blackhole",
      "settings": {},
      "tag": "blocked"
    }
  ],
  "routing": {
    "strategy": "rules",
    "settings": {
      "rules": [
        {
          "type": "field",
          "ip": [
            "0.0.0.0/8",
            "10.0.0.0/8",
            "100.64.0.0/10",
            "127.0.0.0/8",
            "169.254.0.0/16",
            "172.16.0.0/12",
            "192.0.0.0/24",
            "192.0.2.0/24",
            "192.168.0.0/16",
            "198.18.0.0/15",
            "198.51.100.0/24",
            "203.0.113.0/24",
            "::1/128",
            "fc00::/7",
            "fe80::/10"
          ],
          "outboundTag": "blocked"
        }
      ]
    }
  }
}
```

# 8 Nginx配置文件
```
 vi /etc/nginx/conf.d/default.conf
```
配置如下:
```
server {
	listen 443 ssl http2 default_server;
	listen [::]:443 ssl http2 default_server;

	server_name your_domain www.your_domain; #你的服务器域名
	root /var/www/html/your_domain;
	index index.php index.html index.htm;

	ssl_certificate 	/etc/ssl/certs/cloudflare_your_domain; #你的ssl证书， 如果第一次，可能还需要自签一下，
	ssl_certificate_key 	/etc/ssl/private/cloudflare_your_domain;  #你的ssl key
	ssl_protocols		TLSv1 TLSv1.1 TLSv1.2; 	
	ssl_ciphers		HIGH:!aNULL:!MD5:!EXPORT56:!EXP;
	ssl_prefer_server_ciphers on;
	ssl_client_certificate 	/etc/ssl/certs/origin-pull-ca.pem;
	ssl_verify_client on;

	location / {
		try_files $uri $uri/ /index.php?$args;
	}
	
	location /yf321 { #  路径需要和v2ray服务器端，客户端保持一致
		proxy_redirect off;
		proxy_pass http://127.0.0.1:9000;	
		proxy_http_version 1.1;
		proxy_set_header Upgrade $http_upgrade;
		proxy_set_header Connection "upgrade";
		proxy_set_header Host $http_host;
	}
    
	location ~ \.php$ {
		include snippets/fastcgi-php.conf;
		fastcgi_pass unix:/var/run/php/php7.0-fpm.sock; #根据PHP版本进行相应修改
		fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
		include fastcgi_params;
	}
}
```
重启Nginx和v2ray:
```
 systemctl restart nginx
 systemctl restart v2ray
```
# 9.设置代理和TLS
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
# 10.v2rayN Windows客户端
配置如下:

- 地址(address):`你申请的域名` //这里写你申请的域名
- 端口(port):`443`
- 用户(ID): `eb950add-608e-409d-937f-e797324387093z` //客户端生成,拷贝到服务端,保存两边一致
- 额外ID(alterId):`64` //不能大于服务器上面的alterId的值
- 加密方式(security):一般 `aes-128-gcm`
- 传输协议(network):`ws`
- 别名(remarks): v2ray(随便填)
- 伪装域名(host):留空
- 路径(path):`/yf321` //与服务器保持一致
- 底层传输安全:`tls`
- allowInsecure:`true`



