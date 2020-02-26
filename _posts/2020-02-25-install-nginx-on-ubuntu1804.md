---
layout:     post
title:      ubuntu18.04安装nginx1.16.1(源码编译安装)
subtitle:   
date:       2020-02-25
author:     D
header-img: 
catalog: true
mermaid: true
tags:
    - [nginx]
---

# 1.安装依赖
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
sudo apt-get install openssl
```
openssl是一个强大的安全套接字层密码库,囊括主要的密码算法,常用的密钥和证书封装管理功能及SSL协议,并提供丰富的应用程序供测试或其他目的使用.nginx不仅支持http协议,还支持https(即在SSL协议上传输http),所以需要在linux安装openssl库.

# 2.安装nginx
2.1 从官网下载稳定版本nginx1.16.1
```
sudo apt-get install wget
sudo wget http://nginx.org/download/nginx-1.16.1.tar.gz
```
2.2 解压且进入解压目录
```
sudo tar -zxvf nginx-1.16.1.tar.gz
cd nginx-1.16.1
```
2.3 配置且将nginx安装到`/usr/local/nginx`目录

2.3.1 生成Makefile
```
./configure --prefix=/usr/local/nginx \
--with-pcre \
--with-http_ssl_module \
--with-http_v2_module \
--with-http_gunzip_module \
--with-http_gzip_static_module 
```
上面的这些模块可以自己添加或去掉如果什么模块都不加:
```
./configure --prefix=/usr/local/nginx 
```
详细请查看官网文档:[Building nginx from Sources](http://nginx.org/en/docs/configure.html)

2.3.2 编译 
```
make 
```
2.3.3 安装(需要root权限)
```
sudo -i  
make install
```
# 3.启动nginx
3.1 启动nginx
```
sudo /usr/local/nginx/sbin/nginx -c /usr/local/nginx/conf/nginx.conf
```
`-c`是指定配置文件路径,不加的话,nginx会自动加载默认路径的配置文件,可通过`-h`查看帮助命令.

3.2 查看nginx进程
```
ps -ef|grep nginx
```
# 4.关闭nginx
有两种方式:<br>
4.1 方式1:快速停止
```
sudo /usr/local/nginx/sbin/nginx -s stop
```
这种方式想当于先查出nginx进程id再用kill命令强制杀掉进程,不太友好.

4.2 方式2:平缓停止
```
sudo /usr/local/nginx/sbin/nginx -s quit
```
此方式是指允许nginx服务将当前正在处理的网络请求处理完成,但不在接收新的请求,之后关闭连接,停止工作.

# 5.重启nginx
5.1 方式1:先停止再启动
```
sudo /usr/local/nginx/sbin/nginx -s quit
sudo /usr/local/nginx/sbin/nginx 
```
5.2 方式2:重新加载配置文件
```
sudo /usr/local/nginx/sbin/nginx -s reload
```
通常我们使用nginx修改最多的便是其配置文件nginx.conf,修改之后想让配置文件生效而不用重启nginx，便可以使用此命令.

# 6.检测nginx配置文件语法是否正确
6.1 方式1:指定需要检查的配置文件
```
sudo /usr/local/nginx/sbin/nginx -t -c /usr/local/nginx/conf/nginx.conf
```
6.2 方式2:不加`-c`参数,默认检测nginx.conf配置文件
```
sudo /usr/local/nginx/sbin/nginx -t
```

参考:[nginx详解](https://cnblogs.com/qiulovelinux/p/10417477.html)<br>
参考:[Nginx（一）------简介与安装](https://www.cnblogs.com/ysocean/p/9384877.html)<br>
参考:[ubuntu18.04 lnmp编译安装](https://jianshu.com/p/157fb7f28bf7)<br>


