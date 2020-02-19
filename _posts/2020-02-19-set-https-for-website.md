---
layout:     post
title:      Ubuntu18.04安装带有Nginx,MariaDB10和PHP7的WordPress(2) - 配置HTTPS
subtitle:  
date:       2020-02-19
author:     D
header-img: 
catalog: true
mermaid: true
tags:
    - website
---

**前提：以下域名全假设为 `xxx.tk`，请将`xxx.tk`改为自己的域名**

# 1.从letsencrypt生成免费的证书

```
sudo apt install curl
curl  https://get.acme.sh | sh
```
使脚本生效:
```
source ~/.bashrc
```
# 2.设置Key和Email为环境变量:
```
->登录Cloudflare
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
邮箱就是注册Cloudflare的邮箱:
```
export CF_Email="xyz@gmail.com"
```
# 3.acme.sh 配置
利用脚本acme.sh配置
```
~/.acme.sh/acme.sh --issue --dns dns_cf -d xxx.tk -d *.xxx.tk -k ec-256
```
```
~/.acme.sh/acme.sh --installcert -d xxx.tk -d *.xxx.tk --fullchainpath /etc/ssl/certs/xxx.tk.crt --keypath /etc/ssl/certs/xxx.tk.key --ecc
```
设置定期自动更新
```
~/.acme.sh/acme.sh --upgrade --auto-upgrade
```
# 4.配置修改
```
sudo vi /etc/nginx/sites-available/wordpress.conf
```
```
server {
 listen 443 ssl;
 listen [::]:443 ssl;
 root  /home/wwwroot/html/wordpress;
 index index.php index.html index.htm;
 server_name xxx.tk www.xxx.tk;

 client_max_body_size 100M;

 location / {
	try_files $uri $uri/ /index.php?$args;        
 }

 location ~ \.php$ {
	include snippets/fastcgi-php.conf;
	fastcgi_pass             unix:/var/run/php/php7.2-fpm.sock;
	fastcgi_param   SCRIPT_FILENAME $document_root$fastcgi_script_name;
 }

 ssl on;
 ssl_certificate       /etc/ssl/certs/xxx.tk.crt;
 ssl_certificate_key   /etc/ssl/certs/xxx.tk.key;
 ssl_protocols         TLSv1 TLSv1.1 TLSv1.2 TLSv1.3; 
 ssl_ciphers           HIGH:!aNULL:!MD5;
 ssl_prefer_server_ciphers on;
 ssl_session_cache shared:SSL:10m;
 ssl_session_timeout 10m;
 add_header Strict-Transport-Security "max-age=31536000; includeSubDomains" always;
}
# 配置 80 重定向 443 强制 SSL
server {
 listen 80;
 listen [::]:80;
 server_name xxx.tk www.xxx.tk;
 return 301 https://xxx.tk$request_uri;
}
```

# 5.TLS 优化

TLS开启OCSP
```
openssl s_client -connect xxx.tk:443 -status -tlsextdebug < /dev/null 2>&1 | grep -i "OCSP response"
```
开启TCP fastopen
```
echo 3 > /proc/sys/net/ipv4/tcp_fastopen
```
重启Nginx使优化生效
```
sudo systemctl restart nginx
```

# 6.设置代理和TLS

代理:

- 登录 Cloudflare 主页
- 点击域名 `xxx.tk` 
- DNS 
- 点击两朵灰色小云朵,变为橘黄,即Proxy status从 `DNS only` 变 `Proxied`

TLS

- 登录 Cloudflare 主页
- 点击域名 `xxx.tk`
- `SSL/TLS`
- `Full strict`

