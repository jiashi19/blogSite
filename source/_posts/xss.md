---
title: XSS
categories:
  - 渗透
date: 2023-11-16 09:39:52
tags:
---

# XSS

## session管理

- server端session
- cookie-based
- token-based

<!-- more -->

## session攻击

### 会话劫持

中间人攻击。

### 会话固定

含义：想办法重置用户session id，目标用户携带攻击者设定的session id登录。

#### 攻击方法

1. 客户端脚本：

   httponly

```html
<script> document.cookie="PHPSESSID=99999" ;</script>
```

2. 使用HTML的<META>标签加Set-Cookie属性。

服务器可以在返回的HTML文档中增加<meta>标签来设置Cookie

```html
<meta http-equiv='Set-Cookie' content='PHPSESSID=23333'>
```

3. set-cookie

攻击者使用一些方法在web服务器的响应中加入Set-cookie的HTTP响应头部。

#### 防御方法

1、每当用户登陆的时候就进行重置Session lD

2、Session ID闲置过久时，进行重置Session lD

3、大部分防止会话劫持的方法对会话固定攻击同样有效。如设置HttpOnly，关闭透明化Session lD,User-Agent验证，Token校验等。

## Cookie安全

### **Cookie字段**

setcookie()函数用于设置cookie

```
[name][value][expires][path][domain][secure][httponly]
```

### **Cookie存储**

本地Cookie：设置了过期时间

内存Cookie：未设置过期时间，随浏览器关闭消失

攻击者可以将内存Cookie转换为本地Cookie

## XSS类型

反射性

存储型（持久型）

DOM型



（xss数据接收平台：https://xssaq.com/）

## XSS防御

Content Security Policy（CSP）



