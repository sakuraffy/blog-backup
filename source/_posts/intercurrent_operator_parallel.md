---
title: 并发编程（二十三）并行操作
date: 2016-08-06 18:53:13
tags:
	- 并发编程
---
### 并发流水线

我们知道一个产品的生产会经过很多步骤，工厂会让员工每一个步骤都执行，从头到尾的生产这个商品吗？
显然不会，工厂会将产品的生产分为好几个部分，然后每个人分部分完成。如果将其抽象成为线程，每个部门的线程组合起来就是并发流水线

### 并发搜索

搜索是一个很常见的操作，对于有序数组，我们可以采用二分法搜索。当在并发编程中，并发搜索就是将数组分为多个段，每一段用一个线程进行查找

<!--more-->

下面就演示一个实例
``` java
	public class IntercurrentSearch {
		public static final int NUM = 20;
		public static final int THREAD_NUM = 5;
		
		private static int[] array;
		
		public static void makeData() {
			Random r = new Random();
			int[] arr = new int[NUM];
			for(int i = 0; i < NUM; i++) {
				arr[i] = i;
			}
			for(int i = 0; i < NUM; i++) {
				int temp = r.nextInt(NUM);
				int t = arr[i];
				arr[i] = arr[temp];
				arr[temp] = t;
			}
			array = arr;
		}
		
		public static class SearchTask implements Callable<Integer> {
			private int start;
			private int end;
			private int searchValue;
			
			public SearchTask(int start, int end, int searchValue) {
				super();
				this.start = start;
				this.end = end;
				this.searchValue = searchValue;
			}

			@Override
			public Integer call() throws Exception {
				for(int i = start; i < end; i++) {
					if (searchValue == array[i]) {
						return i;
					}
				}
				return -1;
			}
		}
		
		public static void main(String[] args) throws InterruptedException, ExecutionException {
			makeData();
			int searchValue = 1;
			int size = NUM / THREAD_NUM;
			int ret = -1;
			ExecutorService es = Executors.newFixedThreadPool(THREAD_NUM);
			List<Future<Integer>> futures = new ArrayList<>(THREAD_NUM);
			for(int i = 0; i < NUM; i += size) {
				Future<Integer> future = es.submit(new SearchTask(i, i + size, searchValue));
				futures.add(future);
			}
			for(int i = 0; i < THREAD_NUM; i++) {
				if (futures.get(i).get() > 0) {
					ret = futures.get(i).get();
				}
			}
			System.out.println(Arrays.toString(array));
			System.out.println(ret);
		}
	}
```
输出结果 ：
``` java
[13, 19, 12, 2, 0, 4, 10, 11, 9, 16, 8, 14, 5, 7, 18, 1, 6, 15, 17, 3]
15
```

### 并发排序

并发排序的关键是解决分离数据相关性。什么是数据的相关性呢？就冒泡排序而言，每次迭代都会比较前后两个数据的大小，也就是说每一个数据都要和左右数据进行比较。这个就不符合了。那怎么能将其改变为并发的呢？如果我们分为基偶，每个数据只与后面一个数比较这就可以了
先来看看串行的基偶排序
``` java
	public static void sort(int[] arr) {
		// 是否交换
		boolean exchFlag = true;
		int start = 0;
		//当两轮循环都没有进行交换结束
		while(start == 0 || exchFlag) {
			exchFlag = false;
			for(int i = start; i < arr.length - 1; i += 2) {                              
				if (arr[i] > arr[i+1]) {
					int temp = arr[i];
					arr[i] = arr[i+1];
					arr[i+1] = temp;
					exchFlag = true;
				}
			}
			if (start == 0) {
				start = 1;
			}else {
				start = 0;
			}
		}
	}
```
再来看看并行的基偶排序
``` java
	public class IcOddEvenSort {
		private static boolean exchFlag = true;

		public static class OddEvenSort implements Runnable {
			private int i;
			private CountDownLatch cdl;
			private int[] arr;
			
			public OddEvenSort(int[] arr, int i, CountDownLatch cdl) {                    
				super();
				this.arr = arr;
				this.i = i;
				this.cdl = cdl;
			}

			@Override
			public void run() {
				if (arr[i] > arr[i+1]) {
					int temp = arr[i];
					arr[i] = arr[i+1];
					arr[i+1] = temp;
					exchFlag = true;
				}
				cdl.countDown();
			}
		}
		
		public static void main(String[] args) throws InterruptedException {
			int[] arr = new int[]{1,2,3,6,5,4,7,8,9,0};
			int start = 0;
			ExecutorService es = Executors.newCachedThreadPool();
			while(start == 1 || exchFlag) {
				exchFlag = false;
				CountDownLatch cdl = new CountDownLatch(arr.length/2 - 
						(arr.length%2 == 0? start : 0));
				for(int i = start; i < arr.length - 1; i += 2) {
					es.execute(new OddEvenSort(arr, i, cdl));
				}
				cdl.await();
				if (start == 0) {
					start = 1;
				}else {
					start = 0;
				}
			}
			System.out.println(Arrays.toString(arr));
		}
	}
```
输出结果 ：
``` java
[0, 1, 2, 3, 4, 5, 6, 7, 8, 9]
```
