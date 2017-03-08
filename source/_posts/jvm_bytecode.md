---
title: 字节码指令集
date: 2016-09-11 20:25:02
tags:
	- JVM
---
编写程序，对于程序员来说是一行行代码，可是对于JVM来说，可不是如此，它认识的都是字节码指令
我们知道数据类型有基本数据类型（boolean、byte、char、short、int、long、float、double）和引用（reference）数据类型，以及返回值（returnAddress）数据类型
对于大部分与数据类型相关的字节码来说，它们的操作码助记符中都有特殊的字符来表名该指令为哪种数据类型来服务：i代表对int类型的数据操作，l代表long，b代表byte，c代表char，s代表short,f代表float,d代表double,a代表reference
而对于JVM来说会将基本数据类型的char、short、byte、int都当作int来处理

<!--more-->

###  加载和存储指令

- 加载和存储指令用于将数据从栈帧的本地变量表（局部变量表）和操作数栈之间来回传递
将本地变量加载到操作数栈的指令包括 ： iload、iload\_&lt;n&gt;、fload、fload\_&lt;n&gt;、dload、dload\_&lt;n&gt;、aload、aload\_&lt;n&gt;
- 将常量加载到操作数栈的指令包括 ：
bipush、sipush、ldc、ldc\_w、ldc2\_w、aconst\_null、iconst\_ml、iconst\_&lt;i&gt;、fconst\_&lt;f&gt;、lconst\_&lt;l&gt;、dconst_\_&lt;d&gt;
- 将数值从操作数栈存储到局部变量表的指令包括 ：
istore、istore\_&lt;n&gt;、fstore、fstore\_&lt;n&gt;、dstore、dstore\_&lt;n&gt;、astore、astore\_&lt;n&gt;
- 用于补充局部变量表的访问索引或立即数的指令 ： wide

&lt;n&gt; : 表示非负整数
&lt;i&gt; : 表示int类型数据
&lt;l&gt; : 表示long类型数据
&lt;f&gt; : 表示float类型数据
&lt;d&gt; : 表示double类型数据

> 算术指令

- 加法指令 ： iadd,ladd,fadd,dadd
- 加法指令 ： isub,lsub,fsub,dsub
- 乘法指令 ： imul,lmul,fmul,dmul
- 除法指令 ： idiv,ldiv,fdiv,ddiv
- 取余指令 ： irem,lrem,frem,drem
- 求负值指令 ：ineg,lneg,fneg,dneg
- 移位指令 ： ishl,ishr,iushr,lshl,lshr,lushr
- 按位或指令 ：ior,lor
- 按位与指令 ： iand,land
- 按位异或指令 ：ixor,lxor
- 局部变量自增指令 ：iinc
- 比较指令 ： dcmpg,dcmpl,fcmpg,fcmpl,lcmp

### 类型转化指令

类型转换可以分为两种：宽化类型转换和窄化类型转换
#### 宽化类型转换
所谓的宽化类型转换就是从小范围类型向大范围类型，数据安全，不会失真
从int类型可以转换为long、float、double
从long类型可以转换为float、double
从float类型可以转换到double
相关指令i2d("2"(two))表示从int转化到double

#### 窄化类型转换

所谓的窄化类型转换就是从大范围类型向小范围类型，数据不安全，可能会失真
就是将上面倒过来，再加上int类型转化为char、short、byte
对于浮点型数据转换为int或long时，简单来说就是将其小数部分舍去

### 对象创建与操作

- 类对象实例创建的指令 ： new
- 数组创建的指令 ： newarray, anewarray, multianewarray
- 访问类字段和类实例字段的指令 ：getstatic,putstatic,getfield,putfield
- 将一个数组元素加载到操作数栈的指令 ： baload,caload,saload,iaload,laload,faload,daload,aaload
- 将操作数栈的值存储到数组元素中的指令 ： bastore,castore,sastore,iastore,lastore,fastore,dastore,aastore
- 取数组长度指令 ： arraylength
- 检查类实例或数组实例的指令 ： instanceof、checkcast

### 操作数栈管理指令

一些直接控制操作数栈的指令，包括：pop,pop2,dup,dup2,dup\_x1,dup2\_x1,dup2\_x2和swap

### 控制转移指令

-  条件分支：ifeq,iflt,ifne,ifge,ifgt,ifnull,ifnonnull,if_icmpeq,if_icmpne,if_icmplt,if_cmpgt,if_icmpge,if_cmpge,if_icmpeq,if_acmpne
- 复合条件分支：tableswitch,lookupswitch
- 无条件分支：goto,goto_w,jsr,jsr_w,ret

### 方法调用和转移指令

方法调用指令一般有5个 ：
- invokevirtual ： 调用对象的实例方法
- invokeinterface : 调用接口方法
- invokespecial ： 调用特殊实例方法（例如父类的方法）
- invokestatic ： 调用静态方法
- invokedynamic ： 调用多态方法

方法返回指令根据返回值的类型进行区分，包括ireturn，lreturn,freturn,dreturn和areturn，另外return指令供申明为void的方法、实例对象，类初始化方法使用

### 异常

通常用athrow指令抛出异常