---
title: 并发编程（二十二）Future模式
date: 2016-08-04 20:40:50
tags:
	- 并发编程
---
Future模式的核心就是程序的异步。所谓程序的异步就是不再等待执行结果，而是开一个线程去执行并获得结果，而调用线程则继续向下执行
Future模式的角色组成 
- Main ： 系统启动，调用Client发出请求
- Client ：返回Data对象，立即返回FutureData对象，并开启线程装配RelData
- Data ：返回数据的接口
- FutureData ： Future数据，构造速度快，但是虚拟数据，需要装配RelData
- RelDate ：真实数据，构造速度慢

<!--more-->
### 代码实现

Data接口定义
``` java
	public static interface Data {
		public String getResult();
	}
```
FutureData定义
``` java
	public static class FutureData implements Data {
		private RelData relData;
		private boolean isReady = false;
		
		public synchronized final void setRelData(RelData relData) {                          
			if(isReady) {
				return ;
			}
			this.relData = relData;
			isReady = true;
			notifyAll();
		}
		@Override
		public String getResult() {
			while(!isReady) {
				try {
					wait();
				} catch (InterruptedException e) {
					// TODO Auto-generated catch block
					e.printStackTrace();
				}
			}
			return relData.getResult();
		}
	}
```
RelData定义
``` java
	public static class RelData implements Data {
		private String result;
		
		public final void setResult(String result) {                                          
			this.result = result;
		}

		public RelData(String request) {
			StringBuilder sb = new StringBuilder();
			for(int i = 0; i < 3; i++) {
				sb.append(request);
			}
			try {
				Thread.sleep(500);
			} catch (InterruptedException e) {
				// TODO Auto-generated catch block
				e.printStackTrace();
			}
			result = sb.toString();
		}
		
		@Override
		public String getResult() {
			return result;
		}
	}
```
Client定义
``` java
	public static class Client {
		public Data request(String request) {
			final FutureData futureData = new FutureData();
			new Thread() {
				@Override
				public void run() {
					RelData relData = new RelData(request);
					futureData.setRelData(relData);
				};
			}.start();
			return futureData;
		}
	}
```
Main
``` java
	public static void main(String[] args) throws InterruptedException {
		Client client = new Client();
		Data data = client.request("name");
		System.out.println("request finsh");
		Thread.sleep(2000);
		System.out.println("data : " + data.getResult());
	}
```
输出结果 ：
``` java
request finsh
data : namenamename
```

### JDK中的Future模式

下面我们看下Future的源码架构
``` java
	public interface RunnableFuture<V> extends Runnable, Future<V> {}
	public class FutureTask<V> implements RunnableFuture<V> {
		//取消任务
		public boolean cancel(boolean mayInterruptIfRunning) {}
		//是否已经取消
		public boolean isCancelled() {}
		//是否已经完成
		public boolean isDone() {}
		//返回取得对象
		public V get() throws InterruptedException, ExecutionException {}
		//返回取得对象，设置超时时间
		public V get(long timeout, TimeUnit unit) throws InterruptedException, ExecutionException, TimeoutException {}
	}
	public interface Callable<V> {
		V call() throws Exception;
	}
```
Callable接口中只有一个call(),它会返回需要构造的实际数据
``` java
	public class TestFuture {
		public static class RelData implements Callable<String>{
			private String para;
			
			public RelData(String para) {
				super();
				this.para = para;
			}

			@Override
			public String call() throws Exception {
				StringBuilder sb = new StringBuilder();
				for(int i = 0; i < 3; i++) {
					sb.append(para);
				}
				Thread.sleep(500);
				return sb.toString();
			}
		}
		public static void main(String[] args) throws InterruptedException, ExecutionException {
			ExecutorService es = Executors.newCachedThreadPool();
			FutureTask<String> future = new FutureTask<>(new RelData("hello"));
			es.execute(future);
			System.out.println("request finsh");
			Thread.sleep(2000);
			System.out.println(future.get());
		}
	}
```
输出结果 ：
``` java
request finsh
hellohellohello
```

