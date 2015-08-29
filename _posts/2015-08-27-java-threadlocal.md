---
layout: post
title: java多线程之ThreadLocal
tags: 多线程 ThreadLocal
categories: Java
published: false
---




<div class="toc"></div>

#例子
<div class="toc"></div>
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
从上面例子可以体会到ThreadLocal并不神秘，它的核心思想就是为每个线程保存一个本地变量

#源码分析
<div class="toc"></div>
get函数
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
setInitialValue函数
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
set函数,比较简单没
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
ThreadLocal总结图片如下
![ThreadLocalsummary.jpg]({{site.baseurl}}/_posts/ThreadLocalsummary.jpg)
