---
title: 原型模式(Prototype)
date: 2016-11-17 22:42:20
tags:
	- 设计模式
---
在日常开发中，有些类的结构异常复杂，但恰巧我们需要频繁地创建与使用它们。我们知道创建对象会占用大量的CPU资源，这时候我们就可以使用原型模式将该类的对象进行复制，从而减少因对象创建而消耗的资源和时间

### 原型模式的UML图和角色

{% qnimg pattern/prototype/p1.jpg 'class:class1 class2' normal:yes %}

Prototype：抽象原型类。声明克隆自身的接口(这里直接借用Java的Cloneable接口)。 
ConcretePrototype：具体原型类。实现克隆的具体操作。 

<!--more-->

### 源码实现 
``` java
	package cn.sakuraffy.prototype;

	public class ConcretePrototype implements Cloneable {                                       

		private String name;
		private int id;
		
		public ConcretePrototype(String name, int id) {                                       
			super();
			this.name = name;
			this.id = id;
		}

		public final String getName() {
			return name;
		}

		public final void setName(String name) {
			this.name = name;
		}

		public final int getId() {
			return id;
		}

		public final void setId(int id) {
			this.id = id;
		}

		@Override
		protected Object clone() {
			ConcretePrototype cp = null;
			try {
				cp = (ConcretePrototype) super.clone();
			} catch (CloneNotSupportedException e) {
				// TODO Auto-generated catch block
				e.printStackTrace();
			}
			return cp;
		}

		@Override
		public String toString() {
			return "ConcretePrototype [name=" + name + ", id=" + id + "]";
		}
		
	}
```
#### 测试类 ：
``` java
	package cn.sakuraffy.prototype;

	public class Client {
		public static void main(String[] args) {
			ConcretePrototype cp = new ConcretePrototype("sakuraffy",8);
			ConcretePrototype cp1 = (ConcretePrototype) cp.clone();
			System.out.println(cp);
			System.out.println(cp1);
			System.out.println(cp.getClass() == cp1.getClass());
			System.out.println(cp == cp1);
		}
	}
```
#### 输出结果 ：
``` java
ConcretePrototype [name=sakuraffy, id=8]
ConcretePrototype [name=sakuraffy, id=8]
true
false
```

### 原型模式优缺点及应用场景

#### 优点 
- 提高对象创建效率
- 逃避构造函数的约束
#### 缺点 
- 逃避构造函数的约束
- 需要为每一个类装配一个克隆方法。对已有类进行改造，需要修改其源码，违背开关原则
#### 应用场景
- 创建对象成本较大的时候

### 原型模式在Java中的应用

#### 浅复制
浅复制(浅克隆) 简单来说 ： 浅复制仅仅考虑要复制的对象，而不考虑该对象所引用的对象
``` java
	package cn.sakuraffy.prototype;

	public class ShallowClone implements Cloneable {                                              
		private int id;
		private String name;
		private Book book;
		
		private static class Book {
			private String name;

			public Book(String name) {
				super();
				this.name = name;
			}

			@Override
			public String toString() {
				return "Book [name=" + name + "]";
			}
			
		}
		
		public ShallowClone(int id, String name, Book book) {
			super();
			this.id = id;
			this.name = name;
			this.book = book;
		}

		@Override
		public String toString() {
			return "ShallowClone [id=" + id + ", name=" + name + ", book=" + book
					+ "]";
		}

		@Override
		protected Object clone() {
			ShallowClone sc = null;
			try {
				sc = (ShallowClone) super.clone();
			} catch (CloneNotSupportedException e) {
				// TODO Auto-generated catch block
				e.printStackTrace();
			}
			return sc;
		}
		
		public static void main(String[] args) throws InterruptedException {
			Book book = new Book("Goodbye");
			ShallowClone sc = new ShallowClone(1, "sakuraffy", book);
			DeepClone sc1 = (DeepClone) sc.clone();
			Thread.sleep(1000);
			//这里简单使用，用getter和setter更合理
			sc1.name = "sakura";
			sc1.book.name = "Hello";
			sc1.id = 2;
			System.out.println(sc);
			System.out.println(sc1);
			System.out.println(sc.book == sc1.book);
		}
	}
```
输出结果 ：
``` java
ShallowClone [id=1, name=sakuraffy, book=Book [name=Hello]]
ShallowClone [id=2, name=sakura, book=Book [name=Hello]]
true
```

通过对比输出可以看出 ：两个ShallowClone对象所引用的Book是同一对象，String和Date都是实行深复制的

#### 深复制
深复制(深克隆) 简单来说 ： 深复制不仅仅考虑要复制的对象，而且还考虑该对象所引用的对象

``` java
	package cn.sakuraffy.prototype;

	public class DeepClone implements Cloneable {
		private int id;
		private String name;
		private Book book;
		
		//要实现对象引用的深复制，其所引用的对象必须实现实现Cloneable接口                          
		private static class Book implements Cloneable {
			private String name;

			public Book(String name) {
				super();
				this.name = name;
			}

			@Override
			protected Object clone() throws CloneNotSupportedException {                  
				return super.clone();
			}
			
			@Override
			public String toString() {
				return "Book [name=" + name + "]";
			}
			
		}
		
		public DeepClone(int id, String name, Book book) {
			super();
			this.id = id;
			this.name = name;
			this.book = book;
		}

		@Override
		public String toString() {
			return "DeepClone [id=" + id + ", name=" + name + ", book=" + book
					+ "]";
		}

		@Override
		protected Object clone() {
			DeepClone dc = null;
			try {
				dc = (DeepClone) super.clone();
				//重点
				dc.book = (Book) this.book.clone();
			} catch (CloneNotSupportedException e) {
				// TODO Auto-generated catch block
				e.printStackTrace();
			}
			return dc;
		}
		
		public static void main(String[] args) throws InterruptedException {
			Book book = new Book("Goodbye");
			DeepClone dc = new DeepClone(1, "sakuraffy", book);
			DeepClone dc1 = (DeepClone) dc.clone();
			//这里简单使用，用getter和setter更合理
			dc1.name = "sakura";
			dc1.id = 2;
			dc1.book.name = "Hello";
			System.out.println(dc);
			System.out.println(dc1);
			System.out.println(dc.book == dc1.book);
		}
	}
```
输出结果 ：
``` java
DeepClone [id=1, name=sakuraffy, book=Book [name=Goodbye]]
DeepClone [id=2, name=sakura, book=Book [name=Hello]]
false
```

通过对比输出可以看出 ：两个DeepClone对象所引用的Book不是同一对象，它的实现就是再将Book都克隆一份给克隆对象