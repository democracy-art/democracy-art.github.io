---
layout:     post
title:      FastDFS扩展
subtitle:   Nginx和fastdfs-nginx-module安装实现HTTP访问
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
- FastDFS V6.06(版本应该 >=6.03)
- nginx(1.14.0)
- fastdfs-nginx-module V1.22(已通过nginx 1.16.1 测试)

# 1.Nginx安装
安装
```
 apt install nginx
```
设置开机启动及启动Nginx服务
```
 systemctl enable nginx.service
 systemctl start nginx.service
```
Ubuntu18.04安装的nginx版本是1.14.0
# 2.fastdfs-nginx-module安装
github:[fastdfs-nginx-module](https://github.com/happyfish100/fastdfs-nginx-module)

2.1 克隆 fastdfs-nginx-module:
```
apt install git 
git clone https://github.com/happyfish100/fastdfs-nginx-module.git
```
建议**关闭所有nginx进程**再进行fastdfs-nginx-module安装:
```
systemctl stop nginx
```
将`$YOUR_PATH`替换成你存放fastdfs-nginx-module的路径
2.2 依赖安装:
```
sudo apt-get update
sudo apt-get install libpcre3 libpcre3-dev
apt-get install zlib1g-dev
```
若依赖没安装会遇到下面错误提示:
```
./configure: error: the HTTP rewrite module requires the PCRE library.
...
./configure: error: the HTTP gzip module requires the zlib library.
...
```
2.3 安装 fastdfs-nginix-module
```
apt install wget
wget http://nginx.org/download/nginx-1.16.1.tar.gz
tar -zxvf nginx-1.16.1.tar.gz
cd nginx-1.16.1
./configure --add-module=$YOUR_PATH/fastdfs-nginx-module/src   
make && make install
```
2.4 配置 nginx的配置文件,比如 nginx.conf
```
vi /usr/local/nginx/conf/nginx.conf
```
添加下面这段:
```
location /M00 {
            root /home/yuqing/fastdfs/data;
            ngx_fastdfs_module;
        }
```

2.5 建立一个软链接 `${fastdfs_base_path}/data/M00` 到 `${fastdfs_base_path}/data`
请根据自己文件路径进行相应修改:
```
ln -s /home/yuqing/fastdfs/data  /home/yuqing/fastdfs/data/M00
```

2.6 复制源码fastdfs中的`conf/http.conf`和`conf/mime.types`到 `/etc/fdfs`
假设安装fastdfs时使用的是默认路径,则如下:
```
cd ~
git clone https://github.com/happyfish100/fastdfs.git
cd fastdfs
cp conf/http.conf conf/mime.types /etc/fdfs/
```

2.7 复制 `mod_fastdfs.conf` 到 `/etc/fdfs/` 且修改它
```
cp $YOUR_PATH/fastdfs-nginx-module/src/mod_fastdfs.conf  /etc/fdfs/
```
必须改的部分:
- `tracker_server=tracker:22122` 改为自己的IP
- `store_path0=/home/yuqing/fastdfs` 跟 storaged 配置要一致.

2.8 从新启动 nginx
```
systemctl restart nginx
```
查看 nginx 状态
```
systemctl status nginx
```

2.9 查看 nginx
```
tail -n 100 /usr/local/logs/error.log
```



