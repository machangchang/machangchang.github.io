---
layout: post
title: (译)Strings, Literally
categories: Java
description: String
keywords: String
---

在这个月内，我会对 String literals （字面量） 进行研究以及分析在 Java 中是如何处理它们的。如果你阅读过[last month's SCJP Tip Line article](https://javaranch.com/journal/200408/Journal200408.jsp#a1) ，你就会发现这篇文章是关于Strings的一个很好的后续。如果你还没有读过那篇文章，那也没有什么大问题，我会在这篇文章中尽可能的去包含足够的信息以便让你加速理解。String literals 在 Java 中有点被特殊对待。在大多数情况下，

让我们首先抛出一个简单的问题，什么是一个 String Literal？ 一个 String literal 是被**引号**包围的字符序列，比如 "string" 或者 "literal" 。你可能已经在你的程序中多次使用 String literal 了，但你可能还没有意识到 String literal 在 Java 中有多么特殊。

## String 是不可变的

那么到底是什么导致 String literal 如此特殊？首先，知道 String objects 是不可变的。这就意味着一旦创建完成，一个 String object 就不会被改变（除了很少使用的用反射去获取私有数据）。你说是不可变的，那下面这段代码呢？

```java
public class ImmutableStrings
{
    public static void main(String[] args)
    {
        String start = "Hello";
        String end = start.concat(" World!");
        System.out.println(end);
    }
}

// Output

Hello World!
```

好吧，这样看来 String 是被改变了还是没改变？在这段代码中，没有 String object 被改变。我们首先为变量 start 分配了"Hello"，这样就会在堆中创建一个 String 对象以及一个存储在 start 中的引用。接下来，我们在这个对象上调用 concat(String) 方法，如果我们查看 [API Spec for String](https://docs.oracle.com/javase/8/docs/api/java/lang/String.html)，你会看到这些关于 concat(String) 的描述：

> Concatenates the specified string to the end of this string. 
If the length of the argument string is 0, then this String object is returned. **Otherwise, a new String object is created**, representing a character sequence that is the concatenation of the character sequence represented by this String object and the character sequence represented by the argument string. 
Examples: 
    "cares".concat("s") returns "caress"
    "to".concat("get").concat("her") returns "together" 
Parameters:
    str - the String that is concatenated to the end of this String. 
Returns:
    a string that represents the concatenation of this object's characters followed by the string argument's characters.

注意上面加粗的地方，当你将一个 String 和另一个拼接时，实际上它并不会改变该 String，它只是创建了一个新的 String 将原 String 和 其他 String 拼接起来，这就是上面代码所做的。被局部变量 start 所引用的 String object 从没改变，如果你加上一行代码 `System.out.println(start);` ，你会发现 start 依然引用的是"Hello"，而且'+'做的事和和 concat() 方法所作的一样。

Strings 确实是不变的。

## Storage of Strings - The String Literal Pool

什么是 String Literal Pool ？大多数情况下，很多人认为它是 collection of String obejct 。虽然这种说法很接近，但并不是完全正确。实际上，it's a collection of references to String objects 。Strings，尽管是不变的，但它和 Java 中的其他对象一样，对象是被创建在堆中， Strings 也不例外。**So, Strings that are part of the "String Literal Pool" still live on the heap, but they have references to them from the String Literal Pool.**

到现在为止我们并没有解释什么是 pool，或者它有什么用，这是因为 String objects 是不可变的，多个引用可以安全的共享同一个 String object。看下面的例子：

```java
public class ImmutableStrings
{
    public static void main(String[] args)
    {
        String one = "someString";
        String two = "someString";
        
        System.out.println(one.equals(two));
        System.out.println(one == two);
    }
}

// Output

true
true
``` 

在这个例子中，我们没有必要去创建两个完全相同的 String object。如果 String object 可以被改变，比如 一个 StringBuffer 可以被改变，我们就不得不去创建两个单独的对象。但是，就我们所知道的，String objects 不能够被改变，我们可以安全的在两个 String references 分享一个 String object，这是通过 String literal pool完成的，以下是它的实现方式：

将 .java 文件编译成 .class 文件时，任何 String lirerals 都会被以一种特殊的方式所记录，就和所有的常量一样。当一个 class 文件被加载时（注意，加载发生在初始化之前），JVM 会遍历类的代码查找 String literals。当找到一个时，它会检查是否已经从堆中引用了相同的 String。如果没有，它会在堆上创建一个 String 对象，并在 constant table 中保存一个该对象的引用（**译者注：在HotSpot VM 中，String Literal Pool 是通过  StringTable 来实现的，它是一个哈希表，存的是字面量和对象的引用**）。*Once a reference is made to that String object, any references to that String literal throughout your program are simply replaced with the reference to the object referenced from the String Literal Pool. *（**译者注：这段话翻译起来有点别扭，这里说下译者的理解，一旦该对象有了一个引用，那整个程序中对这个 String literal 的所有引用都会被替换为在 String Literal Pool 中该 String literal 所对应的引用**）

因此，在上面的例子中，只会有一个 entry 在 String Literal Pool 中，它会和内容是 "someString" 的 String 对象所关联。变量 one 和 two 都将会被分配同一个引用，也就是这个 String object 的引用。你可以通过上面例子的输出来确认这个事实。equals() 方法会检查 String object 是否包含相同的内容，== 操作符用在object上时，用于检查引用的相同性 - 这意味着两个引用变量引用的是同一个 object 时会返回 true，这种情况下，引用是相等的。通过上面例子的输出，你会发现 one 和 two 引用的 String 不仅内容相同，而且对象相同。

objects 和 references 的关系如下图所示：

![](/images/blog/2018-09-23-String-Literal-Pool/String_Literal_Pool_001.jpg)

注意，String Literals 有一个特殊的操作，使用 "new" 关键字构造 Strings 意味着不同的操作，看下面的例子：

```java
public class ImmutableStrings
{
    public static void main(String[] args)
    {
        String one = "someString";
        String two = new String("someString");
        
        System.out.println(one.equals(two));
        System.out.println(one == two);
    }
}

// Output

true
false
```

在这个例子中，我们确实因为 "new" 关键字得到了不同的结果，在这种情况下，和两个 String literals 关联的引用依然会被放进 constant table (the String Literal Pool)，但是，当你使用"new"关键字时，the JVM is obliged to create a new String object at run-time, rather than using the one from the constant table。

在这种情况下，尽管两个 String 引用指向的 String objects 包含的内容是相同的，但指向了不同的对象，这可以从例子的输出结果可以看出来。equals()返回的是 true， == 操作符返回的是 false，表明String objects是不同的。

如下图所示，注意，String Literal Pool 中引用所指向的 String object 是在 class 被加载时创建的，而另一个 String object 是当"new String..."执行时在 runtime 被创建的。

![](/images/blog/2018-09-23-String-Literal-Pool/String_Literal_Pool_002.jpg)

如果你想让这两个本地变量指向同一个对象，你可以使用 String 中定义的 intern() 方法。调用 two.intern() 会去查找在 String Literal Pool 中的引用所指向的 String 对象，该对象的内容和调用 intern() 方法的对象的内容一样。如果找到一个，则返回该对象的引用，并将其赋值给你的本地变量。如果你这样做了，你就会得到一个和上面类似的图，两个本地变量，one 和 two，指向同一个 String object，这个对象也被 String Literal Pool 中的引用所指向。*此时，在运行时创建的第二个 String Object 将有资格进行垃圾回收*。

## Garbage Collection

什么情况下 an object 有资格进行垃圾回收？当不再有程序的活动部分的引用指向该 object 时，它就有资格被回收。有人发现 String Literal 的垃圾回收有什么特别之处吗？看下面的程序。

```java
public class ImmutableStrings
{
    public static void main(String[] args)
    {
        String one = "someString";
        String two = new String("someString");
        
        one = two = null;
    }
}
```

在 main() 方法结束之前，有多少 objects 可用于垃圾回收？0？1？2？

答案是：1。

不同于大多数的 objects，String Literal Pool 中的 String literal 一直指向它们，这意味着它们一直被引用，所以不符合垃圾回收的条件。这个例子和我上面使用的相同，所以你可以看到下面的图片也是相似的，一旦给变量 one 和 two 赋值为 null，我们最终得到的图片如下：

![](/images/blog/2018-09-23-String-Literal-Pool/String_Literal_Pool_003.jpg)
 
 正如你所看到的，尽管我们的本地变量 one 和two 都没有指向 String object，但仍然有一个 String Literal Pool 里的引用指向它。所以，这个 object 仍然是不可被垃圾回收的。通过 intern() 方法始终可以访问该 object，如前所述。

## Conclusion

- 相同的 String literals (即使它们存储在不同包内的不同classes 中)会关联到同一个 String object。
- 一般来说，String literals 永远不符合垃圾回收的条件。
- run-time 创建的 Strings 总是和从 String literals 创建的不同。
- 你可以在 run-time Strings 通过 intern() 方法重用String literals。
-  检查 String 相等的最好方法是使用 equals() 方法。

> 原文链接：https://javaranch.com/journal/200409/ScjpTipLine-StringsLiterally.html

> 声明：本站采用开放的[知识共享署名-非商业性使用-相同方式共享 许可协议](https://creativecommons.org/licenses/by-nc-sa/3.0/deed.zh)进行许可。
