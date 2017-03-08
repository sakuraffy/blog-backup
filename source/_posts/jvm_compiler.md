---
title: JVM编译器
date: 2016-09-14 10:50:55
tags:
	- JVM
---
当我们写一行行Java代码的时候，你是否想过机器它是否会认识呢？那么JVM编译器又是怎么对数据处理呢？可以Eclipse中安装byteCode，它会展示所对应的byteCode，我这里就一些常用的展示一下。

### 参数接收

#### 实例方法 ：
``` java
	int addTwo(int i, int j) {
		return i + j;
	}
```

<!--more-->

编译后的代码 ：
```
addTwo(II)I
   L0
    LINENUMBER 6 L0
    ILOAD 1
    ILOAD 2
    IADD
    IRETURN 
```

#### 静态方法 ：
``` java
	static int addTwo(int i, int j) {
		return i + j;
	}
```

编译后的代码 ：
```
static addTwo(II)I
   L0
    LINENUMBER 6 L0
    ILOAD 0
    ILOAD 1
    IADD
    IRETURN
```

注意 ：保存变量到局部变量表中，实例方法从1开始（0为this），静态方法则为0开始

### 常量，局部变量和控制结构的使用

``` java
	int addSum() {
		int sum = 0;
		for (int i = 0; i < 100; i++) {
			sum += i;
		}
		return sum;
	}
```
编译后的代码 ：
```
addSum()I
   L0
    LINENUMBER 6 L0
    ICONST_0
    ISTORE 1
   L1
    LINENUMBER 7 L1
    ICONST_0
    ISTORE 2
   L2
    GOTO L3
   L4
    LINENUMBER 8 L4
   FRAME APPEND [I I]                                                                         
    ILOAD 1
    ILOAD 2
    IADD
    ISTORE 1
   L5
    LINENUMBER 7 L5
    IINC 2 1
   L3
   FRAME SAME
    ILOAD 2
    BIPUSH 100
    IF_ICMPLT L4
   L6
    LINENUMBER 10 L6
    ILOAD 1
    IRETURN
```

### 方法调用

``` java
	public class Test {
	
		public int sum(int i, int j) {
			return add(i,j);
		}

		public int add(int i, int j) {
			return i + j;
		}
	}
```
sum()方法编译后的代码 ：
```
public sum(II)I
   L0
    LINENUMBER 7 L0
    ALOAD 0
    ILOAD 1
    ILOAD 2
    INVOKEVIRTUAL Test.add (II)I
    IRETURN
```
调用父类方法和静态方法使用INVOKESPECIAL和INVOKESTASTIC

### 类实例的使用

``` java
	public String test() {
		String str = new String("Hello1");
		return str.toString();
	}
```
编译后的代码 ：
```
public test()Ljava/lang/String;
   L0
    LINENUMBER 5 L0
    NEW java/lang/String
    DUP
    LDC "Hello1"
    INVOKESPECIAL java/lang/String.<init> (Ljava/lang/String;)V
    ASTORE 1
   L1
    LINENUMBER 6 L1
    ALOAD 1
    INVOKEVIRTUAL java/lang/String.toString ()Ljava/lang/String;
    ARETURN
```

### getter和setter方法的编译

``` java
	public class Test {
		private int id;

		public int getId() {
			return id;
		}

		public void setId(int id) {
			this.id = id;
		}
	}
```
编译后的代码 :
```
public getId()I
   L0
    LINENUMBER 7 L0
    ALOAD 0
    GETFIELD Test.id : I
    IRETURN
	
public setId(I)V
   L0
    LINENUMBER 11 L0
    ALOAD 0
    ILOAD 1
    PUTFIELD Test.id : I
   L1
    LINENUMBER 12 L1
    RETURN
```
	
### 数组的使用

``` java
	public int[] array() {
		int[] i = new int[10];
		return i;
	}
```
编译后的代码 ：
```
public array()[I
   L0
    LINENUMBER 5 L0
    BIPUSH 10
    NEWARRAY T_INT
    ASTORE 1
   L1
    LINENUMBER 6 L1
    ALOAD 1
    ARETURN
```

### Exception的使用

``` java
	public void exception() throws Exception {
		throw new Exception();
	}
```
编译后的代码 ；
```
public exception()V throws java/lang/Exception 
   L0
    LINENUMBER 5 L0
    NEW java/lang/Exception
    DUP
    INVOKESPECIAL java/lang/Exception.<init> ()V
    ATHROW
```

### 同步的使用

``` java
	public void syn(){
		String str = new String();
		synchronized (str) {
			str.trim();
		}
	}
```
编译后的代码 ：
```
 public syn()V
    TRYCATCHBLOCK L0 L1 L2 
    TRYCATCHBLOCK L2 L3 L2 
   L4
    LINENUMBER 5 L4
    NEW java/lang/String
    DUP
    INVOKESPECIAL java/lang/String.<init> ()V
    ASTORE 1
   L5
    LINENUMBER 6 L5
    ALOAD 1
    DUP
    ASTORE 2
    MONITORENTER
   L0
    LINENUMBER 7 L0
    ALOAD 1
    INVOKEVIRTUAL java/lang/String.trim ()Ljava/lang/String;
    POP
   L6
    LINENUMBER 6 L6
    ALOAD 2
    MONITOREXIT
   L1
    GOTO L7
   L2
   FRAME FULL [Test java/lang/String java/lang/String] [java/lang/Throwable]                   
    ALOAD 2
    MONITOREXIT
   L3
    ATHROW
   L7
    LINENUMBER 9 L7
   FRAME CHOP 1
    RETURN
```