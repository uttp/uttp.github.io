---
layout: post
title: java之HashMap、HashTable、ConcurrentHashMap分析
tags: java HashMap HashTable ConcurrentHashMap 
categories: java
published: true
---

#目录
一 简介</br>
二 源码分析

#一 简介</br>
> 使用键值对的存储，区别在于HashMap为非线程安全的，HashTable和ConcurrentHashMap为线程安全的，ConcurrentHashMap使用了锁分离技术，它的效率比HashTable更高</br>

#二 源码分分析</br>
>##1 HashMap分析</br>
###1.1 数据结构</br>
HashMap里面的每个元素的键值对都保存在Entry内部类里面，现在对HashMap进行分析</br>
里面有四个变量
~~~java
final K key;
V value;
Entry<K,V> next;
int hash;
~~~
key值是固定不变的，故声明为final类型。next指定下一个相同hash值的位置，hash为键哈希计算出来的值</br>
equals方法的实现</br>
~~~java
public final boolean equals(Object o) {
	if (!(o instanceof Map.Entry))
    	return false;
    Map.Entry e = (Map.Entry)o;
    Object k1 = getKey();
    Object k2 = e.getKey();
    if (k1 == k2 || (k1 != null && k1.equals(k2))) {
    	Object v1 = getValue();
        Object v2 = e.getValue();
        if (v1 == v2 || (v1 != null && v1.equals(v2)))
        	return true;
    }
    return false;
}
~~~
通过源码分析，可以得知，判断两个Entry是否相等是要判断对应的key和value都相等或者equals为true</br>
~~~java
static final int DEFAULT_INITIAL_CAPACITY = 1 << 4;
static final int MAXIMUM_CAPACITY = 1 << 30;
static final float DEFAULT_LOAD_FACTOR = 0.75f;
static final Entry<?,?>[] EMPTY_TABLE = {};
transient Entry<K,V>[] table = (Entry<K,V>[]) EMPTY_TABLE;
transient int size;
int threshold;
final float loadFactor;
transient int modCount;
static final int ALTERNATIVE_HASHING_THRESHOLD_DEFAULT = Integer.MAX_VALUE;
~~~
DEFAULT_INITIAL_CAPACITY为HashMap的默认桶的个数，这里默认的值为16</br>
MAXIMUM_CAPACITY为HashMap的桶的最大的个数，这里定义为2^30</br>
DEFAULT_LOAD_FACTORY的作用是当元素的个数大于桶的个数*load_factory时HashMap就得重新进行hash</br>
table是用一个数组来保存桶中的元素，每个桶是由一个链表组成</br>
modCount记录的是桶更改的次数，这样在遍历时如果对HashMap进行了修改那么就会报ConcurrentModificationException异常</br>
##1.2 











