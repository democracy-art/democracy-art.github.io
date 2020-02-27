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

# 1.IP获取(购买VPS)

VPS 购买:[搬瓦工](https://bwh88.net),也可以选择其他厂商.

# 2.域名申请

2.1 注册[freenom](https://freenom.com)帐号

2.2 注册域名(freenom注册的域名可`免费`试用`一年`),步骤:<br>
- 登录freenom
- `Services`
- `Register a New Domain`
-  在输入框(显示Find your new Domain)中输入域名 
- 点击`Check Availability`
- 如果域名`可用`它会出现,选一个喜欢的 点击`Get it now `
- 跳转到另一页面, `Period` 下面,点下拉框 选 12 Months@FREE(12个月免费) 
- 点击 `Continue`
- `Review & Checkout` 输入一个`有效`的邮箱用来接收信息 
- `Your Details` 页面,填写信息随便填,只要能通过,邮箱`有效`就行,点`Complete Order` 
- 完成后注意查收邮件,登录freenom,点击 `Services`
- `My Domains`
- 看到刚注册的域名 Status是 `ACTIVE` 说明注册成功

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
![done](/img/dong.png)<br>

3.7 SSL/TLS<br>
3.7.1 `SSL/TLS`标签下的`Overview`页选择`Full(strict)`<br>
![full-strict](/img/ssl-tls-tab-overview-full-strict.png)<br>

3.7.2 使用Cloudflare签署的免费TLS证书.<br>
![create-certificate](/img/ssl-tls-tab-origin-server-create-certificate.png)<br>
点击`Create certificate`,出来下面页面:<br>
选择**Let Cloudflare generate a private key and a CSR**,然后点击<kbd>Next</kbd>.<br>
![Origin Certificate Installation](/img/Let-Cloudflare-generate-a-private-key-and-a-CSR.png)<br>
然后将它们复制粘贴到服务器上的文本文件中.<br>

3.7.3 在Ubuntu上，运行下面的命令来创建密钥、证书和源文件,并保存.<br>
对于`key file`运行下面命令并复制粘贴:<br>
```
sudo vi /etc/ssl/private/cloudflare_example.com.pem
```
对于`certificate file`运行下面命令并复制粘贴:
```
sudo vi /etc/ssl/certs/cloudflare_example.com.pem
```
你还需要下载Cloudflare `Origin Pull certificate`，可以从下面的链接下载:<br>
[Origin Pull Certificate](https://support.cloudflare.com/hc/en-us/articles/204899617-Authenticated-Origin-Pulls#section6)<br>
或者使用下面命令下载:<br>
```
cd /etc/ssl/certs/
sudo wget https://support.cloudflare.com/hc/en-us/article_attachments/360044928032/origin-pull-ca.pem
```

3.7.4 总是启用HTTPS<br>
![always-use-https](/img/always-use-https.png)<br>
往下拉还有一个**HTTP Strict Transport Security (HSTS)**可以启用也可以不启用.<br>

往下拉启用 **Opportunistic Encryption** 和 **Automatic HTTPS Rewrites**<br>
切换到 `Origin Server` 这页启用 **Authenticated Origin Pulls**<br>

3.8 Speed标签设置(速度优化)<br>
<kbd>Speed</kbd>标签下面的`Optimization`页的 **Auto Minify** 这栏的3个复选框全勾选.<br>
* [x] JavaScript
* [x] CSS
* [x] HTML

3.9 Page Rules 标签<br>
来到<kbd>Page Rules</kbd>标签,点击`Create Page Rule`选择**Always Use HTTPS**.<br> 
![page-rules-always-use-https](/img/page-rules-always-use-https.png)<br>

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
