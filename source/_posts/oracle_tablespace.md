---
title: Oracle 表空间(tablespace)
date: 2016-8-14 08:39:55
tags:
	- Oracle
---
### 什么是表空间
我们知道数据库真正存放的是数据文件，而数据文件的集合就是表空间，表空间是一个逻辑结构。数据库与表空间是一对多的关系，而表空间与数据文件也是一对多的关系
 <!-- more -->
{% qnimg oracle/tablespace/p1.png 'class:class1 class2' normal:yes %}

### 表空间的组成
对于表空间来说，逻辑结构如图
{% qnimg oracle/tablespace/p2.png 'class:class1 class2' normal:yes %}

#### Segment(段)
段是指占用数据文件空间的通称，或数据库对象使用的空间的集合；段可以有表段、索引段、回滚段、临时段和高速缓存段...

#### Extent(区间)
分配给对象（如表）的任何连续块叫区间；区间也叫扩展，因为当它用完已经分配的区间后，再有新的记录插入就必须在分配新的区间（即扩展一些块）；一旦区间分配给某个对象（表、索引及簇），则该区间就不能再分配给其它的对象

### 表空间的分类
- 永久表空间 ：用来存放永久性数据，如表...
- 临时表空间 ：用来存放排序，分组时产生的临时数据
- undo表空间 ：用来保存修改前的镜像

### 表空间的操作
对表空间的管理分为两种
- 字典管理 ： 表空间内的区间占用与否都存在字典里，分配或释放表空间时，这个表空间内的数据文件都会被修改
- 本地管理 ： 表空间分配不放在数据字典，而在每个数据文件头部的第3到第8个块的位图块，来管理空间分配

#### 默认表空间
当我们创建用户时，不指定表空间，oracle会默认指定users表空间

#### 创建表空间
``` sql
	#create [TEMPORARY] TABLESPACE tablespace_name TEMPFILE|DATAFILE 'fileName.dbf' size xx;  
	# 创建临时表空间 --size 为数据文件大小
	create TEMPORARY TABLESPACE tempts TEMPFILE 'e:\tempts.dbf' size 20M; 
	# 创建表空间 数据文件默认为安装路径
	create TABLESPACE ts DATAFILE 'ts.dbf' size 20M; 
```

#### 修改表空间
可以修改表空间名称，是否联机，文件大小是否可以扩展...
``` sql
	# offline 不可使用，默认为online
	alter tablespace ts offline;
	# 扩展数据文件大小  --不能扩展表空间大小
	alter database datafile 'ts1.dbf' autoextend on next 10M maxsize 100M;
```

> 不能删除表空间第一个数据文件，如果将第一个数据文件删除的话，相当于删除了整个表空间

#### 删除表空间
``` sql
	# drop tablespace tablespace_name [including contents]; --加上including contents则表示连同数据文件都删除
	 drop tablespace ts1 including contents;
```