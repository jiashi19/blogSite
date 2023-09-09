---
title: wireless_security
date: 2023-09-09 19:54:16
tags:
---

# 无线安全

## 相关资源

基础知识和一些攻击::

https://www.scucyber.cn/wireless_sec/

[伪AP检测技术研究 - 掘金 (juejin.cn)](https://juejin.cn/post/6844903572836974599)

https://zsecurity.org/how-to-start-a-fake-access-point-fake-wifi/

一些工具：

- wifiPhisher--设立一个钓鱼AP

  [wifiphisher实现无线渗透WiFi钓鱼 - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/149945656)

- airocrack-ng--实施一些ap攻击

- [(134条消息) 使用kali破解WIFI——Aircrack-ng_kali aircrack-ng_我重来不说话的博客-CSDN博客](https://blog.csdn.net/qq_19623861/article/details/117690103)

- airSnarf

- airbase-ng

- mdk3,mdk4

- hostapd

- fluxion--设立一个钓鱼AP

  [Wifi钓鱼工具——fluxion - freeliver - 博客园 (cnblogs.com)](https://www.cnblogs.com/cmt110/p/15125418.html#:~:text=1. 选择一个攻击方式 专属门户 创建一个 "邪恶的双胞胎",接入点。 2. 选择要扫描的信道，默认选 "1"（若模拟攻击网卡支持5GHz，可选择 "3"）；出现目标AP后CTRL%2BC，这里选择29，选择后即进入攻击配置环节。)

  

### kali设定成桥接模式

[(134条消息) 【虚拟机】解决Kali虚拟机不能联网问题（桥接模式WIFI和本地网线）_江湖one Cat的博客-CSDN博客](https://blog.csdn.net/qq_43633973/article/details/100732758#:~:text=如下图步骤： 1 1..打开虚拟机编辑->虚拟网络编辑器如图： 2 2.选择对应的网卡 参考：控制面板→网络和Internet→查看网络状态和任务→更改适配器设置如图： 3 3.可以右击WLAN选择状态→属性→IPV4设置自动获取网络,-a 我自己的一开始eth0是没有IP地址的，如图： 2.KALI基础配置： 在终端输入： 1）gedit %2Fetc%2Fnetwork%2Finterfaces 这里采用静态配置的方法：（在下面我写了可以设置另一种动态获取DHCP的方法，如果网络稳定，不经常更换网络环境，这里可以选择静态配置） )

## 基本结构

AP：无线接入点

STA站点：每一个连接到无线网络中的终端

SSID：服务集标识符，一个或一组基础架构模式无线网络的标识

- 基本服务集标识符（BSSID），表示的是[AP](https://zh.wikipedia.org/wiki/無線接取器)的[数据链路层](https://zh.wikipedia.org/wiki/数据链路层)的[MAC地址](https://zh.wikipedia.org/wiki/MAC地址)
- 扩展服务集定标识符（ESSID），一个最长32字节区分大小写的字符串，表示无线网络的名称

（bssid就是具体的某个连锁店编号（001）或地址，ssid就是连锁店的名字或者照片，essid就是连锁店的总公司或者招牌or品牌。一般ssid和essid都是相同的）

[(134条消息) 无线网安全威胁：伪AP攻击原理与检测方法综述【转载】_Winds_Up的博客-CSDN博客](https://blog.csdn.net/Winds_Up/article/details/118296162)



## 2.4G/5G标准

2.4G频段共划分为14个信道（能用的仅13个），5G频段共划分为5个信道

## 扫描无线网络

- 主动扫描：客户端主动发送**探测请求（Probe Request）帧**（使用NULL或设置的SSID名称），周围的AP收到该请求后，将会响应该探索信号帧，即使不发送信号的AP也会响应该请求
- 被动扫描：AP每隔一段时间自动发送**信号（Beacon）帧**，STA通过侦听AP定期发送的Beacon帧来发现网络

###  Airodump-ng使用

- BSSID: AP的Mac地址
- PWR：**信号强度**，数字越小，信号越强
- CH：路由器的所在**频道**
- MB：无线所支持的最大速率
- ENC：使用的加密算法体系
- CIPHER：检测到的加密算法
- AUTH：使用的认证协议
- ESSID：路由器的名称

