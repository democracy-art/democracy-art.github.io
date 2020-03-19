---
layout:     post
title:      v2ray+ws+tls+nginx+cdn(on ubunt18.04)
subtitle:
date:       2020-03-19
author:     D
header-img: 
catalog: true
mermaid: true
tags:
    - [v2ray]
---

# 1.IP获取(购买VPS)

VPS 购买:[搬瓦工](https://bwh88.net),也可以选择其他厂商.

# 2.域名申请

2.1 注册[freenom](https://freenom.com)帐号

2.2 注册域名(freenom注册的域名可`免费`试用`一年`),步骤:<br>
- 1.登录freenom
- 2.`Services`
- 3.`Register a New Domain`
- 4.在输入框(显示Find your new Domain)中输入域名 
- 5.点击`Check Availability`
- 6.如果域名`可用`它会出现,选一个喜欢的 点击`Get it now `
- 7.跳转到另一页面, `Period` 下面,点下拉框 选 12 Months@FREE(12个月免费) 
- 8.点击 `Continue`
- 9.`Review & Checkout` 输入一个`有效`的邮箱用来接收信息 
- 10.`Your Details` 页面,填写信息随便填,只要能通过,邮箱`有效`就行,点`Complete Order` 
- 11.完成后注意查收邮件,登录freenom,点击 `Services`
- 12.`My Domains`
- 13.看到刚注册的域名 Status是 `ACTIVE` 说明注册成功

**注意:**下面所有的步骤中涉及到域名 **example.com** 请均改为你自己申请的域名.

# 3.注册Cloudflare且配置

3.1 在这里:<br>
[注册CLoudflare](https://dash.cloudflare.com/sign-up)<br>
![sign-up Cloudflare](/img/sign-up-cloudflare.png)

3.2 Add a Site<br>
注册完成后点击**Add a Site**<br>
![add-a-site](/img/add-a-site.png)

3.3 输入你已注册的域名<br>
![add-your-site](/img/add-your-site.png)

3.4 Cloudflare 查询 DNS<br>
![cloudflare-query-your-dns](/img/cloudflare-query-your-dns.png)

3.5 选择Cloudflare一款套餐<br>
![select-a-plan-of-cloudflare](/img/select-a-plan-of-cloudflare.png)<br>
完成和你应该看到Cloudflare给你分配的两个nameservers.<br>
![change-your-nameservers](/img/change-your-nameservers.png)<br>

3.6 替换为Cloudflare的nameservers.<br>
![use-custom-name-servers](/img/use-custom-name-servers.png)<br>
保存自定义名称服务器更改后，请返回您的Cloudflare帐户并等待Cloudflare看到更改.<br> 
根据您的域提供商的不同，最多需要一个小时才能看到Cloudflare.<br>
成功后你将会看到**Active**.<br>
![active](/img/active.png)<br>
完成上面的所有东西你再查看你的Cloudflare帐户里的DNS标签如下:<br>
![done](/img/done.png)<br>

3.7 SSL/TLS证书获取<br>

3.7.1 acme.sh 实现了 acme 协议, 可以从 letsencrypt 生成免费的证书
```
apt install curl
curl  https://get.acme.sh | sh
```

3.7.2 使其生效:
```
source ~/.bashrc
```

3.7.3 设置 Cloudflare Key 和 Email:
- 1.登录Cloudflare主页最右上角小用户头像
![figure20200319-1](/img/v2ray/figure20200319-1.jpg)
- 2.点击 MyProfile
![figure20200319-2](/img/v2ray/figure20200319-2.jpg)
- 3.点击 **API Tokens**
![figure20200319-3](/img/v2ray/figure20200319-3.jpg)
- 4.**API Keys**里Global API Key的<kbd>View</kbd>按钮
![figure20200319-4](/img/v2ray/figure20200319-4.jpg)
- 5.输入Cloudflare密码和验证码
![figure20200319-5](/img/v2ray/figure20200319-5.jpg)
- 6.复制Key
![figure20200319-6](/img/v2ray/figure20200319-6.jpg)

3.7.4 设置环境变量<br>
`CF_Key`是刚刚获取的`Your API Key`,`CF_Email`注册Cloudflare用的邮箱
```
export CF_Key="Your API Key"
export CF_Email="your_cloudflare@gmail.com"
```

3.7.5 利用脚本acme.sh配置
```
~/.acme.sh/acme.sh --issue --dns dns_cf -d example.com -d *.example.com -k ec-256
~/.acme.sh/acme.sh --installcert -d example.com -d *.example.com --fullchainpath /etc/ssl/certs/example.com.crt --keypath /etc/ssl/private/example.com.key --ecc
```

3.7.6 设置定期自动更新
```
~/.acme.sh/acme.sh --upgrade --auto-upgrade
```

3.7.7 总是启用HTTPS<br>
- 1.Always Use HTTPS
![always-use-https](/img/always-use-https.png)<br>
- 2.往下拉还有一个 **HTTP Strict Transport Security (HSTS)** 可以启用也可以不启用.
![figure20200319-7](/img/v2ray/figure20200319-7.jpg)
- 3.往下拉启用 **Opportunistic Encryption** 和 **Automatic HTTPS Rewrites**
![figure20200319-8](/img/v2ray/figure20200319-8.png)
- 4.切换到 `Origin Server` 这页启用 **Authenticated Origin Pulls**
![figure20200319-9](/img/v2ray/figure20200319-9.png)

3.8 速度优化<br>
![figure20200319-9](/img/v2ray/figure20200319-9.png)

3.9 Page Rules 标签<br>
来到<kbd>Page Rules</kbd>标签,点击`Create Page Rule`选择**Always Use HTTPS**.<br> 
![page-rules-always-use-https](/img/page-rules-always-use-https.png)<br>

# 4.安装nginx
4.1 更新apt-get
```
sudo apt-get update && sudo apt-get upgrade
```
4.2 安装编译依赖包<br>
4.2.1 安装build-essential包
```
sudo apt-get install build-essential
```
该命令安装了一组新的包,包括`gcc`,`g++`,`make`.<br>
安装nginx需要先将官网下载的源码进行编译,编译依赖gcc环境.

4.2.2 安装libtool
```
sudo apt-get install libtool
```
GNU libtool是通用库支持脚本.它将共享库的使用隐藏在一个一致的可移植的接口后面.

4.3 安装pcre依赖库
```
sudo apt-get install libpcre3 libpcre3-dev
```
pcre是一个Perl库,包括perl兼容的正则表达式库.nginx的http模块使用pcre来解析正则表达式,所以需要在linux上安装pcre库.

4.4 安装zlib依赖库
```
sudo apt-get install zlib1g-dev
```
zlib库提供了很多种压缩和解压缩的方式,nginx使用zlib对http包的内容进行gzip,所以需要在linux上安装zlib库.

4.5 安装SSL依赖库
```
sudo apt-get install libssl-dev
```
openssl是一个强大的安全套接字层密码库,囊括主要的密码算法,常用的密钥和证书封装管理功能及SSL协议,并提供丰富的应用程序供测试或其他目的使用.nginx不仅支持http协议,还支持https(即在SSL协议上传输http),所以需要在linux安装openssl库.

4.6 安装
```
sudo apt-get install nginx
```
4.7 设置开机启动nginx且启动nginx
```
sudo systemctl stop nginx.service
sudo systemctl start nginx.service
sudo systemctl enable nginx.service
```

4.8 检测nginx是否安装成功<br>
在浏览器输入你的服务器的IP地址,比如:`192.168.1.100`,出现如下信息说明成功:<br>
**Welcome to nginx!**<br>
...<br>

如果想源码编译安装nginx请查看:[ubuntu18.04安装nginx1.16.1(源码编译安装)](https://dm116.github.io/2020/02/25/install-nginx-on-ubuntu1804/)<br>
**注意:**如果nginx是源码编译安装,那么下面的步骤的内容会不一样.

# 5.安装MariaDB
我们将使用MariaDB作为我们的WordPress数据库,要安装MariaDB，请运行以下命令:
```
sudo apt-get install mariadb-server mariadb-client
```
安装完成后，我们将启动它并将其配置为在系统引导时自动启动:
```
sudo systemctl stop mariadb.service
sudo systemctl start mariadb.service
sudo systemctl enable mariadb.service
```
数据库安全配置
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

检查是否安装成功:
```
sudo mysql -u root -p
```
输入密码后看到如下字符串,说明登录成功.<br>
>MariaDB[(none)]>

# 6.安装PHP7.2-FPM及相关模块

由于WordPress是用PHP编写的应用程序，<br>
我们将安装PHP和运行WordPress所需的PHP包，使用以下命令:<br>
```
sudo apt-get install php-fpm php-common php-mbstring php-xmlrpc php-soap php-gd php-xml php-intl php-mysql php-cli php-ldap php-zip php-curl
```
安装PHP7.2完成后,打开nginx的PHP默认配置文件,进行如下配置:
```
sudo vi /etc/php/7.2/fpm/php.ini
```
下面的配置对大多数基于PHP的CMS来说都是好的配置,当然如果自己的需求可自行配置.
```
file_uploads = On
allow_url_fopen = On
short_open_tag = On
memory_limit = 256M
cgi.fix_pathinfo = 0
upload_max_filesize = 100M
max_execution_time = 360
date.timezone = America/Chicago
```
重启nginx
```
sudo systemctl restart nginx.service
```
安装完成后,启动php-fpm服务器:
```
sudo systemctl start php7.2-fpm
sudo systemctl enable php7.2-fpm
```

# 7.创建wordpress数据库
登录mariadb
```
sudo mysql -u root -p
```
创建wordpress数据库
```
CREATE DATABASE wordpress
```
创建wordpress数据库用户(**secure_password**换成自己安全系数高的密码)
```
CREATE USER 'wp_user'@'localhost' IDENTIFIED BY 'secure_password';
```
给用户授权
```
GRANT ALL ON wordpress.* TO 'wp_user'@'localhost';
```
保存你的更改然后退出.
```
FLUSH PRIVILEGES
EXIT;
```

# 8.下载wordpress最新的版本
使用以下wget命令下载最新的WordPress包:
```
cd /tmp && wget http://wordpress.org/latest.tar.gz
```
然后用以下内容提取存档:
```
sudo tar -xvzf latest.tar.gz
sudo mv wordpress /var/www/html/example.com
```
设置wordpress根目录的正确权限并给予nginx控制权.
```
sudo chown -R www-data:www-data  /var/www/html/example.com/ 
sudo chmod -R 755 /var/www/html/example.com/
```

# 9.配置nginx
创建新的配置文件**example.com**
```
sudo vi /etc/nginx/sites-available/example.com
```
将 **example.com** 替换为自己的域名.
```
server {
    listen 80;
    listen [::]:80;
    listen 443 ssl http2;
    listen [::]:443 ssl http2;

    server_name  example.com www.example.com;
    root   /var/www/html/example.com;
    index  index.php;

    ssl_certificate /etc/ssl/certs/example.com.crt;
    ssl_certificate_key /etc/ssl/private/example.com.key;

    client_max_body_size 100M;
  
    autoindex off;

    location / {
        try_files $uri $uri/ /index.php?$args;
    }

    location ~ \.php$ {
         include snippets/fastcgi-php.conf;
         fastcgi_pass unix:/var/run/php/php7.2-fpm.sock;
         fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
         include fastcgi_params;
    }
}
```

# 10.使能wordpress
```
sudo ln -s /etc/nginx/sites-available/example.com /etc/nginx/sites-enabled/
sudo systemctl restart nginx.service
```
浏览器输入 **https://example.com/** 会出现下图:
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
