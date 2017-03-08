---
title: Oracle DDL 语句
date: 2016-8-22 21:52:58
tags:
	- Oracle
---

### Oracle基本数据类型
对于Oracle来说，基本的数据类型就是对字符，数字，时间，文件这几类数据进行处理的

#### 字符类
- char : 固定长度(空间大小一定)，会用空格填充来达到其最大长度，对已知长度的字符处理，效率极高
- nchar : 与char相同，不过是unicode的形式存储
- varchar ：可变长度，不会用空格填满空间 
- varchar2 ：与varchar相似，但在其基础上进行了修改(一般使用varchar2,而非varchar)  <!-- more -->
- nvarchar2 : 与varchar2相同，不过是unicode的形式存储

#### 数字类
- number ：既可以表示整数(number(n))，也可以表示浮点数(number(m,n))
- float : 浮点型
- binary_float : 32位单精度浮点型
- binary_double : 64位双精度浮点型

#### 时间类
- date : 包含年月日，时分秒
- timestamp ： 比date更加精确，可以精确到9位小数秒

#### 文件类(lob)
- clob : 存储单字节和多字节字符数据,最大存储4G
- nclob : 存储UNICODE类型的数据,最大存储4G
- blob : 存储非结构化的二进制数据大对象,一般是图像、声音、视频等文件,最大存储4G
- nfile : 存储二进制文件，最大存储4G

> oracle不区分大小写，但是字符比较时'name' --> name是区分大小写的

### DDL语句
DDL(Data Definition Language)数据库定义语言,用于定义数据库的三级结构，包括外模式、概念模式、内模式及其相互之间的映像，定义数据的完整性、安全控制等约束，不需要commit

#### create 关键字
create(创建)一般指创建表，视图……，这里以建表为例
``` sql
	# create table table_name(字段...);
	create table aa (
		name varchar2(20),
		age number(3),
		sex char(2),
		birthday date
	);
```
当你用前面创建的用户sakuraffy操作时，会提示你权限不足

> 对于表的操作，需要赋予创建(create table)增(insert any table)删(drop any table)改(update any table)注释(comment any table)的权限

``` sql
	grant create table,insert any table,update any table,delete any table,comment any table to sakuraffy;
```
表复制
```
	#只复制表结构
	create table cc as select * from bb where 1 = 0;
	create table cc like bb;
	#复制表结构和内容
	create table cc as select * from bb;
```

#### alter 关键字
修改表(或者视图...)的结构，如添加，删除字段
``` sql 
	#增加字段
	#alter table table_name add (column_name column_dataStructure);  --可以多个字段，一个可以去除括号
	alter table aa add (address varchar2(20));
	#修改字段
	#alter table table_name modify (column_name column_dataStructure);  --可以多个字段，一个可以去除括号
	alter table aa modify (address varchar2(18));
	#删除字段
	#alter table table_name drop (column_name); --无论单个还是多个字段，都需要加括号
	alter table aa drop (address);
```

#### rename 关键字
修改表(或者视图...)的名称
``` sql
	#alter table table_oldName rename to table_newName;
	alter table aa rename to bb;
```

#### drop 关键字
删除表(或者视图...)的结构
``` sql 
	#drop table table_name;
	drop table aa;
```

#### truncate 关键字
truncate与delete相似，都是删除表(视图...)数据
- delete可以删除单条语句，truncate只能删除全部数据
- delete可以回滚，而truncate不可以
- delete效率低，而truncate效率高
``` sql
	# truncate table table_name
	truncate table aa
```

#### comment 关键字
为表/字段(或者视图...)添加注释
``` sql
	#添加注释
	#comment on table table_name is '内容';
	comment on table aa is 'aa table';
	comment on column aa.name is '姓名';
	#删除注释  --将注释改为空
	comment on column aa.name is ‘’；
```