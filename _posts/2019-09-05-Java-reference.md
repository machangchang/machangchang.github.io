---
layout: post
title: Java reference
categories: Java
description: Java reference
keywords: Java reference
---

Java 传引用还是传值？

先看一段代码：

### 1. 示例1

```java
public class Main {
    public static void test(int age,int weight){
        System.out.println("传入的age: " + age);
        System.out.println("传入的weight: " + weight);
        age = 33;
        weight = 89;
        System.out.println("方法内重新赋值后的age: " + age);
        System.out.println("方法内重新赋值后的weight: " + weight);
    }

    public static void main(String[] args) {
        int a = 25;
        int w = 77;
        test(a,w);
        System.out.println("方法执行后的age: " + a);
        System.out.println("方法执行后的weight: " + w);
    }
}
```

结果：

```java
传入的age: 25
传入的weight: 77.5
方法内重新赋值后的age: 33
方法内重新赋值后的weight: 89.5
方法执行后的age: 25
方法执行后的weight: 77.5
```

接下来的分析涉及到JVM内存中的虚拟机栈，不了解的可以参照 [JVM内存结构](http://machangchang.com//2018/03/24/JVM-1/)。

首先程序运行时，调用mian()方法，此时JVM为main()方法往虚拟机栈中压入一个栈帧，即为当前栈帧，用来存放main()中的局部变量表(包括参数)、操作栈、方法出口等信息，如a和w都是mian()方法中的局部变量，因此可以断定，a和w是在mian()方法所在的栈帧中。

![](/images/blog/2019-09-05-Java-reference/reference_001.jpg)

而当执行到test()方法时，JVM也为其往虚拟机栈中压入一个栈，即为当前栈帧，用来存放test()中的局部变量等信息，因此age和weight是放在着test方法所在的栈帧中，而他们的值是从a和w的值copy了一份副本而得，如图：

![](/images/blog/2019-09-05-Java-reference/reference_002.jpg)

当在方法内重新赋值时，实际流程如图：

![](/images/blog/2019-09-05-Java-reference/reference_003.jpg)

age和weight的改动，只是改变了当前栈帧（test方法所在栈帧）里的内容，当方法执行结束之后，这些局部变量都会被销毁，mian方法所在栈帧重新回到栈顶，成为当前栈帧，再次输出a和w时，依然是初始化时的内容。

再举一个例子：

### 2. 示例2

```java
public class Person {

    private String name;
    private int age;

    public String getName() {
        return name;
    }
    public void setName(String name) {
        this.name = name;
    }
    public int getAge() {
        return age;
    }
    public void setAge(int age) {
        this.age = age;
    }
}
```
```java
public class Main {

    public static void PersonTest(Person person){
        System.out.println("传入的person的name: " + person.getName());
        person.setName("我是张小龙");
        System.out.println("方法内重新复制后的name: " + person.getName());
    }

    public static void main(String[] args) {
        Person p = new Person();
        p.setName("我是马化腾");
        p.setAge(45);
        PersonTest(p);
        System.out.println("方法执行后的name: " + p.getName());
    }
}
```

输出：

```java
传入的person的name: 我是马化腾
方法内重新复制后的name: 我是张小龙
方法执行后的name: 我是张小龙
```

可以看出，person经过PersonTest()方法的执行之后，内容发生了改变。

接着对上面的例子做下修改：

### 3. 示例3

```java
public static void PersonTest(Person person){
        System.out.println("传入的person的name: " + person.getName());
        person = new Person();
        person.setName("我是张小龙");
        System.out.println("方法内重新赋值后的name: " + person.getName());
    }
```

输出：

```java
传入的person的name: 我是马化腾
方法内重新赋值后的name: 我是张小龙
方法执行后的name: 我是马化腾
```

之所以会出现上面的结果是因为程序执行到main（）方法中的下列代码时：

```java
Person p = new Person();
p.setName("我是马化腾");
p.setAge(45);
PersonTest(p);
```

JVM会在堆内开辟一块内存，用来存储p对象的所有内容，同时在main（）方法所在线程的栈区中创建一个引用p存储堆区中p对象的真实地址，如图：

![](/images/blog/2019-09-05-Java-reference/reference_004.jpg)

当执行到PersonTest()方法时，因为方法内有这么一行代码：

```java
person=new Person();
```

JVM需要在堆内另外开辟一块内存来存储new Person()，假如地址为“xo3333”，那此时形参person指向了这个地址。这种情况下类似于第一个例子，此时的局部变量person是在PersonTest()方法所在的栈帧中，它指向的是一个新的Person对象，修改的值也是这个新的对象，不会影响到原来Main()方法栈帧中p所指向的Person对象。

可以参考下图进行理解：

![](/images/blog/2019-09-05-Java-reference/reference_005.jpg)

> 推荐阅读：
http://ifeve.com/stackoverflow-reference-or-value/

> 参考链接：
https://juejin.im/post/5bce68226fb9a05ce46a0476#heading-12
