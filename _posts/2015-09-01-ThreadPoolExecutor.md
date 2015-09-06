---
layout: post
title: java多线程之ThreadPoolExecutor
tags: java 多线程 ThreadPoolExecutor
categories: java
published: true
---

#目录
一、简介</br>
二、源码分析</br>

#一、简介
>ThreadPoolExecutor是java多线框架的基础，可以定制各种各样的多线程。比较常用的有定制线程池可以用Executors，它可以创建的newFixedThreadPool、newSingleThreadExecutor、newCachedThreadPool、newScheduledThreadPool等。理解了ThreadPoolExecutor的原理，就理解了java的线程池。</br>

>ThreadPoolExecutor(int corePoolSize, int maximumPoolSize, long keepAliveTime, TimeUnit unit, BlockingQueue<Runnable> workQueue
, ThreadFactory threadFactory, RejectedExecutionHandler handler)corePoolSize线程池里面线程个数，如果线程池里面的线程个数少于
corePoolSize则新的任务来时就会创建新的线程，如果线程池里面的线程大于等于corePoolSize，当workQueue队列满时，如果线程池里面的
线程小于maximumPoolSize，那么就创建新的线程；keepAliveTime是当线程池里面的线程个数多于corePoolSize，并且线程池里面的线程有
处于idle状态，当线程处于idle状态时间超过keepAliveTime时，线程回收；workQueue是存放线程的队列，主要由这几种:SynchronousQueue--
没有数据缓冲生产者线程对其进行进行put操作必须要等到消费者线程进行消费，LinkedBlockingQueue--无线大的缓冲区， ArrayBlockingQueue
--有限大的缓冲区；handler有四种策略：AbortPolicy CallerRunsPolicy DiscardPolicy DiscardOldestPolicy

#二、源码分析</br>
1）状态</br>
ThreadPoolExecutor中定义了5个状态，分别是running--可以接收任务和处理队列中的任务；shutdown--不接受新的任务，但是要处理在
队列中的任务；stop--不接受新的任务，不处理队列中的任务，中断正在处理的任务；tidying--所有的任务都终止，workerCount为零；terminated--terminated()方法完成</br>
2）字段</br>
``
//ctl用于计算线程池里面线程的个数和辅助计算线程池的状态
private final AtomicInteger ctl = new AtomicInteger(ctlOf(RUNNING, 0));
private static final int COUNT_BITS = Integer.SIZE - 3;
//最大的线程个数
private static final int CAPACITY   = (1 << COUNT_BITS) - 1;
//RUNNING状态时最高三位全为1，即111000...000
private static final int RUNNING    = -1 << COUNT_BITS;
//SHUTDOWN状态000...000
private static final int SHUTDOWN   =  0 << COUNT_BITS;
//STOP状态00100...000
private static final int STOP       =  1 << COUNT_BITS;
//TIDYING状态为010000...000
private static final int TIDYING    =  2 << COUNT_BITS;
//TERMINATED状态为01100...000
private static final int TERMINATED =  3 << COUNT_BITS;
``
用CAPACITY高三位得到状态，地位计算线程池里面线程的个数，而线程池的RUNNING状态转化为数字后为负数比其它状态值都要小，比较容易
得到线程池的状态是否为运行状态

``
//得到运行的状态
private static int runStateOf(int c)     { return c & ~CAPACITY; }
//得到线程池里面线程的个数
private static int workerCountOf(int c)  { return c & CAPACITY; }
private static int ctlOf(int rs, int wc) { return rs | wc; }
private static boolean runStateLessThan(int c, int s) {
	return c < s;
}
private static boolean runStateAtLeast(int c, int s) {
    return c >= s;
}
private static boolean isRunning(int c) {
    return c < SHUTDOWN;
}
//利用CAS，只是增加expect的值，如果增加不成功，就放弃增加
private boolean compareAndIncrementWorkerCount(int expect) {
    return ctl.compareAndSet(expect, expect + 1);
}

private boolean compareAndDecrementWorkerCount(int expect) {
    return ctl.compareAndSet(expect, expect - 1);
}
//利用CAS进行减少线程的计数，此时一定要减少成功，故利用了do while循环
private void decrementWorkerCount() {
    do {} while (! compareAndDecrementWorkerCount(ctl.get()));
}
//阻塞队列
private final BlockingQueue<Runnable> workQueue;
private final ReentrantLock mainLock = new ReentrantLock();
//用HashSet存储工作线程
private final HashSet<Worker> workers = new HashSet<Worker>();
private final Condition termination = mainLock.newCondition();
private int largestPoolSize;
private long completedTaskCount;
private volatile ThreadFactory threadFactory;
private volatile RejectedExecutionHandler handler;
``
3)分析</br>
先从

