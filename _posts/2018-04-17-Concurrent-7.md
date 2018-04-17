---
layout: post
title: Java并发编程：同步容器
categories: Java并发编程
description: 并发
keywords: 并发
---

为了方便编写出线程安全的程序，Java里面提供了一些线程安全类和并发工具，比如：同步容器、并发容器、阻塞队列、Synchronizer（比如CountDownLatch）。今天我们就来讨论下同步容器。

## 一.为什么会出现同步容器？

在Java的集合容器框架中，主要有四大类别：List、Set、Queue、Map。

List、Set、Queue接口分别继承了Collection接口，Map本身是一个接口。

注意Collection和Map是一个顶层接口，而List、Set、Queue则继承了Collection接口，分别代表数组、集合和队列这三大类容器。

像ArrayList、LinkedList都是实现了List接口，HashSet实现了Set接口，而Deque（双向队列，允许在队首、队尾进行入队和出队操作）继承了Queue接口，PriorityQueue实现了Queue接口。另外LinkedList（实际上是双向链表）实现了了Deque接口。

像ArrayList、LinkedList、HashMap这些容器都是非线程安全的。

如果有多个线程并发地访问这些容器时，就会出现问题。

因此，在编写程序时，必须要求程序员手动地在任何访问到这些容器的地方进行同步处理，这样导致在使用这些容器的时候非常地不方便。

所以，Java提供了同步容器供用户使用。

## 二.Java中的同步容器类

在Java中，同步容器主要包括2类：

　　1）Vector、Stack、HashTable

　　2）Collections类中提供的静态工厂方法创建的类

　　Vector实现了List接口，Vector实际上就是一个数组，和ArrayList类似，但是Vector中的方法都是synchronized方法，即进行了同步措施。

　　Stack也是一个同步容器，它的方法也用synchronized进行了同步，它实际上是继承于Vector类。

　　HashTable实现了Map接口，它和HashMap很相似，但是HashTable进行了同步处理，而HashMap没有。

　　Collections类是一个工具提供类，注意，它和Collection不同，Collection是一个顶层的接口。在Collections类中提供了大量的方法，比如对集合或者容器进行排序、查找等操作。最重要的是，在它里面提供了几个静态工厂方法来创建同步容器类，如下图所示：

![](/images/blog/2018-04-17-Concurrent-7/Concurrent_7_001.jpg)

## 三.同步容器的缺陷

1. 从同步容器的具体实现源码可知，同步容器中的方法采用了synchronized进行了同步，那么很显然，这必然会影响到执行性能。

2. 同步容器真的是安全的吗？

也有有人认为Vector中的方法都进行了同步处理，那么一定就是线程安全的，事实上这可不一定。看下面这段代码：

```java
public class Test {
    static Vector<Integer> vector = new Vector<Integer>();
    public static void main(String[] args) throws InterruptedException {
        while(true) {
            for(int i=0;i<10;i++)
                vector.add(i);
            Thread thread1 = new Thread(){
                public void run() {
                    for(int i=0;i<vector.size();i++)
                        vector.remove(i);
                };
            };
            Thread thread2 = new Thread(){
                public void run() {
                    for(int i=0;i<vector.size();i++)
                        vector.get(i);
                };
            };
            thread1.start();
            thread2.start();
            while(Thread.activeCount()>10)   {
                 
            }
        }
    }
}
```

运行结果：

```java
Exception in thread "Thread-337" java.lang.ArrayIndexOutOfBoundsException: Array index out of range: 13
	at java.util.Vector.get(Vector.java:744)
	at concurrent.Test$2.run(Test.java:25)
Exception in thread "Thread-436" java.lang.ArrayIndexOutOfBoundsException: Array index out of range: 14
	at java.util.Vector.remove(Vector.java:827)
	at concurrent.Test$1.run(Test.java:19)
```

正如大家所看到的，这段代码报错了：数组下标越界。

也许有朋友会问：Vector是线程安全的，为什么还会报这个错？很简单，对于Vector，虽然能保证每一个时刻只能有一个线程访问它，但是不排除这种可能：

当某个线程在某个时刻执行这句时：

```java
for(int i=0;i<vector.size();i++)
    vector.get(i);
```

假若此时vector的size方法返回的是10，i的值为9

然后另外一个线程执行了这句：

```java
for(int i=0;i<vector.size();i++)
    vector.remove(i);
```

将下标为9的元素删除了。

那么通过get方法访问下标为9的元素肯定就会出问题了。

因此为了保证线程安全，必须在方法调用端做额外的同步措施，如下面所示：

```java
public class Test {
    static Vector<Integer> vector = new Vector<Integer>();
    public static void main(String[] args) throws InterruptedException {
        while(true) {
            for(int i=0;i<10;i++)
                vector.add(i);
            Thread thread1 = new Thread(){
                public void run() {
                    synchronized (Test.class) {   //进行额外的同步
                        for(int i=0;i<vector.size();i++)
                            vector.remove(i);
                    }
                };
            };
            Thread thread2 = new Thread(){
                public void run() {
                    synchronized (Test.class) {
                        for(int i=0;i<vector.size();i++)
                            vector.get(i);
                    }
                };
            };
            thread1.start();
            thread2.start();
            while(Thread.activeCount()>10)   {
                 
            }
        }
    }
}
```

3. ConcurrentModificationException异常

在对Vector等容器并发地进行迭代修改时，会报ConcurrentModificationException异常，关于这个异常将会在后续文章中讲述。

但是在并发容器中不会出现这个问题。

> 参考链接：http://www.cnblogs.com/dolphin0520/p/3933404.html

> 声明：本站采用开放的[知识共享署名-非商业性使用-相同方式共享 许可协议](https://creativecommons.org/licenses/by-nc-sa/3.0/deed.zh)进行许可。
