---
title: Oracle 复杂查询SQL拼接
date: 2016-08-28 20:00:57
tags:
	- Oracle
---
在日常开发中，我们往往需要连接几个表进行复杂查询，那么SQL语句的编写就显得尤为重要，但我们也知道，一个复杂的语句往往并不能信手拈来，尤其对于新手来说。那么形成一个好的清晰的思路能是我们更快更准确地写出SQL语句。下面就以一个具体的实例来整理一下我的思路

<!-- more -->

> 列出薪金比“SMITH”或“ALLEN”多的所有员工编号，姓名，部门名称，其领导姓名，部门人数，平均工资，最高及最低工资

### SQL拼接准备

#### 确定要使用的数据表
- emp ：员工编号，姓名
- dept ：部门名称
- emp ：领导名称
- emp : 统计信息

#### 确定以知的关联字段
员工表与部门表 ： emp.deptno = dept.deptno
雇员与领导 ： emp.mgr = emp.empno

### SQL拼接

#### 1. 查出“SMITH”或"ALLEN"的薪金
``` sql 
	SELECT sal 
	FROM emp 
	WHERE ename IN ('SMITH','ALLEN'); 
```
{% qnimg oracle/sql/select_complex/p1.png 'class:class1 class2' normal:yes %}

#### 2. 找出比"SMITH"或"ALLEN"薪金多的人，但要刨除他们两个
上面的结果为多行单列，使用>any完成
``` sql 
	SELECT e.empno,e.ename
	FROM emp e
	WHERE e.sal > ANY(SELECT SAL 
			FROM EMP 
			WHERE ENAME IN ('SMITH','ALLEN'))
		AND e.ename NOT IN ('SMITH','ALLEN');
```
{% qnimg oracle/sql/select_complex/p2.png 'class:class1 class2' normal:yes %}

#### 3. 查询部门名称
``` sql 
	SELECT e.empno,e.ename,d.dname
	FROM emp e,dept d
	WHERE e.sal > ANY(SELECT SAL 
			FROM EMP 
			WHERE ENAME IN ('SMITH','ALLEN'))
		AND e.ename NOT IN ('SMITH','ALLEN')
		AND e.deptno = d.deptno;
```
{% qnimg oracle/sql/select_complex/p3.png 'class:class1 class2' normal:yes %}

#### 4. 查询领导名称
``` sql 
	SELECT e.empno,e.ename,d.dname,m.ename mgr
	FROM emp e,dept d，emp m
	WHERE e.sal > ANY(SELECT SAL 
			FROM EMP 
			WHERE ENAME IN ('SMITH','ALLEN'))
		AND e.ename NOT IN ('SMITH','ALLEN')
		AND e.deptno = d.deptno
		AND e.mgr = m.empno(+);
```
{% qnimg oracle/sql/select_complex/p4.png 'class:class1 class2' normal:yes %}

#### 5. 查询分组信息
查询分组信息一般要用到GROUP BY，但是由于员工信息之类的信息不能添加在最后进行分组，这里只能使用子查询
``` sql 
	SELECT e.empno,e.ename,d.dname,m.ename mgr,temp.count,temp.avg_sal,temp.max_sal,temp.min_sal
	FROM emp e,dept d，emp m, (
		SELECT deptno,COUNT(empno) count, AVG(sal) avg_sal, MAX(sal) max_sal,MIN(sal) min_sal
		FROM emp 
		GROUP BY deptno) temp
	WHERE e.sal > ANY(SELECT SAL 
			FROM EMP 
			WHERE ENAME IN ('SMITH','ALLEN'))
		AND e.ename NOT IN ('SMITH','ALLEN')
		AND e.deptno = d.deptno
		AND e.mgr = m.empno(+)
		AND d.deptno = temp.deptno;
```
{% qnimg oracle/sql/select_complex/p5.png 'class:class1 class2' normal:yes %}