---
layout: post
title: java多线程之队列
tags: java 多线程 队列
categories: java
published: true
---

#目录
二、LinkedBlockingQueue分析</br>


#二、LinkedBlockingQueue分析</br>
1)简介</br>
LinkedBlockingQueue一般在多线程框架中用于缓冲任务，继承自AbstractQueue BlockingQueue和Serializable。里面有一个Node用于存储值和
下一个节点引用</br>
LinkedBlockingQueue里面有变量int类型capacity用于保存容量，AtomicInteger类型count保存元素个数（为什么要用AtomicInteger类型
而不用int类型？），Node类型的head和last，ReentrantLock类型的takeLock和putLock，Condition类型的notEmpty和notFull。这里用到的锁
有两个，分别是从队列里面取数据的时候和放数据到队列里面的时候。但是如果只有这两个锁，那么当队列里为空时，取元素的时候，直接返回
空，但是在正常情况下应该是等待队列为非空，可以用两个Condition变量来实现。</br>
2)源码分析</br>
先从构造函数开始分析，LinkedBlockingQueue有三个构造函数分别是：LinkedBlockingQueue() LinkedBlockingQueue(int capacity)和Linked
BlockingQueue(Collection<? extends E> c)</br>
~~~java
public LinkedBlockingQueue(int capacity) {
	if (capacity <= 0) throw new IllegalArgumentException();
	//初始化容量
    this.capacity = capacity;
	//设置dummy节点
    last = head = new Node<E>(null);
}

Collection里面的元素都会放在队列里面
public LinkedBlockingQueue(Collection<? extends E> c) {
	//设置默认的容量
	this(Integer.MAX_VALUE);
    final ReentrantLock putLock = this.putLock;
	//获取put锁
    putLock.lock();
    try {
    	int n = 0;
        for (E e : c) {
			//如果这里发生异常会怎么样？这里的变量count值还是为0，但是队列里面真正的元素个数不为0
        	if (e == null)
            	throw new NullPointerException();
            if (n == capacity)
                throw new IllegalStateException("Queue full");
            //入队列
			enqueue(new Node<E>(e));
            ++n;
         }
         count.set(n);
    } finally {
         putLock.unlock();
    }
}
~~~
放入元素到队列中时，放入的元素不能为null</br>
~~~java
public void put(E e) throws InterruptedException {
	if (e == null) throw new NullPointerException();
  
	int c = -1;
    Node<E> node = new Node<E>(e);
    final ReentrantLock putLock = this.putLock;
    final AtomicInteger count = this.count;
    putLock.lockInterruptibly();
    try {
	    //当容量满时，则不能往里面加入元素
   	    while (count.get() == capacity) {
    	   	notFull.await();
        }
  	    //加入node节点
        enqueue(node);
        c = count.getAndIncrement();
        //为什么要加入？ 
		if (c + 1 < capacity)
         	notFull.signal();
    } finally {
         putLock.unlock();
    }
    if (c == 0)
        signalNotEmpty();
}
~~~
元素从队列中取出来时</br>
~~~java
public E take() throws InterruptedException {
	E x;
    int c = -1;
    final AtomicInteger count = this.count;
    final ReentrantLock takeLock = this.takeLock;
    takeLock.lockInterruptibly();
    try {
		//如果队列为空，则阻塞
    	while (count.get() == 0) {
        	notEmpty.await();
        }
		//取出队列里面的元素
        x = dequeue();
        c = count.getAndDecrement();
        if (c > 1)
        	notEmpty.signal();
    } finally {
        takeLock.unlock();
    }
    if (c == capacity)
    	signalNotFull();
    return x;
}
~~~



