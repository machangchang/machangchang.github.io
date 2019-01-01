---
layout: post
title: Java static
categories: Java
description: static
keywords: static
---

Java static

## 用途

在《Java编程思想》P86页有这样一段话：

“static方法就是没有this的方法。在static方法内部不能调用非静态方法，反过来是可以的。而且可以在没有创建任何对象的前提下，仅仅通过类本身来调用static方法。这实际上正是static方法的主要用途。”

这段话虽然只是说明了static方法的特殊之处，但是可以看出static关键字的基本作用，简而言之，一句话来描述就是：

方便在没有创建对象的情况下来进行调用（方法/变量）。

很显然，被static关键字修饰的方法或者变量不需要依赖于对象来进行访问，只要类被加载了，就可以通过类名去进行访问。

static可以用来修饰类的成员方法、类的成员变量，另外可以编写static代码块来优化程序性能。

- 1.static方法

static方法一般称作静态方法，由于静态方法不依赖于任何对象就可以进行访问，因此对于静态方法来说，是没有this的，因为它不依附于任何对象，既然都没有对象，就谈不上this了。并且由于这个特性，在静态方法中不能访问类的非静态成员变量和非静态成员方法，因为非静态成员方法/变量都是必须依赖具体的对象才能够被调用。

但是要注意的是，虽然在静态方法中不能访问非静态成员方法和非静态成员变量，但是在非静态成员方法中是可以访问静态成员方法/变量的。

如果说想在不创建对象的情况下调用某个方法，就可以将这个方法设置为static。我们最常见的static方法就是main方法，至于为什么main方法必须是static的，现在就很清楚了。因为程序在执行main方法的时候没有创建任何对象，因此只有通过类名来访问。

另外记住，关于构造器是否是static方法可参考：http://blog.csdn.net/qq_17864929/article/details/48006835。

- 2.static变量

static变量也称作静态变量，静态变量和非静态变量的区别是：静态变量被所有的对象所共享，在内存中只有一个副本，它当且仅当在类初次加载时会被初始化。而非静态变量是对象所拥有的，在创建对象的时候被初始化，存在多个副本，各个对象拥有的副本互不影响。

static成员变量的初始化顺序按照定义的顺序进行初始化。

- 3.static代码块

static关键字还有一个比较关键的作用就是 用来形成静态代码块以优化程序性能。static块可以置于类中的任何地方，类中可以有多个static块。在类初次被加载的时候，会按照static块的顺序来执行每个static块，并且只会执行一次。

为什么说static块可以用来优化程序性能，是因为它的特性:只会在类加载的时候执行一次。

## 误区

- 1.static关键字会改变类中成员的访问权限吗？

在Java中能够影响到访问权限的只有private、public、protected（包括包访问权限）这几个关键字。例如：

```java
public class Main {
	public static void main(String[] args) {
		System.out.println(Person.name);
		System.out.println(Person.age);
	}

}
public class Person {

	public static String name = "M";
	private static int age = 10;
}
```

会提示 age 是 not visible。

- 2.能通过this访问静态成员变量吗？

虽然对于静态方法来说没有this，那么在非静态方法中能够通过this访问静态成员变量吗？先看下面的一个例子，这段代码输出的结果是什么？

```java
public class Main {　　
    static int value = 33;
 
    public static void main(String[] args) throws Exception{
        new Main().printValue();
    }
 
    private void printValue(){
        int value = 3;
        System.out.println(this.value);
    }
}
```

输出：

```java
33
```

这里面主要考察队this和static的理解。this代表什么？this代表当前对象，那么通过new Main()来调用printValue的话，当前对象就是通过new Main()生成的对象。而static变量是被对象所享有的，因此在printValue中的this.value的值毫无疑问是33。在printValue方法内部的value是局部变量，根本不可能与this关联，所以输出结果是33。在这里永远要记住一点：静态成员变量虽然独立于对象，但是不代表不可以通过对象去访问，所有的静态方法和静态变量都可以通过对象访问（只要访问权限足够）。

> 声明：本站采用开放的[知识共享署名-非商业性使用-相同方式共享 许可协议](https://creativecommons.org/licenses/by-nc-sa/3.0/deed.zh)进行许可。
