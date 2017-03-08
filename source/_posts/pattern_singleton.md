---
title: 单例模式(Singleton)
date: 2016-10-18 18:43:49
tags: 设计模式
top: 999
---



### 什么是单例模式

在程序和项目中，我们经常会创建并使用对象，但我们知道频繁地创建对象会消耗大量的时间，也会降低程序的效率，而单例这种设计模式就是这种问题的，单例顾名思义就是一个类只有一个对象

### 单例模式的几种方式

<!--more-->

#### 懒汉式
``` java
	package cn.sakuraffy.singleton;

	public class LazySingleton {
		private final static LazySingleton instance = new LazySingleton();
		
		private LazySingleton() {}
		
		public static LazySingleton newInstance() {
			return instance;
		}
	}
```
- 优点 ： 简单
- 缺点 ： 无论这个类是否被用到，都需要创建instance对象

#### 饿汉式
``` java
	package cn.sakuraffy.singleton;

	public class HungrySingleton {
		private static HungrySingleton instance;
		
		private HungrySingleton() {}

		public static HungrySingleton newInstance() {
			if (instance == null) {
				instance = new HungrySingleton();
			} 
			return instance;
		}
	}
```

- 优点 ： 迟缓，用到的时候才加载。
- 缺点 ： 在多线程下会出现问题。

#### 双重检验式
``` java 
	package cn.sakuraffy.singleton;

	public class DoubleCheckSingleton {
		private static DoubleCheckSingleton instance = null;
		
		private DoubleCheckSingleton() {}
		
		public static DoubleCheckSingleton newInstance() {
			if (instance == null){
				synchronized (DoubleCheckSingleton.class) {
					// 避免判断了instance==null的等待线程重复创建instance
					if (instance == null){  
						instance = new DoubleCheckSingleton();
					}
				}
			}
			return instance;
		}
	}
```

- 优点 ： 可以同步，性能较高
- 缺点 ： 在JVM的编译器优化时无法确定先创建栈中的内存还是堆中的内存
	下面我们来考虑这么一种情况：线程A开始创建SingletonClass的实例，此时线程B调用了getInstance()方法，
	首先判断instance是否为null。按照我们上面所说的内存模型，A已经把instance指向了那块内存，
	只是还没有调用构造方法，因此B检测到instance不为null，于是直接把instance返回了——问题出现了，	尽管instance不为null，但它并没有构造完成，就像一套房子已经给了你钥匙，但你并不能住进去，因为里面还没有收拾。此时，如果B在A将instance构造完成之前就是用了这个实例，程序就会出现错误了！

#### 解决了内存分配的问题了两种方式

使用 volatile 关键词
``` java
	package cn.sakuraffy.singleton;

	public class VolatileSingleton {
		private static volatile VolatileSingleton instance = null;
		
		private VolatileSingleton(){}
		
		public static VolatileSingleton newInstance() {
			if (instance == null){
				synchronized (VolatileSingleton.class) {
					// 避免判断了instance==null的等待线程重复创建instance
					if (instance == null){  
						instance = new VolatileSingleton();
					}
				}
			}
			return instance;
		}
	}
```

使用内部类
``` java
	package cn.sakuraffy.singleton;

	public class InnerClassSingleton {
		private static class SingletonInstance {
			private static final InnerClassSingleton instance = new InnerClassSingleton(); 
		}
		
		private InnerClassSingleton() {}
		
		public static InnerClassSingleton newInstance() {
			return SingletonInstance.instance;
		}
	}
```

### enum  就是天然的单例 (推荐)
``` java
	package cn.sakuraffy.singleton;

	public enum EnumSingleton {
		instance
	}

```

### 单例模式的补充

#### 通过反射破坏单例怎么解决
``` java
	package cn.sakuraffy.singleton;

	import java.lang.reflect.Constructor;

	public class RefectSingletonBug {
		public static void main(String[] args) throws Exception {
			HungrySingleton hs = HungrySingleton.newInstance();
			Class<HungrySingleton> clazz = HungrySingleton.class;
			Constructor<HungrySingleton> c = clazz.getDeclaredConstructor();
			c.setAccessible(true);
			HungrySingleton hs1 = c.newInstance();
			
			System.out.println(hs);
			System.out.println(hs1);
			System.out.println(hs == hs1);
		}
	}
```

- 输出结果：
``` java
cn.sakuraffy.singleton.HungrySingleton@2a139a55
cn.sakuraffy.singleton.HungrySingleton@15db9742
false
```
显然获取的不是同一个对象

- 解决方案：

对构造函数做以下处理
``` java
	private Singleton(){
		if (instance != null) {
			throw new RuntimeException();
		}
	}
```

#### 通过反序列化破坏单例
``` java
	package cn.sakuraffy.singleton;

	import java.io.ByteArrayInputStream;
	import java.io.ByteArrayOutputStream;
	import java.io.ObjectInputStream;
	import java.io.ObjectOutputStream;
	import java.io.Serializable;

	public class SerSingletonBug {
		
		private static class Singleton implements Serializable {
			private static final long serialVersionUID = 1L;
			
			private static final Singleton instance = new Singleton();                    
			
			private Singleton() {}
			
			public static Singleton newInstance() {
				return instance;
			}
		}
		
		public static void main(String[] args) throws Exception {
			Singleton s = Singleton.newInstance();
			
			ByteArrayOutputStream baos = new ByteArrayOutputStream();
			ObjectOutputStream oos = new ObjectOutputStream(baos);
			oos.writeObject(s);
			byte[] bytes = baos.toByteArray();
			oos.close();
			
			ByteArrayInputStream bais = new ByteArrayInputStream(bytes);
			ObjectInputStream ois = new ObjectInputStream(bais);
			Singleton s1 = (Singleton) ois.readObject();
			ois.close();
			
			System.out.println(s);
			System.out.println(s1);
			System.out.println(s1 == s);
		}
	}
```

- 输出结果： 
``` java
cn.sakuraffy.singleton.SerSingletonBug$Singleton@33909752
cn.sakuraffy.singleton.SerSingletonBug$Singleton@55f96302
false
```
显然获取的不是同一个对象，这里要注意的Singleton一定要实现序列化接口，关于序列化的问题，请参考[序列化机制](https://sakuraffy.github.io/java_serializable/)

- 解决方案：

对Singleton添加以下方法
``` java
	private Object readResolve() throws ObjectStreamException {
		return instance;
	}
```