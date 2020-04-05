--- 
layout: post
title: Modify repository of FreeBSD that machine is in China
subtitle:
date: 2020-03-28
author: D
header-img:
catalog: true
tags: [FreeBSD]
---

# 1. Create directory

```
mkdir -p /usr/local/etc/pkg/repos
ee /usr/local/etc/pkg/repos/FreeBSD.conf
```
content of FreeBSD.conf
```
FreeBSD: { enabled: no } // disabled file /etc/pkg/FreeBSD.conf
taiwan:{
	url: "pkg+http://pkg0.twn.freebsd.org/${ABI}/quarterly",
	mirror_type: "srv",
	signature_type: "none",
	fingerprints: "/usr/share/keys/pkg",
	enabled: yes	
}
ustc:{
	url: "pkg+http://mirrors.ustc.edu.cn/freebsd-pkg/${ABI}/quarterly", // sometime will time out if you use ustc
	mirror_type: "srv",
	signature_type: "none",
	fingerprints: "/usr/share/keys/pkg",
	enabled: no	
}
```
# 2. Update pkg
```
pkg update -f
```
To update the index to make it take effect.

After that you can try to install `axel`.
```
pkg install axel
```
`axel` is a multi-threaded download tool that will be used when modifying the ports repository.

If install `axel` failed. You should try to update your system time use command 'date'.

# 3. Modify portsnap repository 

```
ee /etc/portsnap.conf
```
Content of portsnap.conf
```
SERVERNAME=portsnap.tw.freebsd.org
```
And then run command:
```
portsnap fetch
```
To get porsts' directory.

Because it is the first time use portsnap, you need to run command:
```
portsnap extract
portsnap update
```
In the future updating the ports directory, you can directly use command:
```
portsnap fetch update
```

# 4. Modify the ports repository 

```
ee /etc/make.conf
```
Content of make.conf
```
FETCH_CMD=axel -n 30 -a  # -n 30 means use 30 threads for download
DISABLE_SIZE=yes
MASTER_SIZE_OVERRIDE?=http://mirrors.ustc.edu.cn/freebsd-ports/distfiles/${DIST_SUBDIR}/
```
Install the ports upgrade tool portmaster to try the effect:
```
cd /usr/ports/ports-mgmt/portmaster
make install clean
```
portmaster is an upgrade tool for ports.
```
portmaster -a
```
References:

[FreeBSD更换国内源(pkg源使用台湾源，中科大源备用）](https://www.cnblogs.com/liujingli1986/p/11774738.html)
