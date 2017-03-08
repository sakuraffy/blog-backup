---
title: 贪心算法
date: 2016-12-17 20:54:45
tags:
	- 算法
---
古人云：鱼与熊掌不可兼得，而现实中人们又往往想获取更多。通过程序设计和相应的算法，有时候我们又可以获取更多一点，而这就是贪心算法。其实贪心算法也并是特别复杂，在现实生活中，我们经常都能使用到，只是我们并没有意识到而已。

<!-- more -->

### 什么是贪心算法
贪心算法是指在求解实际问题中，每一步都采取在当前状态下最好或最优（即最有利）的选择，从而希望导致结果是最好或最优的算法。也就是说，它不从全局加以考虑，而仅仅采用局部最优解

贪心算法并没有固定的框架，其关键在于贪心策略到的选择，当然贪心算法也必须有一定的前提，那就是无后效性，即在当前做出的选择并不会因为后面的选择而改变，只与当前的状态有关

### 贪心算法基本思路
1. 建立数学模型来描述问题。
2. 把求解的问题分成若干个子问题。
3. 对每一子问题求解，得到子问题的局部最优解。
4. 把子问题的解局部最优解合成原来解问题的一个解。

### 案例分析
已知，中华人民共和国的纸币面额分别为：100元、50元、20元、10元、5元、2元、1元，输入钱数，输出最小的货币方案

### 代码实现
``` java
package cn.sakuraffy.greedy;

import java.util.Arrays;
import java.util.Scanner;

/**
 * 
 * @author Sakuraffy
 * @date 2016-12-25 19:11
 * @desc 利用贪心算法，解决找零问题
 */
public class Change {
	public final static int[] change = new int[]{100,50,20,10,5,1};                       
	
	public int[] getChange(int num) {
		int len = change.length;
		int[] count = new int[len];
		// 记录每次遍历的开始
		int tmp = 0;
		while(num > 0) {
			for(int i = tmp; i < len; i++) {
				if(num >= change[i]) {
					num -= change[i];
					count[i]++;
					tmp = i;
					break;
				}
			}
		}
		return count;
	}
	
	public static void main(String[] args) {
		Scanner sc = new Scanner(System.in);
		int num = sc.nextInt();
		sc.close();
		int[] count = new Change().getChange(num);
		System.out.println(Arrays.toString(count));
	}
}
```
输入数据：
``` java
345
```
输出结果：
``` java
[3, 0, 2, 0, 1, 0]
```

### 贪心算法适用场景
- 生产调度问题
- 0-1背包问题
- 次数有限问题
