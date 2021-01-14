---
title: "V2ray使用Vmess+Websocket+TLS+Nginx+CDN(Cloudflare)例子的相关配置"
date: 2021-01-14
draft: true
categories: [v2ray]
tags: [v2ray,vmess,websocket,TLS,nginx,CDN]
keywords: [v2ray,vmess,websocket,TLS,nginx,CDN,Cloudflare]
description : "V2ray使用Vmess+Websocket+TLS+Nginx+CDN(Cloudflare)例子的相关配置.A example how to deplay v2ray with Vmess+Websocket+TLS+Nginx+CDN(Cloudflare) the config of client and server."
---

**注意:**　这是一个**隐藏**你的服务器真实IP的配置,如果你追求**安全**这是一种非常好的配置,如果你追求的是**速度**那么这种配置**不**适合你.

# v2ray 客户端配置 
```
{
        "log": {
                "loglevel": "warning"
        },

        "dns": {
                "hosts":{
                        "domain:baidu.com":"127.0.0.1" //换成相应的域名的IP
                },
                "servers": [
                        {
                                "address": "https://1.1.1.1/dns_query", //DNS over HTTPS,防DNS污染
                                "domains": [
                                        "domain:google.com",
                                        "domain:youtube.com",
                                        "domain:v2fly.org"
                                ]
                        },
                        "8.8.8.8", // 当它的值是一个DNS IP地址时,v2ray会使用此地址的53端口进行查询,这样会被DNS污染
                        "localhost"//表示本机预设的DNS配置
                ],
                "tag": "dns_inbound"
        },

        "routing": {
                "domainStrategy": "AsIs", //只使用域名进行路由选择(默认值).
                "rules": [
                        {
                                "type": "field",
                                "domain": [
                                        "geosite:cn"                                                                            
                                ],                                                                                              
                                "outboundTag": "direct"                                                                         
                        },
                        {
                                "type": "field",
                                "domain": [
                                        "domain:google.com",
                                        "domain:youtube.com",
                                        "domain:github.com",
                                        "domain:v2fly.org"
                                ],
                                "outboundTag": "proxy"
                        }
                ]
        },

        "inbounds": [{
                "port": 1080,
                "listen": "127.0.0.1",
                "protocol": "socks",
                "settings": {
                        "udp": true
                },
                "sniffing": { //防DNS污染
                        "enabled": true, // 开启流量探测
                        "destOverride": ["http", "tls"] //将http的目标地址重置为https
                }
        }],

        "outbounds": [
                {
                        "tag": "proxy",
                        "protocol":"vmess", //使用VMess协议
                        "settings": {
                                "vnext": [{
                                        "address": "your_domain",  //写你自己的域名
                                        "port": 443,               //HTTPS使用的端口443
                                        "users": [{
                                                "id": "your_uuid", //你自己的UUID
                                                "security": "aes-128-gcm",//"aes-128-gcm"推荐PC用,"chacha20-poly1305"推荐手机端用
                                                "alterId": 64      // 值要<或=服务器端的alterId的值
                                        }]
                                }]
                        },
                        "streamSettings": {
                                "network": "ws",//传输方式使用WebSocket
                                "security": "tls",//传输层使用TLS加密
                                "wsSettings": {
                                        "path": "/your_nginx_path" //这个路径要与nginx的上配置的路径一致
                                }
                        }
                },
                {
                        "protocol": "freedom",
                        "tag": "direct"     
                }
        ]
 }

```
# v2ray 服务端配置
```
{
    "log": {
        "loglevel": "warning"
    },

    "routing": {
        "domainStrategy": "AsIs", //只使用域名进行路由选择,默认值.
        "rules": [
            {
                "type": "field",
                "ip": [
                    "geoip:private" //private包含所有私有地址,如127.0.0.1
                ],
                "outboundTag": "block"//阻塞所有对本机私有地址的访问
            }
        ]
    },

    "inbounds": [
        {
            "listen": "0.0.0.0", 
            "port": 12345, //这里的端口要与nginx的端口一致
            "protocol": "vmess",//使用VMess协议与v2ray客户端的协议一致
            "settings": {
                "clients": [
                    {
                        "id": "your_UUID", //你的UUID要与客户端的UUID一致
                        "alterId": 64 
                    }
                ]
            },
            "streamSettings": {
                "network": "ws", //使用WebSocket传输
                "wsSettings": {
                    "path": "your_nginx_path" //路径要与nginx的路径一致
                }
            }
        }
    ],

    "outbounds": [
        {
            "protocol": "freedom",
            "tag": "direct"
        },
        {
            "protocol": "blackhole",
            "tag": "block"
        }
    ]
}
```
# nginx配置

**注意:** 将example.com换成自己的域名

```
server {
        # Redirect all http requests to https.
        listen 80 default_server;
        listen [::]:80 default_server;
        server_name _;
        return 301 https://$host$request_uri;
}
server {
        listen 443 ssl http2;
        listen [::]:443 ssl http2;
        server_name www.example.com example.com;
        server_tokens off;

        ssl_certificate      /path/to/certs/example.com.crt; //将path/to/certs替换为自己的证书所在路径
        ssl_certificate_key  /path/to/certs/example.com.key;
        ssl_client_certificate /path/to/certs/cloudflare.crt;
        ssl_verify_client on; //开启来路认证,判断是否来自cloudflare的流量

        location / {
                root /path/to/example.com; //换成自己example.com站点内容所在路径
                index index.html index.htm;
                try_files $uri $uri $uri/ =404;
        }

        # This path must be as same as v2ray path
        location /your_nginx_path {
                proxy_redirect off;
                proxy_pass http://127.0.0.1:12345; //与v2ray的端口要一致
                proxy_http_version 1.1;
                proxy_set_header Upgrade $http_upgrade;
                proxy_set_header Connection "upgrade";
                proxy_set_header Host $http_host;
        }
}
```
