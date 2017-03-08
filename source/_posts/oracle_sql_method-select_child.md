---
title: Oracle 常用函数与子查询
date: 2016-08-27 17:39:05
tags:
	- Oracle
---
### 常用限定词
``` sql
	select * from student;
```
像上面的查询，我们会查询到所有的列，但是有时候需要对其做一些限定，这时候就需要where限定词 <!-- more -->
#### 通配符(like)
- % : 任意个数字符
- _ : 1个字符

#### 集合限定
- in : 在集合内
- any : > any() <==> 大于集合中最小的
- all : > all() <==> 大于集合中最大的

#### between and
``` sql
	#btween and 是双闭区间
	select ename , sal from emp where sal between 800 and 1600;
```
查询结果 ：
{% qnimg oracle/sql/method-select_child/p3.png 'class:class1 class2' normal:yes %}

### 常用函数
为了方便，所有的查询都已scott用户以及该用户下的表

#### 字符类函数
- upper(char) : 将字符转为大写
- lower(char) : 将字符转为小写
- length(char) : 字符长度
- substr(char,m,n) : 字符截取,从第m位向后截取n位
- replace(char,oldChar,newChar) : 字符替换,用newChar替换oldChar
- to_char(char,'yy') : 将字符以什么的形式展现

``` sql
	# 展现所有员工的姓名，薪水，并且姓名首字母大写，其它小写
	# 薪水以99，999.99的形式展现
	select upper(substr(ename,1,1)) || lower(substr(ename,2,length(ename)-1)) ename,
		to_char(sal,'99,999.99') sal from emp;
```
查询结果 ：
{% qnimg oracle/sql/method-select_child/p1.png 'class:class1 class2' normal:yes %}

#### 时间类函数
- to_date(date,'') :  将date数据以特定字符格式展现
- sysdate() ： 系统时间
- year(date) 获取年份
- add_months(date, n) :  以该日期为几点，向后加n个月
- last_day(date, n) : 月份的最后几天

#### 数字类函数
- count(number) : 总数(分组函数) 
- max(number) ： 最大值(分组函数) 
- min(number) ： 最小值(分组函数) 
- avg(number) ： 平均值
- round(number) ： 四舍五入
- floor(number) ： 向下取整
- ceil(number) ： 向上取整
- mod(number,n) ： 字段对n取模
{% qnimg oracle/sql/method-select_child/p2.png 'class:class1 class2' normal:yes %}

> dual 是用来测试的虚拟表

#### sys_context
- language : 数据库语言
- db_name : 数据库名称
- session_user : 当前用户
- nls_date_format : 时间格式
``` sql
	select sys_context('userevn','language') from dual;
```

#### group by
分组，查询的都是组数据，对于分组数据过滤只能用having，不能用where
``` sql
	# 查询部门编号大于10，所在部门的人数，平均工资
	select count(*) ,avg(sal) avg_sal from emp group by deptno having deptno > 10;
```
{% qnimg oracle/sql/method-select_child/p4.png 'class:class1 class2' normal:yes %}

#### order by
``` sql
	select empno,ename,sal from emp order by sal desc;
```
{% qnimg oracle/sql/method-select_child/p5.png 'class:class1 class2' normal:yes %}

### 子查询
#### 什么是子查询
子查询就是将查询运算后的表再作为一个表供我们查询

#### 单行子查询
``` sql 
	# 查询薪水最高员工的姓名和薪水
	# select ename,sal from emp where sal = (select max(sal) from emp);
```
{% qnimg oracle/sql/method-select_child/p6.png 'class:class1 class2' normal:yes %}

#### 多行子查询
``` sql 
	# 查询与scott同部门员工的姓名，薪水，部门号
	select ename,sal,deptno from emp where deptno = (select deptno from emp where ename = 'SCOTT');
```
{% qnimg oracle/sql/method-select_child/p7.png 'class:class1 class2' normal:yes %}

#### 利用子查询修改数据
``` sql
	# 利用子查询修改数据（将7499的薪水和职位改为7369的）
	update emp set (sal,job) = (select sal,job from emp where empno = 7499) where empno = 7369;
```

#### 分页查询
``` sql
	# 使用rownum (查出1-9条记录)
	select * from 
		(select rownum r, t.* from 
			(select empno,job,sal from emp) t 
		where rownum < 10) 
	where r > 0;
```
分页查询的关键在于子查询与二分法