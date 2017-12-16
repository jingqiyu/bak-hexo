---
title: 再看ZVAL
date: 2017-10-20 17:30:38
tags: php
---



首先在php源码中找到Zend/zend.h文件  其中定义了php对于变量的结构定义 我们先来看下面的两组定义
```
typedef union _zvalue_value {
    long lval;                  /* long value */
    double dval;                /* double value */
    struct {
        char *val;
        int len;
    } str;
    HashTable *ht;              /* hash table value */
    zend_object_value obj;
    zend_ast *ast;
} zvalue_value;
​
struct _zval_struct {
    /* Variable information */
    zvalue_value value;     /* value */  值
    zend_uint refcount__gc;    //引用计数
    zend_uchar type;    /* active type */ 类型
    zend_uchar is_ref__gc;   //是否是引用
};
```
首先定义了个union共用体。 保存 一个变量的 “值” 也就是 _zval_struct 下的值。  

 可以是long类型 -- 对应phpint double -- php float  struct 对应字符串类型 HastTable对应的 数组  zend_object_value 保存对象类型 

现在，这里有一些黑科技。看到那个union的定义吗？那意味着这不是真正的结构体，而是一个单独的类型。但是有多个类型的变量在里面。如果这里面 有多种类型的话，那它怎么能作为单一的类型呢？我很高兴你问了这个问题。要理解这个问题，我们需要先回想我们在第一篇文章谈论的C语言中的类型。

在C里面，变量只是一行内存地址的标签。也可以说类型只是标识哪一块内存将被使用的方式。在C里面没有使用任何东西将4个字节的字符串和整型值分隔 开。它们都只是一整块的内存。编译器会尝试通过”标识”内存段作为变量来解析它，然后将这些变量转换为特定的类型，但这并不是总是成功（顺便说一句，当一 个变量“重写”它得到的内存段，那将会产生段错误）。

那么，据我们所知，union是单独的类型，它根据怎么被访问而使用不同的方式解释。这可以让我们定义一个值来支持多种类型。有一点要注意的是，所 有类型的数据都必须使用同一块内存来存储。这个例子，在64位的编译器，long和double都会占用64个位来保存。字符串结构体会占用96位（64 位存储字符指针，32位保存整型长度）。hash_table会占用64位，还有zend_object_value会占用96位（32位用来存储元素，剩下的64位来存储指针）。而整一个union会占用最大元素的内存大小，因此在这里就是96位。

现在，如果再看清楚这个联合体（union），我们可以看到只有5种PHP数据类型在这里（long == int，double == float，str == string，hashtable == array，zend_object_value == object）。那么剩下的数据类型去了哪里呢？原来，这个结构体已经足够来存储剩余的数据类型。BOOL使用long(int)来存储，NULL不占用数据段，RESOURCE也使用long来存储。

在64位的编译器，long和double都会占用64个位来保存。字符串结构体会占用96位（64位存储字符指针，32位保存整型长度）。hash_table会占用64位，还有zend_object_value会占用96位（32位用来存储元素，剩下的64位来存储指针）。而整一个union会占用最大元素的内存大小，因此在这里就是96位。

#### 细说 zend_uchar type 

TYPE

因为这个value联合体并没有控制它是怎么被访问的，我们需要其他方式来记录变量的类型。这里，我们可以通过数据类型来得出如何访问value的信息。它使用type这个字节来处理这个问题（zend_uchar是一个无符号的字符，或者内存中的一个字节）。它从zend类型常量保留这些信息。这真的是一种魔法，是需要使用zval.type = IS_LONG来定义整型数据。因此这个字段和value字段就足够让我们知道PHP变量的类型和值。

IS_REF

这个字段标识变量是否为引用。那就是说，如果你执行了在变量里执行了$foo = &$bar。如果它是0，那么变量就不是一个引用，如果它是1，那么变量就是一个引用。它并没有做太多的事情。那么，在我们结束_zval_struct之前，再看一看它的第四个成员。

REFCOUNT

这个变量是指向PHP变量容器的指针的计数器。也就是说，如果refcount是1，那就表示有一个PHP变量使用这个容器。如果refcount是2，那就表示有两个PHP变量指向同一个变量容器。单独的refcount变量并没有太多有用的信息，但如果它与is_ref一起使用，就构成了垃圾回收器和写时复制的基础。它允许我们使用同一个zval容器来保存一个或多个PHP变量。refcount的语义解释超出这篇文章的范围，如果你想继续深入，我推荐你查看这篇文档。



#### 它是怎么工作的？

在PHP内部，zval使用跟其他C变量一样，作为内存段或者一个指向内存段的指针（或者指向指针的指针，等等），传递到函数。一旦我们有了变量，我们就想访问它里面的数据。那我们要怎么做到呢？我们使用定义在zend_operators.h文件里面的宏来跟zval一起使用，使得访问数据更简单。有一点很重要的是，每一个宏都有多个拷贝。不同的是它们的前缀。例如，要得出zval的类型，有Z_TYPE(zval)宏，这个宏返回一个整型数据来表示zval参数。但这里还有一个Z_TYPE(zval_p)宏，它跟Z_TYPE(zval)做的事情是一样的，但它返回的是指向zval的指针。事实上，除了参数的属性不一样之外，这两个函数是一样的，实际上，我们可以使用Z_TYPE(*zval_p)，但_P和_PP让事情更简单。

我们可以使用VAL这一类宏来获取zval的值。可以调用Z_LVAL(zval)来得到整型值（比如整型数据和资源数据）。调用Z_DVAL(zval)来得到浮点值。还有很多其他的，到这里到此为止。要注意的关键是，为了在C里面获取zval的值，你需要使用宏（或应该）。因此，当我们看见有函数使用它们时，我们就知道它是从zval里面提取它的值。



  

 
 