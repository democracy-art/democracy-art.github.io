---
layout:     post
title:      (My)SQL基础(5)-DCL
subtitle:   
date:       2020-03-02
author:     D
header-img: 
catalog: true
mermaid: true
tags:
    - [MySQL]
---

DCL语句主要是DBA用来管理系统中的对象权限时所使用,一般开发人员很少用.<br>

例子1:创建一个数据库用户z1,具有对sakila数据库中所有表的`SELECT/INSERT`权限:
```
GRANT SELECT，INSERT ON sakila.* TO ‘z1’@'localhost' IDENTIFIED BY '123';
```
例子2:由于权限变更,需将z1的权限变更,收回`INSERT`,只能对数据进行`SELECT`操作:
```
REVOKE INSERT ON sakila.* FROM 'z1'@'localhost';
```

[(My)SQL基础(4)-DQL](https://dm116.github.io/2020/03/02/sql-basic-dql/)<br>
