---
layout:     post
title:      ubuntu18.04安装wordpress+lnmp(nginx源码编译安装)
subtitle:   ubuntu18.04安装带有nginx(源码编译安装),mariadb10,php7,wordpress
date:       2020-02-26
author:     D
header-img: 
catalog: true
mermaid: true
tags:
    - [wordpress]
---

# 1.安装nginx
[ubuntu18.04安装nginx1.16.1(源码编译安装)](https://dm116.github.io/2020/02/25/install-nginx-on-ubuntu1804/)

# 2.安装mariadb   
wordpress需要数据库来存储它的数据,mariadb是非常好的选择.<br>
2.1 安装
```
sudo apt-get install mariadb-server mariadb-client
```
2.2 停止和启动mariadb
```
sudo systemctl stop mariadb.service
sudo systemctl start mariadb.service
```
2.3 设置开机启动mariadb
```
sudo systemctl enable mariadb.service
```
2.4 安全配置向导命令
```
sudo mysql_secure_installation
```
当提示时,按如下指南回答问题：
- Enter current password for root (enter for none): **按Enter键**
- Set root password? [Y/n]:**Y**
- New password: **输入设置密码**
- Re-enter new password: **再次输入密码**
- Remove anonymous users? [Y/n]: **Y**
- Disallow root login remotely? [Y/n]: **Y**
- Remove test database and access to it? [Y/n]: **Y**
- Reload privilege tables now? [Y/n]:  **Y**

2.5 测试数据库是否安装成功
```
sudo mysql -u root -p
```

# 3.安装php7.2-fpm和相关模块
3.1 安装php
```
sudo apt install php7.2-fpm php7.2-common php7.2-mysql php7.2-gmp php7.2-curl php7.2-intl php7.2-mbstring php7.2-xmlrpc php7.2-gd php7.2-xml php7.2-cli php7.2-zip
```

3.2 配置php.ini
```
sudo vi /etc/php/7.2/fpm/php.ini
```
下面的这些设置对于大多数基于PHP的CMS来说是良好的设置(有特殊要求请自行设置)
```
file_uploads = On
allow_url_fopen = On
short_open_tag = On
memory_limit = 256M
cgi.fix_pathinfo = 0
upload_max_filesize = 100M
max_execution_time = 360
date.timezone = America/Chicago   //时区可改,比如: Asia/Shanghai
```
每次修改php配置文件,请记得重新启动nginx web服务.
```
sudo /usr/local/nginx/sbin/nginx -s reload
```

3.3 检查php是否安装成功
```
sudo vi /usr/local/nginx/html/phpinfo.php
```
`phpinfo.php`内容如下:
```
<?php phpinfo(); ?>
```



