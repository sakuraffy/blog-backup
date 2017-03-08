---
title: Java对象的四种创建方式
date: 2016-09-19 09:40:05
tags:
	- java
---
在Java编程，我们成天与对象打交道，但你又是否知道对象怎么创建，以及又有哪几种创建方式呢？

一般认为对象的创建有四种方式 ： new关键字、利用反射、Clone以及反序列化

### New

这应该是大家最熟悉的
``` java
	Object obj = new Object();
```

<!--more-->

### 反射

通过反射一般有两种方式 ： Class的newInstance()以及Constructor的newInstance()
``` java
	public class Test{
	
		public static void main(String[] args) throws Exception {
			Test test1 = Test.class.newInstance();
			Test test2 = Test.class.getConstructor().newInstance();
			System.out.println(test1);
			System.out.println(test2);
		}
	}
```
输出结果 ：
``` java
Test@2a139a55
Test@15db9742
```

### Clone

关于克隆的问题，请参考[原型模式]()
``` java
	public class Test implements Cloneable{
		public static void main(String[] args) throws CloneNotSupportedException {
			Test test1 = new Test();
			Test test2 = (Test) test1.clone();
			System.out.println(test1);
			System.out.println(test2);
		}
	}
```
输出结果 ：
``` java
	Test@2a139a55
	Test@15db9742
```

### 反序列化

关于序列化的问题，请参考[序列化机制](https://sakuraffy.github.io/pattern_serializable/)
``` java
	public class Test{
		public static class User implements Serializable {
			private static final long serialVersionUID = 1L;
			
			public User(int id) {
				System.out.println("haha");
				this.id = id;
			}
			
			private int id;

			public final int getId() {
				return id;
			}

			public final void setId(int id) {
				this.id = id;
			}
			
		}

		public static void main(String[] args) throws Exception, IOException{
			File file = new File("data.txt");
			ObjectOutputStream oos = new ObjectOutputStream(
					new FileOutputStream(file));
			ObjectInputStream ois = new ObjectInputStream(
					new FileInputStream(file));
			User user1 = new User(1);
			System.out.println(user1);
			oos.writeObject(user1);
			oos.close();
			User user2 = (User) ois.readObject();
			System.out.println(user2);
			ois.close();
		}
	}
```
输出结果 ：
``` java
	haha
	Test$User@2a139a55
	Test$User@55f96302
```

这里值得注意的是：无论是哪一种创建方式，所创建的对象都是不同的。通过观察字节码，除了new采用INVOKESPECIAL，其余都是INVOKEVIRTUAL
