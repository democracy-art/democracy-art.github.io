--- 
layout: post
title: Set PAC of shadowsocks-libev on Kali Linux
subtitle:
date: 2020-04-02
author: D
header-img:
catalog: true
tags: [shadowsocks,pac]
---

**PAC: Proxy auto-config**

1.安装 `genpac`:
```
pip install genpac
pip install --upgrade genpac
```
2.生成PAC文件:
```
mkdir ~/.pac
cd ~/.pac
touch user-rules.txt # 可在此文件中添加用户自定义规则，此处省略
genpac --pac-proxy "SOCKS5 127.0.0.1:1080" --output="ProxyAutoConfig.pac" --user-rule-from="user-rules.txt"
```
3.设置系统自动代理:<br>
**设置->网络->网络代理**，方式改为**自动**, `Configuration URL`改为`file:///root/.pac/ProxyAutoConfig.pac`.<br>
**注意:** 我用的是`root`用户，如果**非**root用户请将`/root`改为`/home/<your username>`.

4.测试：<br>
打开浏览器，输入网址www.google.com看是否访问成功

5.更新:<br>
该方法存在的问题是如果你的 PAC 文件失效了，可能需要**重新**下载 PAC 文件，即重新执行第 **2** 步中的**genpac**步骤



