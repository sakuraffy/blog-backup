---
title: 分治算法
date: 2016-12-27 00:28:34
tags:	
	- 算法
---
我们先不说什么是分治算法，先来看个小问题，找出一个数组中的最小值和最大值
``` java
	package cn.sakuraffy.divide;

	public class SimpleMaxAndMin {
		public static void main(String[] args) {
			int[] arr = new int[]{1,4,6,3,2,9,8,5,7};
			int max = arr[0];
			int min = arr[0];
			
			for(int i = 1; i < arr.length; i++) {
				if(arr[i] > max) {
					max = arr[i];
				}
				if(arr[i] < min) {
					min = arr[i];
				}
			}
			System.out.println("min = " + min);
			System.out.println("max = " + max);
		}
	}
```

<!--more-->

输出结果 ：
``` java
min = 1
max = 9
```

我想绝大部分的人都可以写出以上代码，它也是对的。但在这里，我们提供一个更好的方法，那就是利用分治算法，先来展示一下用分治算法的解法，再来说分治算法
``` java
	package cn.sakuraffy.divide;

	public class DivMaxAndMin {
		public static class Num {
			private int max;
			private int min;
			//...这里省略getter,setter方法
		}

		public Num getNum(int[] arr, int start, int end) {
			Num num = new Num();
			if(start == end) {
				num.setMax(arr[start]);
				num.setMin(arr[start]);
			}else if(start == end-1) {
				if(arr[start] < arr[end]) {
					num.setMax(arr[end]);
					num.setMin(arr[start]);
				}else {
					num.setMin(arr[end]);
					num.setMax(arr[start]);
				}
			}else {
				int mid = (start + end) / 2;
				Num num1 = getNum(arr, start, mid);
				Num num2 = getNum(arr, mid+1, end);
				int max = num1.getMax() > num2.getMax() ? num1.getMax() : num2.getMax();
				int min = num1.getMin() < num2.getMin() ? num1.getMin() : num2.getMin();
				num.setMax(max);
				num.setMin(min);
			}
			return num;
		}
		
		public static void main(String[] args) {
			int[] arr = new int[]{1,4,6,3,2,9,8,5,7};
			DivMaxAndMin dmam = new DivMaxAndMin();
			Num num = dmam.getNum(arr,0,arr.length-1);
			System.out.println("min = " + num.getMin());
			System.out.println("max = " + num.getMax());
		}
		
	}
```
输出结果 ：
``` java
min = 1
max = 9
```

### 分治算法

天下分久必合，合久必分，以史为鉴，我们可以少走很多弯路，同样的分治算法也可帮我们有效的解决很多问题，下面就来揭示一下其庐山真面目

#### 基本思想

分治算法字面上的解释是“分而治之”，就是把一个复杂的问题分成两个或更多的相同或相似的可直接求解的子问题，原问题的解即子问题的解的合并。

#### 使用情况

分治法所能解决的问题一般具有以下几个特征：
- 该问题的规模缩小到一定的程度就可以容易地解决
- 该问题可以分解为若干个规模较小的相同问题，即该问题具有最优子结构性质
- 利用该问题分解出的子问题的解可以合并为该问题的解

#### 解决步骤

分治法的三个步骤是：
1. 分解（Divide）：将原问题分解为若干子问题，这些子问题都是原问题规模较小的实例。
2. 解决（Conquer）：递归地求解各子问题。如果子问题规模足够小，则直接求解。
3. 合并（Combine）：将所有子问题的解合并为原问题的解

### 归并排序

下面就看一下分治算法一个常用的实现-归并排序。同样的按照分治法的解决步骤一步一步来就行了
{% qnimg algorithm/divide/p1.png 'class:class1 class2' normal:yes %}

#### 代码实现
 
``` java
	package cn.sakuraffy.divide;

	public class MergeSort {
		public void sort(int arr[], int start, int end) {
			if(start < end) {
				int mid = (start + end) / 2;
				//分解
				sort(arr,start,mid);
				sort(arr,mid+1,end);
				//合并
				merge(arr,start,mid,end);
			}
		}
		
		public void merge(int arr[], int start, int mid, int end) {                           
			//接收参数，并定义两组起始点
			int start1 = start;
			int start2 = mid + 1;
			int end1 = mid;
			int end2 = end;
			int[] temp = new int[arr.length];
			int k = 0;
			//逐一比较 
			while(start1 <= end1 && start2 <= end2) {
				if(arr[start1] < arr[start2]) {
					temp[k++] = arr[start1++];
				}else {
					temp[k++] = arr[start2++];
				}
			}
			//将多的一组直接落下来 
			while(start1 <= end1) {
				temp[k++] = arr[start1++];
			}		
			while(start2 <= end2) {
				temp[k++] = arr[start2++];
			}
			//将当前排好序的数组合并到前面排好序的数组中
			for(int i = 0; i < k; i++) {
				arr[i + start] = temp[i];
			}
			
		}
		
		public static void main(String[] args) {
			int[] arr = new int[]{2,4,7,5,8,1,3,6};
			MergeSort ms = new MergeSort();
			ms.sort(arr, 0, arr.length-1);
			System.out.println(Arrays.toString(arr));
		}
	}
```
输出结果 ：
``` java
[1, 2, 3, 4, 5, 6, 7, 8]
```

