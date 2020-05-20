---
layout: post
title:  Delete Files and Directories Permanently and Securely in Linux
subtitle:
date: 2020-03-27
author: D
header-img:
catalog: true
tags: [linux_basic]
---

**删除**是删除数据最便捷的方法，如 Linux 用户最经常采用`rm`删除命令。实际上并没有真正的将数据从硬盘上删除，只是将文件的索引删除而已，让操作系统和使用者认为文件已经删除，又可以把腾出空间存储新的数据。数据恢复**极易恢复**此类不见的数据，而且也有很多专门进行数据恢复的软件。

彻底删除的原理:磁盘可以重复使用，前面的数据被后面的**数据覆盖**后，前面的数据被还原的可能性就大大降低了，随着被覆盖次数的增多，能够被还原的可能性就趋于 0，但相应的时间支出也就越多.

覆盖原理（overwriting）:覆盖是指使用预先定义的格式——**无意义**、**无规律**的信息来覆盖硬盘上原先存储的数据.

# wipe
这是迄今为止最彻底的数据删除方法，覆盖数据 **35** 次使得任何还原数据的企图都是徒劳的。。随着硬盘容量愈来愈大，重写时间也会愈来愈长，所以 wipe 也提供了只重复写入 4 次的快速模式。


## 安装
Debian/Ubuntu
```
apt-get install wipe
```
Centos7
```
wget http://ftp.tu-chemnitz.de/pub/linux/dag/redhat/el7/en/x86_64/rpmforge/RPMS/rpmforge-release-0.5.3-1.el7.rf.x86_64.rpm
```
```
rpm -Uvh rpmforge-release*rpm
```
```
yum install wipe
```
## 删除

1.删除文件
```
wipe file.txt
```
2.删除目录
```
wipe -r /home/user/test/
```
3.删除整个分区
```
wipe /dev/sdb1
```

reference:
[3 Ways to Permanently and Securely Delete ‘Files and Directories’ in Linux](https://www.tecmint.com/permanently-and-securely-delete-files-directories-linux/)
