---
layout:     post
title:      (My)SQL基础(2)-DDL
subtitle:   
date:       2020-03-01
author:     D
header-img: 
catalog: true
mermaid: true
tags:
    - [MySQL]
---

# 1. DDL(Data Definition Languages)介绍

DDL就是对数据库内部的对象进行创建、删除、修改的操作语言.它和 DML 语言的最大区别是 <br>
DML 只是对表内部数据的操作，而不涉及到表的定义、结构的修改，更不会涉及到其他对象。<br>
DDL 语句更多的被数据库管理员（DBA）所使用，一般的开发人员很少使用

# 2. 创建数据库
### 2.1 登录mysql
```
mysql -u root -p
```
登录后输入密码成功则显示如下:
```
MariaDB [(none)]>
```

### 2.2 创建数据库
2.2.1 创建
```
CREATE DATABASE test1;
```
2.2.2 显示系统中有什么数据库
```
SHOW DATABASES;
```
2.2.3 选择要操作的数据库
```
USE test1;
```
2.2.4 查看`test1`数据库中创建的所有数据表:
```
show tables;
```

# 3. 删除数据库
```
DROP DATABASE test1;
```

>注意:删除数据库这个动作很危险，请务必先做好备份.

# 4. 创建表
### 4.1 表基本语法及创建表例子
在数据库中创建一张表的基本语法:
```
CREATE TABLE tablename (column_name_1 column_type_1 constraints，
column_name_2 column_type_2 constraints ， ……column_name_n column_type_n
constraints）
```
MySQL的表名是以目录的形式存在于磁盘上，所以表名的字符可以用任何目录名允许的字符.
- column_name 列的名字
- column_type 列的数据类型
- contraints 列的约束条件

例子:
```
create table emp(ename varchar(10),hiredate date,sal decimal(10,2),deptno int(2));
```
查看表信息<br>
方式1:
```
DESC tablename;
```
方式2:
```
show create table tablename \G;
```

# 5. 删除表
```
DROP TABLE tablename;
```
# 6. 修改表
### 6.1 修改表类型
表结构的更改一般使用`ALTER` table 语句.
```
ALTER TABLE tablename MODIFY [COLUMN] column_definition [FIRST|AFTER col_name]
```
例子: `ename varchar(10)` to `ename varchar(20)`
```
desc emp;
+----------+---------------+------+-----+---------+-------+
| Field | Type | Null | Key | Default | Extra |
+----------+---------------+------+-----+---------+-------+
| ename | varchar(10) | YES | | | |
| hiredate | date | YES | | | |
| sal | decimal(10,2) | YES | | | |
| deptno | int(2) | YES | | | |
+----------+---------------+------+-----+---------+-------+
```
ALTER:
```
ALTER TABLE emp MODIFY ename varchar(20);
```
### 6.2 增加表字段
语法
```
ALTER TABLE tablename ADD [COLUMN] column_definition [FIRST|AFTER col_name]
```
例子: 表 emp 新增 `age`类型为`int(3)`
```
ALTER TABLE emp ADD column age int(3);
```
### 6.3 删除表字段
语法:
```
 ALTER TABLE tablename DROP [COLUMN] col_name;
```
例子:
```
ALTER TABLE emp DROP COLUMN age;
```
### 6.4 字段改名
语法:
```
ALTER TABLE tablename CHANGE [COLUMN] old_col_name column_definition [FIRST|AFTER col_name];
```
例子: 将 `age` 改为 `age1`,同时字段类型改为 `int(4)`.

```
ALTER TABLE emp CHANGE age age1 int(4);
```

>注意:`change` 和 `modify` 都可以修改表的定义，不同的是 `change` 后面需要写两次列名，不方便。
但是 change 的优点是可以修改`列名称`，modify则不能.

### 6.5 修改字段排顺序
例子1:<br>
新增字段 `birth`类型`date` 加在 `ename` 之后:
```
ALTER TABLE emp ADD birth date AFTER ename;
```
例子2:<br>
修改已存在字段`age`,将它放在最前面:
```
ALTER TABLE emp MODIFY age int(3) first;
```

>注意:CHANGE/FIRST|AFTER COLUMN 这些关键字都属于MySQL在标准SQL上的扩展,在其他数据库上不一定适用.

### 6.6 表改名
语法:
```
ALTER TABLE tablename RENAME [TO] new_tablename;
```
例子:将表`emp`改为`emp1`.
```
ALTER TABLE emp RENAME emp1;
```

[(My)SQL基础](https://dm116.github.io/2020/03/01/sql-basic/)<br>
[(My)SQL基础(3)](https://dm116.github.io/2020/03/01/sql-basic-dml/)<br>
