--- 
layout: post
title: Configuration of v2ray's client and server (v2ray+ws+tls+nginx+ss+cdn)
subtitle:
date: 2020-04-01
author: D
header-img:
catalog: true
tags: [v2ray,shadowsocks]
---

# Content of v2ray server's config.json
```
{
 "inbound": {
    "port": 10086, //(此端口与nginx配置一样)
    "listen": "127.0.0.1",
    "protocol": "vmess",
    "settings": {
      "clients": [
        {
          "id": "Your UUID ", //你的UUID， 此ID需与客户端保持一致
          "level": 1,
          "alterId": 64 //此ID也需与客户端保持一致
	  //因为服务器和自适应加密算法,所以 security 可以不用
        }
      ]
    },
   "streamSettings":{
      "network": "ws",
      "wsSettings": {
           "path": "/v2ray" //与nginx和client的path要一致
      }
   }
  },
  "outbound": {
    "protocol": "freedom",
    "settings": {}
  }
}
```
# Content of nginx on server
```
server {
	listen 443 ssl http2 default_server;
	listen [::]:443 ssl http2 default_server;

	server_name your_domain www.your_domain;
	root /var/www/your_domain;
	index index.php index.html index.htm;

	ssl_certificate 	/etc/ssl/certs/your_domain.cert;
	ssl_certificate_key 	/etc/ssl/private/your_domain.key;
	ssl_protocols		TLSv1 TLSv1.1 TLSv1.2; 	
	ssl_session_cache 	shared:SSL:1m;
	ssl_session_timeout	10m;
	ssl_ciphers		HIGH:!aNULL:!MD5;
	ssl_prefer_server_ciphers on;
	ssl_client_certificate 	/etc/ssl/certs/origin-pull-ca.pem;
	ssl_verify_client on;

	client_max_body_size 100M;
	
	autoindex off;

	
	location / {
		try_files $uri $uri/ /index.php?$args;
	}

	location /ss {
		access_log off;
		proxy_redirect off;
		proxy_pass http://localhost:110; # Must be the same as the shadowsocks-libe's config.json	
		proxy_http_version 1.1;
		proxy_set_header Upgrade $http_upgrade;
		proxy_set_header Connection "upgrade";
		proxy_set_header Host $http_host;
	}
	
	location /v2ray {
		access_log off;
		proxy_redirect off;
		proxy_pass http://localhost:10086; # Must be the same as the v2ray server's config.json	
		proxy_http_version 1.1;
		proxy_set_header Upgrade $http_upgrade;
		proxy_set_header Connection "upgrade";
		proxy_set_header Host $http_host;
	}


	location ~ \.php$ {
		include snippets/fastcgi-php.conf;
		fastcgi_pass unix:/var/run/php/php7.0-fpm.sock;
		fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
		include fastcgi_params;
	}
}
```
# Content of v2ray client's config.json
```
{                                                                                                                                                                                                                                          
  "log": {                                                                                                                                                                                                                                 
    "loglevel": "warning",                                                                                                                                                                                                                 
    "access": "/var/log/v2ray/access.log",                                                                                                                                                                                                  
    "error": "/var/log/v2ray/error.log"                                                                                                                                                                                                     
  },                                                                                                                                                                                                                                       
                                                                                                                                                                                                                                           
  // in                                                                                                                                                                                                                                    
  "inbounds": [                                                                                                                                                                                                                            
    {                                                                                                                                                                                                                                      
      "port": 1080,                                                                                                                                                                                                                        
      "listen": "127.0.0.1",                                                                                                                                                                                                               
      "protocol": "socks",                                                                                                                                                                                                                 
      "settings": {                                                                                                                                                                                                                        
        "auth": "noauth",                                                                                                                                                                                                                  
        "udp": false,                                                                                                                                                                                                                      
        "ip": "127.0.0.1"                                                                                                                                                                                                                  
      },                                                                                                                                                                                                                                   
      "sniffing": {                                                                                                                                                                                                                        
        "enabled": true,
        "destOverride": ["http", "tls"]
      }
    }
  ],

  // out
  "outbounds": [
    {
      "tag":"proxy",
      "protocol":"vmess",
      "settings": {
        "vnext": [
          {
            "address": "yourdomain", // 你自己的域名
            "port": 443,
            "users": [
              {
                "id": "3ced360a-f524-4119-8d26-84f370034ec0", //ID必须与服务器上面的一致
                "alterId": 64
              }
            ]
          }
        ]
      },
      "streamSettings": {
        "network": "ws",
        "security": "tls",
        "wsSettings": {
          "path": "/v2ray" //填自己的路径和服务器的路径一致
        }
      },
      "mux": {
              "enabled": true
      }
    },
    {
      "protocol": "freedom",
      "settings":{},
      "tag": "direct" //如果要使用路由，这个 tag 是一定要有的，在这里 direct 就是 freedom 的一个标号，在路由中说 direct V2Ray 就知道是这里的 freedom 了    
    },
    {
      "protocol": "blackhole",
      "settings":{},
      "tag": "adblock" //同样的，这个 tag 也是要有的，在路由中说 adblock 就知道是这里的 blackhole（黑洞） 了  
    }
  ],

  // route
  "routing": {
    "domainStrategy": "IPOnDemand",
    "rules": [
      {
        "type": "field",
        "outboundTag": "adblock",
        "domain": [
          "tanx.com",
          "googeadsserving.cn",
          "baidu.com"
        ]
      },
      {
        "type": "field",
        "outboundTag": "direct",
        "ip": [
          "geoip:cn", // 中国大陆的 IP
          "geoip:private" // 私有地址 IP，如路由器等
        ]
      },
      {
        "type": "field",
        "outboundTag": "direct",
        "domain": [ 
          "geosite:cn", // 中国大陆主流网站的域名
          "domain:baidu.com",
          "domain:taobao.com",
          "domain:cnblogs.com",
          "domain:runoob.com",
          "domain:freebuf.com",
          "domain:mozilla.org",
          "domain:csdn.net",
          "domain:jd.com"
        ]
      }
    ]
  }
 }
```

reference:[V2Ray 配置指南](https://toutyrater.github.io/)

