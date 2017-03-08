---
title: 并发编程（十五）线程池（二）
date: 2016-07-23 00:13:58
tags:
	- 并发编程
---
### ThreadFactory

在上节，我们介绍了线程池的作用就是复用线程，那么问题来了，我们一直都没有创建线程，那么线程又是从何而来呢？
还记得上次提到的那个ThreadPoolExecutor最原始的构造方法吗？里面有个参数ThreadFactory，线程池中所有的线程都是有它创造出来的
``` java
	public interface ThreadFactory {
		Thread newThread(Runnable r);
	}
```
下面给出一个自己建造ThreadFactory的示例

<!--more-->

``` java
	public class TestThreadFactory implements Runnable {
		@Override
		public void run() {
			System.out.println("HelloWorld");
		}
		
		public static void main(String[] args) {
			TestThreadFactory ttf = new TestThreadFactory();
			ExecutorService es = new ThreadPoolExecutor(5, 5, 0L, TimeUnit.SECONDS,       
					new LinkedBlockingQueue<Runnable>(), 
					new ThreadFactory() {
						
						@Override
						public Thread newThread(Runnable r) {
							Thread t = new Thread(r);
							t.setDaemon(true);
							System.out.println("create " + t);
							return t;
						}
					});
			for(int i = 0; i < 5; i++) {
				es.execute(ttf);
			}
		}
	}
```
输出结果 ：
``` java
create Thread[Thread-0,5,main]
create Thread[Thread-1,5,main]
create Thread[Thread-2,5,main]
create Thread[Thread-3,5,main]
create Thread[Thread-4,5,main]
```
### 拓展线程池

JDK已经给我们提供了一个相当稳固的线程池，但如果我们还想对其进行拓展，可不可以呢？
JDK的ThreadPoolExecutor还为我们提供了beforeExecute(),afterExecute()和terminated()三个接口供我们拓展
``` java
	//运行前
	protected void beforeExecute(Thread t, Runnable r) { }
	//运行结束
	protected void afterExecute(Runnable r, Throwable t) { }
	//线程池退出
	protected void terminated() { }
```
下面给出一个示例
``` java
	public class ExpandThreadPoolExecutor implements Runnable {
		private String name;
		public ExpandThreadPoolExecutor(String name) {
			this.name = name;
		}
		@Override
		public void run() {
			System.out.println("HelloWorld" + name);
			try {
				Thread.sleep(1000);
			} catch (InterruptedException e) {
				// TODO Auto-generated catch block
				e.printStackTrace();
			}
		}
		public static void main(String[] args) {
			ExecutorService es = new ThreadPoolExecutor(2, 2, 0L, 
					TimeUnit.SECONDS, new LinkedBlockingQueue<Runnable>() ) {
				@Override
				protected void beforeExecute(Thread t, Runnable r) {
					// TODO Auto-generated method stub
					System.out.println("before " + ((ExpandThreadPoolExecutor) r).name);
				}
				
				@Override
				protected void afterExecute(Runnable r, Throwable t) {
					// TODO Auto-generated method stub
					System.out.println("after " + ((ExpandThreadPoolExecutor) r).name);
				}
				
				@Override
				protected void terminated() {
					System.out.println("thread pool exit");
				}
			};

			for(int i = 0; i < 2; i++) {
				es.execute(new ExpandThreadPoolExecutor("thread_" + i));
			}
			es.shutdown();
		}
	}
```
输出结果 ：
``` java
before thread_1
before thread_0
HelloWorldthread_1
HelloWorldthread_0
after thread_0
after thread_1
thread pool exit
```


### 线程池中的幽灵错误

什么都不说，先上代码
``` java
	public class ThreadPoolBug implements Runnable {
		private int a;
		private int b;
		
		public ThreadPoolBug(int a, int b) {
			this.a = a;
			this.b = b;
		}
		
		@Override
		public void run() {
			double ret = a / b;
			System.out.println(ret);
		}
		
		public static void main(String[] args) throws InterruptedException, ExecutionException {
			ExecutorService es = Executors.newCachedThreadPool();
			for(int i = 0; i < 5; i++) {
				es.submit(new ThreadPoolBug(100, i));
				//es.submit(new ThreadPoolBug(100, i)).get();
				//es.execute(new ThreadPoolBug(100, i));
			}
		}
	}
```
输出结果 ：
``` java
100.0
33.0
25.0
50.0
```

不对啊！100/0为什么不报错呢？
当将submit()换成execute()就出现错误了。原来错误也可以这么可爱
``` java
Exception in thread "pool-1-thread-1" java.lang.ArithmeticException: / by zero
	at cn.it.thread.concurrent.ThreadPoolBug.run(ThreadPoolBug.java:17)
	at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1142)
	at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:617)
	at java.lang.Thread.run(Thread.java:745)
100.0
33.0
25.0
50.0
```
先来说一说submit()和execute()区别吧
``` java
	public void execute(Runnable command) {}
	Future<?> submit(Runnable task) {}
```
我们看一下Future怎么才能获取结果，这也就是为什么es.submit(new ThreadPoolBug(100, i)).get()会报错
``` java
	/**
     * Waits if necessary for the computation to complete, and then
     * retrieves its result.
     *
     * @return the computed result
     * @throws CancellationException if the computation was cancelled
     * @throws ExecutionException if the computation threw an
     * exception
     * @throws InterruptedException if the current thread was interrupted
     * while waiting
     */
    V get() throws InterruptedException, ExecutionException;
```

再来说说Exception的问题吧。仔细看看总感觉少点什么？少什么呢？在main()里谁提交的呢？没有指出来，线程池将这个Exception吞没了，下面就应该想办法怎么将它展示出来，代码如下
``` java
	public class ThreadPoolBug implements Runnable {
	
		public static class ThreadPoolTrace extends ThreadPoolExecutor {

			public ThreadPoolTrace(int corePoolSize, int maximumPoolSize,
					long keepAliveTime, TimeUnit unit,
					BlockingQueue<Runnable> workQueue) {
				super(corePoolSize, maximumPoolSize, keepAliveTime, unit, workQueue);
			}
			
			@Override
			public void execute(Runnable command) {
				super.execute(wrap(command, cilentTrace()));
			}
			
			@Override
			public Future<?> submit(Runnable task) {
				// TODO Auto-generated method stub
				return super.submit(wrap(task, cilentTrace()));
			}
			
			private Exception cilentTrace() {
				return new Exception("client stack trace");
			}
			
			private Runnable wrap(final Runnable task, final Exception clientstack) {
				return new Runnable() {
					
					@Override
					public void run() {
						try {
							task.run();
						}catch (Exception e) {
							clientstack.printStackTrace();
							throw e;
						}
					}
				};
			}
		}
		
		private int a;
		private int b;
		
		public ThreadPoolBug(int a, int b) {
			this.a = a;
			this.b = b;
		}
		
		@Override
		public void run() {
			double ret = a / b;
			System.out.println(ret);
		}
		
		public static void main(String[] args) throws InterruptedException, ExecutionException {
			ExecutorService es = new ThreadPoolTrace(5, 5, 0L, TimeUnit.SECONDS,
					new LinkedBlockingQueue<Runnable>());
			for(int i = 0; i < 5; i++) {
				es.submit(new ThreadPoolBug(100, i)).get();
				//es.execute(new ThreadPoolBug(100, i));
			}
		}
	}

```
输出结果 ：
``` java
java.lang.Exception: client stack trace
	at cn.it.thread.concurrent.ThreadPoolBug$ThreadPoolTrace.cilentTrace(ThreadPoolBug.java:35)
	at cn.it.thread.concurrent.ThreadPoolBug$ThreadPoolTrace.submit(ThreadPoolBug.java:30)
	at cn.it.thread.concurrent.ThreadPoolBug.main(ThreadPoolBug.java:73)
Exception in thread "main" java.util.concurrent.ExecutionException: java.lang.ArithmeticException: / by zero
	at java.util.concurrent.FutureTask.report(FutureTask.java:122)
	at java.util.concurrent.FutureTask.get(FutureTask.java:192)
	at cn.it.thread.concurrent.ThreadPoolBug.main(ThreadPoolBug.java:73)
Caused by: java.lang.ArithmeticException: / by zero
	at cn.it.thread.concurrent.ThreadPoolBug.run(ThreadPoolBug.java:65)
	at cn.it.thread.concurrent.ThreadPoolBug$ThreadPoolTrace$1.run(ThreadPoolBug.java:45)
	at java.util.concurrent.Executors$RunnableAdapter.call(Executors.java:511)
	at java.util.concurrent.FutureTask.run(FutureTask.java:266)
	at cn.it.thread.concurrent.ThreadPoolBug$ThreadPoolTrace$1.run(ThreadPoolBug.java:45)
	at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1142)
	at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:617)
	at java.lang.Thread.run(Thread.java:745)
```
说说整体的思路吧
为什么会被吞没呢？因为线程池在调度任务前没有保存任务线程的堆栈信息。那么解决方案就应该是将堆栈信息包装到Runnable对象里面去，而用一个方法正好，而恰巧Exception可以打印，所以方法就选择了返回值为Exception的方法

### Fork/Join框架

我们知道分治算法讲究的分，治，合，关于分治算法，请参考[分治算法](https://sakuraffy.github.io/algorithm_divide/)
JDK中的Fork/Join框架和其异曲同工之妙
``` java
	@since 1.7
	public class ForkJoinPool extends AbstractExecutorService {
		public final ForkJoinTask<V> fork() {}
		public final V join() {}
		public void execute(ForkJoinTask<?> task) {}
		public <T> ForkJoinTask<T> submit(ForkJoinTask<T> task) {}
	}
	@since 1.7
	public abstract class ForkJoinTask<V> implements Future<V>, Serializable {}
	//没有返回值的任务
	public abstract class RecursiveAction extends ForkJoinTask<Void> {
		protected abstract void compute();
	}
	//携带返回值的任务
	public abstract class RecursiveTask<V> extends ForkJoinTask<V> {
		protected abstract V compute();
	}
```
这就JDK提供的类及其重要方法。我们使用Fork/Join创建任务必须继承RecursiveTask和RecursiveAction其中之一
下面给出一个示例
``` java
	public class ForkJoin extends RecursiveTask<Long>{

		private static final long serialVersionUID = 1L;

		private static final int THRESHOLD = 10000;
		private long start;
		private long end;
		
		public ForkJoin(long start, long end) {
			super();
			this.start = start;
			this.end = end;
		}
		
		@Override
		protected Long compute() {
			long sum = 0;
			boolean canComputor = (end - start) < THRESHOLD;
			if (canComputor) {
				for(long i = start; i <= end; i++) {
					sum += i;
				}
			}else {
				long step = (end -start) / 100;
				ArrayList<ForkJoin> subTasks = new ArrayList<>();
				long pos = start;
				for (int i = 0; i < 100; i++) {
					long lastOne = pos + step;
					if (lastOne > end) {
						lastOne = end;
					}
					ForkJoin subTask = new ForkJoin(pos, lastOne);
					pos += step + 1;
					subTasks.add(subTask);
					subTask.fork();
				}
				for (ForkJoin t : subTasks) {
					sum += t.join();
				}
			}
			return sum;
		}

		public static void main(String[] args) throws InterruptedException, ExecutionException {
			ForkJoinPool pool = new ForkJoinPool();
			ForkJoin fj = new ForkJoin(0, 200000L);
			ForkJoinTask<Long> task = pool.submit(fj);
			long ret = task.get();
			System.out.println(ret);
		}
	}

```
输出结果 ：
``` java
20000100000
```
