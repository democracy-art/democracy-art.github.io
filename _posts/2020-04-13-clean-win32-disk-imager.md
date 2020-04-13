--- 
layout: post
title: win32 disk imager使用后u盘容量恢复
subtitle:
date: 2020-04-13
author: D
header-img:
catalog: true
tags: [linux_basic,win32diskimager]
---


1.打开windows10终端 <kbd>Win</kbd>+<kbd>R</kbd> 键,输入 `cmd` 后弹出终端.

2.终端输入:
```
DISKPART
```
3.输入 
```
LIST DISK
```
查看电脑磁盘

4.选择磁盘,假设U盘的磁盘号为 **2**, 则
```
SELECT DISK 2
```
5.清空磁盘
```
CLEAN
```
6.创建主磁盘分区
```
CREATE PARTITION PRIMARY
```
7.激活磁盘分区
```
ACTIVE
```
8.格式化磁盘
```
FORMAT FS=FAT32 QUICK
```

参考:[win32 disk imager使用后u盘容量恢复](https://blog.csdn.net/u012400584/article/details/55506527)
