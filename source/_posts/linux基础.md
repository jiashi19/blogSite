---
title: linux基础-个人学习总结
categories:
  - others
date: 2024-07-11 21:24:04
tags:
---
# linux基础知识点

`ps -ef| grep name`
查看相关进程.

`cat log |grep info`
读取日志，查看相关信息.

<!-- more -->
`tail -n 100 -f log `
-n 100行，-f 实时更新显示.

`lsof -i:3306`
查看对应端口.

`curl -k xx.com`
-k/--insecure 设置允许不安全连接

`curl -v xx.com` 
显示通信的整个过程信息

环境变量相关的两个文件：
`/etc/profile` 
/etc/profile：此文件为系统的**每个用户**设置环境信息，当用户第一次登录时，该文件被执行，并从/etc/profile.d目录的配置文件中搜集shell的设置。

`~/.bash_profile`
每个用户都可使用该文件输入专用于自己使用的shell信息，当用户登录时，该文件仅仅执行一次!默认情况下，他设置一些环境变量，执行用户的.bashrc文件。是交互式login 方式进入 bash 运行的。

环境变量配置：
修改/etc/profile

```bash
export PATH=/usr/local/tomcat/bin/:$PATH
source /etc/profile
```

`netstat -tunlp` 
用于显示 tcp，udp 的端口和进程等相关情况。
-t (tcp) 仅显示tcp相关选项
-u (udp)仅显示udp相关选项
-n 拒绝显示别名，能显示数字的全部转化为数字
-l 仅列出在Listen(监听)的服务状态
-p 显示建立相关链接的程序名

`netstat -tunlp | grep 8000`
查看8000端口