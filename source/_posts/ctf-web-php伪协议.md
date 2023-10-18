---
title: ctf-web-php伪协议
categories:
  - [ctf-web]
  - 渗透
date: 2023-10-16 21:41:42
tags:
---

# 文件包含漏洞和php伪协议

## 文件包含漏洞

什么是文件包含漏洞：

为了更好地使用代码的重用性，可以使用文件包含函数将文件包含进来，直接使用文件中的代码来提高重用性。但是这也产生了**文件包含漏洞**，产生原因是在通过 PHP 的函数引入文件时，为了灵活包含文件会将被包含文件设置为变量，通过动态变量来引入需要包含的文件。此时用户可以对变量的值可控，而服务器端未对变量值进行合理地校验或者校验被绕过，就会导致文件包含漏洞。

<!-- more -->

文件包含示例：

```php
<?php   
$file = $_GET['file'];
include($file); 
?>
```



## 伪协议

### php://filter

```
php://filter/read=convert.base64-encode/resource=index.php
```

```
php://filter/convert.base64-encode/resource=index.php
```

访问本地文件的协议，/read=convert.base64-encode/ 表示读取的方式是 base64 编码后，resource=index.php 表示目标文件为index.php

**转换过滤器**：

1. convert.base64-encode & convert.base64-decode。即上面示例
2. convert.iconv.*。使用有两种方法:

```
convert.iconv.<input-encoding>.<output-encoding> 
or
convert.iconv.<input-encoding>/<output-encoding>
```

支持的编码：

```txt
UCS-4*
UCS-4BE
UCS-4LE*
UCS-2
UCS-2BE
UCS-2LE
UTF-32*
UTF-32BE*
UTF-32LE*
UTF-16*
UTF-16BE*
UTF-16LE*
UTF-7
UTF7-IMAP
UTF-8*
ASCII*
```

### php://input

可以访问请求的原始数据的只读流，可以读取 POST 请求的参数--即可以在**post data中附带上php代码**。

比如，遍历当前目录下所有文件：

```php
<?php
system("ls");  
?>
```

### data伪协议

```
data://text/plain;base64,xxxx(base64编码后的数据)
```

参考链接

https://www.php.net/manual/zh/mbstring.supported-encodings.php

https://blog.csdn.net/woshilnp/article/details/117266628

https://blog.csdn.net/qq_44657899/article/details/109300335

https://blog.csdn.net/cocoaiu/article/details/126319957

https://www.cnblogs.com/linfangnan/p/13535097.html
