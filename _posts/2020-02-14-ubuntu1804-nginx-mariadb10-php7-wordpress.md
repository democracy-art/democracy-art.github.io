---
layout:     post
title:      Ubuntu18.04安装带有Nginx,MariaDB10和PHP7的WordPress
subtitle:
date:       2020-02-14
author:     D
header-img: 
catalog: true
mermaid: true
tags:
    - linux_basic
---

参考:[在Ubuntu 18.04上安装带有Nginx，MariaDB 10和PHP 7的WordPress](https://www.howtoing.com/install-wordpress-with-nginx-mariadb-php-on-ubuntu-18-04)

# 1.在Ubuntu 18.04上安装Nginx Web Server
安装Nginx:<br>
```
sudo apt update && sudo apt upgrade
sudo apt install nginx
```
设置开机启动且启动Nginx服务:<br>
```
sudo systemctl start nginx.service
sudo systemctl enable nginx.service
```
# 2.在Nginx上为WordPress网站创建Vhost

```
sudo vi /etc/nginx/sites-available/wordpress.conf
```
在下面的示例中，使用您要使用的域更改`example.com`: 
```
```
