---
layout: post
title: Java CAS And AtomicLong
categories: Java并发编程
description: Java
keywords: CAS
---

## Java CAS And AtomicLong

## CAS 简介

在Java并发中，我们最初接触的应该就是 `synchronized` 关键字了，但是 `synchronized` 属于重量级锁，很多时候会引起性能问题， `volatile` 也是个不错的选择，但是 `volatile` 不能保证原子性，只能在某些场合下使用。

像 `synchronized` 这种独占锁属于悲观锁，它是在假设一定会发生冲突的，那么加锁恰好有用，除此之外，还有乐观锁，乐观锁的含义就是假设没有发生冲突，那么我正好可以进行某项操作，如果要是发生冲突呢，那我就重试直到成功，乐观锁最常见的就是**CAS**。

 `Concurrent` 包下的各种 `Atomic` 开头的原子类，内部都应用到了CAS。

## CAS and AtomicLong

**AtomicLong 内部是通过CAS实现的。**

CAS是单词compare and set的缩写，意思是指在set之前先比较该值有没有变化，只有在没变的情况下才对其赋值。

CAS操作可以分为以下三个步骤：

* 步骤 1.读旧值(即从系统内存中读取所要使用的变量的值，例如：读取变量i的值)。

* 步骤2.求新值（即对从内存中读取的值进行操作，但是操作后不修改内存中变量的值，例如：i=i+1,这一步只进行                   i+1,没有赋值，不对内存中的i进行修改）

* 步骤3.两个不可分割的原子操作

  第一步：比较内存中变量现在的值与 最开始读的旧值是否相同(即从内存中重新读取i的值，与一开始读取的                              i进行比较)

  第二步：如果这两个值相同的话，则将求得的新值写入内存中（即：i=i+1,更改内存中的i的值）

  如果这两个值不相同的话，则重复步骤1开始

注：这两个操作是不可分割的原子操作，必须两个同时完成

### AtomicLong 介绍和方法列表

AtomicLong是作用是对长整形进行原子操作。

在32位操作系统中，64位的long 和 double 变量由于会被JVM当作两个分离的32位来进行操作，所以不具有原子性。而使用AtomicLong能让long的操作保持原子型。

**AtomicLong方法列表**

```java
// 构造方法
AtomicLong()
// 创建值为initialValue的AtomicLong对象
AtomicLong(long initialValue)
// 以原子方式设置当前值为newValue。
final void set(long newValue) 
// 获取当前值
final long get() 
// 以原子方式将当前值减 1，并返回减1后的值。等价于“--num”
final long decrementAndGet() 
// 以原子方式将当前值减 1，并返回减1前的值。等价于“num--”
final long getAndDecrement() 
// 以原子方式将当前值加 1，并返回加1后的值。等价于“++num”
final long incrementAndGet() 
// 以原子方式将当前值加 1，并返回加1前的值。等价于“num++”
final long getAndIncrement()    
// 以原子方式将delta与当前值相加，并返回相加后的值。
final long addAndGet(long delta) 
// 以原子方式将delta添加到当前值，并返回相加前的值。
final long getAndAdd(long delta) 
// 如果当前值 == expect，则以原子方式将该值设置为update。成功返回true，否则返回false，并且不修改原值。
final boolean compareAndSet(long expect, long update)
// 以原子方式设置当前值为newValue，并返回旧值。
final long getAndSet(long newValue)
// 返回当前值对应的int值
int intValue() 
// 获取当前值对应的long值
long longValue()    
// 以 float 形式返回当前值
float floatValue()    
// 以 double 形式返回当前值
double doubleValue()    
// 最后设置为给定值。延时设置变量值，这个等价于set()方法，但是由于字段是volatile类型的，因此次字段的修改会比普通字段（非volatile字段）有稍微的性能延时（尽管可以忽略），所以如果不是想立即读取设置的新值，允许在“后台”修改值，那么此方法就很有用。如果还是难以理解，这里就类似于启动一个后台线程如执行修改新值的任务，原线程就不等待修改结果立即返回（这种解释其实是不正确的，但是可以这么理解）。
final void lazySet(long newValue)
// 如果当前值 == 预期值，则以原子方式将该设置为给定的更新值。JSR规范中说：以原子方式读取和有条件地写入变量但不 创建任何 happen-before 排序，因此不提供与除 weakCompareAndSet 目标外任何变量以前或后续读取或写入操作有关的任何保证。大意就是说调用weakCompareAndSet时并不能保证不存在happen-before的发生（也就是可能存在指令重排序导致此操作失败）。但是从Java源码来看，其实此方法并没有实现JSR规范的要求，最后效果和compareAndSet是等效的，都调用了unsafe.compareAndSwapInt()完成操作。
final boolean weakCompareAndSet(long expect, long update)
```

### AtomicLong 源码分析(基于JDK1.7.0_40)

AtomicLong的代码很简单，下面仅以incrementAndGet()为例，对AtomicLong的原理进行说明。

`incrementAndGet()` 源码如下：

```java
public final long incrementAndGet() {
    for (;;) {
        // 获取AtomicLong当前对应的long值
        long current = get();
        // 将current加1
        long next = current + 1;
        // 通过CAS函数，更新current的值
        if (compareAndSet(current, next))
            return next;
    }
}
```

说明：

1. incrementAndGet()首先会根据get()获取AtomicLong对应的long值。该值是volatile类型的变量，get()的源码如下：

```java
// value是AtomicLong对应的long值
private volatile long value;
// 返回AtomicLong对应的long值
public final long get() {
    return value;
}
```

2. incrementAndGet()接着将current加1,然后通过CAS函数，将新的值赋值给value。
3. 
compareAndSet()的源码如下：

```java
public final boolean compareAndSet(long expect, long update) {
    return unsafe.compareAndSwapLong(this, valueOffset, expect, update);
}
```

compareAndSet()的作用是更新AtomicLong对应的long值。它会比较AtomicLong的原始值是否与expect相等，若相等的话，则设置AtomicLong的值为update。

## ABA 问题

比如说一个线程one从内存位置V中取出A，这时候另一个线程two也从内存中取出A，并且two进行了一些操作变成了B，然后two又将V位置的数据变成A，这时候线程one进行CAS操作发现内存中仍然是A，然后one操作成功。尽管线程one的CAS操作成功，但是不代表这个过程就是没有问题的。

各种乐观锁的实现中通常都会用版本戳version来进行标记，避免并发操作带来的问题。

> 推荐阅读：https://segmentfault.com/a/1190000014858404

> 参考链接：https://juejin.im/post/5a73cbbff265da4e807783f5
https://my.oschina.net/vshcxl/blog/795495
