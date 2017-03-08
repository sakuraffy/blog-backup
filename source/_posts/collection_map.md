---
title: 集合系列08-Map和AbstractMap的原理与使用
date: 2016-10-19 13:49:06
tags:
	- Collection
---
### Map

我们前面谈到了Collection，其实集合中还有另一种键值对的存在，那就是Map
``` java
	public interface Map<K,V> {
		// ...
		interface Entry<K,V> {
			// ...
		}
		// ...
	}
```
这就是Map接口的结构，至于其抽象方法如下图

<!--more-->

{% qnimg collection/map/p1.png 'class:class1 class2' normal:yes %}

下面就看看jdk8出现的常用默认方法

``` java
	//若键不存在或所对应的值为null，则返回默认值
	default V getOrDefault(Object key, V defaultValue) {
        V v;
        return (((v = get(key)) != null) || containsKey(key))                                 
            ? v
            : defaultValue;
    }
	// 若键所对应的值为null，则将值添加进去
	default V putIfAbsent(K key, V value) {
        V v = get(key);
        if (v == null) {
            v = put(key, value);
        }

        return v;
    }
	//若键所对应的值和给定的值相同则删除
	default boolean remove(Object key, Object value) {
        Object curValue = get(key);
        if (!Objects.equals(curValue, value) ||
            (curValue == null && !containsKey(key))) {
            return false;
        }
        remove(key);
        return true;
    }
	//若键存在且对应的值与oldValue相同，则用新值替代
	default boolean replace(K key, V oldValue, V newValue) {
        Object curValue = get(key);
        if (!Objects.equals(curValue, oldValue) ||
            (curValue == null && !containsKey(key))) {
            return false;
        }
        put(key, newValue);
        return true;
    }
	//若键存在，则用给定值替代原值
	default V replace(K key, V value) {
        V curValue;
        if (((curValue = get(key)) != null) || containsKey(key)) {
            curValue = put(key, value);
        }
        return curValue;
    }
```

比较特别的是Map接口中还有一个接口Entry(包含key和value)，下图是Entry的抽象方法
{% qnimg collection/map/p2.png 'class:class1 class2' normal:yes %}

### AbstractMap

上面提到了Map和Entry接口，下面说说Map的一个抽象实现AbstractMap
``` java
	public abstract class AbstractMap<K,V> implements Map<K,V> {
		transient volatile Set<K>        keySet;
		transient volatile Collection<V> values;
	}
```

通过查看源码发现AbstractMap中只有一个entrySet()抽象方法和一个抛出UnsupportedOperationException的put(),其余的都通过entrySet()方法实现了，可见entrySet()多么重要，entrySet()返回的就是一个包含每一个键值对组合实体的集合