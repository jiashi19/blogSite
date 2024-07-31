---
title: Java泛型,Java IO
categories:
  - java
date: 2024-07-25 00:24:33
tags:
---

<!-- more -->

## Java泛型
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

### 运用场景

**泛型DAO（数据访问对象）模式**

在持久层（如使用JPA或Hibernate）中，泛型DAO可以用来操作各种实体类，但在运行时需要知道实际的实体类类型以便执行查询操作。



## Java-IO

### InputStream
Java标准库提供的最基本的输入流。所有输入流的超类。

原始的int read()：读取下一个字节（返回字节表示的int值（0~255）。

在读取流的时候，一次读取一个字节并不是最高效的方法。->缓冲。

InputStream有两个重载方法read(byte[] b)和read(byte[] b, int off, int len)。

利用上述方法一次读取多个字节时，需要先定义一个**byte[]**数组作为缓冲区，read()方法会尽可能多地读取字节到缓冲区， 但不会超过缓冲区的大小。read()方法的返回值不再是字节的int值，而是返回实际读取了多少个字节。如果返回-1，表示没有更多的数据


```java
public void readFile() throws IOException {
    try (InputStream input = new FileInputStream("src/readme.txt")) {
        // 定义1000个字节大小的缓冲区:
        byte[] buffer = new byte[1000];
        int n;
        while ((n = input.read(buffer)) != -1) { // 读取到缓冲区
            System.out.println("read " + n + " bytes.");
        }
    }
}
```

### InputStream实现类

FileInputStream.
ByteArrayInputStream 模拟一个InputStream（测试）



```java
import java.io.*;
public class Main {
    public static void main(String[] args) throws IOException {
        byte[] data = { 72, 101, 108, 108, 111, 33 };
        try (InputStream input = new ByteArrayInputStream(data)) {
            int n;
            while ((n = input.read()) != -1) {
                System.out.println((char)n);
            }
        }
    }
}
```


读取文件内容存为字符串：

```java
public static String readAsString(InputStream input) throws IOException {
    int n;
    StringBuilder sb = new StringBuilder();
    while ((n = input.read()) != -1) {
        sb.append((char) n);
    }
    return sb.toString();
}
```

**总结：**
总是使用try(resource)来保证InputStream正确关闭：


```java
try (InputStream input=new FileInputStream("src/main/resources/ab.txt");){
    String s=readAsString(input);
    System.out.println(s);
}
```

### OutputStream


```java
public void writeFile() throws IOException {
    try (OutputStream output = new FileOutputStream("out/readme.txt")) {
        output.write("Hello".getBytes("UTF-8")); // Hello
    } // 编译器在此自动为我们写入finally并调用close()
}
```

`flush() `：
数据先会写到缓冲区再等满时一次写入。


### Reader 
字符流（InputStream为字节流）。

普通的Reader实际上是基于InputStream构造的，因为Reader需要从InputStream中读入字节流（byte），然后，根据编码设置，再转换为char就可以实现字符流。FileReader在内部实际上持有一个FileInputStream。 `InputStreamReader`，它可以把任何InputStream转换为Reader。 

```java
try (Reader reader = new InputStreamReader(new FileInputStream("src/readme.txt"), "UTF-8")) {
    // TODO:
}
```

### writer
同样的，字符流。