---
layout: post
title: Java assert
categories: Java
description: Java assert
keywords: Java, assert
---

Java SE 从1.4版本开始增加了断言(assert)的特性。

断言是为了方便调试程序，并不是发布程序的组成部分。理解这一点是很关键的。

默认情况下，JVM是关闭断言的。因此如果想使用断言调试程序，需要手动打开断言功能。在命令行模式下运行Java程序时可增加参数-enableassertions或者-ea打开断言。可通过-disableassertions或者-da关闭断言(默认情况,可有可无)。

断言的使用：

断言是通过关键字assert来定义的，一般的，它有两种形式。

- 1.assert <bool expression>;  比如     boolean isStudent = false; assert isStudent;
- 2.assert <bool expression> : <message>;    比如  boolean isSafe = false;  assert isSafe : "Not Safe at all";

#### 第一种形式：

```java
public class AssertionTest { 
  
    public static void main(String[] args) {  
          
        boolean isSafe = false;  
        assert isSafe;  
        System.out.println("断言通过!");  
    }  
}  
```

如果是在命令行模式下运行，需要指明开启断言功能。

```
java -ea AssertionTest  
```

如果是在IDE下，比如Eclipse，可这样设置: Run as -> Run Configurations -> Arguments -> VM arguments：敲入-ea即可。

输出：

```
Exception in thread "main" java.lang.AssertionError  
    at AssertionTest.main(AssertionTest.java:8)  
```

#### 第二种形式

```java
public class AssertionTest {  
  
    public static void main(String[] args) {  
          
        boolean isSafe = false;  
        assert isSafe : "Not safe at all";  
        System.out.println("断言通过!");  
    }  
}  
```

输出：

```
Exception in thread "main" java.lang.AssertionError: Not safe at all  
    at AssertionTest.main(AssertionTest.java:7)  
```

*第二种形式和第一种的区别在于后者可以指定错误信息*

#### 注意

断言只是为了用来调试程序，切勿将断言写入业务逻辑中。比如考虑下面这个简单的例子：

```java
public class AssertionTest {  
  
    public static void main(String[] args) {  
          
            assert ( args.length > 0);  
            System.out.println(args[1]);  
    }  
}  
```

该句assert (args.length >0)和if(args.length >0)意思相近，但是如果在发布程序的时候（一般都不会开启断言），所以该句会被忽视，因此会导致以下:

```
Exception in thread "main" java.lang.ArrayIndexOutOfBoundsException: 1  
    at AssertionTest.main(AssertionTest.java:7)  
```

> 声明：本站采用开放的[知识共享署名-非商业性使用-相同方式共享 许可协议](https://creativecommons.org/licenses/by-nc-sa/3.0/deed.zh)进行许可。
