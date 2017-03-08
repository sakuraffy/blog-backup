---
title: Date、DateFormat与Calendar的使用
date: 2016-09-12 23:15:16
tags:
	- Java
---
前面我们提到了字符串String，StringBuilder和StringBuffer的使用，平常我们还会经常与另外一类数据打交道，那就是时间，在Java中，与时间处理相关的类一般有三个Date、DateFormat和Calendar

### Date

Date主要与一个long类型的数据打交道。在Java中，这个数是此时距1970-1-1 08:00:00的毫秒数。但还值得注意的一点就是java.util和java.sql两个包里面都有Date这个类

<!--more-->

``` java
	public class Date implements java.io.Serializable, Cloneable, Comparable<Date> {
		public Date() {
			this(System.currentTimeMillis());
		}
		public String toString() {
			//Tue Aug 30 23:24:42 CST 2016 以这种格式输出
		}
	}
	
	public class Date extends java.util.Date {
		//java.sql.Date没有空的构造方法
		public Date(long date) {
			// If the millisecond date value contains time info, mask it out.
			super(date);
		}
		public String toString() {
			//2016-08-30 以这种格式输出
		}
	}
```

也就是说java.sql.Date不能精确到分和秒

### DateFormat

DateFormat主要就是在字符串和Date之间打交道，可以将Date对象转化为指定格式的字符串，同样的，也可以将指定格式的字符串转化为Date对象
``` java
	public abstract class DateFormat extends Format {
		public abstract StringBuffer format(Date date, StringBuffer toAppendTo,
                                        FieldPosition fieldPosition);
		public abstract Date parse(String source, ParsePosition pos);
		public final String format(Date date){
			return format(date, new StringBuffer(),
						  DontCareFieldPosition.INSTANCE).toString();
		}
	}
	
	public class SimpleDateFormat extends DateFormat {}
```

### Calendar

Calendar日历的意思，顾名思义就是和日历相关东西，这里值得注意的Calendar类中对于每一个都是用常量进行表示，也可以用数字，但是就月份而言0代表一月份...... 11代表十二月，就星期来说1代表周日...... 7代表周六
``` java
	public abstract class Calendar implements Serializable, Cloneable, Comparable<Calendar> {
		protected abstract void computeTime();
		protected abstract void computeFields();
	}
	
	public class GregorianCalendar extends Calendar {}
```
下面展示一个关于日历的小示例 ： 
``` java
	public class CalendarDemo {
		public static void main(String[] args) {
			Scanner sc = new Scanner(System.in);
			System.out.println("请输入日期（格式为1111-11-11）");
			String temp = sc.nextLine();
			sc.close();
			
			DateFormat df = new SimpleDateFormat("yyyy-MM-dd");
			try {
				Date date = df.parse(temp);
				Calendar c = new GregorianCalendar();
				c.setTime(date);
				
				//获得第几日，为了将其标注出来
				int day = c.get(Calendar.DATE);
				
				//从1号开始打印
				c.set(Calendar.DATE, 1);
				
				//获得1号为星期几
				int weekDay = c.get(Calendar.DAY_OF_WEEK);

				//获得该月的最大天数
				int maxDay = c.getActualMaximum(Calendar.DATE);
				
				System.out.println("日\t一\t二\t三\t四\t五\t六");			
				for(int i = 1; i < weekDay; i++){
					System.out.print("\t");
				}
				for(int i = 1; i <= maxDay; i++){
					if(c.get(Calendar.DATE) == day){
						System.out.print("&");
					}
					System.out.print(i + "\t");
					if(c.get(Calendar.DAY_OF_WEEK) == Calendar.SATURDAY){         
						System.out.println();
					}
					c.set(Calendar.DATE, i+1);
				}
			} catch (ParseException e) {
				e.printStackTrace();
			}
			
		}
	}
```
输出结果：
{% qnimg java/date/p1.png 'class:class1 class2' normal:yes %}