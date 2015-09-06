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
>ThreadPoolExecutor是java多线框架的基础，可以定制各种各样的多线程。比较常用的有Executors里面可以创建的newFixedThreadPool、newSingleThreadExecutor、newCachedThreadPool、newScheduledThreadPool等。理解了ThreadPoolExecutor的原理，就理解了java的线程池。</br>

>ThreadPoolExecutor(int corePoolSize, int maximumPoolSize, long keepAliveTime, TimeUnit unit, BlockingQueue<Runnable> workQueue
, ThreadFactory threadFactory, RejectedExecutionHandler handler)corePoolSize线程池里面线程个数，如果线程池里面的线程个数少于
corePoolSize则新的任务来时就会创建新的线程，如果线程池里面的线程大于等于corePoolSize，当workQueue队列满时，如果线程池里面的
线程小于maximumPoolSize，那么就创建新的线程；keepAliveTime是当线程池里面的线程个数多于corePoolSize，并且线程池里面的线程有
处于idle状态，当线程处于idle状态时间超过keepAliveTime时，线程回收；workQueue是存放线程的队列，主要由这几种:SynchronousQueue--
没有数据缓冲生产者线程对其进行进行put操作必须要等到消费者线程进行消费，LinkedBlockingQueue--无线大的缓冲区， ArrayBlockingQueue
--有限大的缓冲区；handler有四种策略：AbortPolicy CallerRunsPolicy DiscardPolicy DiscardOldestPolicy

#二、源码分析</br>




