---
title: Oracle 约束与索引
date: 2016-08-30 20:18:37
tags:
	- Oracle
---
### 约束
我们知道数据库里的数据不是随意插入的，而是需要保证有一定的意义与合法性，例如一个人的年龄为1000，这样是不是显得有点滑稽，正是因为这样，我们就有了约束来保证数据的合法性与完整性

<!-- more -->

#### 非空约束
非空（NOT NULL）约束：顾名思义，所约束的列不能为NULL值。否则就会报错
``` sql 
	# 创建表时加入
	create table tt (nickname varchar2(20) not null);
	# 添加非空约束
	alter table tt modify nickname not null;
```

#### 条件约束
条件（CHECK）约束：表中每行都要满足该约束条件。条件约束既可以在表一级定义也可以在列一级定义。在一列上可以定义任意多个条件约束
``` sql 
	# 创建表时加入
	create table tt (age number(3) check(age between 0 and 200));
	# 添加条件约束
	alter table tt modify age check(age between 0 and 200);
```

#### 唯一约束
唯一（UNIQUE）约束：在表中每一行中所定义的这列或这些列的值都不能相同。必须保证唯一性
``` sql 
	# 创建表时加入
	create table tt (name varchar2(20) unique);
	# 添加唯一约束
	alter table tt modify name varchar2(20) unique;
	# 删除唯一约束
	alter table tt drop unique(name);
```

#### 主键约束
主键(PRIMARY KEY)约束：用来唯一标识每一行，与唯一约束的区别在于不能为空
``` sql 
	# 创建表时加入
	create table tt (id number(2) primary key);
	# 添加主键约束
	alter table tt modify id number(2) primary key;
	# 删除主键约束
	alter table tt drop primary key;
```

##### 序列
``` sql
	create sequence myseq increment by 2 start with 10;
	# 查询sequence信息 
	select * from user_sequences;
	# 利用sequence插入数据
	insert into tt(id) values(myseq.nextval);
```
{% qnimg oracle/constraint-index/p1.png 'class:class1 class2' normal:yes %}

> LAST_NUMBER += CACHE_SIZE * INCREMENT_BY

#### 外键约束
``` sql
	# 新建一个额外的表
	create table ttt(id number(2) primary key);
	# 创建表时加入
	create table tt (tid number(2), constraint fk_ttt foreign key (tid) references ttt(id));
	# 添加外键约束
	alter table tt add constraint fk_ttt foreign key(tid) references ttt(id);
	# 删除外键约束
	alter table tt drop constraint fk_ttt;
```

> 创建约束的时候，最好加上名称，便于后续的管理

### 索引

#### ROWID
在数据进行从年初的时候，每一条记录都会生成一个伪列(ROWID)来记录数据，ROWID一般用来创建二叉树形成索引
``` sql
	# 创建索引
	create index ind_name on tt(name);
	# 删除索引
	drop index ind_name;
```

#### 索引优缺点
- 索引可以加快查询速度
- 索引会降低修改速度
- 索引适合经常使用且不易修改的字段