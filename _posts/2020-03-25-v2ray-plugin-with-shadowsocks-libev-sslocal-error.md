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

# 1. Client kali 2019
## 1.1 local.json
path: `/etc/shadowsocks-libev/local.json`
```
{
    "server":"bishe.ga",
    "server_port":443,
    "local_port":1080,
    "password":"d@XRG*D6",
    "timeout":300,
    "method":"aes-256-gcm",
    "plugin":"/usr/bin/v2ray-plugin",
    "plugin_opt":"tls;host=bishe.ga;path=/ss/;loglevel=none",
    "reuse_port":true
}
```
## 1.2 nginx.conf 
path: /etc/nginx/nginx.conf
```
user www-data;
worker_processes auto;
pid /run/nginx.pid;
include /etc/nginx/modules-enabled/*.conf;

events {
	worker_connections 768;
	# multi_accept on;
}

http {

	##
	# Basic Settings
	##
	server {
		listen 80;
		server_name 127.0.0.1;
		location /autoproxy.pac {
			alias /etc/shadowsocks-libev/autoproxy.pac;
		}
	}

	sendfile on;
	tcp_nopush on;
	tcp_nodelay on;
	keepalive_timeout 65;
	types_hash_max_size 2048;
	# server_tokens off;

	# server_names_hash_bucket_size 64;
	# server_name_in_redirect off;

	include /etc/nginx/mime.types;
	default_type application/octet-stream;

	##
	# SSL Settings
	##

	ssl_protocols TLSv1 TLSv1.1 TLSv1.2; # Dropping SSLv3, ref: POODLE
	ssl_prefer_server_ciphers on;

	##
	# Logging Settings
	##

	access_log /var/log/nginx/access.log;
	error_log /var/log/nginx/error.log;

	##
	# Gzip Settings
	##

	gzip on;

	# gzip_vary on;
	# gzip_proxied any;
	# gzip_comp_level 6;
	# gzip_buffers 16 8k;
	# gzip_http_version 1.1;
	# gzip_types text/plain text/css application/json application/javascript text/xml application/xml application/xml+rss text/javascript;

	##
	# Virtual Host Configs
	##

	include /etc/nginx/conf.d/*.conf;
	include /etc/nginx/sites-enabled/*;
}

# proxy chrome 


#mail {
#	# See sample authentication script at:
#	# http://wiki.nginx.org/ImapAuthenticateWithApachePhpScript
# 
#	# auth_http localhost/auth.php;
#	# pop3_capabilities "TOP" "USER";
#	# imap_capabilities "IMAP4rev1" "UIDPLUS";
# 
#	server {
#		listen     localhost:110;
#		protocol   pop3;
#		proxy      on;
#	}
# 
#	server {
#		listen     localhost:143;
#		protocol   imap;
#		proxy      on;
#	}
#}
```

## 1.3 shadowsocks-libev-local@.service 
```
#  This file is part of shadowsocks-libev.
#
#  Shadowsocks-libev is free software; you can redistribute it and/or modify
#  it under the terms of the GNU General Public License as published by
#  the Free Software Foundation; either version 3 of the License, or
#  (at your option) any later version.
#
#  This is a template unit file. Users may copy and rename the file into
#  config directories to make new service instances. See systemd.unit(5)
#  for details.

[Unit]
Description=Shadowsocks-Libev Custom Client Service for %I
Documentation=man:ss-local(1)
After=network-online.target

[Service]
Type=simple
CapabilityBoundingSet=CAP_NET_BIND_SERVICE
AmbientCapabilities=CAP_NET_BIND_SERVICE
DynamicUser=true
ExecStart=/usr/bin/ss-local -c /etc/shadowsocks-libev/%i.json

[Install]
WantedBy=multi-user.target
```

## 1.4 error message of chrome
```
root@kali:~# chrome
[2097:2097:0325/184838.830074:ERROR:edid_parser.cc(102)] Too short EDID data: manufacturer id
[2126:2126:0325/184838.931405:ERROR:sandbox_linux.cc(374)] InitializeSandbox() called with multiple threads in process gpu-process.
[2129:2134:0325/184854.258843:ERROR:ssl_client_socket_impl.cc(941)] handshake failed; returned -1, SSL error code 1, net_error -100
[2129:2134:0325/184854.259257:ERROR:ssl_client_socket_impl.cc(941)] handshake failed; returned -1, SSL error code 1, net_error -100
[2129:2134:0325/184854.276415:ERROR:ssl_client_socket_impl.cc(941)] handshake failed; returned -1, SSL error code 1, net_error -100
```

# 2   
```
```
