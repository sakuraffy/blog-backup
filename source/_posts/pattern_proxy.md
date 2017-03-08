---
title: 代理模式(Proxy)
date: 2016-12-14 21:50:05
tags:
	- 设计模式
---
在生活中，我们经常会需要别人代替我们做一些事情，其可能的原因：没有时间，亦或不方便。而程序中也有这种可能存在，我们不直接使用引用对象，而是用其一个“替身”去完成所需要的工作，这就是代理模式

<!-- more -->

代理模式一般分为两种，静态代理和动态代理

### 代理模式的角色和UML类图
Subject: 抽象角色，声明真实对象和代理对象的共同接口
Proxy: 代理角色，用来代理和封装真实主题
RealSubject: 真实角色，实现真正的业务逻辑
Client： 客户端，实现调用与测试
{% qnimg pattern/proxy/p1.jpg 'class:class1 class2' normal:yes %}

### 静态代理
这里先说下业务，房东将房委托中介出租，中介为租房者提供看房机会和收取中介费

#### 抽象角色 Rentable
``` java
package cn.sakuraffy.proxy;

public interface Rentable {
	public void rent();
}
```

#### 真实角色 Host
``` java
package cn.sakuraffy.proxy;

public class Host implements Rentable {

	@Override
	public void rent() {
		System.out.println("---house rent---");
	}

}
```

#### 代理角色 StaticProxy
``` java
package cn.sakuraffy.proxy;

public class StaticProxy implements Rentable {

	private Host host;
	
	public StaticProxy(Host host) {
		super();
		this.host = host;
	}



	@Override
	public void rent() {
		System.out.println("---visit house----");
		host.rent();
		System.out.println("---get cost----");
	}
	
}
```

#### Client角色 StaticProxyClient
``` java
package cn.sakuraffy.proxy;

public class StaticProxyClient {
	public static void main(String[] args) {
		Host host = new Host();
		StaticProxy proxy = new StaticProxy(host);
		proxy.rent();
	}
}
```
输出结果 ：
``` java
---visit house----
---house rent---
---get cost----
```

### 动态代理
在给出相应代码之前，先来说一下动态代理是基于jdk proxy代理，那么就需要实现相对应的接口InvocationHandler，并重写invoke()方法，这里的代理是针对接口来说，只要获取了相对应的接口，就可以相对应的内容，而不仅仅局限于Rentable接口

#### 动态代理类  DynamicProxy
``` java
package cn.sakuraffy.proxy;

import java.lang.reflect.InvocationHandler;
import java.lang.reflect.Method;
import java.lang.reflect.Proxy;

public class DynamicProxy implements InvocationHandler{
	
	private Object targe;
	
	public DynamicProxy(Object targe) {
		super();
		this.targe = targe;
	}

	/**
	 * 	@param proxy 代理类对象
	 * 	@param method 代理类的调用处理程序的方法对象
	 * 	@param args 代理类的调用处理程序的方法的参数
	 */
	@Override
	public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {   
		System.out.println("---dynamic proxy start---");
		Object result = method.invoke(targe, args);
		System.out.println("---dynamic proxy end---");
		return result;
	}
	
	public Object getProxy(){
		return Proxy.newProxyInstance(this.getClass().getClassLoader(),
				targe.getClass().getInterfaces(), this);
	}
}
```
#### 动态代理测试  DynamicProxyClient
``` java
package cn.sakuraffy.proxy;

import java.util.ArrayList;
import java.util.Collection;

public class DynamicProxyClient {
	@SuppressWarnings("unchecked")
	public static void main(String[] args) {
		Host host = new Host();
		DynamicProxy dp = new DynamicProxy(host);
		Rentable proxy = (Rentable) dp.getProxy();
		proxy.rent();
		
		Collection<String> c = new ArrayList<>();
		dp = new DynamicProxy(c);
		Collection<String> proxy2 = (Collection<String>) dp.getProxy();       
		proxy2.add("Hello");
	}
}
```
输出结果 ：
``` java
---dynamic proxy start---
---house rent---
---dynamic proxy end---
---dynamic proxy start---
---dynamic proxy end---
```

### 代理模式适用场景与优缺点
#### 适用场景
- 远程代理：为一个对象在不同的地址空间提供局部代表。这样可以隐藏一个对象存在于不同地址空间的事实
- 虚拟代理：通过使用过一个小的对象代理一个大对象。这样就可以减少系统的开销
- 保护代理：用来控制对真实对象的访问权限

#### 优缺点
- 能够协调调用者和被调用者，在一定程度上降低了系统的耦合度
- 在客户端和真实对象之间增加了代理对象，因此有些类型的代理模式可能会造成请求的处理速度变慢