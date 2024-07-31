---
title: JVM
categories:
  - java
date: 2024-07-31 17:48:01
tags:
---

<!-- more -->
# JVM
## JVM背景



- 编译器负责把Java源代码编译成**字节码**，JVM负责把字节码转换成机器码。转换的时候，可以做一些压缩或者优化（**JIT**），提高程序运行效率。
- JIT: just-in-time:即时编译器。
- 如项目遇到内存泄露、CPU飙升的问题，需要通过 JVM 的性能监控进行定位和解决。
- 目前最常见最广泛：HotSpot VM—— 热点代码探测 和 准确式内存管理。

## JVM结构
1. **类加载器**（Class Loader） **加载->连接->实例化**
2. **运行时数据区**(类比国库)       [结构](https://javabetter.cn/jvm/what-is-jvm.html#%E8%BF%90%E8%A1%8C%E6%97%B6%E6%95%B0%E6%8D%AE%E5%8C%BA)       
3. **执行引擎**
解释器， JIT， 垃圾回收器
![](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/overview/seven-06.png)

![](https://cdn.tobebetterjavaer.com/stutymore/what-is-jvm-20231030185742.png)


## java代码运行
编译期->运行时
编译期：编译成字节码文件
运行时：类加载器把bytecode加载到运行时数据区，然后执行引擎执行：
- 解释执行：逐条
- JIT即时编译，**运行时**将热点代码优化并缓存，下次直接用缓存起来的的**机器码**

## 运行时数据区
**运行时数据区哪些部分是线程私有，哪些是共享数据区**：
运行数据区有什么：
- 虚拟机内存：堆，栈（虚拟机栈，本地方法栈），PC(程序计数器)等
- 本地内存：元空间等

[方法区->(jdk1.8)元空间]

线程私有：程序计数器、虚拟机栈、本地方法栈；
共享：虚拟机内存中堆的字符串常量池，本地内存中的元空间

![](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2019/11/29/16eb7c5ca9064ab4~tplv-t2oaga2asx-jj-mark:3024:0:0:0:q75.png)

**堆区**
堆中存对象，栈中存基本数据类型和堆中对象的引用
**元空间区（方法区）**
存放已被加载的类信息、常量、静态变量、即编译器编译后的代码等。

## 垃圾收集机制
垃圾回收问题主要发生在 Java 堆上。主要内容：死亡的对象。
1. 引用计数法
在对象中添加一个引用计数器，对象每次被引用时，该计数器加一；当引用失效时，计数器的值减一；只要计数器的值为零，则代表对应的对象不可能再被使用。该方法的缺点在于无法避免相互引用的问题：



```java
objA.instance = objB
objB.instance = objA    
objA = null;
objB = null;
System.gc();
```
如上所示，此时两个对象已经不能再被访问，但其互相持有对对方的引用，如果采用引用计数法，则两个对象都无法被回收。
2. 可达性分析
主流的虚拟机采用的可达性分析.