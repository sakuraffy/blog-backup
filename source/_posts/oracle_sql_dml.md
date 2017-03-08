---
title: Oracle 基本DML语句
date: 2016-8-24 21:56:08
tags:
	- Oracle
---
### 什么是DML语句
DML(Data Manipulation Language)数据操纵语言,用户或程序员实现对数据库数据的操作

这里为了方便测试，我们先建一个简单的表student
``` sql
	create table student (
		sid number(4),
		ssex char(2),
		sname varchar2(20),
		sbirthday date
	);
```

<!-- more -->

### select 关键字
``` sql 
	select column_name from table_name;
	select sid,sname from student;
```

### insert 关键字
``` sql
	# insert into table_name(column_name...) values(column_value);
	# column可以写任意个数，也可以不写(不写默认全部添加),但必须一一对应
	insert into student(sid,ssex,sname,sbirthday) values(1001,'男','jack','11-3月-1995');
```

> 时间必须以11-3月-1995这种格式录入，当然也可以用to_date(),后面再说

当我们用前面创建的用户sakuraffy插入数据的时候，会出现ORA-01950: 对表空间 'USERS' 无权限，这又是怎么回事呢？

这是因为我们没有使用表空间的权限，关于表空间，请参考[Oracle 表空间](https://sakuraffy.github.io/oracle_tablespace)
``` sql
	# 这里提供两个方案(必须是管理员用户)
	# alter user user_name quota unlimited on tablespace_name(建用户时，没有指定，默认为users)
	alter user sakuraffy quota unlimited on users;
	# grant resource to user_name  
	grant resource to sakuraffy
```

### update 关键字
``` sql 
	# update table_name set column_name = newValue [where column_name = oldValue] 
	# where 限制条件，没有where全部更新
	update student set sname = 'jackjack' where sid = 101;
```

### delete 关键字
``` sql 
	# delete from table_name [where column_name = oldValue] 
	delete from student where sid = 101;
```

### merge 关键字
merge into 解决用B表跟新A表数据，如果A表中没有，则把B表的数据插入A表

DML语句的关键字还有call、explain plan、lock table 这些后面用到了在详细说