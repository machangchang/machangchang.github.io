---
layout: post
title: 【译】Java Memory Model
categories: JVM
description: JVM
keywords: JVM
---

【译】Java Memory Model (Java 内存模型)

Java内存模型指定了Java虚拟机如何与计算机的内存（RAM）一起使用。Java虚拟机是整个在计算机的一个模块，所以该模块也有一个内存模型 - AKA Java 内存模型。（**译者注：本文提到的 Java 内存模型严格上来说应该叫 JVM 内存模型**）。

如果想设计正确的并发程序，理解 Java 内存模型是非常重要的。Java 内存模型规定了不同线程如何以及何时可以看到其他线程写入共享变量的值，以及如何在必要时同步对共享变量的访问。

原始的 Java 内存模型存在缺陷，因为在 Java 1.5 版本中进行了修订。此版本的 Java 内存模型仍适用于 Java 8。

## 内部的 Java 内存模型

JVM 内部使用 Java 内存模型来将内存划分为线程栈和堆。下面的图从逻辑角度说明了 Java 内存模型。

![](/images/blog/2018-10-11-Java-Memory-Model/Java_Memory_Model_001.jpg)

每个运行在 Java 虚拟机中的线程都有它独有的线程栈。线程栈包含有关于哪些方法被调用了以到达当前执行点的信息。我将其称为“调用栈”，当线程执行时，其调用栈也会随之改变。

线程栈还包含正在执行的每个方法的所有局部变量（调用栈上的所有方法）。每个线程只能访问它自己的线程栈。由线程创建的局部变量对于除创建它的线程之外的所有其他线程是不可见的。即使两个线程正在执行完全相同的代码，这两个线程仍将在每个自己的线程栈中创建该代码中的局部变量。因此，每个线程都有自己的每个局部变量的版本。

所有原始类型的局部变量（boolean，byte，short，char，int，long，float，double）都存储在线程栈中，因此对其他线程是不可见的。一个线程可以将一个原始变量的副本传递给另一个线程，但它不能共享原始变量本身。

堆包含你 Java 程序中所创建的所有对象，不管是哪个线程创建的。其中包括原始类型的对象版本（比如 Byte, Integer, Long 等等）。创建的对象不管是将其分配给局部变量，还是作为其他对象的成员变量，该对象仍然保存在堆中。

下图说明了调用栈和局部变量存储在线程栈中，而对象存储在堆中。

![](/images/blog/2018-10-11-Java-Memory-Model/Java_Memory_Model_002.jpg)

局部变量可以是基本类型，这种情况下，它完全保存在线程栈上。

局部变量也可以是对象的引用，这种情况下，引用（局部变量）存储在线程栈上，但对象本身存储在堆中。

对象可能包含方法，这些方法可能包含局部变量。即使方法所属的对象存储在堆上，但这些局部变量也存储在线程栈中。

一个对象的成员变量与对象本身一起被存储在堆中。不管成员变量是基本类型还是引用类型。

静态类变量随着类定义也被存储在堆上。

所有拥有对象引用的线程都可以访问堆上的对象。当一个线程可以访问一个对象时，它也可以访问该对象的成员变量。如果两个线程同时在同一个对象上调用一个方法，它们都可以访问该对象的局部变量，但每个线程都拥有自己的局部变量的副本。

如下图所示：

![](/images/blog/2018-10-11-Java-Memory-Model/Java_Memory_Model_003.jpg)

两个线程都有一组局部变量，其中一个局部变量（Local  Variable 2）指向堆上的一个共享对象（Object 3）。两个线程对同一个对象具有不同的引用。它们的引用是局部变量，因此存储在每个线程的线程栈中。但是，这两个不同的引用指向的是堆上的同一个对象。

注意共享变量（Object 3）如何将 Object 2 和 Object 4 作为成员变量的。通过 Object 3 中的这些成员变量引用，两个线程可以访问 Object 2 和 Object 4。

上图还显示了一个局部变量指向了堆中的两个不同的对象。在这种情况下，引用指向了两个不同的对象。理论上，如果两个线程都引用了两个对象，则两个线程都可以访问 Object 1 和 Object 5。但在上图中，每个线程只引用了两个对象中的一个。

因此，什么样的 Java 代码可以导致上面的内存图？代码如下：

```java
public class MyRunnable implements Runnable() {

    public void run() {
        methodOne();
    }

    public void methodOne() {
        int localVariable1 = 45;

        MySharedObject localVariable2 =
            MySharedObject.sharedInstance;

        //... do more with local variables.

        methodTwo();
    }

    public void methodTwo() {
        Integer localVariable1 = new Integer(99);

        //... do more with local variable.
    }
}
```

```java
public class MySharedObject {

    //static variable pointing to instance of MySharedObject

    public static final MySharedObject sharedInstance =
        new MySharedObject();


    //member variables pointing to two objects on the heap

    public Integer object2 = new Integer(22);
    public Integer object4 = new Integer(44);

    public long member1 = 12345;
    public long member1 = 67890;
}
```

如果两个线程执行 run() 方法，那结果就是上图所示的内容。run() 调用 methodOne()，然后methodOne() 调用了 methodTwo()。

methodOne() 声明了一个原始类型的本地变量 localVariable1 和一个引用类型的本地变量 localVariable2。

每个线程在执行 methodOne() 时都会在各自的线程栈里创建自己的 localVariable1 和 localVariable2 副本。本地变量 localVariable1 会完全独立，仅存于各自的线程栈中。一个线程无法知道另一个线程对自己线程栈的的 localVariable1 变量做了什么操作。

每个线程执行 methodOne() 时也会创建 localVariable2 的副本，然而，两个不同的 localVariable2 副本最终都指向堆上同一个对象。代码将 localVariable2 设置为指向一个对象的静态变量。该静态变量只有一个副本，而且存储在堆上。因此 localVariable2 的两个副本最终都指向静态变量所指向的 MySharedObject 的实例。MySharedObject 的实例也存储在堆中。它对应上图的 Object3。

注意 MySharedObject 类包含两个成员变量。成员变量本身与对象一起存储在堆上。这两个成员变量指向另外两个 Integer 对象。这些 Integer 对象对应于上图中的 Object2 和 Object4。

另外注意 methodTwo() 创建了一个本地变量 localVariable1。这个本地变量是 Integer 对象的引用。这个方法设置了 localVariable1 引用变量指向了一个新的 Integer 实例。localVariable1 这个引用将会被存储在执行 methodTwo() 方法的每一个线程的副本中。实例化的两个 Integer 对象将存储在堆上，但由于每次该方法执行时都会创建一个新的 Integer 对象，因此执行此方法的两个线程将创建单独的 Integer 实例。在 methodTwo() 中创建的 Integer 对象对应于上图中的 Object1 和 Object5。

还要注意 MySharedObject 中的类型为 long 的两个成员变量，它们是基本类型。由于这些变量是成员变量，因此它们仍与对象一起被存储在堆上。只有局部变量存储在线程栈中。

## 硬件内存架构

> 原文链接：http://tutorials.jenkov.com/java-concurrency/java-memory-model.html

> 声明：本站采用开放的[知识共享署名-非商业性使用-相同方式共享 许可协议](https://creativecommons.org/licenses/by-nc-sa/3.0/deed.zh)进行许可。
