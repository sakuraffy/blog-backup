---
title: Java运算符
date: 2016-09-08 12:55:38
tags:
	- Java
---
我们知道计算机最开始的时候就是用来计算的，对于Java来说有很多的运算符，有些我们还是要记一下的，虽说用我们所知的一样可以实现，但是性能却远远没有那么高。Java运算符一般分为算数运算符，关系运算符，位运算符，逻辑运算符，其他运算符。

### 算数运算符

算数运算符一般指 ： 加减乘除，取余、自增、自减，这里重点说一下自增

<!--more-->
	
``` java
	package cn.sakuraffy;

	public class Test {
		public static void main(String[] args) {
			int a = 0; 
			int b = 0;
			int c = 0;
			int d = 0;
			// 先赋值，后增加
			c = a++;
			// 先增加，后赋值
			d = ++b;
			System.out.println(a);
			System.out.println(b);
			System.out.println(c);
			System.out.println(d);
		}
	}
```
输出结果 ： 
``` java
	1
	1
	0
	1
```

### 关系运算符

关系运算符一般指 ： 等于，大于，小于，不等于这些

### 位运算符

位运算符只能用于byte，char，short，int，long 这些数据类型，都是对其二进制数字进行操作

#### 位与(&)

按位与操作符，当且仅当两个操作数的某一位都非0时候结果的该位才为1

#### 位或(|)

按位或操作符，只要两个操作数的某一位有一个非0时候结果的该位就为1

#### 位异或(^)

按位异或操作符，两个操作数的某一位不相同时候结果的该位就为1

#### 位非(~)

按位补运算符翻转操作数的每一位

#### 左移(<<)

左侧操作数的值根据右侧操作数指定的位的数量移至左侧

#### 右移(>>)

左侧操作数的值根据右侧操作数指定的位的数量移至右侧

#### 无符号右移(>>>)

左侧操作数的值根据右侧操作数指定的位的数量移至右，并且转移的值用零补满

这里先说一下负数的编码，我们知道以8位为例3的二进制为00000011，那么-3的二进制是不是10000011呢，因为首位表示正负嘛

其实不是这样的，因为如果这样，那么计算时，加法和减法就有两套规则，而计算机采用的是补码的形式，那么又该怎么表示负数的编码呢？以8位-5为例步骤如下 ：
1. 写出负数绝对值的二进制 -->  00000101
2. 对每一位取反			  -->  11111010
3. 对取反的数加1		  -->  11111011

``` java
	package cn.sakuraffy;

	public class Test {
		public static void main(String[] args) {
			int a = 60;  // a = 0011 1100
			int b = 13;  // b = 0000 1101
			System.out.println(a & b); //00001100 = 12
			System.out.println(Integer.toBinaryString(a & b));
			
			System.out.println(a | b); //00111101 = 61
			System.out.println(Integer.toBinaryString(a | b));
			
			System.out.println(a ^ b); //00110001 = 49
			System.out.println(Integer.toBinaryString(a ^ b));
			
			System.out.println(~a);    //11000011 = -61
			System.out.println(Integer.toBinaryString(~a));
			
			int c = 16;
			int d = -16;
			System.out.println(c >> 2); //4
			System.out.println(c << 2); //64
			System.out.println(c >>> 2); // 4
			
			System.out.println(d >> 2); //-4
			System.out.println(d << 2); //-64
			System.out.println(d >>> 2); // 1073741820
			
		}
	}
```

### 逻辑运算符

逻辑运算符一般指 ： 
&&(且)  
||(或)
！(非)