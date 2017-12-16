---
title: NIO学习01概述
date: 2017-10-09 17:30:38
tags: java
---


什么JAVA NIO
------

先来看看什么是IO。
I/O即输入输出，是计算机与外界世界的一个借口。IO操作的实际主题是操作系统。在java编程中，一般使用流的方式来处理IO，所有的IO都被视作是单个字节的移动，通过stream对象一次移动一个字节。流IO负责把对象转换为字节，然后再转换为对象。

那么什么又是NIO呢?

java.nio全称java non-blocking IO，是指jdk1.4 及以上版本里提供的新api（New IO） ，为所有的原始类型（boolean类型除外）提供缓存支持的数据容器，使用它可以提供非阻塞式的高伸缩性网络。**NIO主要用到的是块**，所以NIO的效率要比IO高很多。


NIO和传统IO（一下简称IO）之间第一个最大的区别是，IO是面向**流**的，NIO是面向**缓冲区**的。 Java IO面向流意味着每次从流中读一个或多个字节，直至读取所有字节，它们没有被缓存在任何地方。此外，它不能前后移动流中的数据。如果需要前后移动从流中读取的数据，需要先将它缓存到一个缓冲区。NIO的缓冲导向方法略有不同。数据读取到一个它稍后处理的缓冲区，需要时可在缓冲区中前后移动。这就增加了处理过程中的灵活性。但是，还需要检查是否该缓冲区中包含所有您需要处理的数据。而且，需确保当更多的数据读入缓冲区时，不要覆盖缓冲区里尚未处理的数据。


IO的各种流是阻塞的。这意味着，当一个线程调用read() 或 write()时，该线程被阻塞，直到有一些数据被读取，或数据完全写入。该线程在此期间不能再干任何事情了。 NIO的非阻塞模式，使一个线程从某通道发送请求读取数据，但是它仅能得到目前可用的数据，如果目前没有数据可用时，就什么都不会获取。而不是保持线程阻塞，所以直至数据变的可以读取之前，该线程可以继续做其他的事情。 非阻塞写也是如此。一个线程请求写入一些数据到某通道，但不需要等待它完全写入，这个线程同时可以去做别的事情。 线程通常将非阻塞IO的空闲时间用于在其它通道上执行IO操作，所以一个单独的线程现在可以管理多个输入和输出通道（channel）。

<!--more-->

NIO的核心部分
------
- Channels (通道)
- Buffers
- Selectors

当然除了这三大模块还有很多模块，但是这三者构成了核心的API。

先来看Channels 和 Buffers

我们可以看到你既可以把数据从channel中读到buffer中，也可以将数据从buffer写到channel中

![image](http://ifeve.com/wp-content/uploads/2013/06/overview-channels-buffers1.png) 

如果把channel理解为IO的Stream，那么channel提供了一个双向读写的功能来代替InputStream和OutputStream

Channels钟主要提供了如下几个实现

- FileChannel   从文件中读写数据。
- DatagramChannel 能通过UDP读写网络中的数据。
- SocketChannel 能通过TCP读写网络中的数据。
- ServerSocketChannel 可以监听新进来的TCP连接，像Web服务器那样。对每一个新进来的连接都会创建一个SocketChannel。

可以看到基本涵盖了UDP、TCP、以及文件的IO操作

Buffers提供了

- ByteBuffer
- CharBuffer
- DoubleBuffer
- FloatBuffer
- IntBuffer
- LongBuffer
- ShortBuffer 

也涵盖了基本数据类型 byte, char, int, long, double, float 等。

Selector允许单线程处理多个 Channel。如果你的应用打开了多个连接（通道），但每个连接的流量都很低，使用Selector就会很方便。例如，在一个聊天服务器中。

简单的看一个图理解一下

![image](http://ifeve.com/wp-content/uploads/2013/06/overview-selectors.png)

要使用Selector，得向Selector注册Channel，然后调用它的select()方法。这个方法会一直阻塞到某个注册的通道有事件就绪。一旦这个方法返回，线程就可以处理这些事件，事件的例子有如新连接进来，数据接收等。
