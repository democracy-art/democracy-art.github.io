--- 
layout: post
title: FreeBSD改台湾源
subtitle:
date: 2020-06-14
author: D
header-img:
catalog: true
tags: [FreeBSD]
---

# 1.修改pkg源

原本源的路径 `/etc/pkg/FreeBSD.conf`

**1.1 创建用户级pkg源目录**
```
# mkdir -p /usr/local/etc/pkg/repos
# ee /usr/local/etc/pkg/repos/FreeBSD.conf
```
`/usr/local/etc/pkg/repos/FreeBSD.conf`的内容如下:
```
FreeBSD.conf:{ enabled: no } //禁用FreeBSD原本的源
taiwan:{
	url: "pkg+http://pkg.tw.freebsd.org/${ABI}/quarterly",
	mirror_type: "srv",
	signature_type: "fingerprints",
	fingerprints: "/usr/share/keys/pkg",
	enabled: yes; //表示启用
}
```
**1.2 更新源**
```
# pkg update -f
```
**1.3 测试国内源**
```
# pkg install axel
```
axel是下面修改ports源时会用到的一个多线程下载工具

# 2.修改portsnap源
```
# ee /etc/portsnap.conf
```
内容如下
```
SERVERNAME=portsnap.tw.freebsd.org
```
之后运行
```
# portsnap fetch
```
以获取ports目录
因为是第一次用portsnap,之后还需要
```
# portsnap extract
# portsnap update
```
以后更新ports目录,直接
```
# portsnap fetch update
```
就可以了.

# 3.修改ports源
**3.1 编辑`/etc/make.conf`**
```
# ee /etc/make.conf
```
make.conf内容如下
```
FETCH_CMD=axel -n 30 -a  # -n 30 表示使用30个线程下载
DISABLE_SIZE=yes
MASTER_SITE_BACKUP?=\
http://ftp.tw.freebsd.org/pub/FreeBSD/distfiles/${DIST_SUBDIR}/
MASTER_SITE_OVERRIDE?=${MASTER_SITE_BACKUP}
```
**3.2 安装ports升级工具portmaster**
```
# cd /usr/ports/ports-mgmt/portmaster
# make install clean
```
portmaster是ports的升级工具,使用基本上就用:
```
# portmaster -a
```

Reference:[設定FreeBSD update & ports & package使用local mirror server](https://www.peterdavehello.org/2014/05/config-freebsd-update-ports-package-use-local-mirror-server/)
