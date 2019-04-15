---
layout: post
title: Java Type Conversion
categories: Java
description: Java
keywords: Type Conversion
---

基本类型转换

## 自动类型转换

自动类型转换是指：数字表示范围小的数据类型可以自动转换成范围大的数据类型。

```java
long l = 100;

int i = 200;
long ll = i;
```

具体自动转换如如下图所示。

![](/images/blog/2019-04-15-Type-Conversion/Type_Conversion_001.jpg)

实线表示自动转换时不会造成数据丢失，虚线则可能会出现数据丢失问题。丢失精度是因为不同类型的存储结构不一样，[相关链接](https://blog.csdn.net/koreyoshi326/article/details/71513208)。

自动转换也要小心数据溢出问题，看下面的例子。

```java
int count = 100000000;
int price = 1999;
long totalPrice = count * price;
```

编译没任何问题，但结果却输出的是负数，这是因为两个 int 相乘得到的结果是 int, 相乘的结果超出了 int 的代表范围。

另外，向下转换时可以直接将 int 常量字面量赋值给 byte、short、char 等数据类型，而不需要强制转换，只要该常量值不超过该类型的表示范围都能自动转换。

## 强制类型转换

强制类型转换我们再清楚不过了，即强制显示的把一个数据类型转换为另外一种数据类型。

```java
double d = 10.24;
long ll = (long) d;// 10
```

以上的转换结果都在我们的预期之内，属于正常的转换和丢失精度的情况，下面的例子就一样属于数据溢出的情况。

```java
int ii = 300;
byte b = (byte)ii;
```

300 已经超出了 byte 类型表示的范围，所以会转换成一个毫无意义的数字。

## 类型提升

所谓类型提升就是指在多种不同数据类型的表达式中，类型会自动向范围表示大的值的数据类型提升。

把上面的溢出的例子再改下。

```java
long count = 100000000;
int price = 1999;
long totalPrice = price * count;
```

price 为 int 型，count 为 long 型，运算结果为 long 型，运算结果正常，没有出现溢出的情况。

> 声明：本站采用开放的[知识共享署名-非商业性使用-相同方式共享 许可协议](https://creativecommons.org/licenses/by-nc-sa/3.0/deed.zh)进行许可。
