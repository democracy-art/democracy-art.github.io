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
## 2.1 单条记录
语法:
```
INSERT INTO tablename(field1,field2,...,fieldn) VALUES(value1,value2,...,valusen);
```
例子1:向表`emp`插入记录:`ename`为`zzx1`,`hiredate`为`2000-01-01`,`sal`为`2000`,`deptno`为`1`.
```
INSERT INTO emp (ename,hiredate,sal,deptno) VALUES('zzx1','2000-01-01','2000',1);
```
例子2:含空字段,非空但是含默认值的字段,自增字段.
```
INSERT INTO emp (ename,sal) VALUES('dony',1000);
```
>注意:例2的VALUES后面的顺序要和字段排列顺序一致.

查看表`emp`记录:
```
select * from emp;
```
## 2.2 多条记录
语法:
```
INSERT INTO tablename (field1,field2,...,fieldn)
VALUES
(record1_value1,record1_value2,...,record1_valuen),
(record2_value1,record2_value2,...,record2_valuen),
...
(recordn_value1,recordn_value2,...,recordn_valuen);
```
例子3:
```
INSERT INTO dept values(5,'dept5'),(6, 'dept6');
```
[(My)SQL基础(2)](https://dm116.github.io/2020/03/01/sql-basic-ddl/)<br>
