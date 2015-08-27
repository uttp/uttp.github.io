---
layout: post
title: java多线程之ThreadLocal
tags: 多线程 ThreadLocal
categories: Java
---

<div class="toc"></div>

#例子
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




