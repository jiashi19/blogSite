---
title: dns欺骗--kali实现
date: 2023-09-09 19:16:01
tags:
---

# DNS 欺骗

## 基础知识

环境：

攻击实施：kali linux on vmware 

靶机：windows 7 on vmware 

工具:

- arpspoof
- ettercap

相关原理：

在一个局域网内，攻击者首先使用ARP欺骗，使目标主机的所有网络流量都通过攻击者的主机。之
后攻击者通过嗅探目标主机发出的DNS请求分组，分析数据分组的ID和端口号后，向目标主机发
送攻击者构造好的DNS 返回分组，目标主机收到 DNS 应答后，发现ID和端口号全部正确，即
把返回的数据分组中的域名和对应的IP地址保存进DNS缓存，而后到达的真实DNS应答分组则被
丢弃。

## 攻击实施--使用ettercap

### 1.基本用法（ARP欺骗）

**修改etter.dns**：

```bash
$ vim /etc/ettercap/etter.dns
```



- 若写 *则匹配所有（未试验，应该是适用正则） 
- A：所有域名指向192.168.83.157（kali的IP）



具体操作：参照以下链接可以实行ARP欺骗

[(134条消息) Kali Linux渗透测试小实践——DNS欺骗_kali dns欺骗_PICACHU+++的博客-CSDN博客](https://blog.csdn.net/qq_60503432/article/details/128353116)

### 2.DNS劫持

基于以上的arp欺骗，可以进而实施dns欺骗

#### 开启端口转发

ip_forward：为0时是断网攻击，为1时开启流量转发。

- 以下命令可以获取当前的ip_forward的值

```b
$ cat /proc/sys/net/ipv4/ip_forward
```

- 以下命令将其设置为1

```bash
$ echo 1 >  /proc/sys/net/ipv4/ip_forward 
```



#### kali机启动apache服务

```bash
$ service apache2 start
```

之后访问**http**网页，则会看到apache的页面，说明劫持成功

#### 更换靶机访问页面的内容

```bash
$ vim /var/www/html/index.html
```



参考：

[(134条消息) Kali利用Ettercap实现中间人攻击之DNS劫持（DNS欺骗）_ettercap dns欺骗_call me pascal的博客-CSDN博客](https://blog.csdn.net/m0_46745664/article/details/126276629#:~:text=开始攻击 1.打开 Ettercap 2.网卡默认eth0，也没什么好说的%2C点√,3.添加攻击目标 4.对目标进行ARP欺骗%2C点击后确认就行 5.打开Plugins，如图 6.打开dns_spoof 7.攻击基本完成，使用被攻击的主机选择baidu进行ping)

[(134条消息) Ettercap中间人攻击——DNS劫持、替换网页内容与ARP欺骗_kali替换网页_Mr. Wanderer的博客-CSDN博客](https://blog.csdn.net/Mr_Wanderer/article/details/107985245)

