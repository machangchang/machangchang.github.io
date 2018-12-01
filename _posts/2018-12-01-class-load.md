---
layout: post
title: Java 类加载机制
categories: Java
description: Java
keywords: Java
---

Java 类加载机制。

关于 JVM 的加载机制，这里不再赘述。主要讲下**准备**和**初始化**阶段。

## 准备

在准备阶段，JVM 只会为「类变量」分配内存。在准备阶段，JVM 会为类变量分配内存，并为其初始化。但是这里的初始化指的是为变量赋予 Java 语言中该数据类型的零值，而不是用户代码里初始化的值。但如果一个变量是常量（被 static final 修饰）的话，那么在准备阶段，属性便会被赋予用户希望的值。因为 final 修饰的变量一旦被赋值就不会再改变，所以一开始赋的就是程序设置的值。而没有被 final 修饰的类变量，其可能在**初始化**阶段或者**运行**阶段发生变化，所以就没有必要在准备阶段对它赋予用户想要的值。

## 初始化

到了初始化阶段，用户定义的 Java 程序代码才真正开始执行。一般来说当 JVM 遇到下面 5 种情况的时候会触发初始化：

- 遇到 new、getstatic、putstatic、invokestatic 这四条字节码指令时，如果类没有进行过初始化，则需要先触发其初始化。生成这4条指令的最常见的Java代码场景是：使用new关键字实例化对象的时候、读取或设置一个类的静态字段（被final修饰、已在编译器把结果放入常量池的静态字段除外）的时候，以及调用一个类的静态方法的时候。
- 使用 java.lang.reflect 包的方法对类进行反射调用的时候，如果类没有进行过初始化，则需要先触发其初始化。
- 当初始化一个类的时候，如果发现其父类还没有进行过初始化，则需要先触发其父类的初始化。
- 当虚拟机启动时，用户需要指定一个要执行的主类（包含main()方法的那个类），虚拟机会先初始化这个主类。
- 当使用 JDK1.7 动态语言支持时，如果一个 java.lang.invoke.MethodHandle实例最后的解析结果 REF_getstatic,REF_putstatic,REF_invokeStatic 的方法句柄，并且这个方法句柄所对应的类没有进行初始化，则需要先出触发其初始化。

## 示例

```java
public class Book {
    public static void main(String[] args)
    {
        System.out.println("Hello ShuYi.");
    }

    Book()
    {
        System.out.println("书的构造方法");
        System.out.println("price=" + price +",amount=" + amount);
    }

    {
        System.out.println("书的普通代码块");
    }

    int price = 110;

    static
    {
        System.out.println("书的静态代码块");
    }

    static int amount = 112;
}
```

输出：

```java
书的静态代码块
Hello ShuYi.
```

下面我们来简单分析一下，首先根据上面说到的触发初始化的5种情况的第4种（当虚拟机启动时，用户需要指定一个要执行的主类（包含main()方法的那个类），虚拟机会先初始化这个主类），我们会进行类的初始化。

在我们代码中，我们只知道有一个构造方法，但实际上Java代码编译成字节码之后，是没有构造方法的概念的，只有类初始化方法 和 对象初始化方法 。

那么这两个方法是怎么来的呢？

- 类初始化方法。编译器会按照其出现顺序，收集类变量的赋值语句、静态代码块，最终组成类初始化方法。类初始化方法一般在类初始化的时候执行。

上面的这个例子，其类初始化方法就是下面这段代码了：

```java
static
    {
        System.out.println("书的静态代码块");
    }
    static int amount = 112;
```

- 对象初始化方法。编译器会按照其出现顺序，收集成员变量的赋值语句、普通代码块，最后收集构造函数的代码，最终组成对象初始化方法。对象初始化方法一般在实例化类对象的时候执行。

```java
{
        System.out.println("书的普通代码块");
    }
    int price = 110;
    System.out.println("书的构造方法");
    System.out.println("price=" + price +",amount=" + amount);
```

类初始化方法 和 对象初始化方法 之后，我们再来看这个例子，我们就不难得出上面的答案了。

但细心的朋友一定会发现，其实上面的这个例子其实没有执行对象初始化方法。

因为我们确实没有进行 Book 类对象的实例化。如果你在 main 方法中增加 new Book() 语句，你会发现对象的初始化方法执行了！

接下来看一个更复杂的例子：

```java
class Grandpa
{
    static
    {
        System.out.println("爷爷在静态代码块");
    }
}    
class Father extends Grandpa
{
    static
    {
        System.out.println("爸爸在静态代码块");
    }

    public static int factor = 25;

    public Father()
    {
        System.out.println("我是爸爸~");
    }
}
class Son extends Father
{
    static 
    {
        System.out.println("儿子在静态代码块");
    }

    public Son()
    {
        System.out.println("我是儿子~");
    }
}
public class InitializationDemo
{
    public static void main(String[] args)
    {
        System.out.println("爸爸的岁数:" + Son.factor);  //入口
    }
}
```

输出：

```java
爷爷在静态代码块
爸爸在静态代码块
爸爸的岁数:25
```

也许会有人问为什么没有输出「儿子在静态代码块」这个字符串？

这是因为对于静态字段，只有直接定义这个字段的类才会被初始化（执行静态代码块）。因此通过其子类来引用父类中定义的静态字段，只会触发父类的初始化而不会触发子类的初始化。

对面上面的这个例子，我们可以从入口开始分析一路分析下去：

- 首先程序到 main 方法这里，使用标准化输出 Son 类中的 factor 类成员变量，但是 Son 类中并没有定义这个类成员变量。于是往父类去找，我们在 Father 类中找到了对应的类成员变量，于是触发了 Father 的初始化。
- 但根据我们上面说到的初始化的 5 种情况中的第 3 种（当初始化一个类的时候，如果发现其父类还没有进行过初始化，则需要先触发其父类的初始化）。我们需要先初始化 Father 类的父类，也就是先初始化 Grandpa 类再初始化 Father 类。于是我们先初始化 Grandpa 类输出：「爷爷在静态代码块」，再初始化 Father 类输出：「爸爸在静态代码块」。
- 最后，所有父类都初始化完成之后，Son 类才能调用父类的静态变量，从而输出：「爸爸的岁数：25」。

接下来看一个更复杂的例子：

```java
class Grandpa
{
    static
    {
        System.out.println("爷爷在静态代码块");
    }

    public Grandpa() {
        System.out.println("我是爷爷~");
    }
}
class Father extends Grandpa
{
    static
    {
        System.out.println("爸爸在静态代码块");
    }

    public Father()
    {
        System.out.println("我是爸爸~");
    }
}
class Son extends Father
{
    static 
    {
        System.out.println("儿子在静态代码块");
    }

    public Son()
    {
        System.out.println("我是儿子~");
    }
}
public class InitializationDemo
{
    public static void main(String[] args)
    {
        new Son();  //入口
    }
}
```

输出：

```java
爷爷在静态代码块
爸爸在静态代码块
儿子在静态代码块
我是爷爷~
我是爸爸~
我是儿子~
```

让我们仔细来分析一下上面代码的执行流程：

- 首先在入口这里我们实例化一个 Son 对象，因此会触发 Son 类的初始化，而 Son 类的初始化又会带动 Father 、Grandpa 类的初始化，从而执行对应类中的静态代码块。因此会输出：「爷爷在静态代码块」、「爸爸在静态代码块」、「儿子在静态代码块」。
- 当 Son 类完成初始化之后，便会调用 Son 类的构造方法，而 Son 类构造方法的调用同样会带动 Father、Grandpa 类构造方法的调用，最后会输出：「我是爷爷~」、「我是爸爸~」、「我是儿子~」。

最后看一个特殊的例子：

```java
public class Book {
    public static void main(String[] args)
    {
        staticFunction();
    }

    static Book book = new Book();

    static
    {
        System.out.println("书的静态代码块");
    }

    {
        System.out.println("书的普通代码块");
    }

    Book()
    {
        System.out.println("书的构造方法");
        System.out.println("price=" + price +",amount=" + amount);
    }

    public static void staticFunction(){
        System.out.println("书的静态方法");
    }

    int price = 110;
    static int amount = 112;
}
```

输出：

```java
书的普通代码块
书的构造方法
price=110,amount=0
书的静态代码块
书的静态方法
```

下面我们一步步来分析一下代码的整个执行流程。

在上面两个例子中，因为 main 方法所在类并没有多余的代码，我们都直接忽略了 main 方法所在类的初始化。

但在这个例子中，main 方法所在类有许多代码，我们就并不能直接忽略了。

- 当 JVM 在准备阶段的时候，便会为类变量分配内存和进行初始化。此时，我们的 book 实例变量被初始化为 null，amount 变量被初始化为 0。
- 当进入初始化阶段后，因为 Book 方法是程序的入口，根据我们上面说到的类初始化的五种情况的第四种（当虚拟机启动时，用户需要指定一个要执行的主类（包含main()方法的那个类），虚拟机会先初始化这个主类）。所以JVM 会初始化 Book 类，即执行类构造器 。
- JVM 对 Book 类进行初始化首先是执行类构造器（按顺序收集类中所有静态代码块和类变量赋值语句就组成了类构造器 ），后执行对象的构造器（按顺序收集成员变量赋值和普通代码块，最后收集对象构造器，最终组成对象构造器 ）。
对于 Book 类，其类构造方法（）可以简单表示如下：

```java
static Book book = new Book();
static
{
    System.out.println("书的静态代码块");
}
static int amount = 112;
```

于是首先执行`static Book book = new Book();`这一条语句，这条语句又触发了类的实例化。于是 JVM 执行对象构造器 ，收集后的对象构造器 代码：

```java
{
    System.out.println("书的普通代码块");
}
int price = 110;
Book()
{
    System.out.println("书的构造方法");
    System.out.println("price=" + price +", amount=" + amount);
}
```

于是此时 price 赋予 110 的值，输出：「书的普通代码块」、「书的构造方法」。而此时 price 为 110 的值，而 amount 的赋值语句并未执行，所以只有在准备阶段赋予的零值，所以之后输出「price=110,amount=0」。

当类实例化完成之后，JVM 继续进行类构造器的初始化：

```java
static Book book = new Book();  //完成类实例化
static
{
    System.out.println("书的静态代码块");
}
static int amount = 112;
```

即输出：「书的静态代码块」，之后对 amount 赋予 112 的值。

- 到这里，类的初始化已经完成，JVM 执行 main 方法的内容。

```java
public static void main(String[] args)
{
    staticFunction();
}
```

即输出：「书的静态方法」。

## 总结

从上面几个例子可以看出，分析一个类的执行顺序大概可以按照如下步骤：

1. 确定类变量的初始值。在类加载的准备阶段，JVM 会为类变量初始化零值，这时候类变量会有一个初始的零值。如果是被 final 修饰的类变量，则直接会被初始成用户想要的值。
2. 初始化入口方法。当进入类加载的初始化阶段后，JVM 会寻找整个 main 方法入口，从而初始化 main 方法所在的整个类。当需要对一个类进行初始化时，会首先初始化类构造器（），之后初始化对象构造器（）。
3. 初始化类构造器。JVM 会按顺序收集类变量的赋值语句、静态代码块，最终组成类构造器由 JVM 执行。
4. 初始化对象构造器。JVM 会按照收集成员变量的赋值语句、普通代码块，最后收集构造方法，将它们组成对象构造器，最终由 JVM 执行。
5. 如果在初始化 main 方法所在类的时候遇到了其他类的初始化，那么就先加载对应的类，加载完成之后返回。如此反复循环，最终返回 main 方法所在类。

> 参考链接：http://www.cnblogs.com/chanshuyi/p/the_java_class_load_mechamism.html

> 声明：本站采用开放的[知识共享署名-非商业性使用-相同方式共享 许可协议](https://creativecommons.org/licenses/by-nc-sa/3.0/deed.zh)进行许可。
