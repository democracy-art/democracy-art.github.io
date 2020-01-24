---
layout:     post
title:      kali window10 双系统修改kali grub设置默认启动window10为第一项
subtitle:   
date:       2020-01-24
author:     D
header-img: 
catalog: true
mermaid: true
tags:
    - linux_basic
---
### 1. 更新kali grub
```
update-grub
```

### 2. 修改kali grub
2.1 给予写权限
```
chmod +w /boot/grub/grub.cfg
```
2.2 修改 grub.cfg
```
vim /boot/grub/grub.cfg
```
找到:
```
### BEGIN /etc/grub.d/30_os-prober ###
menuentry 'Windows 10 (on /dev/sda1)' --class windows --class os $menuentry_id_option 'osprober-chain-3636428B36424BD5' {
	insmod part_msdos
	insmod ntfs
	set root='hd0,msdos1'
	if [ x$feature_platform_search_hint = xy ]; then
	  search --no-floppy --fs-uuid --set=root --hint-bios=hd0,msdos1 --hint-efi=hd0,msdos1 --hint-baremetal=ahci0,msdos1  3636428B36424BD5
	else
	  search --no-floppy --fs-uuid --set=root 3636428B36424BD5
	fi
	parttool ${root} hidden-
	drivemap -s (hd0) ${root}
	chainloader +1
}
### END /etc/grub.d/30_os-prober ###
```
剪切，把它放在 kali 的前面，放在 theme 的后面，即如下：
```
### END /etc/grub.d/05_debian_theme ###

### BEGIN /etc/grub.d/30_os-prober ###
menuentry 'Windows 10 (on /dev/sda1)' --class windows --class os $menuentry_id_option 'osprober-chain-3636428B36424BD5' {
	insmod part_msdos
	insmod ntfs
	set root='hd0,msdos1'
	if [ x$feature_platform_search_hint = xy ]; then
	  search --no-floppy --fs-uuid --set=root --hint-bios=hd0,msdos1 --hint-efi=hd0,msdos1 --hint-baremetal=ahci0,msdos1  3636428B36424BD5
	else
	  search --no-floppy --fs-uuid --set=root 3636428B36424BD5
	fi
	parttool ${root} hidden-
	drivemap -s (hd0) ${root}
	chainloader +1
}
### END /etc/grub.d/30_os-prober ###

### BEGIN /etc/grub.d/10_linux ###
function gfxmode {
	set gfxpayload="${1}"
}
set linux_gfx_mode=
export linux_gfx_mode
menuentry 'Kali GNU/Linux' --class kali --class gnu-linux --class gnu --class os $menuentry_id_option
...
```
保存.

### 3 重启
```
reboot
```


