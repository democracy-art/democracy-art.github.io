---
layout:     post
title:      绑定域名到IP
subtitle:
date:       2020-02-19
author:     D
header-img: 
catalog: true
mermaid: true
tags:
    - website
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

**假设注册的域名为:** `xxx.tk`

# 3.获取 Cloudflare 的 DNS
- 登录Cloudflare
- 点 `+Add site`
- 输入在freenom注册的域名
- 来到另一个页面 显示We're querying your DNS records 点击 `Next`
- 来到另一个页面 Select a Plan 选择 FREE $0/month 点击 `Confire Plan`
- Add DNS records:(2条记录)

|Type|Name|value|TTL|Proxy status|
|-|-|-|-|-|
|A|www|IPv4地址(例如:192.168.1.8)|Automatic TTL|DNS only|
|A|@|IPv4地址(例如:192.168.1.8)|Automatic TTL|DNS only|

填完一条记录就点击 `Add Record`.

-  点击`Continue`
-  来到一个页面显示你当前域名使用的DNS服务器和Cloudflare要你使用的它的DNS服务器 

复制且保存 Replace with Cloudflare’s nameservers:<br>
```
xxx.ns.cloudflare.com
yyy.ns.cloudflare.com
```

- 复制保存好后 点击 Done,check nameservers

# 4.设置DNS

- 登录freenom
- Services标签
- My Domains
- Manage Domain
- Management Tools
- Nameservers 
- Use custom nameservers (enter below)输入cloudflare的DNS,
```
Nameserver 1: xxx.ns.cloudflare.com 
Nameserver 2: yyy.ns.cloudflare.com 
```
- 点击 `Change Nameservers`
- 24小时之内更新好DNS服务器,一般10分钟内.

- 登录cloudflare 看到域名 xxx.tk 下面的 `ACTIVE` 打勾 说明成功

