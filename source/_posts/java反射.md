---
title: java反射
categories:
  - java
date: 2024-07-21 20:48:48
tags:
password: 12345f
---

<!-- more -->

# java反射

获取**class对象**的三种方法：

```java
package com.reflect;


public class Main {
    public static void main(String[] args) throws ClassNotFoundException {
        Class c1=Class.forName("com.reflect.Student");

        Class c2=Student.class;
    
        Student s=new Student();

        Class c3=s.getClass();
    }

}
```

获取对象的构造方法（仅public，getDeclaredConstructor方法可以获取全部）：

```java
package com.reflect;

import java.lang.reflect.Constructor;

public class Main {
    public static void main(String[] args) throws ClassNotFoundException {
        Class c1=Class.forName("com.reflect.Student");
        
        Constructor[] c1Constructors = c1.getConstructors();
        for(Constructor con:c1Constructors){
            System.out.println(con);
        }
    }
}
```

获取对象的私有构造方法，且设置取消权限，强行调用私有方法创建对象：

```java
package com.reflect;

import java.lang.reflect.Constructor;
import java.lang.reflect.InvocationTargetException;

public class Main {
    public static void main(String[] args) throws ClassNotFoundException, NoSuchMethodException, InvocationTargetException, InstantiationException, IllegalAccessException {
        Class c1=Class.forName("com.reflect.Student");

        Constructor c1DeclaredConstructor = c1.getDeclaredConstructor(int.class);
        System.out.println(c1DeclaredConstructor);
        c1DeclaredConstructor.setAccessible(true);
        Student s=(Student) c1DeclaredConstructor.newInstance(33);
        System.out.println(s.getAge());
    }
}
//private com.reflect.Student(int)
//33
```

反射获取成员变量：

```java
package com.reflect;

import java.lang.reflect.Field;
import java.lang.reflect.InvocationTargetException;

public class Main {
    public static void main(String[] args) throws ClassNotFoundException, IllegalAccessException, NoSuchFieldException  {
        Class c1=Class.forName("com.reflect.Student");

        Field[] c1Fields = c1.getDeclaredFields();
        for (Field field : c1Fields) {
            System.out.println(field);
        }
        Student s=new Student(1,2);
        Field c1DeclaredField = c1.getDeclaredField("num");
        System.out.println(c1DeclaredField.getName());
        System.out.println(c1DeclaredField.get(s));
        c1DeclaredField.set(s,3);
        System.out.println(s.num);
    }
}
/*
public int com.reflect.Student.num
private int com.reflect.Student.age
private int com.reflect.Student.sex
num
1
3
*/
```

反射获取成员方法：略



反射作用：

1. 获取类中所有信息；
2. 结合配置文件 动态创建对象（代码打包好后，进行一些修改or else？以后结合具体场景进行理解吧）



[Java中的getResource()方法，及路径相关问题-腾讯云开发者社区-腾讯云 (tencent.com)](https://cloud.tencent.com/developer/article/1901321)

## Spring AOP中的反射

在 Spring AOP 中，切面可以通过 JoinPoint 对象获取到连接点的相关信息，其中包括方法签名信息，可以通过 JoinPoint 对象的 getSignature() 方法获取到 Signature 对象。如果连接点是一个方法，那么可以将 Signature 对象转换为 MethodSignature 对象，通过 MethodSignature 对象可以获取到更加详细的方法签名信息，比如方法返回类型、参数类型等。


例：从已有的对象获取其class对象——在aop编程中,获取到如参数只能看作Object对象，那么通过**getClass()**，之后再通过后续的一系列方法，获取成员变量、成员方法等。

学习链接：

[揭秘 Spring AOP 中 Signature 对象：把握连接点精髓 - ByteZoneX社区](https://www.bytezonex.com/archives/gxNXhKj-.html)
