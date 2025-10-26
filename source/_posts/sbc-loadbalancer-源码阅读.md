---
title: CAS原子操作与实际源码分析
categories:
  - others
date: 2025-6-22 21:50:08
tags:
---

## 1、CAS操作

**原子计数器**：`AtomicInteger position` 确保多线程环境下计数安全，**线程安全**，避免锁竞争；

<!-- more -->

```java
final AtomicInteger position;//原子性
```

```java
//例子：spring cloud lb部分源码--轮询策略
int pos = this.position.incrementAndGet() & Integer.MAX_VALUE;
ServiceInstance instance = (ServiceInstance)instances.get(pos % instances.size());
```

**无状态**：不依赖外部存储。



分析：AtomicInteger为什么能保证线程安全？

1、Unsafe类；

`Unsafe`类是Java中的一个底层类，提供了硬件级别的原子操作，这个类通常不直接暴露给普通Java应用开发者使用，而是被内部类如`AtomicInteger`所使用，`Unsafe`类提供了如`compareAndSwapInt`等方法，这些方法可以原子地更新内存中的值。

 2、CAS（Compare-And-Swap）操作；

CAS操作是一个原子操作，它包括三个操作数——内存位置（V）、预期原值（A）和新值（B），如果内存位置V的值与预期原值A相匹配，那么处理器会自动将该位置的值更新为新值B，否则，处理器不做任何操作；

3、**volatile**内存可见性保障；

`AtomicInteger`中的值被声明为`volatile`，这意味着这个值的读取和写入操作会从主内存中直接进行，而不是从线程的本地缓存中进行，这确保了所有线程都能看到最新的值。





from AI think：

原子操作的关键在于CAS（Compare-And-Swap）机制。CAS是一种硬件级别的指令，用于实现多线程下的原子操作。在Java中，AtomicInteger使用了Unsafe类提供的CAS操作来保证变量的原子性。 当多个线程同时调用incrementAndGet()时，每个线程都会尝试用CAS更新值。假设初始值是0，线程A和线程B同时读取到0，然后各自加1得到1。线程A先执行CAS，比较当前值是否为0，是的话就设置为1，成功。线程B再执行CAS时，发现当前值已经是1，不等于它之前读到的0，所以操作失败，会重新读取当前值（现在是1），再加1得到2，再次尝试CAS，直到成功。 这样，虽然多个线程可能竞争，但最终每个incrementAndGet()都会正确执行，不会出现数据不一致的问题。这就是为什么AtomicInteger能保证线程安全。 另外，Java的内存模型（JMM）中，volatile关键字保证了变量的可见性。AtomicInteger内部的value被声明为volatile，所以当一个线程修改了值，其他线程能立即看到最新的值。结合CAS的原子性操作，就实现了线程安全。 

## 2、CAS在高并发下的分析

```java
private final Random random = new Random();
```

这个random会有并发问题吗：

在Java中，使用 `private final Random random = new Random();` 的写法本身是线程安全的，因为 `Random` 类的实现通过CAS操作保证了内部状态的原子性。但在**高并发场景下**，这种方式**可能存在性能问题**，具体原因如下：

1. **线程竞争问题**： 
   当多个线程共享同一个 `Random` 实例时，它们会竞争内部的原子变量（如 `seed`），导致CAS操作频繁失败和重试，从而降低并发性能。尽管线程安全，但这种竞争会显著增加延迟，尤其在高并发场景中（如抢红包接口）可能成为性能瓶颈。

2. **替代方案推荐**：  
   - **`ThreadLocalRandom`**（JDK 1.7+）： 
     每个线程持有独立的随机数生成器，避免竞争。通过 `ThreadLocalRandom.current()` 获取实例，性能更高且无需手动管理线程隔离。  
   - **`Random` 实例的线程隔离**： 
     若需兼容旧版本，可通过 `ThreadLocal` 为每个线程分配独立的 `Random` 实例（需用 `static` 修饰防止内存泄漏）。  
   - **`Random.Shared`（JDK 17+）**： 
     直接通过 `Random.Shared` 获取线程安全的共享实例，底层优化了并发性能。

**总结**： 
若代码中 `Random` 实例会被多线程共享（如单例Bean或静态变量），即使线程安全，也建议替换为 `ThreadLocalRandom` 或线程隔离的 `Random`，以避免高并发下的性能问题。若为单线程或低并发场景，当前写法没有问题。

实际源码例子：

spring cloud 的随机轮询用的是ThreadLocalRandom

```java
//例子：spring cloud lb部分源码--随机策略
int index = ThreadLocalRandom.current().nextInt(instances.size());
ServiceInstance instance = (ServiceInstance)instances.get(index);
```

















