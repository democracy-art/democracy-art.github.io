--- 
layout: post
title: 在V2ray服務器上Nginx的配置
subtitle:
date: 2020-09-18
author: D
header-img:
catalog: true
tags: [v2ray,nginx]
---
v2ray模式vmess+tls+nginx+cdn,網絡的連接流程如下：
>客戶端瀏覽器(443)-->CDN(Cloudflare)轉發-->Nginx再轉發-->v2ray對應的端口和path下-->外網

其中nginx和v2ray都是在同一個vps下面的，爲了安全，nginx需要將http的流量全部都轉爲https.
那麼nginx究竟該如何配置呢?如下:
nginx.conf
```
# 如果這裏定義了用戶是 www 和用戶組是 www 那麼，那麼 /path/to/example 目錄的擁有者和擁有組需要改
# 修改命令 chown www -R /path/to/example 和 chgrp www -R /path/to/example 
user www www;

http{
...
  include /path/to/example.com.conf #包含你改該域名的配置文件 
  server_tokens off; #如果不想人家看到你的nginx版本號可以加這一行
...
}
```
你的域名的配置文件如下,一般域名的配置文件是單獨分開，這樣一個vps就可以單獨建好幾個站了.假設你的域名是`example.com`,example.com.conf如下:
```
server {
 listen 80;
 server_name example.com www.example.com;
 rewrite ^(.*)$ https://${server_name}$1 permanent; #意思是永久跳轉到443端口
}
server {
 listen 443 ssl;
 ssl_certificate         /path/to/example.com.crt;
 ssl_certificate_key     /path/to/example.com.key;
 
 # Authenticated Origin Pulls 允许源 Web 服务器以加密方式验证 Web 请求是否来自 CDN(Cloudflare)
 # 我们使用大多数 Web 服务器支持的 TLS 客户端证书身份验证功能，并在 Cloudflare 和源站之间建立连接时提供 Cloudflare 证书。 
 # 通过在源站配置中验证此证书，可以将访问限制为 Cloudflare 连接
 ssl_client_certificate  /path/to/origin-pull-ca.pem;
 ssl_verify_client       on;
 
 ssl_protocols         TLSv1 TLSv1.1 TLSv1.2 TLSv1.3; 
 ssl_ciphers           ECDHE-RSA-AES128-GCM-SHA256:ECDHE:ECDH:AES:HIGH:!NULL:!aNULL:!MD5:!ADH:!RC4;
 ssl_prefer_server_ciphers on;
 ssl_session_cache shared:SSL:5m;
 ssl_session_timeout 5m;
 server_name www.example.com example.com;
 server_tokens off; #去掉nginx版本信息

 location / {
   root  /path/to/example.com;
   index index.html index.htm index.php;
 }

 location / path_to_v2ray 
 { 
     proxy_redirect off;
     proxy_pass http://127.0.0.1:10086;  // 將10086 改爲v2ray運行用端口 
     proxy_http_version 1.1;
     proxy_set_header Upgrade $http_upgrade;
     proxy_set_header Connection "upgrade";
     proxy_set_header Host $http_host;
  }
}

```
[Authenticated Origin Pulls 解析(中文版)](https://support.cloudflare.com/hc/zh-cn/articles/204899617)<br>
[Authenticated Origin Pulls 解析(英文版)](https://support.cloudflare.com/hc/en-us/articles/204899617-Authenticated-Origin-Pulls)
