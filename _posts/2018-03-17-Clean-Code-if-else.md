---
layout: post
title: 代码简洁之道：多态代替条件语句
categories: Java
description: Java Clean Code
keywords: Java, Clean Code
---

## 代码简洁之道：用多态替代条件语句

**大部分的条件语句是可以用多态代替的。**


### 为什么要用多态替代条件语句

* 没有 if 语句的函数更容易阅读。
* 没有 if 语句的函数更容易测试。
* 多态的系统更容易维护。

### 多态和条件语句的使用场景

#### 使用多态的场景

* 当对象要根据不同的状态表现不同的行为时。
* 当你需要在很多地方检查相同的条件时。

#### 使用条件语句的场景

* 主要用于原始对象的比较：<，>，==，!=。
* 其他。

这篇文章主要着重于如何避免 if 语句。

#### 如何避免使用 if 语句

* 不要返回 null，而是返回一个空的对象，例如说空的链表。
* 不要返回错误码，而是直接在运行时抛出异常。

### 如何用多态代替条件语句

如果你有一个条件语句，它根据对象的类型选择不同的行为。那么如何用多态来替代它呢？下面，我们来看一个例子。

#### 条件语句实现的类

```java
Class Update {
    execute() {
        if (FLAG_i18n_ENABLE) {
            //DO A;
        } else {
            //DO B;
        }
    }
    render() {
        if (FLAG_i18n_ENABLE) {
            //render A;
        } else {
            //render B;
        }
    }
}
```

上面的类根据 FLAG_i18n_ENABLE 来执行不同的操作。这样写一点问题都没有。

下面我们来看看一般的测试方法。

```java
void testExecuteDoA() {
    FLAG_i18n_ENABLE = true;
    Update u = new Update();
    u.execute();
    assertX();
}
void testExecuteDoB() {
    FLAG_i18n_ENABLE = false;
    Update u = new Update();
    u.execute();
    assertX();
}
```

看完上面的代码，你可能也觉得似曾相识，也觉得没什么问题。

实际上，这样写的类有以下几个问题：

* 大量的条件语句判断让代码可读性急剧下降。就好像你在高速公路上行驶的时候，开的正开心，前面一个此路不通的公示牌出现，于是你不得不走别的路。看代码也是同样道理，太多的分支语句会让读者晕头转向。
* 条件语句的存在让测试更加困难。在写测试的时候，你不得不去考虑它的状态码。上面的类只有两个状态，如果有五个状态呢。光是搞清楚状态之间关系就已经够呛了。

#### 多态实现的类

那么，如何用多态来重写上面的类呢？

我们可以分为两步来操作：

* 让 `Update` 成为抽象类，方法也抽象。
* 在子类中的覆盖方法实现条件语句的分支操作。

代码如下

```java
abstract class Update {
    abstract execute();
    abstract render();
}

class I18NUpdate extends Update {
    execute() {
        //Do A;
    }
    render() {
        //render A;
    }
}

class NonI18NUpdate extends Update {
    execute() {
        //Do B;
    }
    render() {
        //render B;
    }
}
```

测试方法：

```java
void testExecuteDoA() {
    Update u = new I18NUpdate();
    u.execute();
    assertX();
}
void testExecuteDoB() {
    Update u = new NonI18NUpdate();
    u.execute();
    assertX();
}
```

用多态实现的类，通过继承抽象类，重写抽象方法的方式，避免使用了条件语句。在测试的时候，不需要关心它的状态码，子类本身就已经承载了状态信息。所以你可以看到，在测试的时候，代码非常的清晰易懂。

#### 总结

使用多态实现的类有两个好处：

* 我们可以通过增加新的子类来添加新的行为，而且不会影响到原来的代码。
* 不同的操作和概念在不同的类中，容易理解和阅读。

### 在哪里决定要创建什么子类

咋一看上面的标题有点绕，我们来详细讨论一下问题的来源。上面我们讲了如何多态代替条件语句，但是有一个问题是无法回避的：我们怎么判断要创建哪一种子类？

实际上，我们还是要依靠 `FLAG_i18n_ENABLE` 来决定实例化哪个子类，也就是说，我们仍然要用到条件语句。写程序你肯定要用到条件语句，但是用的太多会有前面我们说到的问题。把多态和条件语句结合，才是正道。

我们回到前面的问题，既然条件语句无法避免，因为我们要根据条件决定使用哪个子类。我们先来对类做一个粗略的划分。

* 负责业务逻辑的类：例如说我们上面的 `Update`
* 负责创建类的类：例如说工厂模式中的 `Factory`

通过上面的划分，我们可以把子类创建交给工厂类。

```java
class Consumer {
    Cosumer(Update u) {...}
}

class Factory {
    Consumer build() {
        Update u = FLAG_i18n_ENABLE
                    ? new I18NUpdate()
                    : new NonI18NUpdate;
        return new Consumer(u);
    }
}
```

现在我们可以回答上面的问题了：在工厂类中根据条件决定要创建哪个子类。这样处理有以下的好处：

* 条件语句集中在了一个地方。
* 没有多余的重复，除了工厂类，其他地方不需要用到条件语句。
* 分离了职责和全局状态。
* 相同的代码集中在一个地方。
* 独立测试变得简单，而且能同时进行。
* 在子类中可以清楚地看到实现的不同。

### 什么情况下使用多态

* 类的行为根据状态进行变化的时候。
* 同样的条件语句在多个地方出现的时候。

### 总结

**该用条件语句的时候不要强行用多态。**
