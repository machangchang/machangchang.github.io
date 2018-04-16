---
layout: post
title: Java并发编程：创建线程
categories: Java并发编程
description: 并发
keywords: 并发
---

## 一.Java中如何创建线程

在java中如果要创建线程的话，一般有两种方式：1）继承Thread类；2）实现Runnable接口。

### 1.继承Thread类

继承Thread类的话，必须重写run方法，在run方法中定义需要执行的任务。

```java
class MyThread extends Thread{
    private static int num = 0;
     
    public MyThread(){
        num++;
    }
     
    @Override
    public void run() {
        System.out.println("主动创建的第"+num+"个线程");
    }
}
```

创建好了自己的线程类之后，就可以创建线程对象了，然后通过start()方法去启动线程。注意，不是调用run()方法启动线程，run方法中只是定义需要执行的任务，如果调用run方法，即相当于在主线程中执行run方法，跟普通的方法调用没有任何区别，此时并不会创建一个新的线程来执行定义的任务。

```java
public class Test {
    public static void main(String[] args)  {
        MyThread thread = new MyThread();
        thread.start();
    }
}
 
 
class MyThread extends Thread{
    private static int num = 0;
     
    public MyThread(){
        num++;
    }
     
    @Override
    public void run() {
        System.out.println("主动创建的第"+num+"个线程");
    }
}
```

在上面代码中，通过调用start()方法，就会创建一个新的线程了。为了分清start()方法调用和run()方法调用的区别，请看下面一个例子：

```java
public class Test {
    public static void main(String[] args)  {
        System.out.println("主线程ID:"+Thread.currentThread().getId());
        MyThread thread1 = new MyThread("thread1");
        thread1.start();
        MyThread thread2 = new MyThread("thread2");
        thread2.run();
    }
}
 
 
class MyThread extends Thread{
    private String name;
     
    public MyThread(String name){
        this.name = name;
    }
     
    @Override
    public void run() {
        System.out.println("name:"+name+" 子线程ID:"+Thread.currentThread().getId());
    }
}
```

```
主线程ID:1
name:thread2 子线程ID:1
name:thread1 子线程ID:9
```

从输出结果可以得出以下结论：

　　1）thread1和thread2的线程ID不同，thread2和主线程ID相同，说明通过run方法调用并不会创建新的线程，而是在主线程中直接运行run方法，跟普通的方法调用没有任何区别；

　　2）虽然thread1的start方法调用在thread2的run方法前面调用，但是先输出的是thread2的run方法调用的相关信息，说明新线程创建的过程不会阻塞主线程的后续执行。

　　2.实现Runnable接口

在Java中创建线程除了继承Thread类之外，还可以通过实现Runnable接口来实现类似的功能。实现Runnable接口必须重写其run方法。

下面是一个例子：

```java
public class Test {
    public static void main(String[] args)  {
        System.out.println("主线程ID："+Thread.currentThread().getId());
        MyRunnable runnable = new MyRunnable();
        Thread thread = new Thread(runnable);
        thread.start();
    }
}
 
 
class MyRunnable implements Runnable{
     
    public MyRunnable() {
         
    }
     
    @Override
    public void run() {
        System.out.println("子线程ID："+Thread.currentThread().getId());
    }
}
```

Runnable的中文意思是“任务”，顾名思义，通过实现Runnable接口，我们定义了一个子任务，然后将子任务交由Thread去执行。注意，这种方式必须将Runnable作为Thread类的参数，然后通过Thread的start方法来创建一个新线程来执行该子任务。如果调用Runnable的run方法的话，是不会创建新线程的，这根普通的方法调用没有任何区别。

　　事实上，查看Thread类的实现源代码会发现Thread类是实现了Runnable接口的。

　　在Java中，这2种方式都可以用来创建线程去执行子任务，具体选择哪一种方式要看自己的需求。直接继承Thread类的话，可能比实现Runnable接口看起来更加简洁，但是由于Java只允许单继承，所以如果自定义类需要继承其他类，则只能选择实现Runnable接口。

> 参考链接：http://www.cnblogs.com/dolphin0520/p/3913517.html

> 声明：本站采用开放的[知识共享署名-非商业性使用-相同方式共享 许可协议](https://creativecommons.org/licenses/by-nc-sa/3.0/deed.zh)进行许可。
