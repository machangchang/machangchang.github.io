---
layout: post
title: 阻塞队列：BlockingQueue
categories: Java
description: Java
keywords: Java
---

阻塞队列：BlockingQueue

BlockingQueue有四种状况，抛出异常、返回特殊值、阻塞、超时，下表总结了这些方法：

| --  | 抛出异常 | 特殊值 | 阻塞 | 超时 |
|---|---|---|
| 插入 | add(e) | offer(e) | put(e) | offer(e, time, unit) |
| 移除 | remove() | poll() | take() | poll(time, unit) |
| 检查 | element() | peek() | 不可用 | 不可用 |

BlockingQueue是个接口，有如下实现类：

1. ArrayBlockQueue：一个由数组支持的有界阻塞队列。此队列按 FIFO（先进先出）原则对元素进行排序。创建其对象必须明确大小，像数组一样。

2. LinkedBlockQueue：一个可改变大小的阻塞队列。此队列按 FIFO（先进先出）原则对元素进行排序。创建其对象如果没有明确大小，默认值是Integer.MAX_VALUE。链接队列的吞吐量通常要高于基于数组的队列，但是在大多数并发应用程序中，其可预知的性能要低。

3. PriorityBlockingQueue：类似于LinkedBlockingQueue，但其所含对象的排序不是FIFO，而是依据对象的自然排序顺序或者是构造函数所带的Comparator决定的顺序。

4. SynchronousQueue：同步队列。同步队列没有任何容量，每个插入必须等待另一个线程移除，反之亦然。

下面使用`ArrayBlockQueue`来实现之前实现过的生产者消/费者模式，代码如下：

```
package Test;

import java.util.concurrent.ArrayBlockingQueue;
import java.util.concurrent.BlockingQueue;

public class BigPlate {  
    
    /** 装鸡蛋的盘子，大小为5 */  
    private BlockingQueue eggs = new ArrayBlockingQueue(5);  
      
    /** 放鸡蛋 */  
    public void putEgg(Object egg) {  
        try {  
            eggs.put(egg);// 向盘子末尾放一个鸡蛋，如果盘子满了，当前线程阻塞  
        } catch (InterruptedException e) {  
            e.printStackTrace();  
        }  
  
        // 下面输出有时不准确，因为与put操作不是一个原子操作  
        System.out.println("放入鸡蛋");  
    }  
      
    /** 取鸡蛋 */  
    public Object getEgg() {  
        Object egg = null;  
        try {  
            egg = eggs.take();// 从盘子开始取一个鸡蛋，如果盘子空了，当前线程阻塞  
        } catch (InterruptedException e) {  
            e.printStackTrace();  
        }  
  
        // 下面输出有时不准确，因为与take操作不是一个原子操作  
        System.out.println("拿到鸡蛋");  
        return egg;  
   }  
      
    /** 放鸡蛋线程 */  
    static class AddThread extends Thread {  
        private BigPlate plate;  
        private Object egg = new Object();  
  
        public AddThread(BigPlate plate) {  
            this.plate = plate;  
        }  
  
        public void run() {  
            plate.putEgg(egg);
        }  
    }  
  
    /** 取鸡蛋线程 */  
    static class GetThread extends Thread {  
        private BigPlate plate;  
  
        public GetThread(BigPlate plate) {  
            this.plate = plate;  
        }  
 
        public void run() {  
            plate.getEgg();  
        }  
    }  
      
    public static void main(String[] args) {  
        BigPlate plate = new BigPlate();  
        // 先启动10个放鸡蛋线程  
        for(int i = 0; i<10; i++) {  
            new Thread(new AddThread(plate)).start();  
        }  
        // 再启动10个取鸡蛋线程  
        for(int i = 0; i<10; i++) {  
            new Thread(new GetThread(plate)).start();  
        }  
    }  
}  
```
从结果看，启动10个放鸡蛋线程和10个取鸡蛋线程，前5个放入鸡蛋的线程成功执行，到第6个，发现盘子满了，阻塞住，这时切换到取鸡蛋线程执行，成功实现了生产者/消费者模式。

> 声明：本站采用开放的[知识共享署名-非商业性使用-相同方式共享 许可协议](https://creativecommons.org/licenses/by-nc-sa/3.0/deed.zh)进行许可。
