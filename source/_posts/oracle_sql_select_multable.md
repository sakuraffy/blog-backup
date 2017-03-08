---
title: Oracle 多表连接查询
date: 2016-8-26 21:57:10
tags:
	- Oracle
---
前面说到了单表查询，但在现实开发中，数据处理往往需要依赖多个表，这时候，我们就需要用到多表连接查询

### 笛卡尔集

#### 什么是笛卡尔积

简单来说，两个集合相乘，形成XY形式，就是笛卡儿积

<!-- more -->

``` sql
	select empno,ename,sal,dept.* from emp,dept;
```
{% qnimg oracle/sql/select_multable/p1.png 'class:class1 class2' normal:yes %}

#### 笛卡尔积产生条件
- 省略连接条件
- 选择条件无效
- 所有表中的所有行互相连接

### 查询连接
去除重复的记录就需要添加连接条件，连接 n个表,至少需要 n-1个连接条件

#### 等值连接
``` sql
	select empno, ename, d.deptno, dname from emp e, dept d 
		where e.deptno = d.deptno;
```
{% qnimg oracle/sql/select_multable/p2.png 'class:class1 class2' normal:yes %}

#### 非等值连接
``` sql
	select empno, ename,sal, grade from emp, salgrade
		where sal between losal and hisal;
```
{% qnimg oracle/sql/select_multable/p3.png 'class:class1 class2' normal:yes %}

> 当最后查询结果只需要一个表的数据，却依赖另一个表的数据，此时用子查询(m+n)而非多表查询(m*n)

### 内连接

#### 等值内连接
``` sql
	# join on的形式
	select empno, ename, d.deptno, dname from emp e [inner] join dept d on e.deptno = d.deptno;

	# join Using的形式
	# 只适合外键名与该外键所属表主键名相同且不能给选中的列中加上表名缀或别名
	select empno, ename, deptno, dname from emp [inner] join dept using (deptno);
```
{% qnimg oracle/sql/select_multable/p2.png 'class:class1 class2' normal:yes %}

#### 非等值内连接
``` sql
	select empno, ename,grade from emp e [inner] join salgrade s on e.sal between s.losal and s.hisal;
```

#### 自然连接
``` sql 
	select empno,ename,deptno,dname from emp natural join dept;
```
{% qnimg oracle/sql/select_multable/p2.png 'class:class1 class2' normal:yes %}

### 外连接
一般来说外连接分为三类 ：
- 左外连接(左边的表不加限制)
- 右外连接(右边的表不加限制)
- 全外连接(左右两边的表不加限制)

#### 左外连接(left [outer] join)
``` sql 
	select e.empno,e.ename,d.deptno,d.dname from dept d left join emp e on e.deptno = d.deptno;
```

#### 右外连接(right [outer] join)
``` sql 
	select e.empno,e.ename,d.deptno,d.dname from emp e right join dept d on e.deptno = d.deptno;
```

#### 全外连接(full [outer] join)
``` sql 
	select e.empno,e.ename,d.deptno,d.dname from emp e full join dept d on e.deptno = d.deptno;
```
#### 使用(+)
``` sql 
	select e.empno,e.ename,d.deptno,d.dname from dept d,emp e where e.deptno(+) = d.deptno;
```

> 全外连接不能使用(+)

由于emp表和dept表的原因，外连接查询的输出都如下
{% qnimg oracle/sql/select_multable/p4.png 'class:class1 class2' normal:yes %}


