---
layout: post
title: java并发：ThreadLocal
tags: 多线程 ThreadLocal java
categories: Java
published: true
---




<div class="toc"></div>
#大纲
</br>
一、ThreadLocal初探</br>
二、ThreadLocal源码分析</br>
三、ThreadLocal总结</br>
<br/>
#ThreadLocal初探
<div class="toc"></div>
ThreadLocal也叫做本地对象，是为每一个访问它的线程分配一个对象，这样就为了防止线程之间对对象并发访问产生竞争
~~~java
public class Test {
	public static class ThreadLocalTest implements Runnable{
		ThreadLocal<Integer> threadLocal = new ThreadLocal<Integer>();
		
		@Override
		public void run(){
			Integer i = threadLocal.get();
			if(i == null){
				threadLocal.set((int)(Math.random()*100D));
			}
			System.out.println("the current thread name is:" + Thread.currentThread().getName() + " the threadLocal val is:" + threadLocal.get());
		}
		
	}
	public static void main(String[] args){
		ThreadLocalTest test = new ThreadLocalTest();
		Thread t1 = new Thread(test);
		Thread t2 = new Thread(test);
		t1.start();
		t2.start();
	}
}
~~~
上面得到的结果是两次访问的值都是不一样的

#源码分析
<div class="toc"></div>
ThreadLocal里面的get主要功能是得到存储在当前线程中ThreadLocalMap，以当前ThreadLocal为键的值
~~~java
public T get() {
	//取得当前线程
	Thread t = Thread.currentThread();
	//每个线程都有一个ThreadLocalMap数据结构，其中键为ThreadLocal变量，值为T类型
    ThreadLocalMap map = getMap(t);
    if (map != null) {
		//直接取出值
    	ThreadLocalMap.Entry e = map.getEntry(this);
        if (e != null) {
        	@SuppressWarnings("unchecked")
            T result = (T)e.value;
            return result;
        }
    }
    return setInitialValue();
}
~~~
setInitialValue主要功能是初始化当前线程里面的值
~~~java
private T setInitialValue() {
	//默认返回为null，可以覆盖这个方法
	T value = initialValue();
    Thread t = Thread.currentThread();
    ThreadLocalMap map = getMap(t);
    if (map != null)
    	map.set(this, value);
    else
		//如果当前线程中的threadLocals为空，则可以初始化当前线程的值
        createMap(t, value);
    return value;
}
~~~
set设置当前线程对应的值
~~~java
void set(T value) {
	Thread t = Thread.currentThread();
    ThreadLocalMap map = getMap(t);
    if (map != null)
    	map.set(this, value);
    else
         createMap(t, value);
}
~~~
ThreadLocalMap分析
~~~java
//创建对象
ThreadLocalMap(ThreadLocal<?> firstKey, Object firstValue){
	//初始化数组，这里使用的是和HashMap不同的Hash算法
	table = new Entry[INITIAL_CAPACITY];
	//这里的hashcode是由ThreadLocal产生的，是递增的，保证均匀分布到桶中间去
    int i = firstKey.threadLocalHashCode & (INITIAL_CAPACITY - 1);
    table[i] = new Entry(firstKey, firstValue);
    size = 1;
	//设置门限值为初始容量的2/3
    setThreshold(INITIAL_CAPACITY);
}

//获得当前线程的对象
private Entry getEntry(ThreadLocal<?> key) {
	int i = key.threadLocalHashCode & (table.length - 1);
    Entry e = table[i];
    if (e != null && e.get() == key)
    	return e;
    else
    	return getEntryAfterMiss(key, i, e);
}

//hash算法碰撞处理，线性探测
private Entry getEntryAfterMiss(ThreadLocal<?> key, int i, Entry e) {
	Entry[] tab = table;
    int len = tab.length;
    while (e != null) {
    	ThreadLocal<?> k = e.get();
        if (k == key)
        	return e;
        if (k == null)
			//弱引用的对象回收后
            expungeStaleEntry(i);
        else
            i = nextIndex(i, len);
        e = tab[i];
     }
     return null;
}
//去掉弱应用的对象
private int expungeStaleEntry(int staleSlot) {
	Entry[] tab = table;
    int len = tab.length;
	//设置value和table[staleSlot]为空，不然出现内存泄露
    tab[staleSlot].value = null;
    tab[staleSlot] = null;
    size--;

    // Rehash until we encounter null
    Entry e;
    int i;
    for (i = nextIndex(staleSlot, len);
    	(e = tab[i]) != null;
        	i = nextIndex(i, len)) {
    	ThreadLocal<?> k = e.get();
		//下一个对象中得弱引用为空，则继续回收Entry对象
        if (k == null) {
        	e.value = null;
            tab[i] = null;
            size--;
        } else {
            int h = k.threadLocalHashCode & (len - 1);
			// 重新hash
            if (h != i) {
           		 tab[i] = null;

                 // Unlike Knuth 6.4 Algorithm R, we must scan until
                 // null because multiple entries could have been stale.
                 while (tab[h] != null)
                 	h = nextIndex(h, len);
                 tab[h] = e;
            }
        }
   }
   return i;
}
~~~

#ThreadLocal图解
![ThreadLocal] [ThreadLocal]

[ThreadLocal]: {{"/ThreadLocalsummary.jpg" | prepend: site.imgrepo }}

