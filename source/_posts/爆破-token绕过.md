---
title: 爆破-token绕过
date: 2023-10-03 01:47:54
tags:
categories:
- 渗透
---
# token绕过

## 前言

- 知识点：

  token 验证：客户端每次请求服务器时，服务器会返回一个 token，下次请求时，客户端需要带上这个 token，否则服务器认为请求不合法。且这个 token 不可重复使用，每次请求都将生成新的 token，并导致旧的 token 失效。
<!-- more -->
- 实例：

  pikachu渗透测试平台--暴力破解--token防爆破

## 过程

### 开始

使用burp抓包，可以注意到post请求携带参数token。repeater重复发送第二次就会发现出现“csrf token error”的提示符。

![image-20231003005919263](https://s2.loli.net/2023/10/03/goLb2NI5PfKUZCD.png)

每次请求，token都会改变，而这个下一次请求的token值从每次的response中可以得到。

具体：在response中搜索token找到有一个type为“hidden“的input标签，可以发现其value正是下一次的token值。

<img src="https://s2.loli.net/2023/10/03/YOcp3Tw5ksz2NRS.png" alt="image-20231003010510211" style="zoom:60%;" />

### 具体实施- with Intruder

利用burp抓包转入intruder模块，设置两个变量：password和token。选择**pitchfork**模式（为什么不设置username也为变量：这里已经默认自己知道username为admin了。然后选取pitchfork是因为token每个只能使用一次，其实是有不妥：如果在不知道username的情况下，设置三个变量，username和password的list就要额外设计，因为使用不重复的list就会错过很多username和password的组合，而这都是因为**一一对应**）

![image-20231003011542400](https://s2.loli.net/2023/10/03/oP4JavFzq3NwxcY.png)

然后设置payload。重点在于设置token变量的payload时需要选择为**recursive grep**

![image-20231003012417416](https://s2.loli.net/2023/10/03/PfAgn5hc78pt2xQ.png)

然后在intruder的setting部分找到grep-extract部分

<img src="https://s2.loli.net/2023/10/03/wC1oJyc5SMjgEAL.png" alt="image-20231003012916995" style="zoom: 33%;" />

点击add：

<img src="https://s2.loli.net/2023/10/03/zRYIEVKTcWXbfC4.png" alt="image-20231003013125589" style="zoom: 53%;" />

下框中若无内容则点击fetch response/refetch response

<img src="https://s2.loli.net/2023/10/03/Fw7OLCruMJWxsgd.png" alt="image-20231003013146189" style="zoom: 40%;" />

之后利用search框找到token，则会自动生成正则，然后选择ok填入。

<img src="https://s2.loli.net/2023/10/03/xVR9ELwoI4gycF7.png" alt="image-20231003013417675" style="zoom:40%;" />

之后还得设置线程数量为1

<img src="https://s2.loli.net/2023/10/03/sbfiAc3xZt7DlH4.png" alt="image-20231003013601503" style="zoom: 50%;" />



一切就绪之后开始爆破，可以看到payload2确实加载成功了，而其中的正确的response也显示login success

![](https://s2.loli.net/2023/10/03/xm9VFlWYhkRo6sI.png)