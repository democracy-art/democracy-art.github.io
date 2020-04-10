--- 
layout: post
title: 安装kali linux在固态硬盘(SSD 120GB)+机械硬盘(HDD 1TB)如何分区?
subtitle:
date: 2020-04-10
author: D
header-img:
catalog: true
tags: [linux_basic,SSD,HDD]
---

想要安装kali linux在自己的台式电脑上，电脑本身有一块固态硬盘(120GB)和一块机械硬盘(1TB).<br>
我没有打算装双系统,我是换一种解决方案，在kali里面装virtualbox，然后在virtualbox里面再装<br>
windows系统,然后再在windows里面装一些windows才方便装的软件.

**因为kali跟ubuntu同是来自Debian,所以这里的安装对ubuntu同样适用.**

这里只写主要的步骤,以及关键点:

- 使用`Win32 Disk imager`将系统烧入U盘
- 设置电脑从U盘启动
- 安装时选择系统语言`English`,因为文件目录是中文,进入文件目录还要切换输入法,很烦. 
- **分区**是重点

**分区**
分区依据的原则是**读取频繁**的放在SSD，**写入频繁**的放在HDD，不清楚的放在HDD.

固态硬盘(SSD 120GB):
- `EFI`: 分配 **64MB**, (系统提示要大于等于35MB)(我是强制使用UEFI)
- `/boot`: 分配 **512MB** (其实这里用不了那么多,我装好后看我的`/boot`才用了140MB,那么这样看来256MB足够了), 系统引导,只是在系统启动之后就没有用了.(读取多，放SSD) 
- `/usr`: 分配 **80GB** 这里存放很多可执行文件在`/usr/bin`读取多
- `/usr/local`: **10GB** 这里也是存放可执行文件比较多,其实Centos和FreeBSD的蛮多可执行文件都是放这里.
- `/root`: SSD剩下的空间全给它

机械硬盘(HDD 1TB):
- 
- `swap`: 分配 **8GB**, 因为我的内存是8GB,那为什么不把swap放在SSD呢？其实放在SSD也行。我说一下我放HDD的理由.早期的SSD写入失败的情况比HDD要多,现在的SSD比以前的好多了，所以我说放在SSD也行。因为,我的内存是8GB，运行程序一般很少用到swap，所以会有点浪费，所以我把swap放在HDD。当然如果你的内存够大，可以只分配1-2GB，swap就够了.  
- `/tmp`: 分配 **10GB**
- `/var`: 分配 **150GB**,这里存放log,lib,cache,www,mail,backups需要空间比较home要大
- `/home`: 分配 **100GB** 
- `/data`: 分配 HDD余下的空间,可以存放视频，音乐等等.

如果显示写入磁盘失败，从新启动再进入.
