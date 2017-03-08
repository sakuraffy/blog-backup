---
title: Oracle用户与权限
date: 2016-8-17 14:24:25
tags:
	- Oracle
---
我们知道Oracle与其它数据库（如mysql）其中一个很大的不同就是mysql只有一个root用户，而Oracle则是多用户处理，甚至可以自己新建用户
Oracle安装完成后，用户的默认密码 :
- sys(系统管理员，拥有最高权限) --> chang_on_install
- system(本地管理员) --> manager
- scott(普通用户) --> tiger

<!--more-->

### 用户连接与退出

#### 连接
``` sql
	# 本地管理员可以加上 as sysdba
	# 系统管理员必须加上 as sysdba
	# conn[ect] user_name/user_password as sysdba
	conn[ect] sys/chang_on_install as sysdba
```
#### 显示当前用户
``` sql 
	show user
```
#### 退出
``` sql
	# 退出页面
	exit;
	# 断开连接
	# disconn[ect] user_name
	disconn[ect] sys
```

### 用户创建和删除

只用管理员用户才可以创建和删除用户，普通用户不可以
``` sql
	# create user
	# create user user_name identified by user_password;
	create user sakuraffy identified by root;
	
	# drop user
	# drop user user_name [cascade]
	# drop user sakura			# 只删除用户
	# drop user sakura cascade  # 删除用户并删除数据对象
```
但是当你用sakuraffy用户登录发现报错了，这又是为什么呢？

### 权限管理

这是因为sakuraffy用户没有登陆的权限，这里先来简单说下权限，权限分为两类：系统权限和对象权限
- 系统权限 ： 用户对数据库操作(创建用户，表空间，存储过程……)的权限
- 对象权限 ： 用户对其他用户的数据对象操作(查询，修改其它用户的表，存储过程……)的权限

这里登陆的权限为CREATE SESSION

### 权限的授予与收回

#### 授予权限
``` sql
	# 系统权限
	# grant privilege_name to user_name;
	grant create session to sakuraffy;
	
	# 对象权限
	# grant dml on data_object to user_name;
	grant select on emp to sakuraffy; #scott用户下
	grant select on scott.emp to sakuraffy; #管理员用户下
```
#### 收回权限
``` sql
	# 系统权限
	# revoke privilege_name from user_name;
	revoke create session from sakuraffy;
	
	# 对象权限
	# revoke dml on data_object from user_name;
	revoke select on emp from sakuraffy; #scott用户下
	revoke select on scott.emp from sakuraffy; #管理员用户下
```
#### 授予权限,并拥有所有权（它可以将其权限再授予其它用户）
``` sql
	# 系统权限
	# grant privilege_name to user_name with admin option;
	grant create session to sakuraffy with admin option;
	
	# 对象权限
	# grant dml on data_object to user_name with grant option;
	grant select on emp to sakuraffy with grant option; #scott用户下
	grant select on scott.emp to sakuraffy with grant option; #管理员用户下
```
这里值得说的一点是当system将CREATE SESSION授予user1，user1赋予user2。当user1 CREATE SESSION授予收回时，user1授予user2的CREATE SESSION也将被收回

### 角色

简单一点来说 : 角色就是一系列权限的集合

对于上面登陆权限，也可以用角色授予
``` sql
	# grant role_name to user_name;
	grant connect to sakuraffy;
```

### 用户锁定与解锁

#### profile
``` sql
	# 创建profile
	# create profile profile_name limit opretion num password_lock_time num(day);
	create profile lock_account limit fail_login_attempt 3 password_lock_time 2;
	# 删除profile
	# drop profile profile_name [cascade]; 有用户使用这个文件，必须带有 cascade
	drop profile lock_account;
```
#### 用户锁定
``` sql
	# 直接锁定
	# alter user user_name account lock
	alter user sakuraffy account lock;

	# 满足了一定profile的用户锁定
	# alter user user_name profile profile_name;
	alter user sakuraffy profile lock_account;
```
#### 用户解锁
``` sql
	# alter user user_name account unlock;
	alter user sakuraffy account unlock;
```