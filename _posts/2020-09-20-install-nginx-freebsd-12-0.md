--- 
layout: post
title: 在FreeBSD12.0上安裝Nginx 
subtitle:
date: 2020-09-20
author: D
header-img:
catalog: true
tags: [freebsd,nginx]
---
# 1.安裝Nginx
```
# pkg install nginx
```
使能Nginx
```
# sysrc nginx_enable=yes
```
# 2.設置防火牆PF允許Nginx服務
```
# vi /etc/pf.conf
```
允許Nginx的服務的流量進和出,這裏HTTP在80端口，HTTPS在443端口
```
pass in on vtnet0 proto tcp to port { 80 443 }
pass out proto tcp to port {80 443}
```
使用pfctl工具空運行，以檢查修改後的pf.conf的語法是否正確
```
# pfctl -nf /etc/pf.conf
```
# 3.設置服務器塊
使用Nginx Web服务器时，服务器块可用于封装配置详细信息，并在一台服务器中托管多个域。 
我们将建立一个名为example.com的域名，但是您应该使用自己的域名替换它。

3.1 使用 `-p` 标志创建所有必需的父目录，如下所示为example.com创建目录：
```
# mkdir -p /usr/local/www/example.com/html
```

3.2 接下来，将目录的所有权分配给www用户（默认的Nginx运行时用户配置文件）：
```
# chmod -R 755 /usr/local/www/example.com
```
3.3 创建一个示例index.html页面
```
# vi /usr/local/www/example.com/html/index.html
```
`index.html` 內容如下:
```
<!DOCTYPE html>
<html>
<head>
    <meta charset="utf-8">
    <title>Welcome to example.com</title>
</head>

<body>
    <h1>Welcome to example.com</h1>
</body>
</html>
```

3.4 創建域名相關配置文件
```
# touch /usr/local/etc/nginx/example.com.conf
```
example.com.conf內容如下:
```
server {
        access_log /var/log/nginx/example.com.access.log;
        error_log /var/log/nginx/example.com.error.log;
        listen       80;
        server_name  example.com www.example.com;
        server_tokens off;

        location / {
            root   /usr/local/www/example.com/html;
            index  index.html index.htm;
        }
}
```

3.5 配置 nginx.conf
```
# vi /usr/local/etc/nginx/nginx.conf
```
內容修改和添加如下:
```
user www;
worker_processes auto;
http{
...
        include         example.com.conf; # 包含域名文件
        server_tokens   off;  # 不顯示nginx的版本信息
...
}
```
将user 从 nobody 更改为`www`,  然后更新worker_processes指令，该指令允许您选择
Nginx将使用多少个工作**进程**。 在此处输入的最佳值并不总是很明显或很难找到。 
将其设置为自动告诉Nginx将其设置为每个CPU内核一个工作线程，这在大多数情况下就足够了.

3.6 檢查nginx語法
```
# nginx -t
```
如果正確則輸出:
```
nginx: the configuration file /usr/local/etc/nginx/nginx.conf syntax is ok
nginx: configuration file /usr/local/etc/nginx/nginx.conf test is successful
```
3.7 啓用nginx的修改
```
# service nginx reload
```

# 4.管理Nginx的命令
4.1 停止nginx
```
# service nginx stop
```
4.2 啓動nginx
```
# service nginx start
```
4.3 停止且再啓動nginx
```
# service nginx restart
```
4.4 如果只是更改配置，则可以重新加载Nginx，而无需断开任何连接
```
# service nginx reload
```
