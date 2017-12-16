---
title: NIO学习04Selector
date: 2017-10-09 17:30:38
tags: java
---

# 重要组成部分Selector

Selector（选择器）是Java NIO中能够检测一到多个NIO通道，并能够知晓通道是否为诸如读写事件做好准备的组件。这样，一个单独的线程可以管理多个channel，从而管理多个网络连接。

为什么使用Selector?
------

仅用单个线程来处理多个Channels的好处是，只需要更少的线程来处理通道。事实上，可以只用一个线程处理所有的通道。对于操作系统来说，线程之间上下文切换的开销很大，而且每个线程都要占用系统的一些资源（如内存）。因此，使用的线程越少越好。


下面是单线程使用一个Selector处理3个channel的示例

**创建**

首先我们需要创建一个Selector.

```java
Selector selector = Selector.open();
```
我们通过Selector的一个静态方法得到了一个Selector实例.接下来我们向他注册Channel的实例.

**注册**

```java
channel.configureBlocking(false);
SelectionKey key = channel.register(selector,
	Selectionkey.OP_READ);
```

与Selector一起使用时，Channel必须处于非阻塞模式下。**这意味着不能将FileChannel与Selector一起使用，因为FileChannel不能切换到非阻塞模式。而套接字通道都可以。**

注意register()方法的第二个参数。这是一个“interest集合”，意思是在通过Selector监听Channel时对什么事件感兴趣。可以监听四种不同类型的事件：

- Connect (SelectionKey.OP_CONNECT)
- Accept (SelectionKey.OP_ACCEPT)
- Read (SelectionKey.OP_READ)
- Write (SelectionKey.OP_WRITE)

如果你对不止一种事件感兴趣，那么可以用“位或”操作符将常量连接起来

```java
int interestSet = SelectionKey.OP_READ | SelectionKey.OP_WRITE;
```

**SelectionKey**
当我们完成注册我们就得到一个**SelectionKey**实例。

SelectionKey中包含了如下内容

- interest集合
- ready集合
- Channel
- Selector
- 附加的对象（可选）

**interest集合**

```
int interestSet = selectionKey.interestOps();

boolean isInterestedInAccept  = (interestSet & SelectionKey.OP_ACCEPT) == SelectionKey.OP_ACCEPT；
boolean isInterestedInConnect = interestSet & SelectionKey.OP_CONNECT;
boolean isInterestedInRead    = interestSet & SelectionKey.OP_READ;
boolean isInterestedInWrite   = interestSet & SelectionKey.OP_WRITE;
```

我们可以通过interestOps方法能够得到一个整型数字. 再通过位运算得到一个是否感兴趣的bool值。

**ready集合**

```
selectionKey.isAcceptable();
selectionKey.isConnectable();
selectionKey.isReadable();
selectionKey.isWritable();
```

我们可以直接通过这些方法，拿到已经就绪的某些状态，这些方法将返回一个bool型

还可以通过他直接获取对应的Channel和Selector

```java
Channel  channel  = selectionKey.channel();
Selector selector = selectionKey.selector();
```

通过Selector选择通道
------
一旦向Selector注册了一或多个通道，就可以调用几个重载的select()方法。这些方法返回你所感兴趣的事件（如连接、接受、读或写）已经准备就绪的那些通道。换句话说，如果你对“读就绪”的通道感兴趣，select()方法会返回读事件已经就绪的那些通道。

select()阻塞到至少有一个通道在你注册的事件上就绪了。

select(long timeout)和select()一样，除了最长会阻塞timeout毫秒(参数)。

selectNow()不会阻塞，不管什么通道就绪都立刻返回（译者注：此方法执行非阻塞的选择操作。如果自从前一次选择操作后，没有通道变成可选择的，则此方法直接返回零。）。

select()方法返回的int值表示有多少通道已经就绪。亦即，自上次调用select()方法后有多少通道变成就绪状态。如果调用select()方法，因为有一个通道变成就绪状态，返回了1，若再次调用select()方法，如果另一个通道就绪了，它会再次返回1。如果对第一个就绪的channel没有做任何操作，现在就有两个就绪的通道，但在每次select()方法调用之间，只有一个通道就绪了。

一旦调用了select()方法，并且返回值表明有一个或更多个通道就绪了，然后可以通过调用selector的selectedKeys()方法，访问“已选择键集（selected key set）”中的就绪通道。如下所示：

```java
	Set selectedKeys = selector.selectedKeys();
```

**wake up**

某个线程调用select()方法后阻塞了，即使没有通道已经就绪，也有办法让其从select()方法返回。只要让其它线程在第一个线程调用select()方法的那个对象上调用Selector.wakeup()方法即可。阻塞在select()方法上的线程会立马返回。

如果有其它线程调用了wakeup()方法，但当前没有线程阻塞在select()方法上，下个调用select()方法的线程会立即“醒来（wake up）”。

**close()**

用完Selector后调用其close()方法会关闭该Selector，且使注册到该Selector上的所有SelectionKey实例无效。通道本身并不会关闭。


最后我们看一段完整的实例代码

```java
Selector selector = Selector.open();
channel.configureBlocking(false);
SelectionKey key = channel.register(selector, SelectionKey.OP_READ);
while(true) {
  int readyChannels = selector.select();
  if(readyChannels == 0) continue;
  Set selectedKeys = selector.selectedKeys();
  Iterator keyIterator = selectedKeys.iterator();
  while(keyIterator.hasNext()) {
    SelectionKey key = keyIterator.next();
    if(key.isAcceptable()) {
        // a connection was accepted by a ServerSocketChannel.
    } else if (key.isConnectable()) {
        // a connection was established with a remote server.
    } else if (key.isReadable()) {
        // a channel is ready for reading
    } else if (key.isWritable()) {
        // a channel is ready for writing
    }
    keyIterator.remove();
  }
}
```