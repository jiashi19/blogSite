---
title: Java泛型
categories:
  - java
date: 2024-07-25 00:24:33
tags:
---

<!-- more -->

# Java泛型
例子：

`Class<T>`：泛型类。即`ArrayList<T>`，就是泛型类

**T**（Type）: java类

**?** :  泛型通配符



碰到过的与java反射结合的运用：

```java
Class clz = ((Class<T>) ((ParameterizedType) getClass().getGenericSuperclass())).getActualTypeArguments()[0];
```
**getGenericSuperclass()**：返回表示由此类表示的实体（类、接口、基元类型或void）的直接超类的`Type`。
如果超类是泛型类，则返回`ParameterizedType`。

**getActualTypeArguments()**：返回表示此类型的实际类型参数的Type对象的数组。

代码例子：

```java
//BaseTest
public class BaseTest<T, I>{

}
```

```java

//ResultTest
import java.lang.reflect.ParameterizedType;

public class ResultTest extends BaseTest<String,Integer>{
    public static void main(String[] args) {
        System.out.println(((ParameterizedType)new ResultTest().getClass().getGenericSuperclass()).getActualTypeArguments()[0]);
    }
}

//输出
//class java.lang.String

```

## 运用场景

**泛型DAO（数据访问对象）模式**

在持久层（如使用JPA或Hibernate）中，泛型DAO可以用来操作各种实体类，但在运行时需要知道实际的实体类类型以便执行查询操作。

