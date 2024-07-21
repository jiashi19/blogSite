---
title: java多线程
categories:
  - others
date: 2024-07-21 20:49:06
tags:
password: 12345d
---

# 多线程-java实现

## 三种实现方法

<!-- more -->

### 1. 继承Thread类

重写run方法，并调用start方法启动线程。

```java
public class Mythread1 extends Thread{
    public Mythread1() {
    }
    @Override
    public void run(){
        for(int i=0;i<10;i++){
            System.out.println("hello world from "+getName());
        }
    }
}
```

```java
public class Demo1 {
    public static void main(String[] args) {
       Mythread1 thread1=new Mythread1();
       Mythread1 thread2=new Mythread1();
       thread1.start();
       thread2.start();
    }
}
```

### 2. 实现Runnable接口

重写run方法；

定义自己的类，并创建对象，其表示多线程要执行的任务；

创建线程对象。

```java
public class MyRun implements Runnable{
    @Override
    public void run() {
        for(int i=0;i<10;i++){
            System.out.println("hello world");
        }
    }
}
```

```java
public class Demo2 {
    public static void main(String[] args){
       	MyRun mr=new MyRun();
		Thread t1=new Thread(mr);
        t1.start();
    }
}
```

### 3. 实现Callable接口

特点：可获取多线程执行结果；

实现callable接口，重写call；

创建该类的对象，并创建FutureTask（管理结果）和Thread的对象。

```java
public class MyCall implements Callable<Integer> {

    @Override
    public Integer call() throws Exception {
        return 100;
    }
}
```

```java
public class Demo3 {
    public static void main(String[] args) throws ExecutionException, InterruptedException {

        MyCall mc=new MyCall();
        FutureTask<Integer> ft=new FutureTask<>(mc);
        Thread t1=new Thread(ft);
        t1.start();
        System.out.println(ft.get());
    }
}

```

## 多线程中常见成员方法

![image-20240414153243437](D:/myData/javaStudy/img/image-20240414153243437.png)



### 守护线程

JAVA语言中无论是线程还是线程池，默认都是用户线程，因此用户线程也被称为普通线程。守护线程也被称之为后台线程、服务线程或精灵线程，**守护线程是为用户线程服务**的，当线程中的用户线程都执行结束后，守护线程也会跟随结束。**The Java Virtual Machine exits when the only threads running are all daemon threads.**

场景：聊天框以及其传输文件的线程（后者为前者的守护进程）

## 线程安全问题

### 同步代码块

同步代码块将`一段代码`用一把`锁`给锁起来, 只有获得了这把锁的线程才访问, 并且同一时刻, 只有一个线程能持有这把锁, 这样就保证了同一时刻只有一个线程能执行被锁住的代码.

```java
synchronized(this) {
    //code
}
```

而`synchronized` 也可以用来修饰一个方法。

```java
public class Foo {
    public synchronized void doSomething() {
        // code
    }
}
```

**锁**：拿什么当作锁？

在java中可以拿一个对象作为锁。



StringBuilder 和StringBuffer 的区别：StringBuffer 是线程安全的，而前者不是。

**参考链接**：[java - 线程间的同步与通信(1)——同步代码块Synchronized - Keep Coding - SegmentFault 思否](https://segmentfault.com/a/1190000015979202)

## 多线程-线程池

参考链接：[Java 多线程：彻底搞懂线程池_java线程池-CSDN博客](https://blog.csdn.net/u013541140/article/details/95225769)



Executors 中封装好了四种常见的功能线程池： 

- FixedThreadPool 定长线程池
- ScheduledThreadPool  定时线程池
- CachedThreadPool 可缓存线程池
- SingleThreadExecutor 单线程化线程池



这4 个功能线程池虽然方便，但现在已经不建议使用了。

java中的线程池真正的实现类是**ThreadPoolExecutor**。

- corePoolSize（必需）：核心线程数。默认情况下，核心线程会一直存活，但是当将 allowCoreThreadTimeout 设置为 true 时，核心线程也会超时回收。
- maximumPoolSize（必需）：线程池所能容纳的最大线程数。当活跃线程数达到该数值后，后续的新任务将会阻塞。
- keepAliveTime（必需）：线程闲置超时时长。如果超过该时长，**非核心线程**就会被回收。如果将 allowCoreThreadTimeout 设置为 true 时，核心线程也会超时回收。
- unit（必需）：指定 keepAliveTime 参数的时间**单位**。常用的有：TimeUnit.MILLISECONDS（毫秒）、TimeUnit.SECONDS（秒）、TimeUnit.MINUTES（分）。
- workQueue（必需）：任务队列。通过线程池的 execute() 方法提交的 Runnable 对象将存储在该参数中。其采用阻塞队列实现。
- threadFactory（可选）：线程工厂。用于指定为线程池创建新线程的方式。
- handler（可选）：拒绝策略。当达到最大线程数时需要执行的饱和策略。

**任务超过核心线程数和队列长度，才会创建临时线程。**

