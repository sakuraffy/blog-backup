---
title: Oracle_11g安装与卸载
date: 2016-08-9 16:06:49
tags:
	- Oracle
---
### 下载

1. 官网下载地址 ： http://www.oracle.com/technetwork/cn/database/enterprise-edition/downloads/index.html
2. 同意协议
3. 下载64位的oracle11g的两个压缩包 <br>
{% qnimg oracle/install/p1.png 'class:class1 class2' normal:yes %}
4. 将两个压缩包解压到同一目录，我这里是database

<!--more-->

### 安装

1. 点击step.exe进入安装引导界面 <br>
{% qnimg oracle/install/p2.png 'class:class1 class2' normal:yes %}
2. 不要管最低要求，点击"是" <br>
{% qnimg oracle/install/p3.png 'class:class1 class2' normal:yes %}
3. 取消更新 <br>
{% qnimg oracle/install/p4.png 'class:class1 class2' normal:yes %}
4. 忽略警告，点击"是" <br>
{% qnimg oracle/install/p5.png 'class:class1 class2' normal:yes %}
5. 选择“创建和配置数据库”， 即默认， 下一步 <br>
{% qnimg oracle/install/p6.png 'class:class1 class2' normal:yes %}
6. 选择"桌面类"，即默认，下一步 <br>
{% qnimg oracle/install/p7.png 'class:class1 class2' normal:yes %}
7. 配置安装路径和数据库管理口令 <br>
{% qnimg oracle/install/p8.png 'class:class1 class2' normal:yes %}
8. 完成 <br>
{% qnimg oracle/install/p9.png 'class:class1 class2' normal:yes %}
9. 设置管理口令，解锁scott账户，并为sys、system设置口令 <br>
{% qnimg oracle/install/p10.png 'class:class1 class2' normal:yes %}
{% qnimg oracle/install/p11.png 'class:class1 class2' normal:yes %}
10. 忽略口令的复杂度，点击“是” <br>
{% qnimg oracle/install/p12.png 'class:class1 class2' normal:yes %}
11. 安装完成，关闭 <br>
{% qnimg oracle/install/p13.png 'class:class1 class2' normal:yes %}

### 删除和创建数据库

由于Oracle出于安全与系统的考虑使得自带的全局数据库orcl，在操作时有时会出现异常，下面删除并创建一个数据库（当然，不一定非得删除并创建数据库）
1. 打开开始，所有程序，找到Oracle，点击Database Configuration Assistant <br>
{% qnimg oracle/install/p34.png 'class:class1 class2' normal:yes %}
{% qnimg oracle/install/p14.png 'class:class1 class2' normal:yes %}
2. 先删除原来的数据库 <br>
{% qnimg oracle/install/p15.png 'class:class1 class2' normal:yes %}
3. 输入用户（sys或system）和口令 <br>
{% qnimg oracle/install/p16.png 'class:class1 class2' normal:yes %}
{% qnimg oracle/install/p17.png 'class:class1 class2' normal:yes %}
4. 删除完成后，创建一个数据库 <br>
{% qnimg oracle/install/p18.png 'class:class1 class2' normal:yes %}
5. 默认选择，下一步 <br>
{% qnimg oracle/install/p19.png 'class:class1 class2' normal:yes %}
6. 创建数据库名和SID <br>
{% qnimg oracle/install/p20.png 'class:class1 class2' normal:yes %}
7. 默认选择，下一步 <br>
{% qnimg oracle/install/p21.png 'class:class1 class2' normal:yes %}
8. 为所有用户配置统一口令 <br>
{% qnimg oracle/install/p22.png 'class:class1 class2' normal:yes %}
9. 默认选择，下一步 <br>
{% qnimg oracle/install/p23.png 'class:class1 class2' normal:yes %}
10. 默认选择，下一步 <br>
{% qnimg oracle/install/p24.png 'class:class1 class2' normal:yes %}
11. 默认选择，下一步 <br>
{% qnimg oracle/install/p25.png 'class:class1 class2' normal:yes %}
12. 默认选择，下一步 <br>
{% qnimg oracle/install/p26.png 'class:class1 class2' normal:yes %}
13. 默认选择，下一步 <br>
{% qnimg oracle/install/p27.png 'class:class1 class2' normal:yes %}
14. 点击完成 <br>
{% qnimg oracle/install/p28.png 'class:class1 class2' normal:yes %}
{% qnimg oracle/install/p29.png 'class:class1 class2' normal:yes %}
{% qnimg oracle/install/p30.png 'class:class1 class2' normal:yes %}
15. 解锁scott账户，并为sys、system设置口令 <br>
{% qnimg oracle/install/p31.png 'class:class1 class2' normal:yes %}
{% qnimg oracle/install/p32.png 'class:class1 class2' normal:yes %}
16. 数据库创建完成，退出 <br>
{% qnimg oracle/install/p33.png 'class:class1 class2' normal:yes %}


### 登陆检验

oracle有三种登陆使用方式，分别是SQL Plus、SQL Developer和EM Web
在登陆之前必须开启Oracle 服务 OracleServiceKCXX、OracleDBConsoleorcl和OracleOraDb11g_home2TNSListener（这里名称因人而异，Service，DBConsole和TNSListener这三个关键字）
{% qnimg oracle/install/p35.png 'class:class1 class2' normal:yes %}
#### SQL Plus
1. 打开开始，所有程序，找到Oracle，点击SQL Plus <br>
{% qnimg oracle/install/p36.png 'class:class1 class2' normal:yes %}
2. 输入用户名和口令登陆 <br>
{% qnimg oracle/install/p37.png 'class:class1 class2' normal:yes %}

#### SQL Developer
1. 由于Oraclre自带的SQL Developer为32位，我电脑64位的。下载SQL Develper地址 ： http://www.oracle.com/technetwork/cn/developer-tools/sql-developer/downloads/index.html
2. 同意协议，点击下载 <br>
{% qnimg oracle/install/p38.png 'class:class1 class2' normal:yes %}
3. 解压文件，点击sqldeveloper.exe <br>
{% qnimg oracle/install/p39.png 'class:class1 class2' normal:yes %}
4. 连接数据库 <br>
{% qnimg oracle/install/p41.png 'class:class1 class2' normal:yes %}
{% qnimg oracle/install/p40.png 'class:class1 class2' normal:yes %}

#### EM Web
1. 打开浏览器
	输入https://localhost:1158/em
	或https://IP地址:1158/em
	或https://计算机名:1158/em

2. 点击高级，继续前往 <br>
{% qnimg oracle/install/p99.png 'class:class1 class2' normal:yes %}
{% qnimg oracle/install/p98.png 'class:class1 class2' normal:yes %}
3. 登陆 <br>
{% qnimg oracle/install/p97.png 'class:class1 class2' normal:yes %}
{% qnimg oracle/install/p96.png 'class:class1 class2' normal:yes %}

### 卸载

按照上面可以正常安装，如不幸安装失败，那就只能卸载Oracle
1. 关闭Oracle服务
2. 打开注册表：regedit 打开路径：HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\ 		删除该路径下的所有以oracle开始的服务名称，这个键是标识Oracle在windows下注册的各种服务 <br>
{% qnimg oracle/install/p42.png 'class:class1 class2' normal:yes %}
3. 找到路径：HKEY_LOCAL_MACHINE\SOFTWARE\ORACLE 删除该oracle目录，该目录下注册着Oracle数据库的软件安装信息和ODBC\ODBCINST.INI\Oracle in OraDb11g_home1 <br>
{% qnimg oracle/install/p43.png 'class:class1 class2' normal:yes %}
4. 删除注册的oracle事件日志，打开注册表HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\Eventlog\Application 删除注册表的以oracle开头的所有项目。<br>
{% qnimg oracle/install/p44.png 'class:class1 class2' normal:yes %}
5. 删除环境变量path中关于oracle的内容 <br>
{% qnimg oracle/install/p45.png 'class:class1 class2' normal:yes %}
6. 删除Oracle安装目录
7. 删除开始菜单中的oracle项
8. 删除C:\Program Files\Oracle
9. 重启电脑