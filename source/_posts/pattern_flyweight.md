---
title: 享元模式(Flyweight)
date: 2016-12-21 15:14:28
tags:
	- 设计模式
---
在实际开发中，大量的对象存在，有可能会产生内存溢出，有时候，相同的业务，我们只需要调用内存中已有的对象完成，而不需要创建新的对象，这时候，我们就可以使用享元模式。所谓享元模式就是运行共享技术有效地支持大量细粒度对象的复用。在说到享元模式之前，我们先来了解两个概念

- 内部状态 ： 不受外部环境改变而改变的共享部分
- 外部状态 ： 由客户端控制不共享部分

<!--more-->

### 享元模式的UML和角色

{% qnimg pattern/flyweight/p1.jpg 'class:class1 class2' normal:yes %}

- Flyweight : 抽象享元类。规定出所有具体享元角色需要实现的方法
- ConcreteFlyweight : 具体享元类。指定内部状态，为内部状态增加存储空间 
- UnsharedConcreteFlyweight : 非共享具体享元类。指出那些不需要共享的Flyweight子类（一般不怎么用）
- FlyweightFactory : 享元工厂类。用来创建并管理Flyweight对象，它主要用来确保合理地共享Flyweight，当用户请求一个Flyweight时，若存在则返回对象，反之则创建新对象并返回

### 享元模式的实现

#### Flyweight
``` java
	package cn.sakuraffy.flyweight;

	public interface Flyweight {
		void draw();
	}
```
#### ConcreteFlyweight
``` java
	package cn.sakuraffy.flyweight;

	//这里以画方块为例
	public class ConcreteFlyweight implements Flyweight {
		private int width;
		private int hight;
		private Color color;
		
		private ConcreteFlyweight(int width, int hight) {
			super();
			this.width = width;
			this.hight = hight;
		}
		
		//构造时，客户端只能控制颜色
		public ConcreteFlyweight(Color color) {
			this(1,3);
			this.color = color;
		}

		public final int getWidth() {
			return width;
		}

		public final int getHight() {
			return hight;
		}

		public final Color getColor() {
			return color;
		}

		public final void setColor(Color color) {
			this.color = color;
		}

		//打印方块属性
		@Override
		public void draw() {
			System.out.println("color =" + color + " width=" + width + " hight=" + hight);
		}	
	}
```
#### 枚举型的颜色，当然，也可以用String代替
``` java
	package cn.sakuraffy.flyweight;

	public enum Color {
		YELLOW,BLUE,GREEN,RED
	}
```
#### ConcreteFlyweight
``` java
	package cn.sakuraffy.flyweight;

	import java.util.HashMap;

	public class FlyweightFactory {
		private HashMap<Color,Flyweight> flyweights = new HashMap<>();
		
		public Flyweight getFlyweight(Color color) {
			Flyweight fw = flyweights.get(color);
			if (fw == null) {
				fw = new ConcreteFlyweight(color);
				flyweights.put(color, fw);
			}
			return fw;
		}
	}
```
#### Client
``` java
	package cn.sakuraffy.flyweight;

	public class Client {
		public static void main(String[] args) {
			FlyweightFactory ff = new FlyweightFactory();
			Flyweight bf = ff.getFlyweight(Color.BLUE);
			Flyweight bf1 = ff.getFlyweight(Color.BLUE);
			Flyweight rf = ff.getFlyweight(Color.RED);
			bf.draw();
			bf1.draw();
			rf.draw();
			System.out.println(bf == bf1);
		}
	}
```
输出结果 ：
``` java
color =BLUE width=1 hight=3
color =BLUE width=1 hight=3
color =RED width=1 hight=3
true
```

### 享元模式的优缺点

- 优点 ： 降低内存中对象的数量
- 缺点 ： 使其程序逻辑异常复杂

### 享元模式在JDK中的应用

JDK中的String类就很好的使用了享元模式，下面看一下实例
``` java
	package cn.sakuraffy.flyweight;

	public class StringDemo {
		public static void main(String[] args) {
			String s1 = "abc";
			String s2 = new String("abc"); 
			System.out.println(s1 == s2);   //false
			System.out.println(s1.intern() == s2.intern()); //true
		}
	}
```
String中每使用一个字符串都会创建一个String对象，而intern()方法则是通过对象引用去找到那个对象