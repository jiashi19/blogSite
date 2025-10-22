---
title: sbc loadbalancer 源码阅读
categories:
  - others
date: 2025-10-22 21:50:08
tags:
---

<!-- more -->
```java
final AtomicInteger position;//原子性
```

```java
int pos = this.position.incrementAndGet() & Integer.MAX_VALUE;
ServiceInstance instance = (ServiceInstance)instances.get(pos % instances.size());
```

**原子计数器**：`AtomicInteger position` 确保多线程环境下计数安全，**线程安全**，避免锁竞争；

**无状态**：不依赖外部存储。



分析：AtomicInteger为什么能保证线程安全？

1、Unsafe类；

`Unsafe`类是Java中的一个底层类，提供了硬件级别的原子操作，这个类通常不直接暴露给普通Java应用开发者使用，而是被内部类如`AtomicInteger`所使用，`Unsafe`类提供了如`compareAndSwapInt`等方法，这些方法可以原子地更新内存中的值。

 2、CAS（Compare-And-Swap）操作；

CAS操作是一个原子操作，它包括三个操作数——内存位置（V）、预期原值（A）和新值（B），如果内存位置V的值与预期原值A相匹配，那么处理器会自动将该位置的值更新为新值B，否则，处理器不做任何操作；

3、**volatile**内存可见性保障；

`AtomicInteger`中的值被声明为`volatile`，这意味着这个值的读取和写入操作会从主内存中直接进行，而不是从线程的本地缓存中进行，这确保了所有线程都能看到最新的值。





AI think：

原子操作的关键在于CAS（Compare-And-Swap）机制。CAS是一种硬件级别的指令，用于实现多线程下的原子操作。在Java中，AtomicInteger使用了Unsafe类提供的CAS操作来保证变量的原子性。 当多个线程同时调用incrementAndGet()时，每个线程都会尝试用CAS更新值。假设初始值是0，线程A和线程B同时读取到0，然后各自加1得到1。线程A先执行CAS，比较当前值是否为0，是的话就设置为1，成功。线程B再执行CAS时，发现当前值已经是1，不等于它之前读到的0，所以操作失败，会重新读取当前值（现在是1），再加1得到2，再次尝试CAS，直到成功。 这样，虽然多个线程可能竞争，但最终每个incrementAndGet()都会正确执行，不会出现数据不一致的问题。这就是为什么AtomicInteger能保证线程安全。 另外，Java的内存模型（JMM）中，volatile关键字保证了变量的可见性。AtomicInteger内部的value被声明为volatile，所以当一个线程修改了值，其他线程能立即看到最新的值。结合CAS的原子性操作，就实现了线程安全。 