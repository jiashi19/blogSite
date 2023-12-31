---
title: burpsuite学习
date: 2023-09-09 19:56:56
tags:
categories:
- [渗透]
---

# BurpSuite

## Intruder模块

### 模式介绍

Intruder共有4种模式，如下图：

![image-20230811154651214](https://s2.loli.net/2023/09/10/J8FYCvIRSrzEGc7.png)

1. Sniper:一个一个的替换；（单个list）
2. battering ram：一对一对的替换；（单个list）
3. pitchfork：**一一对应**；（**多个list**）
4. cluster bomb：尝试到每个排列组合；（**多个list**）

以crapi中的challenge3为例，在intruder中设置两个变量

![image-20230811163011823](https://s2.loli.net/2023/09/10/fu2JhWnRKU9vAdw.png)

- #### **Sniper**:

  选择"payload type":Simple list；创建一个简单的payload list

  <img src="https://s2.loli.net/2023/09/10/axLUJ5FTe9PqzAR.png" alt="image-20230811163348523" style="zoom:60%;" />

  暂且称第一个变量为1，第二个为2，则替换过程为：先将1用 **list** 中的值逐次替换，2不变；再将2逐次替换，1不变

  <img src="https://s2.loli.net/2023/09/10/1wvP4L35YixpCgD.png" alt="image-20230811163526674" style="zoom:50%;" />

  RequestCount= payloadCount*变量个数

- #### **Battering ram:**

  将所有变量同时用 **list** 中的一个值替换

  <img src="https://s2.loli.net/2023/09/10/45mg3MYHk7xNyec.png" alt="image-20230811164142769" style="zoom: 65%;" />

  RequestCount= payloadCount

- #### **pitchfork:**

  由于有两个变量，所以需要分别将两个变量的list（payload set）设置好，最上方的数字表示的是set，为1则表示当时在设置第一个set，为2则表示在设置第二个set

  <img src="https://s2.loli.net/2023/09/10/4TZ8R7UiFLtjnO2.png" alt="image-20230811164936140" style="zoom:67%;" />

  <img src="https://s2.loli.net/2023/09/10/YoTOLp4MSuAUjRr.png" alt="image-20230811165016766" style="zoom:67%;" />

  由上可以看到，本人将两个set分别设置为小写和大写的26个字母。

  而启动attack则可以看到，每个元素是**一一对应**的

  ![image-20230811165126891](https://s2.loli.net/2023/09/10/1zcwWti7DJGHX2E.png)

  RequestCount= payloadCount（单个set/list）

- #### **Cluster bomb:**

  类似于上一个，但是会尝试到每一个组合

  ![image-20230811165343094](https://s2.loli.net/2023/09/10/57kiSCZfuIsjEp8.png)

  RequestCount= payloadCountOf_1*payloadCountOf_2(......)

## 定长数字爆破

首先将type设置成Numbers

<img src="https://s2.loli.net/2023/09/10/sCUngh7l8pouZGV.png" alt="image-20230811165750670" style="zoom:70%;" />

比如这里的需求是需要实现0001到9999的爆破，那么就要如下图来设置，保证生成的数字如0001 为“0001”而不是“1“

<img src="https://s2.loli.net/2023/09/10/lNTP4btSQvzBVi8.png" alt="image-20230811165944026" style="zoom:67%;" />



## 相关参考链接

https://blog.csdn.net/zbj18314469395/article/details/115429675

https://www.cnblogs.com/tysec/p/15781784.html