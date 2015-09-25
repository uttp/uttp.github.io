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
###1.2 操作分析
~~~java
public HashMap(int initialCapacity, float loadFactor) {
	if (initialCapacity < 0)
    	throw new IllegalArgumentException("Illegal initial capacity: " +
        	initialCapacity);
    if (initialCapacity > MAXIMUM_CAPACITY)
        initialCapacity = MAXIMUM_CAPACITY;
    if (loadFactor <= 0 || Float.isNaN(loadFactor))
        throw new IllegalArgumentException("Illegal load factor: " +
        	loadFactor);
    this.loadFactor = loadFactor;
    threshold = initialCapacity;
    init();
}
~~~ 
上面代码只是简单的对locadFactor和threshold进行初始化</br>
~~~java
public V put(K key, V value) {
	if (table == EMPTY_TABLE) {
    	inflateTable(threshold);
    }
    if (key == null)
    	return putForNullKey(value);
    int hash = hash(key);
   	int i = indexFor(hash, table.length);
    for (Entry<K,V> e = table[i]; e != null; e = e.next) {
    	Object k;
        if (e.hash == hash && ((k = e.key) == key || key.equals(k))) {
        	V oldValue = e.value;
            e.value = value;
            e.recordAccess(this);
            return oldValue;
        }
    }
    modCount++;
    addEntry(hash, key, value, i);
    return null;
}
~~~
如果table为空,则用inflateTable进行初始化</br>
HashMap对应的键和值都可以为空，当键值为空时用，putForNullKey进行初始化</br>
计算key的hash值找到对应的桶，判断是否可以存在对应相等的key值</br>
如果没有找到key，则把对应的key和value值加入到对应的桶中去</br>
~~~java
private void inflateTable(int toSize) {
	int capacity = roundUpToPowerOf2(toSize);
	threshold = (int) Math.min(capacity * loadFactor, MAXIMUM_CAPACITY + 1);
    table = new Entry[capacity];
    initHashSeedAsNeeded(capacity);
}
~~~
对桶进行初始化</br>
capacity设置为2的指数次方</br>
threshold设置为capacity和loadFactor的乘积</br>
初始化桶数组</br>
~~~java
private V putForNullKey(V value) {
	for (Entry<K,V> e = table[0]; e != null; e = e.next) {
    	if (e.key == null) {
        	V oldValue = e.value;
            e.value = value;
            e.recordAccess(this);
            return oldValue;
        }
    }
    modCount++;
    addEntry(0, null, value, 0);
    return null;
}
~~~
key为null的时候，直接对第一个桶进行操作
~~~java
void addEntry(int hash, K key, V value, int bucketIndex) {
	if ((size >= threshold) && (null != table[bucketIndex])) {
    	resize(2 * table.length);
        hash = (null != key) ? hash(key) : 0;
        bucketIndex = indexFor(hash, table.length);
    }
    createEntry(hash, key, value, bucketIndex);
}
~~~
在插入元素的时候，需要对size和threshold的大小进行比较，如果超过了threshold的值，那就应该进行重新hash后再插入对应的桶中</br>
~~~java
void resize(int newCapacity) {
	Entry[] oldTable = table;
    int oldCapacity = oldTable.length;
    if (oldCapacity == MAXIMUM_CAPACITY) {
    	threshold = Integer.MAX_VALUE;
        return;
    }
    Entry[] newTable = new Entry[newCapacity];
    transfer(newTable, initHashSeedAsNeeded(newCapacity));
    table = newTable;
    threshold = (int)Math.min(newCapacity * loadFactor, MAXIMUM_CAPACITY + 1);
}
void transfer(Entry[] newTable, boolean rehash) {
    int newCapacity = newTable.length;
    for (Entry<K,V> e : table) {
    	while(null != e) {
        	Entry<K,V> next = e.next;
            if (rehash) {
            	e.hash = null == e.key ? 0 : hash(e.key);
            }
            int i = indexFor(e.hash, newCapacity);
            e.next = newTable[i];
            newTable[i] = e;
            e = next;
        }
    }
}
~~~
进行桶的容量增倍的时候，需要对每个元素的hash值重新和桶的大小进行与操作</br></br>
下面对移除元素进行的操作分析</br>
~~~java
final Entry<K,V> removeEntryForKey(Object key) {
	if (size == 0) {
		return null;
    }
    int hash = (key == null) ? 0 : hash(key);
   	int i = indexFor(hash, table.length);
    Entry<K,V> prev = table[i];
    Entry<K,V> e = prev;
    while (e != null) {
    	Entry<K,V> next = e.next;
       	Object k;
        if (e.hash == hash &&
       		((k = e.key) == key || (key != null && key.equals(k)))) {
        	modCount++;
            size--;
            if (prev == e)
            	table[i] = next;
            else
                prev.next = next;
            e.recordRemoval(this);
            return e;
        }
        prev = e;
        e = next;
   }
   return e;
}
~~~
由源码可得每次进行移除元素操作的时候，没有再对数据结构进行修改（指的是对capacity的大小进行增倍对每个元素进行重新hash）






