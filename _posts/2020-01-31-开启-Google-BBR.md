---
layout:     post
title:      开启 Google BBR
subtitle:   
date:       2020-01-31
author:     D
header-img: 
catalog: true
mermaid: true
tags:
    - shadowsocks
---

参考:[Ubuntu 18.04 / CentOS 7开启Google BBR](https://www.24kplus.com/linux/150.html)<br>

**BBR**
>Google 开源了其 TCP BBR 拥塞控制算法，并提交到了 Linux 内核，从 4.9 开始，Linux 内核已经用上了该算法。根据以往的传统，Google 总是先在自家的生产环境上线运用后，才会将代码开源，此次也不例外。 根据实地测试，在部署了最新版内核并开启了 TCP BBR 的机器上，网速甚至可以提升好几个数量级。

# 开启BBR
运行 `lsmod | grep bbr`，如果结果中没有 `tcp_bbr`，则先运行：
```
modprobe tcp_bbr
echo "tcp_bbr" >> /etc/modules-load.d/modules.conf
```
运行：
```
echo "net.core.default_qdisc=fq" >> /etc/sysctl.conf
echo "net.ipv4.tcp_congestion_control=bbr" >> /etc/sysctl.conf
```
运行：
```
sysctl -p
```
保存生效。运行：
```
sysctl net.ipv4.tcp_available_congestion_control
sysctl net.ipv4.tcp_congestion_control
```
若均有bbr，则开启BBR成功。
