---
layout:     post
title:      (My)SQL基础(4)-DQL
subtitle:   
date:       2020-03-02
author:     D
header-img: 
catalog: true
mermaid: true
tags:
    - [MySQL]
---

数据在插入数据库后,就可用`SELECT`命令进行查询,因`SELECT`语法很复杂,<br>
故这里只介绍基本的语法.

# 1. 全部记录查询
查询最简单的方式是将记录全部选出.例子:将表emp中的记录全部查询出来:
```
SELECT * FROM emp;
```
# 2. 查询不重复的记录
将表中的记录去掉重复后显示出来,用`DISTINCT`关键字.<br>
例子:将表emp中字段deptno重复的去掉,再显示出deptno字段.
```
SELECT DISTINC deptno FROM emp;
```
# 3. 条件查询
很多情况,只需要根据限定条件查询一部分数据,用`WHERE`关键字.<br>
例子1:查询所有`deptno`为 1 的记录.
```
SELECT * FROM emp WHERE deptno=1;
```
WHERE后面除了`=`外,还可用`>`,`<`,`>=`,`<=`,`!=`等比较运算符.<br>
多个条件之间还可用`OR`,`AND`等逻辑运算符进行多条件联合查询.<br>
例子2:
```
SELECT * FROM emp WHERE deptno=1 AND sal<3000;
```
# 4. 排序和限制
## 4.1 排序
我们经常有取出按照某个字段进行排序后的记录结果集的需求.<br>
这就用到关键字`ORDER BY`来实现.语法如下:
```
SELECT * FROM tablename [WHERE CONDITION] [ORDER BY field1 [DESC|ASC], field2
[DESC|ASC], ..., fieldn[DESC|ASC]] 
```
- DESC:表示按照字段进行降序排序
- ASC:表示升序排列
- 如果不写此关键字, `默认`, 是升序排列.

例子:把emp表中的记录按照工资高低进行显示.
```
SELECT * FROM emp ORDER BY sal;
```
>如果排序字段的值一样,则值相同的字段按照第二个排序字段进行排序,以此类推.<br>
例如,对于`deptno`相同的记录按照工资(`sal`)由高到低排序如下:
```
SELECT * FROM emp ORDER BY deptno, sal DESC;
```
## 4.2 限制
对于排序后的记录,如果希望只显示一部分,可用`LIMIT`关键字.语法如下:
```
SELECT ... [LIMIT offset_start, row_count];
```
- offset_start:表示记录的起始偏移量,默认为0.
- row_count:表示显示的行数,显示的实际就是前面n条记录.

例子1:显示emp表中按照`sal`排序的前3条记录.
```
SELECT * FROM emp ORDER BY sal LIMIT 3;
```
例子2:显示emp表中按照`sal`排序的3条记录,从第`2`条`开始`显示.
```
SELECT * FROM emp ORDER BY sal LIMIT 1, 3;
```

>注意:LIMIT属于MySQL扩展SQL92后的语法,其他数据库不能通用.

# 5. 聚合
语法:
```
SELECT [field1,field2,...,fieldn] fun_name
FROM tablename
[WHERE where_condition]
[GROUP BY field1,field2,...,fieldn
[WITH ROLLUP]]
[HAVING where_condition]
```
- fun_name:表示要做的聚合操作,常用的有`sum`,`count(*)`,max,min.
- `GROUP BY`:表示要进行分类聚合的字段.
- `WITH ROLLUP`:表示是否对分类聚合后的结果进行再汇总.
- `HAVING`关键字表示对分类 **后** 的结果再进行条件的过滤.

例子1:要emp表中统计公司的总人数.
```
SELECT count(1) FROM emp;
```
例子2:在例子1的基础上,统计各个部门的人数.
```
SELECT deptno,count(1) FROM emp GROUP BY deptno;
```
例子3:既要统计各部门人数,又要统计总人数.
```
SELECT deptno,count(1) FROM emp GROUP BY deptno WITH ROLLUP;
```
例子4:统计人数大于1的部门.
```
SELECT deptno,count(1) FROM emp GROUP BY deptno HAVING count(1)>1;
```
例子5:统计公司所有员工的薪水总额,最高和最低薪水.
```
SELECT sum(sal),max(sal),min(sal) FROM emp;
```
# 6. 表连接
当需要同时显示多个表中的字段时,可用表连接来实现.表连接分为`内连接`和`外连接`.

- `内连接`:选出两张表中互相匹配的记录(常用).
- `外连接`:会选出其他不匹配的记录
	- `左连接`:包含所有`左`边表中的记录甚至是`右`边表中没有和它匹配的记录
	- `右连接`:包含所有`右`边表中的记录甚至是`左`边表中没有和它匹配的记录

## 6.1 内连接
例子:查询所有雇员的名字和所在部门名称,雇员名称和部门分别存放在表emp和dept中.
```
SELECT ename,deptname FROM emp,dept WHERE emp.deptno=dept.deptno;
```
## 6.2 外连接
### 6.2.1 左连接
例子:查询emp中所有用户名和所在部门名称.
```
SELECT ename,deptname FROM emp LEFT JOIN dept ON emp.deptno=dept.deptno;
```
### 6.2.2 右连接
例子:查询emp中所有用户名和所在部门名称.
```
SELECT ename,deptname FROM dept RIGHT JOIN emp ON dept.deptno=emp.deptno;
```

# 7. 子查询

## 7.1 子查询
当我们查询需要的条件是另外一个SELECT语句的**结果**,这个时候,要用到子查询.<br>
子查询的关键字包括`IN`,`NOT IN`, `=`, `!=`, `EXISTS`,`NOT EXISTS`等.

例子1:从emp表中查询出所有部门在dept表中的所有记录.
```
SELECT * FROM emp WHERE deptno IN(SELECT deptno FROM dept);
```
若子查询记录数**唯一**,还可以用`=`代替`IN`.例子2:
```
SELECT * FROM emp WHERE deptno = (SELECT deptno FROM dept);
```
## 7.2 子查询转化为表连接
例子:
```
SELECT * FROM emp WHERE deptno IN(SELECT deptno FROM dept);
```
转为表连接:
```
SELECT emp.* FROM emp, dept WHERE emp.deptno=dept.deptno;
```

**注意**:子查询和表连接之间的转换主要应用在两个方面:<br>
- MySQL4.1以前的版本**不**支持子查询,需要用表连接来实现子查询的功能
- 表连接很多情况下用于优化子查询

# 8. 记录联合
将两个表的数据按照查询条件查询出来后,将结果合并到一起显示出来,需要<br>
关键字`UNION`和`UNION ALL`. 语法如下:
```
SELECT * FROM t1
UNION|UNION ALL
SELECT * FROM t2
...
UNION|UNION ALL
SELECT * FROM tn;
```
`UNION`和`UNION ALL`区别:<br>
- `UNION ALL`:是把结果集直接合并在一起.
- `UNION`:是将`UNION ALL`后的结果进行一次`DISTINCT`,即去除重复记录的结果.

例子1:将emp和dept表中的部门编号的集合显示出来.
```
SELECT deptno FROM emp UNION ALL SELECT deptno FROM dept;
```
例子2:将emp和dept表中的部门编号的结婚显示出来,且去除结果中重复记录后.
```
SELECT deptno FROM emp UNION SELECT deptno FROM dept;
```

[(My)SQL基础(3)](https://dm116.github.io/2020/03/01/sql-basic-dml/)<br>
[(My)SQL基础(5)](https://dm116.github.io/2020/03/02/sql-basic-dcl/)<br>
