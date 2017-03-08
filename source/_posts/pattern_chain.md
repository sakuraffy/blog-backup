---
title: 责任链模式(Chain of Responsibility)
date: 2016-12-05 18:43:49
tags: 
	- 设计模式
---
### 什么是责任链

责任链，顾名思义就是将责任以链的形式连接起来，什么是责任，我们可以理解为，对某件事情进行操作，专业一点说就是使多个对象都能对同一对象进行操作，我们每个对象都有下一对象的引用，直到链上的某一个对象处理此请求，但客户并不知道是哪个对象进行处理的，这使得系统可以在不影响客户端的情况下动态地重新组织和分配责任。

责任链模式一个很重要的应用就是对Servlet，request的操作

<!--more-->

### 责任链所涉及的角色

抽象处理者 ： 定义请求接口，一般用抽象类或接口实现
具体处理者 ： 具体实现接口的实现者

### 纯的责任链模式

传统的责任链模式就是被处理对象会被多个对象处理，直到有一个对其进行处理就结束。举一个很简单的例子：公司聚餐费用申请，小组长假设只能批500以内，部门经理可以批1000，而CEO则可以任意额度

下面就对小马虎的申请额度进行审核

#### 抽象处理
``` java
	package cn.sakuraffy.cor;

	public interface SimpleFilter {
		boolean doFilter(int num) ;
	}
```

#### 小组长处理
``` java
	package cn.sakuraffy.cor;

	public class GroupFilter implements SimpleFilter {
		private SimpleFilter next;
		
		public final SimpleFilter getNext() {
			return next;
		}
		public final void setNext(SimpleFilter next) {                                        
			this.next = next;
		}
		
		@Override
		public boolean doFilter(int num) {
			if(num < 500) {
				System.out.println("group deal");
				return true;
			}else {
				return getNext().doFilter(num);
			}
		}
	}
```

#### 经理处理
``` java
	package cn.sakuraffy.cor;

	public class ManagerFilter implements SimpleFilter {
		private SimpleFilter next;
		
		@Override
		public boolean doFilter(int num) {
			if(num < 1000) {
				System.out.println("manager deal");
				return true;
			}
			return getNext().doFilter(num);
		}

		public final SimpleFilter getNext() {
			return next;
		}

		public final void setNext(SimpleFilter next) {
			this.next = next;
		}
	}
```

#### CEO处理
``` java
	package cn.sakuraffy.cor;

	public class CEOFilter implements SimpleFilter {
		private SimpleFilter next;
		
		@Override
		public boolean doFilter(int num) {
			System.out.println("ceo deal");
			return true;
		}

		public final SimpleFilter getNext() {
			return next;
		}

		public final void setNext(SimpleFilter next) {
			this.next = next;
		}
	}
```

#### 小马虎的测试
``` java
	package cn.sakuraffy.cor;

	public class SimpleClient {
		public static void main(String[] args) {
			int num = 800;
			GroupFilter gf = new GroupFilter();
			ManagerFilter mf = new ManagerFilter();
			CEOFilter cf = new CEOFilter();
			gf.setNext(mf);
			mf.setNext(cf);
			System.out.println(gf.doFilter(num));
		}
	}
```

输出结果 ： 
``` java
manager deal
true
```

源码解析 ：
- 这里是为了演示过程，其实小马虎根本不知道是谁处理申请
- 测试中将审核顺序自己设置，也可以在具体的实现中写死（不推荐）

### 不纯的责任链

下面简单模拟一下Servlet的request

#### 抽象处理,FilterChain下面会介绍
``` java 
	package cn.sakuraffy.cor;

	public interface Filter {
		void doFilter(String request, String response, FilterChain chain);
	}
```
#### WorldFilter
``` java
	package cn.sakuraffy.cor;

	public class WorldFilter implements Filter {

		@Override
		public void doFilter(String request, String response,FilterChain chain) {
			request = request.replace("hello", "nihao");
			System.out.println(request);
			System.out.println("WordFilter request");
			chain.doFilter(request, response, chain);
			response = request.replace("nihao", "hello");
			System.out.println(response);
			System.out.println("WordFilter response");
		}
		
	}
```
#### NumberFilter
``` java
	package cn.sakuraffy.cor;

	public class NumberFilter implements Filter {
		
		@Override
		public void doFilter(String request, String response, FilterChain chain) {
			request = request.replace('6', '4');
			System.out.println(request);
			System.out.println("NumberFilter request");
			chain.doFilter(request, response, chain);
			response = request.replace('4', '6');
			System.out.println(response);
			System.out.println("NumberFilter response");
		}

	}
```
#### FilterChain,我们依然可以用引用下一个对象进行多个对象处理，但这里我们可以一下小小的改动，增加一个FilterChain专门用来管理Filter，同时它也实现Filter接口，而它的doFilter()则获取Filter，并调用该Filter相对应的doFilter()

``` java
	package cn.sakuraffy.cor;

	import java.util.ArrayList;
	import java.util.List;

	public class FilterChain implements Filter {
		private int index = 0;
		private List<Filter> filters = new ArrayList<Filter>();

		@Override
		public void doFilter(String request, String response, FilterChain chain) {           
			if(index < filters.size()) {
				Filter filter = filters.get(index);
				index ++;
				filter.doFilter(request, response, chain);
			}
		}
		
		//链式编程
		public FilterChain add(Filter filter) {
			filters.add(filter);
			return this;
		}
	}
```

#### 小马虎的测试
``` java
	package cn.sakuraffy.cor;

	public class Client {
		public static void main(String[] args) {
			String request = "hello everyone 2016";
			String response = "";
			NumberFilter nf = new NumberFilter();
			WorldFilter wf = new WorldFilter();
			FilterChain fc = new FilterChain();
			fc.add(nf).add(wf);
			fc.doFilter(request, response, fc);
		}
	}
```

输出结果 ：
``` java
hello everyone 2014
NumberFilter request
nihao everyone 2014
WordFilter request
hello everyone 2014
WordFilter response
hello everyone 2016
NumberFilter response
```

源码解析 ：
- FilterChain 里面的 index一定要是成员变量，并且使用后要自增，否则会抛出java.lang.StackOverflowError
- public FilterChain add(Filter filter ) 采用的是链式编程，也可以不用，这里只是为了简便
- 在具体的实现类都是调用FilterChain的doFilter(),因为已经将Filter加进FilterChain，我们只需要对链进行操作就行

### 责任链的补充

有时候，我们不需要太拘泥责任链的纯与否，也不要局限于设计模式的条条框框，跳出来可能看到更广阔的天空。我们最终的目的还是业务。另外说一下，过滤器和拦截器都是用的都是责任链模式，只是侧重点不同。
