---
layout:     post
title:      nginx添加fastdfs模块(非覆盖安装)
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
- FastDFS V6.06(版本应该 >=6.03)
- nginx(1.16.1)
- fastdfs-nginx-module V1.22(已通过nginx 1.16.1 测试)

# 1.安装nginx

[ubuntu18.04安装nginx1.16.1(自定义安装)](https://dm116.github.io/2020/02/25/install-nginx-on-ubuntu1804/)

# 2.fastdfs-nginx-module安装

github:[fastdfs-nginx-module](https://github.com/happyfish100/fastdfs-nginx-module)

2.1 克隆 fastdfs-nginx-module:
```
apt install git 
git clone https://github.com/happyfish100/fastdfs-nginx-module.git
```
建议**关闭所有nginx进程**再进行fastdfs-nginx-module安装.<br>
2.2 给nginx添加fastdfs模块
```
cd nginx-1.16.1
./configure --add-module=$YOUR_PATH/fastdfs-nginx-module/src   
make
```
注意:如果之前**已经**安装过nginx,**不**要`make install`,否则就是覆盖安装.

2.3 配置 nginx的配置文件,比如 nginx.conf
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

2.4 建立一个软链接 `${fastdfs_base_path}/data/M00` 到 `${fastdfs_base_path}/data`
请根据自己文件路径进行相应修改:
```
ln -s /home/yuqing/fastdfs/data  /home/yuqing/fastdfs/data/M00
```

2.5 复制源码fastdfs中的`conf/http.conf`和`conf/mime.types`到 `/etc/fdfs`
假设安装fastdfs时使用的是默认路径,如下:
```
cd fastdfs
cp conf/http.conf conf/mime.types /etc/fdfs/
```

2.6 复制 `mod_fastdfs.conf` 到 `/etc/fdfs/` 且修改它
```
cp $YOUR_PATH/fastdfs-nginx-module/src/mod_fastdfs.conf  /etc/fdfs/
```
必须改的部分:
- `tracker_server=tracker:22122` 改为自己的IP
- `store_path0=/home/yuqing/fastdfs` 跟 storaged 配置要一致.

2.7 重启nginx
```
/usr/local/nginx/sbin/nginx -s quit
/usr/local/nginx/sbin/nginx 
```

参考:<br>
[happyfish100/fastdfs-nginx-module/INSTALL](https://github.com/happyfish100/fastdfs-nginx-module/blob/master/INSTALL)<br>
[nginx添加模块(非覆盖安装)](https://cnblogs.com/chaolinux/p/5473950.html)
