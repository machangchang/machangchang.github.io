---
layout: post
title: Java异常
categories: Java
description: 异常
keywords: Exception
---

异常。

## 1. 异常简介

异常的英文单词是exception，字面翻译就是“意外、例外”的意思，也就是非正常情况。事实上，异常本质上是程序上的错误，包括程序逻辑错误和系统错误。比如使用空的引用、数组下标越界、内存溢出错误等，这些都是意外的情况，背离我们程序本身的意图。错误在我们编写程序的过程中会经常发生，包括编译期间和运行期间的错误，在编译期间出现的错误有编译器帮助我们一起修正，然而运行期间的错误便不是编译器力所能及了，并且运行期间的错误往往是难以预料的。

假若程序在运行期间出现了错误，如果置之不理，程序便会终止或直接导致系统崩溃，显然这不是我们希望看到的结果。因此，如何对运行期间出现的错误进行处理和补救呢？Java提供了异常机制来进行处理，通过异常机制来处理程序运行期间出现的错误。通过异常机制，我们可以更好地提升程序的健壮性。

在Java中异常被当做对象来处理，根类是`java.lang.Throwable`类，在Java中定义了很多异常类（如`OutOfMemoryError`、`NullPointerException`、`IndexOutOfBoundsException`等），这些异常类分为两大类：**Error**和**Exception**。

Error是无法处理的异常，比如OutOfMemoryError，一般发生这种异常，JVM会选择终止程序。因此我们编写程序时不需要关心这类异常。

Exception，也就是我们经常见到的一些异常情况，比如NullPointerException、IndexOutOfBoundsException，这些异常是我们可以处理的异常。

Exception类的异常包括checked exception和unchecked exception（unchecked exception也称运行时异常RuntimeException，当然这里的运行时异常并不是前面我所说的运行期间的异常，只是Java中用运行时异常这个术语来表示，Exception类的异常都是在**运行期间**发生的）。

**unchecked exception**（非检查异常），也称运行时异常（RuntimeException），比如常见的NullPointerException、IndexOutOfBoundsException。对于运行时异常，java编译器不要求必须进行异常捕获处理或者抛出声明，由程序员自行决定。

**checked exception**（检查异常），也称非运行时异常（运行时异常以外的异常就是非运行时异常），java编译器强制程序员必须进行捕获处理，比如常见的IOExeption和SQLException。对于非运行时异常如果不进行捕获或者抛出声明处理，编译都不会通过。

在Java中，异常类的结构层次图如下图所示：

![](/images/blog/2018-08-05-Exception/Exception_001.jpg)

在Java中，所有异常类的父类是Throwable类，Error类是error类型异常的父类，Exception类是exception类型异常的父类，RuntimeException类是所有运行时异常的父类，RuntimeException以外的并且继承Exception的类是非运行时异常。

典型的`RuntimeException`包括`NullPointerException`、`IndexOutOfBoundsException`、`IllegalArgumentException`等。

典型的`非RuntimeException`包括`IOException`、`SQLException`等。

## 2. Java是如何处理异常的

在Java中如果需要处理异常，必须先对异常进行捕获，然后再对异常情况进行处理。如何对可能发生异常的代码进行异常捕获和处理呢？使用try和catch关键字即可，如下面一段代码所示：

```java
try {
  File file = new File("d:/a.txt");
  if(!file.exists())
    file.createNewFile();
} catch (IOException e) {
  // TODO: handle exception
}
```

被try块包围的代码说明这段代码可能会发生异常，一旦发生异常，异常便会被catch捕获到，然后需要在catch块中进行异常处理。

　　这是一种处理异常的方式。在Java中还提供了另一种异常处理方式即抛出异常，顾名思义，也就是说一旦发生异常，我把这个异常抛出去，让调用者去进行处理，自己不进行具体的处理，此时需要用到throw和throws关键字。　

**throw**

```java
public static void main(String[] args) {
		String s = "abc";
		if(s.equals("abc")) {
			throw new NumberFormatException();
		} else {
			System.out.println(s);
		}
}
```

**throws**

```java
public class TestException {

    public static void f () throws NumberFormatException {

        String s = "abc";
        System.out.println(Double.parseDouble(s));
    }

    public static void main(String[] args) {

        try {
            f();
        } catch (NumberFormatException e) {
            System.out.println("异常！！！");
        }
    }
}
```

**throw和throws的比较**

1. throws出现在方法函数头；而throw出现在函数体。

2. throws表示出现异常的一种可能性，并不一定会发生这些异常；throw则是抛出了异常，执行throw则一定抛出了某种异常对象。

3. 两者都是消极处理异常的方式（这里的消极并不是说这种方式不好），只是抛出或者可能抛出异常，但是不会由函数去处理异常，真正的处理异常由函数的上层调用处理。
 
也就说在Java中进行异常处理的话，对于可能会发生异常的代码，可以选择三种方法来进行异常处理：

1. 对代码块用try..catch进行异常捕获处理。
2. 在 该代码的方法体外用throws进行抛出声明，告知此方法的调用者这段代码可能会出现这些异常，你需要谨慎处理。此时有两种情况：如果声明抛出的异常是非运行时异常，此方法的调用者必须显示地用try..catch块进行捕获或者继续向上层抛出异常。如果声明抛出的异常是运行时异常，此方法的调用者可以选择地进行异常捕获处理。
3. 在代码块用throw手动抛出一个异常对象，此时也有两种情况，跟2中的类似：如果抛出的异常对象是非运行时异常，此方法的调用者必须显示地用try..catch块进行捕获或者继续向上层抛出异常。如果抛出的异常对象是运行时异常，此方法的调用者可以选择地进行异常捕获处理。

（如果最终将异常抛给main方法，则相当于交给JVM自动处理，此时JVM会简单地打印异常信息）。

## 3. try,catch,finally

try关键字用来包围可能会出现异常的逻辑代码，它单独无法使用，必须配合catch或者finally使用。Java编译器允许的组合使用形式只有以下三种形式：

try...catch...;       try....finally......;    try....catch...finally...

当然catch块可以有多个，注意try块只能有一个,finally块是可选的（但是最多只能有一个finally块）。

三个块执行的顺序为try—>catch—>finally。

当然如果没有发生异常，则catch块不会执行。但是finally块无论在什么情况下都是会执行的（这点要非常注意，因此部分情况下，都会将释放资源的操作放在finally块中进行）。

在有多个catch块的时候，是按照catch块的先后顺序进行匹配的，一旦异常类型被一个catch块匹配，则不会与后面的catch块进行匹配。

在使用try..catch..finally块的时候，注意千万不要在finally块中使用return，因为finally中的return会覆盖已有的返回值。

## 4. 类继承时的异常

1. 父类的方法没有声明异常，子类在重写该方法的时候不能声明异常。

2. 如果父类的方法声明一个异常exception1，则子类在重写该方法的时候声明的异常不能是exception1的父类。

3. 如果父类的方法声明的异常类型只有非运行时异常（运行时异常），则子类在重写该方法的时候声明的异常也只能有非运行时异常（运行时异常），不能含有运行时异常（非运行时异常）。

> 声明：本站采用开放的[知识共享署名-非商业性使用-相同方式共享 许可协议](https://creativecommons.org/licenses/by-nc-sa/3.0/deed.zh)进行许可。
