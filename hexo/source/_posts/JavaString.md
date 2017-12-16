---
title: String,StringBuffer,StringBuilder
date: 2017-10-13 17:30:38
tags: java
---


### String,StringBuffer,StringBuilder

- String **不可变的字符序列**
- StringBuffer **线程安全的可变字符序列**
- StringBuilder **非线程安全的可变字符序列 **

这三个类都是Java提供的用来操作使用字符串的。也是争议很多的三个类。下面就来仔细聊一聊他们之间的区别。

可变？不可变？
------

从刚学习java的时候就知道String是不可变的。当时并不理解所谓什么是可变不可变。我们来看一段String的源码
```java
/** The value is used for character storage. */
    private final char value[];
```
我们看到，我们想存储的字符串实际被存放到了一个不可变的char数组中。既我们创建一个String对象如“ABC”后，这个对象的内容就不能够再更改了。 这时候可能有一个疑问，我们经常会出现如下操作 
```java
String s = "abc";
s = s + "123";
```
既然是不可以再改变，为什么可以使用+操作符进行更改呢，其实这个过程并不是直接改变了原来s所指向的那块内存地址。而是新申请了一段地址，里面存着“ABC123“,并将s指向了新的这块内存地址而已。暂且我们先这样简单理解，其中+运算符的实现机制我们会再进行讨论。

至此我们总结一下，一个String对象是不可变的。

<!--more-->

String
------
```java
public class Main {
    public static void main(String[] args) {
        String str1 = "hello world";
        String str2 = new String("hello world");
        String str3 = "hello world";
        String str4 = new String("hello world");
         
        System.out.println(str1==str2); //false
        System.out.println(str1==str3); // true
        System.out.println(str2==str4); // false
    }
}
```
上述代码的输出结果可能和我们想象的不一样，也揭秘了两种创建一个String对象的区别。

首先，众所周知的java中使用 **new** 关键字生成的对象是存在堆中进行的。也就是我们使用 String str2 = new String("hello world"); 生成的对象是存放在堆中的。 

但是使用String str1 = "hello world"; 时却不相同，这部分涉及到了JVM底层的实现机制。 简单的说就是 **"hello world"字符串实际被存储在了运行时常量池中，而且相同的字符串仅存了一份，固然地址相同** 

而s1,s2两个对象一个在堆中，一个在运行常量池中，固然不同。
而s2,s4本质上就是两个对象，当然不同。

StringBuffer与StringBuilder的线程安全性问题 
------

 StringBuffer和StringBuilder可以算是双胞胎了，这两者的方法没有很大区别。但在线程安全性方面，StringBuffer允许多线程进行字符操作。这是因为在源代码中StringBuffer的很多方法都被关键字synchronized 修饰了，而StringBuilder没有。

我在学习时，有一个疑问，是不是String也不安全呢？事实上不存在这个问题，String是不可变的。线程对于堆中指定的一个String对象只能读取，无法修改。试问：还有什么不安全的呢？ 


你最关心的性能
------
上面我们介绍了StringBuffer、StringBuilder、String的特性，但是问题来了。他们的性能怎么样呢

```java
public class Main {
    private static int time = 50000;
    public static void main(String[] args) {
        testString();
        testStringBuffer();
        testStringBuilder();
        test1String();
        test2String();
    }
     
     
    public static void testString () {
        String s="";
        long begin = System.currentTimeMillis();
        for(int i=0; i<time; i++){
            s += "java";
        }
        long over = System.currentTimeMillis();
        System.out.println("操作"+s.getClass().getName()+"类型使用的时间为："+(over-begin)+"毫秒");
    }
     
    public static void testStringBuffer () {
        StringBuffer sb = new StringBuffer();
        long begin = System.currentTimeMillis();
        for(int i=0; i<time; i++){
            sb.append("java");
        }
        long over = System.currentTimeMillis();
        System.out.println("操作"+sb.getClass().getName()+"类型使用的时间为："+(over-begin)+"毫秒");
    }
     
    public static void testStringBuilder () {
        StringBuilder sb = new StringBuilder();
        long begin = System.currentTimeMillis();
        for(int i=0; i<time; i++){
            sb.append("java");
        }
        long over = System.currentTimeMillis();
        System.out.println("操作"+sb.getClass().getName()+"类型使用的时间为："+(over-begin)+"毫秒");
    }
     
    public static void test1String () {
        long begin = System.currentTimeMillis();
        for(int i=0; i<time; i++){
            String s = "I"+"love"+"java";
        }
        long over = System.currentTimeMillis();
        System.out.println("字符串直接相加操作："+(over-begin)+"毫秒");
    }
     
    public static void test2String () {
        String s1 ="I";
        String s2 = "love";
        String s3 = "java";
        long begin = System.currentTimeMillis();
        for(int i=0; i<time; i++){
            String s = s1+s2+s3;
        }
        long over = System.currentTimeMillis();
        System.out.println("字符串间接相加操作："+(over-begin)+"毫秒");
    }
     
}
```

使用String类型共计耗时8000+毫秒，而使用StringBuffer和StringBuilder差不多大概都在10ms内。可以看到再加操作比较频繁的代码中，使用String的性能是比较底下的，如果理解了String是补可变的，其实可以很容易想明白，因为虚拟机在实例化对象和垃圾回收的过程是很浪费时间的。

我们从上面的代码中可以得出的结论

- 对于直接相加字符串，效率很高，因为在编译器便确定了它的值，也就是说形如"I"+"love"+"java"; 的字符串相加，在编译期间便被优化成了"Ilovejava"
- String、StringBuilder、StringBuffer三者的执行效率：StringBuilder > StringBuffer > String
-  当字符串相加操作或者改动较少的情况下，建议使用 String str="hello"这种形式；
-  当字符串相加操作较多的情况下，建议使用StringBuilder，如果采用了多线程，则使用StringBuffer