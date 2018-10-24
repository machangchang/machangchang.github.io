---
layout: post
title: Java 方法区
categories: JVM
description: Method Area
keywords: Method Area
---

The Method Area

本文是篇译文，不过首先我摘抄了维基百科上的一段话，有助于理解该译文。

引用自维基百科：

> In casual use, people often refer to the "class" of an object, but narrowly speaking objects have type: the interface, namely the types of member variables, the signatures of member functions (methods), and properties these satisfy. At the same time, a class has an implementation (specifically the implementation of the methods), and can create objects of a given type, with a given implementation.[3] In the terms of type theory, a class is an implementation‍—‌a concrete data structure and collection of subroutines‍—‌while a type is an interface. Different (concrete) classes can produce objects of the same (abstract) type (depending on type system); for example, the type Stack might be implemented with two classes – SmallStack (fast for small stacks, but scales poorly) and ScalableStack (scales well but high overhead for small stacks). Similarly, a given class may have several different constructors.
Types generally represent nouns, such as a person, place or thing, or something nominalized, and a class represents an implementation of these. For example, a Banana type might represent the properties and functionality of bananas in general, while the ABCBanana and XYZBanana classes would represent ways of producing bananas (say, banana suppliers or data structures and functions to represent and draw bananas in a video game). The ABCBanana class could then produce particular bananas: instances of the ABCBanana class would be objects of type Banana. Often only a single implementation of a type is given, in which case the class name is often identical with the type name.

以下为正文翻译：

在 Java 虚拟机内部，已经被加载的类型的信息是被存储在成为方法区的内存的逻辑区域中。当 Java 虚拟机加载类型时，它使用类加载器来定位相应的类文件。类加载器读入类文件-二进制的线性流-并将其传递给虚拟机。虚拟机从二进制数据中提取类型信息，并将其存储在方法区中。类中声明的类（静态）变量也取自方法区中。

Java 虚拟机内部实现表示类型信息的方式是由设计者决定的。比如，类文件中的多字节是以big-endian（最重要的字节优先）顺序存储的。但是，当数据导入方法区时，虚拟机可以以任何方式存储数据。如果实现位于little-endian 处理器上，则设计人员可能决定以 little-endian 顺序在方法区上储存多字节值。

当虚拟机执行程序时，它将搜索并使用存储在方法区中的类型信息。设计人员必须尝试设计有助于快速执行 Java 应用程序的数据结构，但也必须考虑紧凑性。如果设计一个可以在低内存限制下运行的实现，设计人员可能决定牺牲一些执行速度来支持紧凑性。另一方面，如果设计将在虚拟机存储系统上运行的实现，则设计者可以决定在方法区中存储冗余信息以便于执行速度（如果底层主机不提供虚拟内存，但提供硬盘，则设计人员可以创建自己的虚拟内存系统作为其实现的一部分）。设计人员可以根据需求选择他们认为优化其实施性能的任何数据结构和组织。

所有线程共享相同的方法区域，因此必须将对方法区域的数据结构的访问设计为线程安全的。例如，如果两个线程试图找到一个名为Lava的类，并且尚未加载Lava，则只允许一个线程加载它而另一个线程等待。

方法区域的大小不是固定的。在Java应用程序运行时，虚拟机可以扩展和收缩方法区域以满足应用程序的需要。此外，方法区域的内存不需要是连续的。它可以在堆上分配 - 甚至在虚拟机自己的堆上。实现可以允许用户或程序员指定方法区域的初始大小，以及最大或最小大小。

方法区域也可以被垃圾回收。因为Java程序可以通过用户定义的类加载器动态扩展，所以类可以被应用程序“取消引用”。如果类未被引用，则Java虚拟机可以卸载该类（垃圾收集它）以使方法区域占用的内存保持最小。

## 类型信息（Type information）

对于它加载的每种类型，Java虚拟机必须在方法区域中存储以下类型的信息：

- The fully qualified name of the type
- The fully qualified name of the type's direct superclass (unless the type is an interface or class java.lang.Object, neither of which have a superclass)
- Whether or not the type is a class or an interface
- The type's modifiers ( some subset of` public, abstract, final)
- An ordered list of the fully qualified names of any direct superinterfaces

在Java类文件和Java虚拟机中，类型名称始终存储为完全限定名称。 在Java源代码中，完全限定名称是类型包的名称，加上一个点，加上类型的简单名称。 例如，包java.lang中的类Object的完全限定名称是java.lang.Object。 在类文件中，点用斜杠替换，如在java / lang / Object中。 在方法区域中，完全限定名称可以用设计者选择的任何形式和数据结构表示。

除了前面列出的基本类型信息之外，虚拟机还必须为每个加载的类型存储以下内容：

- The constant pool for the type
- Field information
- Method information
- All class (static) variables declared in the type, except constants
- A reference to class ClassLoader
- A reference to class Class

下面会分别介绍这些内容。

## 常量池（The Constant Pool）

对于它加载每个类型，Java虚拟机必须保存一个常量池。 常量池是类型使用的有序常量集，包括文字（字符串，整数和浮点常量）以及对类型，字段和方法的符号引用。 常量池中的条目由索引引用，非常类似于数组的元素。 因为它包含对类型使用的所有类型，字段和方法的符号引用，所以常量池在Java程序的动态链接中起着核心作用。

## 字段信息（Field Information）

对于在类型中声明的每个字段，必须将以下信息存储在方法区域中。 除了每个字段的信息之外，还必须记录类或接口声明字段的顺序。 这是字段列表：

- The field's name
- The field's type
- The field's modifiers (some subset of public, private, protected, static, final, volatile, transient)

## 方法信息（Method Information）

对于在类型中声明的每个方法，必须将以下信息存储在方法区域中。 与字段一样，必须记录类或接口声明方法的顺序以及数据。 这是列表：

- The method's name
- The method's return type (or void)
- The number and types (in order) of the method's parameters
- The method's modifiers (some subset of public, private, protected, static, final, synchronized, native, abstract)

除了前面列出的项目之外，还必须将每个非抽象或本机方法存储以下信息：

- The method's bytecodes
- The sizes of the operand stack and local variables sections of the method's stack frame (these are described in a later section of this chapter)
- An exception table (this is described in Chapter 17, "Exceptions")

## 类变量（Class Variables）

类变量在类的所有实例之间共享，即使在没有任何实例的情况下也可以访问。 这些变量与类相关联 - 而不是与类的实例相关联 - 因此它们在逻辑上是方法区域中类数据的一部分。 在Java虚拟机使用类之前，它必须在方法区域中为类中声明的每个非最终类变量分配内存。

常量（声明为final的类变量）的处理方式与非final类变量的处理方式不同。 每个使用final类变量的类型都会在其自己的常量池中获取常量值的副本。 作为常量池的一部分，最终类变量存储在方法区域中 - 就像非最终类变量一样。 但是，非final类变量作为声明它们的类型的数据的一部分存储，而最终类变量作为使用它们的任何类型的数据的一部分存储。

## 类加载器的引用（A Reference to Class ClassLoader）

对于它加载的每种类型，Java虚拟机必须跟踪是否通过引导类加载器或用户定义的类加载器加载了类型。 对于通过用户定义的类加载器加载的那些类型，虚拟机必须存储对加载该类型的用户定义的类加载器的引用。 此信息存储为方法区域中类型数据的一部分。

虚拟机在动态链接期间使用此信息。 当一种类型引用另一种类型时，虚拟机从加载引用类型的同一类加载器请求引用的类型。 这种动态链接过程也是虚拟机形成单独名称空间的核心。 为了能够正确执行动态链接并维护多个名称空间，虚拟机需要知道哪个类加载器在其方法区域中加载了每个类型。

## Class的引用（A Reference to Class Class）

Java虚拟机为其加载的每种类型创建类java.lang.Class的实例。 虚拟机必须以某种方式将对类型实例的引用与方法区域中类型的数据相关联。

您的Java程序可以获取和使用对Class对象的引用。 类Class中的一个静态方法允许您获取对任何已加载类的Class实例的引用：

```java
// A method declared in class java.lang.Class:
public static Class forName(String className);
```

例如，如果调用forName（“java.lang.Object”），您将获得对表示java.lang.Object的Class对象的引用。 如果调用forName（“java.util.Enumeration”），您将获得对java.util包中表示Enumeration接口的Class对象的引用。 您可以使用forName（）从任何包中获取任何加载类型的Class引用，只要该类型可以（或已经）加载到当前名称空间中即可。 如果虚拟机无法将请求的类型加载到当前名称空间，则forName（）将抛出ClassNotFoundException。

获取Class引用的另一种方法是在任何对象引用上调用getClass（）。 这个方法由Object类本身的每个对象继承：

```java
// A method declared in class java.lang.Object:
public final Class getClass();
```

例如，如果您对类java.lang.Integer的对象有引用，则可以通过在对Integer对象的引用上调用getClass（）来获取java.lang.Integer的Class对象。

给定Class对象的引用，您可以通过调用类Class中声明的方法来找到有关该类型的信息。 如果查看这些方法，您将很快意识到类Class使正在运行的应用程序可以访问存储在方法区域中的信息。 以下是类Class中声明的一些方法：

```java
// Some of the methods declared in class java.lang.Class:
public String getName();
public Class getSuperClass();
public boolean isInterface();
public Class[] getInterfaces();
public ClassLoader getClassLoader();
```

这些方法只返回有关加载类型的信息。 getName（）返回类型的完全限定名称。 getSuperClass（）返回类型的直接超类的Class实例。 如果类型是类java.lang.Object或接口，其中没有一个具有超类，则getSuperClass（）返回null。 如果Class对象描述了一个接口，则isInterface（）返回true;如果它描述一个接口，则返回false。 getInterfaces（）返回一个Class对象数组，每个直接超接口一个。 超接口按照类型声明为超接口的顺序出现在数组中。 如果类型没有直接的超接口，则getInterfaces（）返回一个长度为零的数组。 getClassLoader（）返回对加载此类型的ClassLoader对象的引用，如果引导类加载器加载了类型，则返回null。 所有这些信息直接来自方法区域。

## 方法表（Method Tables）

必须组织存储在方法区域中的类型信息以便快速访问。 除了先前列出的原始类型信息之外，实现还可以包括加速对原始数据的访问的其他数据结构。 这种数据结构的一个例子是方法表。 对于Java虚拟机加载的每个非抽象类，它可以生成方法表并将其作为它存储在方法区域中的类信息的一部分包含在内。 方法表是对可以在类实例上调用的所有实例方法的直接引用的数组，包括从超类继承的实例方法。 （方法表在抽象类或接口的情况下没有用，因为程序永远不会实例化这些。）方法表允许虚拟机快速定位在对象上调用的实例方法。 方法表在第8章“链接模型”中有详细描述。

## 方法区的使用案例

作为Java虚拟机如何使用它存储在方法区域中的信息的示例，请考虑以下类：

```java
// On CD-ROM in file jvm/ex2/Lava.java
class Lava {

    private int speed = 5; // 5 kilometers per hour

    void flow() {
    }
}

// On CD-ROM in file jvm/ex2/Volcano.java
class Volcano {

    public static void main(String[] args) {
        Lava lava = new Lava();
        lava.flow();
    }
}
```

以下段落描述了实现如何执行Volcano应用程序的main（）方法的字节码中的第一条指令。 Java虚拟机的不同实现可以以非常不同的方式操作。 以下描述说明了一种方法 - 但不是唯一的方法 - Java虚拟机可以执行Volcano的main（）方法的第一条指令。

要运行Volcano应用程序，请以依赖于实现的方式将名称“Volcano”提供给Java虚拟机。 鉴于名称Volcano，虚拟机会在文件Volcano.class中查找并读取。 它从导入的类文件中的二进制数据中提取Volcano类的定义，并将信息放入方法区域。 然后，虚拟机通过解释存储在方法区域中的字节码来调用main（）方法。 当虚拟机执行main（）时，它维护一个指向当前类（类Volcano）的常量池（方法区域中的数据结构）的指针。

请注意，这个Java虚拟机已经开始在类Volcano中执行main（）的字节码，即使它还没有加载类Lava。 与Java虚拟机的许多（可能是大多数）实现一样，此实现不会等到应用程序使用的所有类在开始执行main（）之前加载。 它只在需要时才加载类。

main（）的第一条指令告诉Java虚拟机为常量池条目1中列出的类分配足够的内存。 虚拟机使用其指针进入Volcano的常量池以查找条目1并找到对类Lava的符号引用。 它检查方法区域以查看是否已加载Lava。

符号引用只是一个字符串，给出了类的完全限定名称：“Lava”。 在这里，您可以看到必须组织方法区域，以便尽可能快地定位类，只给出类的完全限定名称。 实现设计人员可以选择最适合他们需求的算法和数据结构 - 哈希表，搜索树，任何东西。 类Class的静态forName（）方法可以使用相同的机制，它返回给定完全限定名称的Class引用。

当虚拟机发现它尚未加载名为“Lava”的类时，它继续查找并读入文件Lava.class。 它从导入的二进制数据中提取类Lava的定义，并将信息放入方法区域。

然后，Java虚拟机替换Volcano的常量池条目1中的符号引用，该条目只是字符串“Lava”，带有指向Lava的类数据的指针。 如果虚拟机必须再次使用Volcano的常量池条目，那么只需要一个符号引用，字符串“Lava”，就不必经历相对较慢的搜索类Lava的方法区域的过程。 它可以使用指针更快地访问Lava的类数据。 使用直接引用（在本例中为本机指针）替换符号引用的过程称为常量池分辨率。 通过搜索方法区域将符号引用解析为直接引用，直到找到引用的实体，并在必要时加载新类。

最后，虚拟机已准备好为新的Lava对象实际分配内存。 虚拟机再次查询存储在方法区域中的信息。 它使用指针（刚刚放入Volcano的常量池条目中）到Lava数据（刚刚导入到方法区域中）以找出Lava对象需要多少堆空间。

Java虚拟机总是可以通过查看存储在方法区域中的类数据来确定表示对象所需的内存量。 但是，特定对象所需的实际堆空间量取决于实现。 Java虚拟机内对象的内部表示是实现设计者的另一个决定。 本章稍后将更详细地讨论对象表示。

一旦Java虚拟机确定了Lava对象所需的堆空间量，它就会在堆上分配该空间并将实例变量speed初始化为零，即其默认初始值。 如果类Lava的超类Object具有任何实例变量，那么它们也被初始化为默认初始值。 （类和对象的初始化细节在第7章“类型的生命周期”中给出。）

main（）的第一条指令通过将对新Lava对象的引用推送到堆栈来完成。 稍后的指令将使用该引用来调用Java代码，该代码将speed变量初始化为其正确的初始值5。 另一条指令将使用该引用来调用引用的Lava对象上的flow（）方法。

> 参考链接：https://en.wikipedia.org/wiki/Class_(computer_programming)#Class_vs._type
https://www.artima.com/insidejvm/ed2/jvm5.html

> 声明：本站采用开放的[知识共享署名-非商业性使用-相同方式共享 许可协议](https://creativecommons.org/licenses/by-nc-sa/3.0/deed.zh)进行许可。
