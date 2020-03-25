---
layout: post
title: v2ray-plugin with shadowsocks-libev
subtitle:
date: 2020-03-25
author: D
header-img:
catalog: true
tags: [shadowsocks,v2ray-plugin]
---

# 1. Prepare

Your VPS system had install **v2ray+nginx+cdn**. 

References:<br>
[Ubuntu18.04安装v2ray(用CDN避免被墙)](https://dm116.github.io/2020/02/14/ubuntu1804-install-v2ray-cdn/)<br>
[Ubuntu18.04安装带有Nginx,MariaDB10和PHP7的WordPress且使用Cloudflare的CDN/SSL](https://dm116.github.io/2020/02/14/ubuntu1804-nginx-mariadb10-php7-wordpress/)<br>

# 2. Install shadowsocks-libev

Reference:<br>
[Ubuntu 18.04 安装 shadowsocks-libev 服务端(支持多用户)](https://dm116.github.io/2020/01/31/Ubuntu-18.04-install-shadowsocks-libev-server/)<br>

# 3. v2ray-plugin download.

Download:[v2ray-plugin](https://github.com/shadowsocks/v2ray-plugin/releases)<br>

# 4. Configuration

### 4.1 nginx (server side)
Here is an example configuration for nginx:
```
server {
        listen       443 ssl http2;
        listen       [::]:443 ssl http2;
        server_name  example.com;     # Your domain.
        root         /usr/share/nginx/html/;
        ssl_certificate "/path/to/cert";     # Path to certificate
        ssl_certificate_key "/path/to/key";     # Path to private key
        ssl_session_cache shared:SSL:1m;
        ssl_session_timeout  10m;
        ssl_ciphers HIGH:!aNULL:!MD5;
        ssl_prefer_server_ciphers on;

        location /ss {
	    access_log 	off;
            proxy_redirect off;
            proxy_http_version 1.1;
            proxy_pass http://localhost:8008;     # Port of v2ray-plugin
            proxy_set_header Host $http_host;
            proxy_set_header Upgrade $http_upgrade;
            proxy_set_header Connection "upgrade";
        }
}
```
### 4.2 shadowsocks-libev (server side)
First you should put software `v2ray-plugin` inot directory `/usr/bin/`.<br>
The corresponding configuration for shadowsocks-libev with v2ray-plugin.
```
{
    "server":"localhost",
    "server_port":8008,
    "password":"password",
    "timeout":300,
    "method":"aes-256-gcm",
    "plugin":"/usr/bin/v2ray-plugin",
    "plugin_opts":"server;path=/ss/;loglevel=none"
}
```
### 4.3 shadowsocks-libev (client)
#### 4.3.1 linux client config:
First you should put software `v2ray-plugin` inot directory `/usr/bin/`.
```
{
    "server": "example.com",
    "server_port": 443,
    "password": "password",
    "method": "aes-256-gcm",
    "local_address": "0.0.0.0",
    "plugin": "/usr/bin/v2ray-plugin",
    "plugin_opts": "tls;host=example.com;path=/ss/;loglevel=none",
    "timeout": 300,
    "reuse_port": true
}

```
#### 4.3.2 windows client config:
First you should put software `v2ray-plugin` and `shadowsocks` into the same directory.
```
{
	"server":"example.com",
	"server_port":443,
	"password": "password",
	"encryption":"aes-256-gcm",
	"plugin progam":"v2ray-plugin",
	"plugin options":"tls;host=example.com;path=/ss/;loglevel=none"
	"timeout":300
}
```
References:<br>
[Use v2ray-plugin after Nginx #48](https://github.com/shadowsocks/v2ray-plugin/issues/48)
