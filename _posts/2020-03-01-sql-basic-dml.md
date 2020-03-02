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
# 3. 更新记录
## 3.1 单表更新
语法:
```
UPDATE tablename SET field1=value1, field2=value2, ..., fieldn=valuen [WHERE CONDITION];
```
例子:将表`emp`中`ename`为lisa的薪水(`sal`)从3000更改为4000.
```
UPDATE emp SET sal=4000 WHERE ename='lisa';
```
## 3.2 多表更新
语法:
```
UPDATE t1,t2,...,tn SET t1.field1=expr1,tn.fieldn=exprn [WHERE CONDITION];
```
例子:更新表emp中字段sal和表dept中的字段deptname. (例子中用了表的**别名**) 
```
UPDATE emp a, dept b SET a.sal=a.sal*b.deptno, b.deptname=a.ename WHERE a.deptno=b.deptno;
```

>注意:多表更新的语法更多地用在根据一个表的字段,来动态的更新另外一个表的字段.

# 4. 删除记录
## 4.1 删除单表记录
语法:
```
DELETE FROM tablename [WHERE CONDITION];
```
例子:将表emp中ename为`dony`的记录全部删除.
```
DELETE FROM emp WHERE ename='dony';
```
## 4.2 删除多表记录
语法:
```
DELETE t1,t2,...,tn FROM t1,t2,...,tn [WHERE CONDITION];
```
例子:将表emp和 dept中deptno为3的记录同时都删除.(例子中用了表的**别名**)
```
DELETE a, b from emp a, dept b WHERE a.deptno=b.deptno AND a.deptno=3;
```

>注意:不管是单表还是多表,不加WHERE条件将会把表的所有记录删除,所以操作时一定要小心.

[(My)SQL基础(2)](https://dm116.github.io/2020/03/01/sql-basic-ddl/)<br>
[(My)SQL基础(4)](https://dm116.github.io/2020/03/02/sql-basic-dql/)<br>
