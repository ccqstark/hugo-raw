---
title: "一次理解String的不可变性"
date: 2021-04-06T21:07:00+08:00
draft: true
image: java.png
slug: "string"
tags:
    - Java
--- 

今天在看有关StringBuilder和StringBuffer的文章的时候看到里面提及了有关String中的final字段和不可变的性质，发现这个知识点不是很熟悉，去查了很多文章之后整理出这篇。

## 新建字符串与缓冲池

新建一个String我们一般有下面2种方式：

```java
String a = "ok";
String b = new String("ok");
```

这两种写法都可以创建一个String对象。

第一种用赋值运算符进行字符串初始化时，JVM自动为每个字符串生成一个String类的实例。

第二种就是创建String类的对象，因为String本来就是一个类，而不是像int和double那样的基本数据类型。

Java的字符串采用了缓冲池的技术，我们新建一个字符串的时候会去缓冲池寻找是否有已经存在的相同的字符串，如果有的话直接指向它即可；没有的话再创建，缓冲池是在`堆`里面的。

如下图，是下面代码的结果：

```java
// one和two内容相同，指向同一String对象
String one = "someString";
String two = "someString";
```

![https://pic1.zhimg.com/80/71d827920f870b82c93d06f8a9c0b247_720w.jpg?source=1940ef5c](https://pic1.zhimg.com/80/71d827920f870b82c93d06f8a9c0b247_720w.jpg?source=1940ef5c)

关于更深入的创建对象和之间的比较可以看下面这篇：

[java 字符串缓冲池 String缓冲池_天天的专栏-CSDN博客](https://blog.csdn.net/tiantiandjava/article/details/46309163)

## 哪里不可变？

那为什么说String不可变呢？我们明明可以通过给字符串变量赋一个新值来改变它的内容。

```java
String str = "aaaaaaa";
System.out.println(str);
str = "bbbbbbbbb";
System.out.println(str);

// 输出
//aaaaaaa
//bbbbbbbbb
```

实际上，当我们给字符串重新赋值的时候，它并不是去改变这个String对象中的字符数组`char[] value`的值（下面会讲到），而是去缓冲池里寻找有没有已经存在这个值的`String对象`，有的话就直接指向它，没有的话创建一个对象再指向它。

![%E4%B8%80%E6%AC%A1%E7%90%86%E8%A7%A3String%E7%9A%84%E4%B8%8D%E5%8F%AF%E5%8F%98%E6%80%A7%207dd788fbcd57410b913c52ecf7b450c0/Untitled.png](%E4%B8%80%E6%AC%A1%E7%90%86%E8%A7%A3String%E7%9A%84%E4%B8%8D%E5%8F%AF%E5%8F%98%E6%80%A7%207dd788fbcd57410b913c52ecf7b450c0/Untitled.png)

str只是一个引用，指向的String对象是在堆中的。**改变字符串的值其实只是改变整个对象的引用**。

而原来的String对象还是在那里没有被改变，之后要是有别的变量赋这个值可以继续指向它。

## 为什么不可变？

我们看下String的部分源码：

```java
public final class String
    implements java.io.Serializable, Comparable<String>, CharSequence
{
    /** The value is used for character storage. */
    private final char value[];
 
    /** Cache the hash code for the string */
    private int hash; // Default to 0
}
```

重点在这个字符数组`value`就是存放字符串内容的地方，注意它是被`final` 修饰的，也就是一旦初始化之后它的值就不能改变。

不过它value也只是个引用，引用不可变，但是如果真的想改是可以改变数组中元素但值的，但由于String中没有setValue方法，我们要改的话就要通过`反射`了。

### 利用反射改变

下面这段代码就利用了反射改变了value数组中的元素的值，但是数组的地址是没有改变的哦。

```java
String s="123";
Field valueArray=String.class.getDeclaredField("value");
valueArray.setAccessible(true);
char[] array=(char[]) valueArray.get(s);
array[0]='2';
System.out.println(s);//223
```

## 设计成不可变的原因？

### 代码安全

String类本身也是`final` 的，这样设计使得String不可被继承，不可扩展与重写方法，主要是为了安全。因为String作为一个比较底层的类使用得也比较频繁，而且很多实现是通过系统调用的，如果让开发者可以随意修改一些东西就很可能带来问题甚至注入病毒。

### 为了实现字符串缓冲池

**只有当字符串是不可变的，字符串池才有可能实现**。字符串池的实现可以在运行时节约很多heap空间，因为不同的字符串变量都指向池中的同一个字符串。但如果字符串是可变的，那改变一个字符串的值就会导致其它指向这个String对象的字符串引用对应的值也会改变。

### 作为HashMap的键

因为字符串是不可变的，所以在它创建的时候**HashCode**就被缓存了，不需要重新计算。这就使得字符串很适合作为Map中的键，字符串的处理速度要快过其它的键对象。这就是HashMap中的键往往都使用字符串。

### 线程安全

因为字符串是不可变的，所以是多线程安全的，同一个字符串实例可以被多个线程共享。这样便不用因为线程安全问题而使用同步。字符串自己便是线程安全的。

### 参考文章

[Java String类为什么是final的？](https://www.jianshu.com/p/9c7f5daac283)

[如何理解 String 类型值的不可变？](https://www.zhihu.com/question/20618891)