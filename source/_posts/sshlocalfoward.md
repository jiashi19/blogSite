---
title: 用ssh访问内网服务器的服务
categories:
  - others
date: 2024-12-22 14:01:02
tags:
---

<!-- more -->

# 用ssh访问内网服务器的服务

情况：服务器是虚拟化分出来的，本身没有公网地址，ssh的22端口是通过端口转发/映射到公网IP上的某个端口，可以`ssh user@公网IP -p 33333`，但想从外面访问服务器的一些服务（如3306的mysql，80的nginx），因为这些端口没有设置映射（自己没有权限），没有直接映射到公网上。

solv：**ssh本地转发**。

本地转发（local forwarding）指的是，SSH 服务器作为中介的跳板机，建立本地计算机与特定目标网站之间的加密连接。本地转发是在本地计算机的 SSH 客户端建立的转发规则。

效果：指定一个本地端口（local-port），所有发向那个端口的请求，都会转发到 SSH 跳板机（tunnel-host），然后 SSH 跳板机作为中介，将收到的请求发到目标服务器（target-host）的目标端口（target-port）。

```bash
ssh -L local-port:target-host:target-port tunnel-host 
```

实际应用如下，访问内网服务器的8080开放的http服务。

```bash
ssh -L 8080:127.0.0.1:8080 user@222.222.222.222 -p 55322  -N
#222.222.222.222指代公网IP，55322是内网服务器映射的外网端口
```

- `-N`参数，表示不在 SSH 跳板机执行远程命令，让 SSH 只充当隧道。
- `-f`参数表示 SSH 连接在后台运行。

**参考学习**：

[SSH 端口转发](https://www.cainiaojc.com/ssh/ssh-port-forwarding.html)







