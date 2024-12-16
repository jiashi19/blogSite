---
title: HTTPS 协议浅析
categories:
  - others
date: 2023-11-16 09:43:54
tags:
---

<!-- more -->

# HTTPS协议（TLS v1.2）

## 概括介绍

总体流程：

四次握手

<img src="https://s2.loli.net/2023/11/08/V4uDORrZh6xEP81.png" alt="image-20231108200139162" style="zoom:55%;" />



TLS/SSL协议结构分层：

网图如下：

<img src="https://s2.loli.net/2023/11/08/YrgFohC96buVGNw.png" alt="image-20231108201059958" style="zoom:55%;" />

<img src="https://s2.loli.net/2023/11/08/QPN3vGwd9sgJYph.png" alt="image-20231108201124713" style="zoom:55%;" />



Handshake 协议有很多种类：Client Hello，Server Hello，Server Certificate 等，是握手的核心。

Change Cipher Spec 协议是有点独立的协议，也是握手必须的。用于告诉对方，我要使用我们商量好的会话秘钥了。

Alert 协议用于警告双方握手过程没有成功。

## 具体握手过程

注，一次握手可包含多个 Message，比如 server 一次握手可能包含 Server Hello、Server Certificate、Server Key Exchange 等，以此类推。

主要是RSA和ECDHE两种密钥交换算法。



下面介绍**ECDHE**:

### client第一次握手：Client Hello

- Version

- Random：**客户端随机数**，我们称之为随机数 1，这个随机数是为了后面生成对称的**会话秘钥**。

- Session ID：可选的字段。

- Cipher Suites：加密套件。提供一组，可供选择

- Compression Methods：压缩算法，**在加密之前数据压缩**。取值 00 代表不压缩。因为会弱化加密数据的安全性，所以在未来的版本已经不推荐使用了。

  

### server第一次握手

####  Server Hello

- Version：选定的协议。没协商好会有alert协议
- Cipher Suite：选定的加密套件。
- random：**服务端随机数**。

- compression method

#### server Certificate

- the hostname of the server
- the public key used by this server
- proof from a trusted third party that the owner of this hostname holds the private key for this public key



#### （Server Key Exchange）

交换公钥

#### Server Hello Done

后面三条封装在一个包中

### client第二次握手

#### Client Key Exchange

交换公钥

The client provides information for key exchange. As part of the key exchange process both the server and the client will have a keypair of public and private keys, and will send the other party their public key. The shared encryption key will then be generated using a combination of each party's private key and the other party's public key.

The parties have agreed on a cipher suite using ECDHE, meaning the keypairs will be based on a selected **E**lliptic **C**urve, **D**iffie-**H**ellman will be used, and the keypairs are **E**phemeral (generated for each connection) rather than using the public/private key from the certificate.

#### Change Cipher Spec

{

客户端计算密钥：

The client now has the information to calculate the encryption keys that will be used by each side. It uses the following information in this calculation:

- server random (from Server Hello)
- client random (from Client Hello)
- server public key (from Server Key Exchange)
- client private key (from Client Key Generation)

}

告诉 server，要使用商量好的对称秘钥了。请注意，这里是独立的协议，即 Change Cipher Spec Protocol。

#### Encrypted Handshake Message

为了验证握手是否成功且未被篡改。加密之前的握手消息的哈希值。

### server第二次握手

#### Change Cipher Spec

#### Encrypted Handshake Message

验证数据根据所有握手消息的哈希值构建，用于验证握手过程的完整性

**使用RSA密钥交换算法**：

使用 RSA 秘钥交换算法的四次握手的过程和 ECDHE 过程大体一样，如下：

- Bob 第一次握手：Bob 请求建立 TLS 连接，发送**协议版本、加密套件、**一个随机数 **client random** 以及支持的压缩算法给 Alice；
- Alice 第一次握手：Alice 根据 Bob 提供的加密套件和自己支持的情况，选择其中的一种加密套件，选定协议版本，加上**第二个随机数 server random**，和**数字证书（其中有公钥）**，发送给 Bob；
- Bob 第二次握手：Bob 确认这个**数字证书是有效的**，并且再生成第三个随机数-**预秘钥**，即PreMaster key（premastersecret）。将这个PreMaster key用服务器发送给它的数字证书中的**公钥进行加密**发送给服务器；以及放一个**ChangeCipherSpec消息即编码改变**的消息，还有<u>整个前面所有消息的hash值</u>。客户端使用前面的两个随机数以及刚刚新生成的新随机数，使用与服务器确定的加密算法，生成会话秘钥Session key。

这里，没有发送 ECDHE 算法交换中的消息 **Server Key Exchange**。

![img](https://pic1.zhimg.com/80/v2-ba6918091113c294b1ae0a94500e1d24_1440w.webp)

- Alice 第二次握手：Alice 收到 Bob 的回复，利用自己的**私钥进行解密**，获得这个随机数，然后通过将前面这**三个随机数**以及他们协商的加密方式，计算生成一个会话密钥 session key（session secret）。服务端也会使用 Session key 加密一段 Finish 消息发送给客户端，以验证之前通过握手建立起来的加解密通道是否成功。

## 参考链接

https://zhuanlan.zhihu.com/p/421446218

https://tls12.xargs.org/

https://www.cnblogs.com/wusanga/p/17383505.html
