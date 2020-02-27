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
    - [wordpress]
---

# 1.安装nginx
1.1 更新apt-get
```
sudo apt-get update && sudo apt-get upgrade
```
1.2 安装编译依赖包<br>
1.2.1 安装build-essential包
```
sudo apt-get install build-essential
```
该命令安装了一组新的包,包括`gcc`,`g++`,`make`.<br>
安装nginx需要先将官网下载的源码进行编译,编译依赖gcc环境.

1.2.2 安装libtool
```
sudo apt-get install libtool
```
GNU libtool是通用库支持脚本.它将共享库的使用隐藏在一个一致的可移植的接口后面.

1.3 安装pcre依赖库
```
sudo apt-get install libpcre3 libpcre3-dev
```
pcre是一个Perl库,包括perl兼容的正则表达式库.nginx的http模块使用pcre来解析正则表达式,所以需要在linux上安装pcre库.

1.4 安装zlib依赖库
```
sudo apt-get install zlib1g-dev
```
zlib库提供了很多种压缩和解压缩的方式,nginx使用zlib对http包的内容进行gzip,所以需要在linux上安装zlib库.

1.5 安装SSL依赖库
```
sudo apt-get install libssl-dev
```
openssl是一个强大的安全套接字层密码库,囊括主要的密码算法,常用的密钥和证书封装管理功能及SSL协议,并提供丰富的应用程序供测试或其他目的使用.nginx不仅支持http协议,还支持https(即在SSL协议上传输http),所以需要在linux安装openssl库.

1.6 安装
```
sudo apt-get install nginx
```
1.7 设置开机启动nginx且启动nginx
```
sudo systemctl stop nginx.service
sudo systemctl start nginx.service
sudo systemctl enable nginx.service
```

如果想源码编译安装nginx请查看:[ubuntu18.04安装nginx1.16.1(源码编译安装)](https://dm116.github.io/2020/02/25/install-nginx-on-ubuntu1804/)<br>
**注意:**如果nginx是源码编译安装,那么下面的步骤的内容会不一样.

# 2.在Nginx上为WordPress网站创建Vhost

```
sudo vi /etc/nginx/sites-available/wordpress.conf
```
在下面的示例中，使用您要使用的域更改`example.com`: 
```
server {
    listen 80;
    listen [::]:80;
    root /var/www/html/wordpress;
    index  index.php index.html index.htm;
    server_name example.com www.example.com;

     client_max_body_size 100M;

    location / {
        try_files $uri $uri/ /index.php?$args;        
    }

    location ~ \.php$ {
    include snippets/fastcgi-php.conf;
    fastcgi_pass             unix:/var/run/php/php7.2-fpm.sock;
    fastcgi_param   SCRIPT_FILENAME $document_root$fastcgi_script_name;
    }
}
```
保存文件并退出。 然后启用该站点:
```
sudo ln -s /etc/nginx/sites-available/wordpress.conf  /etc/nginx/sites-enabled/
```
然后重新加载nginx:
```
sudo systemctl reload nginx
```

# 3.在Ubuntu 18.04上安装MariaDB 10
我们将使用MariaDB作为我们的WordPress数据库。 要安装MariaDB，请运行以下命令:
```
sudo apt-get install mariadb-server mariadb-client
```
安装完成后，我们将启动它并将其配置为在系统引导时自动启动:
```
sudo systemctl start mariadb.service
sudo systemctl enable mariadb.service
```
接下来，通过运行以下命令来保护MariaDB安装:
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

# 4.为网站创建WordPress数据库

之后，我们将为该用户准备数据库，数据库用户和密码。<br>
它们将由我们的WordPress应用程序使用，因此它可以连接到MySQL服务器。<br>
```
sudo mysql -u root -p
```
使用下面的命令，我们将首先创建数据库，然后创建数据库用户及其密码。<br>
然后我们将授予用户对该数据库的权限。<br>
创建数据库`wordpress`
```
CREATE DATABASE wordpress;
```
创建用户.<br>
注意将下面 `secure_password` 换成你的密码.
```
CREATE USER 'wp_user'@'localhost' IDENTIFIED BY 'secure_password';
```
授权
```
GRANT ALL ON wordpress.* TO 'wp_user'@'localhost';
FLUSH PRIVILEGES;
EXIT;
```

# 5.在Ubuntu 18.04上安装PHP 7

由于WordPress是用PHP编写的应用程序，<br>
我们将安装PHP和运行WordPress所需的PHP包，使用以下命令:<br>
```
sudo apt-get install php-fpm php-common php-mbstring php-xmlrpc php-soap php-gd php-xml php-intl php-mysql php-cli php-ldap php-zip php-curl
```
安装完成后,启动php-fpm服务器:
```
sudo systemctl start php7.2-fpm
sudo systemctl enable php7.2-fpm
```

# 6.在Ubuntu 18.04上安装WordPress 5

使用以下wget命令下载最新的WordPress包:
```
cd /tmp && wget http://wordpress.org/latest.tar.gz
```
然后用以下内容提取存档:
```
sudo tar -xvzf latest.tar.gz -C /var/www/html
```
以上将创建我们在vhost中设置的文档根目录,即`/var/www/html/wordpress`然后,<br>
我们需要更改该目录中文件和文件夹的所有权:<br>
```
sudo chown www-data: /var/www/html/wordpress/ -R
```

在进行下一步之前你需要[绑定域名到IP](https://dm116.github.io/2020/02/19/bind-the-domain-to-ip)<br>

现在我们准备运行WordPress的安装.如果您使用了未注册或不存在的域,<br>
则可以使用以下记录配置hosts `/etc/hosts` 文件:<br>
```
192.168.1.100 example.com
```
假设您的服务器的IP地址是192.168.1.100，并且您使用的域是example.com，<br>
那么您的计算机将在给定的IP地址上解析example.com <br>

现在将您的域加载到浏览器中，您应该看到WordPress安装页面：<br>

>若输入没出现下图，登录Cloudflare,去DNS,把Proxy status从DNS only改为Proxied试试。

![Select-WordPress-Install-Language](/img/Select-WordPress-Install-Language.png)<br>

选择WordPress安装语言

在下一页上输入我们之前设置的数据库凭据：<br>

![WordPress-Database-Settings](/img/WordPress-Database-Settings.png)<br>

WordPress数据库设置

提交表单，然后在下一个屏幕上配置您的网站标题，管理员用户和电子邮件：<br>

![WordPress-Website-Setup](/img/WordPress-Website-Setup.png)<br>

WordPress网站设置

您的安装现已完成，您可以开始管理您的WordPress网站。 您可以先安装一些全新的主题或通过插件扩展网站功能。

结论:
就是这样。 在Ubuntu 18.04上设置自己的WordPress安装的过程。 我希望这个过程简单明了。


参考:[How To Setup WordPress With Nginx And Cloudflare CDN / SSL On Ubuntu 16.04 | 18.04](https://websiteforstudents.com/how-to-setup-wordpress-with-nginx-and-cloudflare-cdn-ssl-on-ubuntu-16-04-18-04/)<br>
参考:[英文原版:Install WordPress with Nginx, MariaDB 10 and PHP 7 on Ubuntu 18.04](https://www.tecmint.com/install-wordpress-with-nginx-mariadb-php-on-ubuntu-18-04/)<br>
参考:[在Ubuntu 18.04上安装带有Nginx，MariaDB 10和PHP 7的WordPress](https://www.howtoing.com/install-wordpress-with-nginx-mariadb-php-on-ubuntu-18-04)
