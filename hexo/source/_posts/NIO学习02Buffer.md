---
title: NIO学习02Buffer
date: 2017-10-09 17:30:38
tags: java
---


参考文章 

[Java NIO 教程buffer](http://ifeve.com/buffers/)

[详尽的Buffer机制](http://zachary-guo.iteye.com/blog/1457542)

什么是buffer
------

Buffer从本质上是一块可以写入数据，然后可以从中读取数据的内存。这块内存被封装成了 NIO Buffer 对象。并提供了一组方法，方便用来访问该快内存。

Buffer的基本用法
------

一般遵照如下步骤
1. 写入数据到buffer
2. 调用flip()方法
3. 从buffer中读取数据
4. 调用clear()或者compact()

<!--more-->

flip的中文翻译是翻转. 那么翻转了什么呢？其实就是读写过程的翻转。

当我们从读状态切换到写状态，或者从写状态切换到读状态时，都需要进行flip。

一旦读完了所有的数据，就需要清空缓冲区，让它可以再次被写入。有两种方式能清空缓冲区：调用clear()或compact()方法。clear()方法会清空整个缓冲区。compact()方法只会清除已经读过的数据。任何未读的数据都被移到缓冲区的起始处，新写入的数据将放到缓冲区未读数据的后面。

看一个段简单的代码

```java
RandomAccessFile aFile = new RandomAccessFile("data/nio-data.txt", "rw");
FileChannel inChannel = aFile.getChannel();

//create buffer with capacity of 48 bytes
ByteBuffer buf = ByteBuffer.allocate(48);

int bytesRead = inChannel.read(buf); //read into buffer.
while (bytesRead != -1) {

  buf.flip();  //make buffer ready for read

  while(buf.hasRemaining()){
      System.out.print((char) buf.get()); // read 1 byte at a time
  }

  buf.clear(); //make buffer ready for writing
  bytesRead = inChannel.read(buf);
}
aFile.close();
```

我们可以看到每当调用写操作之后，都进行了flip操作。同时，在读取完所有缓冲区内容时，执行了close来归还内存空间


既然被封装成了对象，我们来看看他都有哪些属性？

* capacity
* position
* limit

position和limit的含义取决于Buffer处在读模式还是写模式。不管Buffer处在什么模式，capacity的含义总是一样的。

![image1](http://ifeve.com/wp-content/uploads/2013/06/buffers-modes.png)

capacity

作为一个内存块，Buffer有一个固定的大小值，也叫“capacity”.你只能往里写capacity个byte、long，char等类型。一旦Buffer满了，需要将其清空（通过读数据或者清除数据）才能继续写数据往里写数据。

position

当你写数据到Buffer中时，position表示当前的位置。初始的position值为0.当一个byte、long等数据写到Buffer后， position会向前移动到下一个可插入数据的Buffer单元。position最大可为capacity – 1.

当读取数据时，也是从某个特定位置读。当将Buffer从写模式切换到读模式，position会被重置为0. 当从Buffer的position处读取数据时，position向前移动到下一个可读的位置。


limit

在写模式下，Buffer的limit表示你最多能往Buffer里写多少数据。 写模式下，limit等于Buffer的capacity。

当切换Buffer到读模式时， limit表示你最多能读到多少数据。因此，当切换Buffer到读模式时，limit会被设置成写模式下的position值。换句话说，你能读到之前写入的所有数据（limit被设置成已写数据的数量，这个值在写模式下就是position）

**Buffer的分配**


我们看到demo代码中有一段 ByteBuffer.allocate(48);

没错正如你看到的那样，我们通过allocate来申请内存空间

**写入数据**

既可以 int bytesRead = inChannel.read(buf); 从一个Channel中读取数据写入到buff中，也可以buf.put(127); 将想要存入的数据直接放进内存（put有很多形式，参照javaDoc

**读取数据**

可以直接读入到Channel中，也可以说把Buffer写入到Channel
```java
int bytesWritten = inChannel.write(buf);
```
或者直接从中读取数据 byte aByte = buf.get(); 

**rewind()方法**

Buffer.rewind()将position设回0，所以你可以重读Buffer中的所有数据。limit保持不变，仍然表示能从Buffer中读取多少个元素（byte、char等）

**clear()与compact()方法**

一旦读完Buffer中的数据，需要让Buffer准备好再次被写入。可以通过clear()或compact()方法来完成。

如果调用的是clear()方法，position将被设回0，limit被设置成 capacity的值。换句话说，Buffer 被清空了。Buffer中的数据并未清除，只是这些标记告诉我们可以从哪里开始往Buffer里写数据。

如果Buffer中有一些未读的数据，调用clear()方法，数据将“被遗忘”，意味着不再有任何标记会告诉你哪些数据被读过，哪些还没有。

如果Buffer中仍有未读的数据，且后续还需要这些数据，但是此时想要先先写些数据，那么使用compact()方法。

compact()方法将所有未读的数据拷贝到Buffer起始处。然后将position设到最后一个未读元素正后面。limit属性依然像clear()方法一样，设置成capacity。现在Buffer准备好写数据了，但是不会覆盖未读的数据。

**mark()与reset()方法**

通过调用Buffer.mark()方法，可以标记Buffer中的一个特定position。之后可以通过调用Buffer.reset()方法恢复到这个position。