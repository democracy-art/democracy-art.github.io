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
    "loglevel": "warning"
  },

  "inbounds": [{
    "port": 1080,
    "listen": "127.0.0.1",
    "tag": "socks-inbound",
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
  }],

  "outbounds": [{
    "tag":"proxy",
    "protocol":"vmess",
    "settings": {
      "vnext": [{
	  "address": "your_domain", // Write your domain here 
	  "port": 443,
	  "users": [{
	      "id": "my clien uuid", // The UUID  Must be the same as the server
	      "alterId": 64,
	      "security": "aes-128-gcm" // Must be the same as the server
	    }
	  ]
        }
      ],
      "servers":null,
      "response":null
    },
    "streamSettings": {
      "network": "ws",
      "security": "tls",
      "tlsSettings": {
        "allowInsecure": true,
	"serverName": null
      },
      "tcpSettings": null,
      "kcpSettings": null,
      "wsSettings": {
        "connectionReuse": true,
	"path": "/v2ray", //The UUID  Must be the same as the server
	"headers": null
      },
      "httpSettings": null,
      "quicSettings": null
    },
    "mux": {
      "enabled": true
    }
  },
  {
    "protocol": "freedom",
    "settings": {},
    "tag": "direct"
  },{
    "protocol": "blackhole",
    "settings": {},
    "tag": "blocked"
  }],
  "routing": {
    "domainStrategy": "AsIs",
    "rules":[
      {
        "type": "field",
        "port": null,
        "inboundTag": null,
        "outboundTag": "proxy",
        "ip": null,
        "domain": [
          "geosite:google",
          "geosite:youtube",
          "geosite:github",
          "geosite:netflix",
          "geosite:steam",
          "geosite:telegram",
          "geosite:speedtest",
          "domain:bleepingcomputer.com",
          "domain:freenom.com",
          "domain:gvt1.com",
          "domain:github.io",
          "domain:hackerone.com",
          "domain:justmysocks.net",
          "domain:textnow.com",
          "domain:twitch.tv",
          "domain:wikileaks.org",
          "domain:naver.com",
          "domain:gnews.org",
          "domain:twitter.com",
          "domain:youtube.com"
        ]
      },
      {
        "type": "field",
        "port": null,
        "inboundTag": null,
        "outboundTag": "direct",
        "ip": null,
        "domain": [
          "domain:12306.com",
          "domain:51ym.me",
          "domain:52pojie.cn",
          "domain:8686c.com",
          "domain:abercrombie.com",
          "domain:adobesc.com",
          "domain:air-matters.com",
          "domain:air-matters.io",
          "domain:airtable.com",
          "domain:akadns.net",
          "domain:apache.org",
          "domain:api.crisp.chat",
          "domain:api.termius.com",
          "domain:appshike.com",
          "domain:appstore.com",
          "domain:aweme.snssdk.com",
          "domain:bababian.com",
          "domain:baidu.com",
          "domain:battle.net",
          "domain:beatsbydre.com",
          "domain:bet365.com",
          "domain:bilibili.cn",
          "domain:ccgslb.com",
          "domain:ccgslb.net",
          "domain:chinaz.com",
          "domain:chunbo.com",
          "domain:chunboimg.com",
          "domain:clashroyaleapp.com",
          "domain:cloudsigma.com",
          "domain:cloudxns.net",
          "domain:cmfu.com",
          "domain:cnblogs.com",
          "domain:csdn.net",
          "domain:culturedcode.com",
          "domain:dct-cloud.com",
          "domain:didialift.com",
          "domain:douyutv.com",
          "domain:duokan.com",
          "domain:dytt8.net",
          "domain:easou.com",
          "domain:ecitic.net",
          "domain:eclipse.org",
          "domain:eudic.net",
          "domain:ewqcxz.com",
          "domain:fir.im",
          "domain:frdic.com",
          "domain:fresh-ideas.cc",
          "domain:freebuf.com",
          "domain:godic.net",
          "domain:goodread.com",
          "domain:haibian.com",
          "domain:hdslb.net",
          "domain:hollisterco.com",
          "domain:hongxiu.com",
          "domain:hxcdn.net",
          "domain:images.unsplash.com",
          "domain:img4me.com",
          "domain:ipify.org",
          "domain:ixdzs.com",
          "domain:jd.hk",
          "domain:jianshuapi.com",
          "domain:jomodns.com",
          "domain:jsboxbbs.com",
          "domain:knewone.com",
          "domain:kuaidi100.com",
          "domain:lemicp.com",
          "domain:letvcloud.com",
          "domain:lizhi.io",
          "domain:localizecdn.com",
          "domain:lucifr.com",
          "domain:luoo.net",
          "domain:mai.tn",
          "domain:maven.org",
          "domain:miwifi.com",
          "domain:moji.com",
          "domain:moke.com",
          "domain:mtalk.google.com",
          "domain:mxhichina.com",
          "domain:myqcloud.com",
          "domain:myunlu.com",
          "domain:netease.com",
          "domain:nfoservers.com",
          "domain:nssurge.com",
          "domain:nuomi.com",
          "domain:oschina.net",
          "domain:ourdvs.com",
          "domain:overcast.fm",
          "domain:paypal.com",
          "domain:paypalobjects.com",
          "domain:pgyer.com",
          "domain:qdaily.com",
          "domain:qdmm.com",
          "domain:qin.io",
          "domain:qingmang.me",
          "domain:qingmang.mobi",
          "domain:qqurl.com",
          "domain:rarbg.to",
          "domain:rrmj.tv",
          "domain:ruguoapp.com",
          "domain:sm.ms",
          "domain:snwx.com",
          "domain:soku.com",
          "domain:startssl.com",
          "domain:store.steampowered.com",
          "domain:symcd.com",
          "domain:teamviewer.com",
          "domain:tmzvps.com",
          "domain:trello.com",
          "domain:trellocdn.com",
          "domain:ttmeiju.com",
          "domain:udache.com",
          "domain:uxengine.net",
          "domain:weather.bjango.com",
          "domain:weather.com",
          "domain:webqxs.com",
          "domain:weico.cc",
          "domain:wenku8.net",
          "domain:werewolf.53site.com",
          "domain:windowsupdate.com",
          "domain:wkcdn.com",
          "domain:workflowy.com",
          "domain:xdrig.com",
          "domain:xiaojukeji.com",
          "domain:xiaomi.net",
          "domain:xiaomicp.com",
          "domain:ximalaya.com",
          "domain:xitek.com",
          "domain:xmcdn.com",
          "domain:xslb.net",
          "domain:xteko.com",
          "domain:yach.me",
          "domain:yixia.com",
          "domain:yunjiasu-cdn.net",
          "domain:zealer.com",
          "domain:zgslb.net",
          "domain:zimuzu.tv",
          "domain:zmz002.com",
          "domain:samsungdm.com"
        ]
      },
      {
        "type": "field",
        "ip": ["geoip:private"],
        "outboundTag": "blocked"
      }
      ]
  },
  "dns": {
    "hosts": {
      "domain:v2ray.com": "www.vicemc.net",
      "domain:github.io": "pages.github.com",
      "domain:wikipedia.org": "www.wikimedia.org",
      "domain:shadowsocks.org": "electronicsrealm.com"
    },
    "servers": [
      "8.8.4.4",
      {
        "address": "114.114.114.114",
        "port": 53,
        "domains": [
          "geosite:cn",
	  "domain:baidu.com"
        ]
      },
      "8.8.8.8",
      "localhost"
    ]
  },
  "policy": {
    "levels": {
      "0": {
        "uplinkOnly": 0,
        "downlinkOnly": 0
      }
    },
    "system": {
      "statsInboundUplink": false,
      "statsInboundDownlink": false
    }
  },
  "other": {}
}
```

