---
title: JVM内存分区
date: 2016-09-17 01:58:23
tags:
	- JVM
---
我们知道对于Java而言，.java文件是不能运行的，需要编译成为.class文件才能被load到内存被执行，那么JVM(Java Virtual Machine)的内存又是怎么进行分配及各部分又该实现什么功能的呢？
一般认为JVM内存主要分为PC寄存器，堆，栈，方法区，本地方法区五大部分
### PC寄存器

JVM可以同时支持多个线程同时执行，每一个线程都有自己的PC寄存器，在任意时刻，一个线程只能执行一个方法的代码，这个正在被执行的方法叫做当前方法，若这个方法是native，那PC寄存器就保存JVM正在执行的字节码指令的地址，如果这个方法不是native的，那么PC寄存器的值为undefined，PC寄存器的大小至少应当为一个与平台相关的本地指针的值或一个返回值类型的数据

<!--more-->

### 堆

堆的容量可以是固定的，也可以是动态分配的，它所占用内存不需要保证是连续的

堆是供各个线程共享的运行时内存区域，它是存放所有类实例和数组对象分配的区域

###  栈

栈的容量可以是固定的，也可以是动态分配的，它所占用内存不需要保证是连续的
每个线程都拥有自己的栈，栈是存放局部变量和一些尚未计算好的值

#### 栈帧
栈帧是用来存储数据和部分过程结构的数据结构，同时也用来处理动态链接、方法返回值和异常分派
栈帧随着方法的调用而创建，随着方法的结束而销毁。栈帧的存储空间由创建它的线程分配在栈中，每一个栈帧都有自己的局部变量表，操作数栈和指向当前方法所属的类的常量池的引用

	- 局部变量表
	简单来说局部变量就是用来存储变量的表，局部变量使用索引来进行定位访问。首个局部变量的索引值是0，所存储的是该实例方法所在对象的引用（即Java语言中的this关键字）
	一个局部变量所占4个字节，也就是说一个局部变量可以保存一个类型为boolean、char、byte、short、int、float、reference或returnAddress的数据，而对于long和double类型的数据则需要两个局部变量进行保存

	- 操作数栈
	每个栈帧中包含一个先进后出的操作数栈，操作数栈的作用就是将局部变量表中的数据进行字节码指令操作，其中一个long或double类型数据占两个单位的栈深度 

	- 动态链接
	每个栈帧内部都有一个指向当前方法所在类型的常量池的引用，以便对当前方法的代码实现动态链接，在class文件中一个方法要想调用其它方法或者成员常量，则需要通过符号引用来表示，动态链接的作用就是这些符号引用所表示的方法或成员变量转换为对实际方法的直接引用，通过类加载加载调用

### 方法区

- 方法区的容量可以是固定的，也可以是动态分配的，它所占用内存不需要保证是连续的。它是堆的逻辑组成部分
- 在JVM中，方法区是可供各个线程共享的运行时内存区域。方法区存储每个类的结构信息，例如，字段、方法数据、构造函数、普通方法和常量池的字节码内容

### 常量池

运行时常量池是class文件中每一个接口或类运行时的表现形式，它包含若干种不同的常量，从编译期可知的数值字面量到必须在运行期解析后才能获得的方法或字段
下面展示一个Java初学者容易犯的错误
``` java
public class TestString {
	public static void main(String[] args) {
		String str1 = "abc";
		String str2 = new String("abc");
		String str3 = "a" + "bc";
		String str4 = new String("a") + new String("bc");
		String s5 = "a";
		String s6 = "bc";
		String str5 = s5 + s6;
		System.out.println(str1 == str2);
		System.out.println(str1 == str3);
		System.out.println(str1 == str4);
		System.out.println(str1 == str5);
		System.out.println(str2 == str3);
		System.out.println(str2 == str4);
		System.out.println(str2 == str5);
	}
}
```
输出结果 ：
``` java
false
true
false
false
false
false
false
```

通过对比分析可知 ：
- new出来的对象==比较只有同一对象才会为true
- String str3 = "a" + "bc";编译器默认合并，与String str3 = "abc"等价
- String str1 = "abc";在常量池中分配，String str2 = new String("abc");在堆中分配

### 本地方法栈

JVM实现可能会使用到传统栈（例如C stack）来支持native方法（使用Java以外的其它语言编写的方法）的执行，这个栈就是本地方法栈。当JVM使用其他语言来实现指令解释器时，也可以使用本地方法栈
