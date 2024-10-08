应用程序实际上只是发起了 **IO 操作的调用**而已，具体 IO 的执行是由**操作系统的内核**来完成的。

因此常见的IO模型也是在操作系统已有的 5 种IO模型之上的：**同步阻塞 I/O**、**同步非阻塞 I/O**、**I/O 多路复用**、**信号驱动 I/O** 和**异步 I/O**。

## BIO（Blocking I/O）
**BIO 属于同步阻塞 IO 模型** 。
## NIO (Non-blocking/New I/O)
IO 多路复用模型中，线程首先发起 select 调用，询问内核数据是否准备就绪，等内核把数据准备好了，用户线程再发起 read 调用。read 调用的过程（数据从内核空间 -> 用户空间）还是阻塞的。

**IO 多路复用模型，通过减少无效的系统调用，减少了对 CPU 资源的消耗。**

> 使用 NIO 并不一定意味着高性能，它的性能优势主要体现在高并发和高延迟的网络环境下。当连接数较少、并发程度较低或者网络传输速度较快时，NIO 的性能并不一定优于传统的 BIO 。

## AIO (Asynchronous I/O)
它是异步 IO 模型。

基于事件和回调机制实现的，也就是应用操作之后会直接返回，不会堵塞在那里，当后台处理完成，操作系统会通知相应的线程进行后续的操作。

## NIO具体介绍
[Java NIO 核心知识总结 | JavaGuide](https://javaguide.cn/java/io/nio-basis.html#nio-%E7%AE%80%E4%BB%8B)

[面试官：Java NIO 了解？ (qq.com)](https://mp.weixin.qq.com/s/mZobf-U8OSYQfHfYBEB6KA)
### 小结
> java API 中的 NIO 和IO模型中的NIO有什么区别？

首先明确两个是完全不同的东西
- IO模型中的NIO是同步非阻塞模型（Non-blocking IO）
- java API 中的 NIO 是一套java API，底层基于 IO模型中的多路复用模型，使用Selector、Channel和Buffer三个部分，让Selector和Channel一对多的绑定，通过Buffer完成读写。网络调用的还是基于操作系统的IO模型，也可以用于其他的IO，比如文件等

> NIO 由什么组成？
- **Buffer（缓冲区）**：NIO 读写数据都是通过缓冲区进行操作的。读操作的时候将 Channel 中的数据填充到 Buffer 中，而写操作时将 Buffer 中的数据写入到 Channel 中。
- **Channel（通道）**：Channel 是一个双向的、可读可写的数据传输通道，NIO 通过 Channel 来实现数据的输入输出。通道是一个抽象的概念，它可以代表文件、套接字或者其他数据源之间的连接。
- **Selector（选择器）**：允许一个线程处理多个 Channel，基于事件驱动的 I/O 多路复用模型。所有的 Channel 都可以注册到 Selector 上，由 Selector 来分配线程来处理事件。
- 易用的API

> 简单说说NIO是什么？

NIO将选择器（Selector）和通道（Channel）进行一对多的绑定，通过缓冲区（Buffer）实现读写，底层调用的是操作系统层面IO模型的多路复用模型。java的NIO提供了一系列方便的API使用，适用于高并发、高性能的网络编程和文件处理等场景。

> NIO 的优点
- 事件驱动模型
- 避免多线程
- 单线程处理多任务
- 非阻塞I/O，I/O读写不再阻塞，而是返回0
- 基于block的传输，通常比基于流的传输更高效
- IO多路复用大大提高了Java网络应用的可伸缩性和实用性