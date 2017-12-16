## 非阻塞式服务器


非阻塞式IO管道(Pipelines)
------

一个非阻塞式IO管道是由各个处理非阻塞式IO组件组成的链。其中包括读/写IO。下图就是一个简单的非阻塞式IO管道组成。

![管道](http://ifeve.com/wp-content/uploads/2017/04/non-blocking-server-1.png)

一个组件使用 Selector 监控 Channel 什么时候有可读数据。然后这个组件读取输入并且根据输入生成相应的输出。最后输出将会再次写入到一个Channel中。


非阻塞式vs. 阻塞式管道
------
非阻塞和阻塞IO管道两者之间最大的区别在于他们如何从底层Channel(Socket或者file)读取数据。

IO管道通常从流中读取数据（来自socket或者file）并且将这些数据拆分为一系列连贯的消息。这和使用tokenizer（这里估计是解析器之类的意思）将数据流解析为token（这里应该是数据包的意思）类似。相反你只是将数据流分解为更大的消息体。我将拆分数据流成消息这一组件称为“消息读取器”（Message Reader）

虽然阻塞式Message Reader容易实现，但是也有一个不幸的缺点：**每一个要分解成消息的流都需要一个独立的线程**。必须要这样做的理由是每一个流的IO接口会阻塞，直到它有数据读取。这就意味着一个单独的线程是无法尝试从一个没有数据的流中读取数据转去读另一个流。一旦一个线程尝试从一个流中读取数据，那么这个线程将会阻塞直到有数据可以读取。

如果IO管道是必须要处理大量并发链接服务器的一部分的话，那么服务器就需要为每一个链接维护一个线程。对于任何时间都只有几百条并发链接的服务器这确实不是什么问题。但是如果服务器拥有百万级别的并发链接量，这种设计方式就没有良好收放。每个线程都会占用栈32bit-64bit的内存。所以一百万个线程占用的内存将会达到1TB！

为了将线程数量降下来，许多服务器使用了服务器维持线程池（例如：常用线程为100）的设计，从而一次一个地从入站链接（inbound connections）地读取。入站链接保存在一个队列中，线程按照进入队列的顺序处理入站链接。这一设计如下图所示：(译者注：Tomcat就是这样的)

![1](http://ifeve.com/wp-content/uploads/2017/04/non-blocking-server-3.png)

我们可以看到左侧的Stream数据可以看做是一个队列，线程下的MessageReader会从中读取数据并处理。然而，这一设计需要入站链接合理地发送数据。如果入站链接长时间不活跃，那么大量的不活跃链接实际上就造成了线程池中所有线程阻塞。这意味着服务器响应变慢甚至是没有反应。


基础非阻塞式IO管道设计
------

一个非阻塞式IO管道可以使用一个单独的线程向多个流读取数据。这需要流可以被切换到非阻塞模式。在非阻塞模式下，当你读取流信息时可能会返回0个字节或更多字节的信息。如果流中没有数据可读就返回0字节，如果流中有数据可读就返回1+字节。

为了避免检查没有可读数据的流我们可以使用 Java NIO Selector. 一个或多个SelectableChannel 实例可以同时被一个Selector注册.。当你调用Selector的select()或者 selectNow() 方法它只会返回有数据读取的SelectableChannel的实例. 下图是该设计的示意图：

![enter image description here](http://ifeve.com/wp-content/uploads/2017/04/non-blocking-server-4.png)

读取部分消息
当我们从一个SelectableChannel读取一个数据包时，我们不知道这个数据包相比于源文件是否有丢失或者重复数据（原文是：When we read a block of data from a SelectableChannel we do not know if that data block contains less or more than a message）。一个数据包可能的情况有：缺失数据（比原有消息的数据少）、与原有一致、比原来的消息的数据更多（例如：是原来的1.5或者2.5倍）。数据包可能出现的情况如下图所示：

![enter image description here](http://ifeve.com/wp-content/uploads/2017/04/non-blocking-server-5.png)

判断消息的完整性需要消息读取器（Message Reader）在数据包中寻找是否存在至少一个完整消息体的数据。如果一个数据包包含一个或多个完整消息体，这些消息就能够被发送到管道进行处理。寻找完整消息体这一处理可能会重复多次，因此这一操作应该尽可能的快。

判断消息完整性和存储部分消息都是消息读取器(Message Reader)的责任。为了避免混合来自不同Channel的消息，我们将对每一个Channel使用一个Message Reader。设计如下图所示:

![enter image description here](http://ifeve.com/wp-content/uploads/2017/04/non-blocking-server-6.png)

在从Selector得到可从中读取数据的Channel实例之后, 与该Channel相关联的Message Reader读取数据并尝试将他们分解为消息。这样读出的任何完整消息可以被传到读取通道(read pipeline)任何需要处理这些消息的组件中。

一个Message Reader一定满足特定的协议。Message Reader需要知道它尝试读取的消息的消息格式。如果我们的服务器可以通过协议来复用，那它需要有能够插入Message Reader实现的功能 – 可能通过接收一个Message Reader工厂作为配置参数。



