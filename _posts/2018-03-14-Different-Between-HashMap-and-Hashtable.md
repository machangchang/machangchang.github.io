---
layout: post
title: Java Different between HashMap and Hashtable
categories: Java
description: HashMap Hashtable
keywords: HashMap Hashtable
---

## HashMap和Hashtable的区别

### （一）

HashTable基于Dictionary类，而HashMap是基于AbstractMap。Dictionary是什么？它是任何可将键映射到相应值的类的抽象父类，而AbstractMap是基于Map接口的骨干实现，它以最大限度地减少实现此接口所需的工作。

### （二）

HashMap可以允许存在一个为null的key和任意个为null的value，但是HashTable中的key和value都不允许为null。

### （三）

Hashtable的方法是同步的，HashMap未经同步，所以在多线程场合要手动同步HashMap，这个区别就像Vector和ArrayList一样。

### （四）

哈希值的使用不同，Hashtable直接使用对象的hashCode，而HashMap重新计算hash值，而且用与代替求模。

### （五）

Hashtable中hash数组默认大小是11，增加的方式是 old*2+1。HashMap中hash数组的默认大小是16，而且一定是2的指数。
