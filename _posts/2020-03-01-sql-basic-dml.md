---
layout:     post
title:      (My)SQL基础(3)-DML
subtitle:   
date:       2020-03-01
author:     D
header-img: 
catalog: true
mermaid: true
tags:
    - [MySQL]
---

# 1. DML语句简介
DML操作是指对数据库中表记录的操作,主要包括表记录的插入(`insert`),更新(`update`),删除(`delete`)和查询(`select`),是开发人员日常使用最频繁的操作.

# 2. 插入记录
语法:
```
INSERT INTO tablename(field1,field2,...,fieldn) VALUES(value1,value2,...,valusen);
```
例子:向表`emp`插入记录:`ename`为`zzx1`,`hiredate`为`2000-01-01`,`sal`为`2000`,`deptno`为`1`.
```
INSERT INTO emp (ename,hiredate,sal,deptno) VALUES('zzx1','2000-01-01','2000',1);
```
[(My)SQL基础(2)](https://dm116.github.io/2020/03/01/sql-basic-ddl/)<br>
