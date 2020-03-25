---
layout: post
title: Chrome设置PAC模式无效的解决方案
subtitle:
date: 2020-03-24
author: D
header-img:
catalog: true
tags: [shadowsocks,linux_basic]
---

1. 发现问题
以kali2019系统为例，我们通过genpac生成autoproxy.pac文件，然后点击系统设置->网络->代理设置->自动，在输入框中输入file://绝对路径/autoproxy.pac。设置好以后，Chrome应当可以自动切换网络，但是Chrome无法访问google的搜索引擎，而火狐浏览器可以正常访问。

2. 分析问题
出现上面问题的唯一可能就是Chrome设置有误，因此点击设置->高级设置->打开代理设置，打开的就是Ubuntu系统的网络设置。因此，我们可以判断自动代理模式设置失效，这里我们就要仔细分析为什么会失效？

3. 解决方案
出现上面问题的主要原因是：Chrome移除对file://和data:协议的支持，目前只能使用http://协议。因此，我们打算使用nginx实现对本地文件的http映射。

3.1 安装nginx
```
apt install nginx
```
3.2 修改nginx.conf配置文件
```
vi /etc/nginx/nginx.conf
```
在nginx.conf的 `http{…}`代码块中加入:
```
server{
    listen 80; #注意这里不用":"隔开，listen后面没有冒号, 这里端口号一定为80，pac文件中代理的端口号可以为1080等
    server_name 127.0.0.1; #注意这里不用":"隔开，server_name后面没有冒号
    location /autoproxy.pac {
        alias 绝对路径/autoproxy.pac;
    }
}
```
3.3 替换系统代理中大 autoproxy.pac 路径

填写到系统设置->网络->代理设置->自动代理中:
```
http://127.0.0.1/autoproxy.pac 
```
3.4 重启nginx
```
systemctl restart nginx
```

3.5 给chrome设置别名
```
vi ~/.bashrc
```
在 `~/.bashrc` 中添加:
```
alias chrome='google-chrome --proxy-server="socks://127.0.0.1:1080" --no-sandbox --user-data-dir'
```
3.6 启动 chrome
```
chrome
```

参考：[Chrome设置PAC模式无效的解决方案](https://blog.csdn.net/u013241245/article/details/100713859)
