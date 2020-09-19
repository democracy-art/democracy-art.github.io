--- 
layout: post
title: FreeBSD12.0 安裝防火牆PF(Packet Filter) 
subtitle:
date: 2020-09-19
author: D
header-img:
catalog: true
tags: [FreeBSD, PF, Packet Filter]
---

# 1.創建防火牆PF的配置文件
```
# vi /etc/pf.conf
```

1.1 配置內容及解析如下:
```
# 設置宏變量, 引用時在變量名前加美元符合即可 
vtnet0 = "vtnet0"                 # 虛擬網卡接口 vtnet0 

# ICMP是联网设备用于各种类型通信的多用途消息传递协议。 例如，ping实用程序使用一种称为
# 回显请求(echoreq)的消息类型，该消息已添加到icmp_type列表中。
# 分段的另一个重要方面是称为最大传输单位（MTU）的术语。 TCP / IP协议使设备能够协商用于建立连接的数据包大小。 
# 目标主机使用ICMP消息将其MTU通知源IP，此过程称为MTU路径发现。 特定的ICMP消息类型是目标不可访问。 
# 您可以通过将未到达消息(unreach)类型添加到icmp_types列表中来启用MTU路径发现。
icmp_types = "{ echoreq unreach}" # icmp協議(ping命令使用的協議)

# persist : 关键字允许规则集中存在一个空表。 如果没有它，PF将抱怨表中没有IP地址。
table <bruteforce> persist        # 記錄嘗試暴力破解服務器ssh的ip名單表

#table <webcrawlers> persista     # 記錄多次連接服務器網站的ip名單表,如果服務器沒有網站則用不到

# 让我们为不可路由的IP地址创建一个表，该表通常在拒绝服务攻击（DOS）中起作用
# 你也可以使用RFC6890中指定的IP地址，该RFC6890定义了专用IP地址注册表
# 您的服务器不应通过面向公众的接口向这些地址发送数据包或从这些地址接收数据包 
table <rfc6890> { 0.0.0.0/8 10.0.0.0/8 100.64.0.0/10 127.0.0.0/8 169.254.0.0/16          \
                  172.16.0.0/12 192.0.0.0/24 192.0.0.0/29 192.0.2.0/24 192.88.99.0/24    \
                  192.168.0.0/16 198.18.0.0/15 198.51.100.0/24 203.0.113.0/24            \
                  240.0.0.0/4 255.255.255.255/32 }

# 爲回送设备(loopback)创建一个设置跳过规则，因为它不需要过滤流量，并且很可能使您的服务器爬网.
set skip on lo0

# scrub : 用于规范数据包; fragment reassemble(片段重新)组装选项，该选项可防止片段进入系统
# max-mss 1440 : 该选项表示重新组合的TCP数据包（也称为有效负载）的最大段大小。
scrub in all fragment reassemble max-mss 1440

# antispoof : 反ip欺骗规则表示，来自vtnet0网络的所有流量只能通过vtnet0接口，否则将使用quick关键字立即将其丢弃。
antispoof quick for $vtnet0

# return : 選項是對 block out(阻擋)規則的補充.这将丢弃数据包，
# 还将向尝试建立这些连接的主机发送RST消息，这对于分析主机活动非常有用。
# egress : 关键字将自动在任何给定接口上找到默认路由。 这通常是一种查找默
# 认路由的更干净的方法，尤其是在复杂的网络中。
# quick : 关键字可立即执行规则，而无需考虑其余规则集。 例如，如果IP地址不正确
# 的数据包试图连接到服务器，则您希望立即断开连接，并且没有理由在规则集的其余部分中运行该数据包。
# in : 進到服務器的  out: 從服務器出去的  from : 從哪裏來的 to : 到哪裏去的 
block in quick on $vtnet0 from <rfc6890>
block return out quick on egress to  <rfc6890>

# 先阻擋所有流量流入流出，再去精準指定允許流入和流出
block all

# 使用PF，可以限制单个主机允许的極短時間內(如1秒內)连接尝试次数。如果主机超出这些限制，则连接将被断开，
# 并且将禁止服务器连接。 为此，您将使用PF的过载机制，该机制维护一个被禁止IP地址的表。
# keep state : 该选项允许您定义过载表的状态条件。
# max-src-conn : 指定每秒单个主机允许的同時连接数; max-src-conn-rate : 指定每秒从单个主机允许的"新"连接数
# 如果主机超过了这些限制，则 overload (过载)机制会将源IP添加到 <bruteforce> 表中，从而将其禁止在服务器中使用。 
# 最后，flush global 选项立即删除连接。
pass in on $vtnet0 proto tcp to port { 22 } keep state (max-src-conn 5, max-src-conn-rate 1/1, overload <bruteforce> flush global)

# 保護網站的端口,第一條規則比較嚴格,現在是啓用第二條
# pass in on $vtnet0 proto tcp to port { 80 433 } keep state (max-src-conn 45, max-src-conn-rate 9/1, overload <webcrawlers> flush global)
pass in on $vtnet0 proto tcp to port { 80 443 }

# 下面兩條規則時允許從服務器往外的流量:
# 協議對應的端口 SSH:22 DNS:53 HTTP:80 NTP:123 HTTPS:443
# 规则集允许出站SSH，DNS，HTTP，NTP和HTTPS通信，并阻止所有流入流量.  
# 允許端口53和123上的UDP通过，因为DNS和NTP经常在TCP和UDP协议之间切换.
pass out proto { tcp udp } to port { 22 53 80 123 443 }
# inet : 代表IPv4地址家族.
pass inet proto icmp icmp-type $icmp_types
```

1.2 檢查PF配置內容是否有語法錯誤,运行以下pfctl命令以进行空运行：
```
# pfctl -nf /etc/pf.conf
```

# 2.管理过载表,比如 brute-force 過載表

2.1 您将使用pfctl通过以下命令手动清除过载表中存储48小时或更长时间的IP地址：
```
# pfctl -t bruteforce -T expire 172800
```
>它也許會輸出 0/0 addresses expired.

2.2 編寫自動清除過載表腳本
```
# vi /usr/local/bin/clear_overload.sh
```
過載表清除腳本內如下:
```
#!/bin/sh
pfctl -t bruteforce -T expire 172800
```
修改腳本權限
```
chmod 755 /usr/local/bin/clear_overload.sh
```
創建  cron job
```
# crontab -e
```
添加下面內容到crontab文件
```
# minute    hour    mday    month   wday    command
*           0       *       *       *       /usr/local/bin/clear_overload.sh
```
# 3.使能PF
```
# sysrc pf_enable=yes
# sysrc pflog_enable=yes
```
從新啓動 pf
```
# service pf restart
```
