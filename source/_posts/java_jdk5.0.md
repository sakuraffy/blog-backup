---
title: JDK5.0新特性
date: 2016-09-20 11:31:59
tags:
	- Java
---
jdk5.0（jdk1.5）相对于jdk1.4来说，是一场重大的革命，下面就来谈谈jdk5.0增加的一些新特性

### for-each循环

在日常编程中，我们会经常用到遍历，我们知道的有Iterator遍历，主要用于集合，索引式遍历。而在jdk5.0中为我们提供了for-each循环

<!--more-->

``` java
	int[] arr = new int[]{1,2,3,4,5};
	for(int i : arr) {
		System.out.println(i);
	}
```
for-each循环简化了遍历，同时也使得索引不可见，因此无法对内容进行修改，只能单纯的遍历

### 静态导入

在jdk中提供了很多工具类，如Math、Executors......那么每次使用都必须用Math.而静态导入则省去了这个麻烦
``` java
	package cn.sakuraffy;

	import static java.lang.Math.*;

	public class Test {
		public static void main(String[] args) {
			int a = 10;
			System.out.println(abs(a));
		}
	}
```
这里值得说的是import 引入的是类，而import static 引入的则是方法，但过度使用则会降低代码的可读性

### 泛型

``` java
	// 这就是jdk1.4所采用的
	ArrayList al = new ArrayList();
	al.add(new String("123"));
	String str = (String) al.get(0);
	System.out.println(str);
	
	//这是jdk1.5后采用的
	ArrayList<String> al = new ArrayList<String>();
	al.add(new String("123"));
	String str = al.get(0);
	System.out.println(str);
```
相对于以前的，只是少了一个强制转换，其实不然，他将问题提前到了编译期

### 自动拆箱装箱

说到拆装箱之前，先来说一下基本数据类型的包装类型，我们知道基本数据类型是没有对象的，而Java是面向对象编程的，那么就引入了基本数据类型的包装类型取完成，但引用类型和基本数据类型是两个完全不同的东西，使用就必须相互转换
``` java
	Integer i = 1;
	//Integer i = new Integer(1);
	int a = new Integer(3);
	//int a = new Integer(3).intValue();
```
上面两句语句在jdk1.5之前肯定是错的，基本数据类型的值怎么能赋值给对象呢？但在jdk1.5之后，它是可以，这倒不是说Java可以将基本数据类型数据赋值给对象，而是编译器私底下帮你做了这些工作

### 可变参数

``` java
	public static void main(String[] args) {
		System.out.println(add(1,2,3,4));
	}
	
	public static int add(int... arr) {
		int sum = 0;
		for (int i : arr) {
			sum += i;
		}
		return sum;
	}
```
而在jdk1.5之前则采用的是数组的形式，如main()方法

### 内省

是Java对Bean类属性和实现的一种缺省处理，他可以通过属性，通过getName，setName来修改，其本质就是通过Introspector来获取beanInfo，进而获取PropertyDescriptor，最后获得getter/setter方法

### 枚举

``` java
	public enum Color {
		RED,GREEN,BLUE;
	}
```
枚举和类一样，也可定义方法之类的，但是枚举的元素都是static final，枚举就是一个天然的单例

### 编程并发库

线程并发库是Java1.5提出的关于多线程处理的高级功能，所在包：java.util.concurrent，这里我就不过多说了，详情可以看并发编程系列

