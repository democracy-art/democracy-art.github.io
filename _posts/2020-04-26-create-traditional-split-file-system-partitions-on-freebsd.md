--- 
layout: post
title: Create Traditional Split File System Partitions on FreeBSD(12.1)
subtitle:
date: 2020-04-26
author: D
header-img:
catalog: true
tags: [FreeBSD]
---

I have SSD+HDD(SSD 112GB, HDD 963GB), Partition as following:

|Partition Type|Size|Mountpoint|Lable|SSD/HDD|
|-|-|-|-|-|
|freebsd-boot|512K|||SSD|
|freebsd-ufs|2G|/|exrootfs|SSD|
|freebsd-swap|16G||exswap|HDD|
|freebsd-ufs|110G|/usr|exusrfs|SSD|
|freebsd-ufs|16G|/var|exvarfs|HDD|
|freebsd-ufs|2G|/tmp|extmpfs|HDD|
|freebsd-ufs|930G|/home|exhomefs|HDD|

reference:[Creating Traditional Split File System Partitions](https://www.freebsd.org/doc/en_US.ISO8859-1/books/handbook/bsdinstall-partitioning.html#bsdinstall-part-manual-splitfs)
