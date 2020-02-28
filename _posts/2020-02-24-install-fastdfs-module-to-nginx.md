---
layout:     post
title:      nginx添加fastdfs模块(非覆盖安装)on ubuntu18.04
subtitle:   
date:       2020-02-24
author:     D
header-img: 
catalog: true
mermaid: true
tags:
    - [FastDFS]
---

若没安装FastDFS,请先实现[FastDFS安装](https://dm116.github.io/2020/02/23/install-fastdfs)

环境及版本:
- Ubuntu18.04
- FastDFS V6.06(应该 >=6.03)
- nginx(1.14.0)
- fastdfs-nginx-module V1.22(已通过nginx 1.16.1 测试)

# 1. 安装nginx且下载nginx官网源码
## 1.1 未安装nginx
请按照下面步骤安装nginx
### 1.1.1 安装nginx依赖
```
sudo apt-get update && sudo apt-get upgrade
sudo apt-get install build-essential
sudo apt-get install libtool
sudo apt-get install zlib1g-dev
sudo apt-get install libssl-dev
```

- `build-essential`安装了一组新的包,包括`gcc`,`g++`,`make`.
- GNU `libtool`是通用库支持脚本.它将共享库的使用隐藏在一个一致的可移植的接口后面.
- `pcre`是一个Perl库,包括perl兼容的正则表达式库.nginx的`http模块`使用pcre来解析`正则表达式`.
- `zlib`库提供了很多种压缩和解压缩的方式,nginx使用zlib对http包的内容进行gzip.
- `openssl`是一个强大的安全套接字层密码库,囊括主要的密码算法,常用的密钥和证书封装管理功能及`SSL`协议,并提供丰富的应用程序供测试或其他目的使用.nginx不仅支持http协议,还支持`https`(即在SSL协议上传输http)

### 1.1.2 安装nginx
```
sudo apt-get install nginx
```

## 1.2 已安装nginx
如果你之前的系统已经安装了nginx则下载与之前安装的版本一致的源码.查看nginx版本信息:
```
nginx -V
```
我的已安装nginx版本信息如下:
```
nginx version: nginx/1.14.0 (Ubuntu)
built with OpenSSL 1.1.1  11 Sep 2018
TLS SNI support enabled
configure arguments: --with-cc-opt='-g -O2 -fdebug-prefix-map=/build/nginx-GkiujU/nginx-1.14.0=. -fstack-protector-strong -Wformat -Werror=format-security -fPIC -Wdate-time -D_FORTIFY_SOURCE=2' --with-ld-opt='-Wl,-Bsymbolic-functions -Wl,-z,relro -Wl,-z,now -fPIC' --prefix=/usr/share/nginx --conf-path=/etc/nginx/nginx.conf --http-log-path=/var/log/nginx/access.log --error-log-path=/var/log/nginx/error.log --lock-path=/var/lock/nginx.lock --pid-path=/run/nginx.pid --modules-path=/usr/lib/nginx/modules --http-client-body-temp-path=/var/lib/nginx/body --http-fastcgi-temp-path=/var/lib/nginx/fastcgi --http-proxy-temp-path=/var/lib/nginx/proxy --http-scgi-temp-path=/var/lib/nginx/scgi --http-uwsgi-temp-path=/var/lib/nginx/uwsgi --with-debug --with-pcre-jit --with-http_ssl_module --with-http_stub_status_module --with-http_realip_module --with-http_auth_request_module --with-http_v2_module --with-http_dav_module --with-http_slice_module --with-threads --with-http_addition_module --with-http_geoip_module=dynamic --with-http_gunzip_module --with-http_gzip_static_module --with-http_image_filter_module=dynamic --with-http_sub_module --with-http_xslt_module=dynamic --with-stream=dynamic --with-stream_ssl_module --with-mail=dynamic --with-mail_ssl_module
```

## 1.3 下载nginx官网源码

### 1.3.2 下载nginx源码
从上面版本信息可以看到我安装的nginx是1.14.0版本的，所以我尽量下载一样的版本:
```
sudo apt-get install wget
sudo wget http://nginx.org/download/nginx-1.14.0.tar.gz
```
### 1.3.2 解压nginx源码包
```
sudo tar -zxvf nginx-1.14.0.tar.gz
```

# 2.fastdfs-nginx-module安装

我们现在要做的是将fastdfs模块添加到nginx,然后再重新编译一个新的nginx,<br>
这样新的nginx也就包含了fastdfs-nginx-module这个模块.<br>
github:[fastdfs-nginx-module](https://github.com/happyfish100/fastdfs-nginx-module)

## 2.1 克隆fastdfs-nginx-module:
```
sudo apt-get install git 
sudo git clone https://github.com/happyfish100/fastdfs-nginx-module.git
```
## 2.2 给nginx添加fastdfs模块
### 2.2.2 进入nginx源码目录
```
cd nginx-1.14.0
```
### 2.2.3 生成Makefile<br>
复制步骤 1.2 中`configure arguments:`后面的参数放到,添加在`./configure`后面,<br>
同时额外**添加** `--add-module=/root/fastdfs-nginx-module/src` <br>
**注意:**把上面的`/root/`替换为你存放`fastdfs-nginx-module`的路径.
```
./configure --with-cc-opt='-g -O2 -fdebug-prefix-map=/build/nginx-GkiujU/nginx-1.14.0=. -fstack-protector-strong -Wformat -Werror=format-security -fPIC -Wdate-time -D_FORTIFY_SOURCE=2' --with-ld-opt='-Wl,-Bsymbolic-functions -Wl,-z,relro -Wl,-z,now -fPIC' --prefix=/usr/share/nginx --conf-path=/etc/nginx/nginx.conf --http-log-path=/var/log/nginx/access.log --error-log-path=/var/log/nginx/error.log --lock-path=/var/lock/nginx.lock --pid-path=/run/nginx.pid --modules-path=/usr/lib/nginx/modules --http-client-body-temp-path=/var/lib/nginx/body --http-fastcgi-temp-path=/var/lib/nginx/fastcgi --http-proxy-temp-path=/var/lib/nginx/proxy --http-scgi-temp-path=/var/lib/nginx/scgi --http-uwsgi-temp-path=/var/lib/nginx/uwsgi --with-debug --with-pcre-jit --with-http_ssl_module --with-http_stub_status_module --with-http_realip_module --with-http_auth_request_module --with-http_v2_module --with-http_dav_module --with-http_slice_module --with-threads --with-http_addition_module --with-http_geoip_module=dynamic --with-http_gunzip_module --with-http_gzip_static_module --with-http_image_filter_module=dynamic --with-http_sub_module --with-http_xslt_module=dynamic --with-stream=dynamic --with-stream_ssl_module --with-mail=dynamic --with-mail_ssl_module --add-module=/root/fastdfs-nginx-module/src
```
生产Makefile如果缺少库就安装缺少的库,比如我就遇到,错误1:
```
./configure: error: the HTTP XSLT module requires the libxml2/libxslt
libraries. You can either do not enable the module or install the libraries.
```
执行:
```
sudo apt-get install libxslt-dev
```
遇到错误2:
```
./configure: error: the HTTP image filter module requires the GD library.
You can either do not enable the module or install the libraries.
```
执行:
```
sudo apt-get install libgd-dev
```
遇到错误3:
```
./configure: error: the GeoIP module requires the GeoIP library.
You can either do not enable the module or install the library.
```
执行:
```
sudo apt-get install libgeoip-dev
```
### 2.2.4 编译且替换nginx可执行文件
2.2.4.1 编译nginx<br>
注意:**不**要`make install`,否则就是覆盖安装.
```
make
```
make完之后在objs目录下就多了个nginx，这个就是新版本的程序了.

2.2.4.2 查找nginx可执行程序
```
find / -name "nginx" -type f -executable -print
```
找到三个nginx可执行
```
/etc/init.d/nginx
/root/nginx-1.14.0/objs/nginx
/usr/sbin/nginx
```
`/root/nginx-1.14.0/objs/nginx`是刚新编译生成的,用它来替换其他两个旧的nginx.

2.2.4.3 替换调原来的可执行nginx.
```
nginx -s quit
cp /etc/init.d/nginx  /etc/init.d/nginx.bak
cp /usr/sbin/nginx  /usr/sbin/nginx.bak
cp /root/nginx-1.14.0/objs/nginx  /etc/init.d/nginx
cp /root/nginx-1.14.0/objs/nginx  /usr/sbin/nginx
```

### 2.2.5 测试新的nginx程序是否正确
```
nginx -t
```
```
nginx: theconfiguration file /usr/local/nginx/conf/nginx.conf syntax is ok
nginx:configuration file /usr/local/nginx/conf/nginx.conf test issuccessful
```
### 2.2.6 重启nginx
```
systemctl restart nginx
```
### 2.2.7 查看ngixn版本极其编译参数
```
nginx -V
```
```
nginx version: nginx/1.14.0
built by gcc 7.4.0 (Ubuntu 7.4.0-1ubuntu1~18.04.1) 
built with OpenSSL 1.1.1  11 Sep 2018
TLS SNI support enabled
configure arguments: --with-cc-opt='-g -O2 -fdebug-prefix-map=/build/nginx-GkiujU/nginx-1.14.0=. -fstack-protector-strong -Wformat -Werror=format-security -fPIC -Wdate-time -D_FORTIFY_SOURCE=2' --with-ld-opt='-Wl,-Bsymbolic-functions -Wl,-z,relro -Wl,-z,now -fPIC' --prefix=/usr/share/nginx --conf-path=/etc/nginx/nginx.conf --http-log-path=/var/log/nginx/access.log --error-log-path=/var/log/nginx/error.log --lock-path=/var/lock/nginx.lock --pid-path=/run/nginx.pid --modules-path=/usr/lib/nginx/modules --http-client-body-temp-path=/var/lib/nginx/body --http-fastcgi-temp-path=/var/lib/nginx/fastcgi --http-proxy-temp-path=/var/lib/nginx/proxy --http-scgi-temp-path=/var/lib/nginx/scgi --http-uwsgi-temp-path=/var/lib/nginx/uwsgi --with-debug --with-pcre-jit --with-http_ssl_module --with-http_stub_status_module --with-http_realip_module --with-http_auth_request_module --with-http_v2_module --with-http_dav_module --with-http_slice_module --with-threads --with-http_addition_module --with-http_geoip_module=dynamic --with-http_gunzip_module --with-http_gzip_static_module --with-http_image_filter_module=dynamic --with-http_sub_module --with-http_xslt_module=dynamic --with-stream=dynamic --with-stream_ssl_module --with-mail=dynamic --with-mail_ssl_module --add-module=/root/fastdfs-nginx-module/src
```
`configure arguments`与步骤 2.2.3 中的`./configure`一致就OK.

## 2.3 配置 nginx的配置文件,比如 nginx.conf
```
vi /etc/nginx/nginx.conf
```
添加下面这段:
```
	location /M00 {
            root /home/yuqing/fastdfs/data;
            ngx_fastdfs_module;
        }
```

## 2.4 建立一个软链接 `${fastdfs_base_path}/data/M00` 到 `${fastdfs_base_path}/data`
请根据自己文件路径进行相应修改:
```
ln -s /home/yuqing/fastdfs/data  /home/yuqing/fastdfs/data/M00
```

## 2.5 复制源码fastdfs中的`conf/http.conf`和`conf/mime.types`到 `/etc/fdfs`
假设安装fastdfs时使用的是默认路径,如下:
```
cd $YOUR_PATH/fastdfs
cp conf/http.conf conf/mime.types /etc/fdfs/
```

## 2.6 复制 `mod_fastdfs.conf` 到 `/etc/fdfs/` 且修改它
```
cp $YOUR_PATH/fastdfs-nginx-module/src/mod_fastdfs.conf  /etc/fdfs/
```
修改`/etc/fdfs/mod_fastdfs.conf`
```
vi /etc/fdfs/mod_fastdfs.conf
```

- `tracker_server=tracker:22122` 改为自己的IP
- `store_path0=/home/yuqing/fastdfs` 跟 storaged 配置要一致.

## 2.7 重启nginx
```
/usr/local/nginx/sbin/nginx -s quit
/usr/local/nginx/sbin/nginx 
```

## 2.8 检测HTTP访问(浏览访问存储的数据)
客户端上传数据
```
/usr/bin/fdfs_test /etc/fdfs/client.conf upload /usr/include/stdlib.h
```
假设返回链接如下:
```
http://192.168.1.100/group1/M00/00/00/aIBdvl5Th7iAM1PwAACLyBo1AoQ37510.h
```
浏览器输入下面的URL就可以访问刚上传的 `stdlib.h`数据.
```
http://192.168.1.100/M00/00/00/aIBdvl5Th7iAM1PwAACLyBo1AoQ37510.h
```

参考:<br>
[happyfish100/fastdfs-nginx-module/INSTALL](https://github.com/happyfish100/fastdfs-nginx-module/blob/master/INSTALL)<br>
[nginx添加模块(非覆盖安装)](https://cnblogs.com/chaolinux/p/5473950.html)
