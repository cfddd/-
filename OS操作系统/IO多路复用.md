## 简述
单线程或单进程同时监测若干个文件描述符是否可以**执行IO操作**的能力。
## I/O模型
1. 阻塞IO
2. 非阻塞IO
3. IO多路复用
4. 信号驱动IO
5. 异步IO
### 阻塞IO

这是最常用的简单的IO模型。阻塞IO意味着当我们发起一次IO操作后一直等待成功或失败之后才返回，在这期间程序不能做其它的事情。

阻塞IO操作只能对单个文件描述符进行操作，比如[read](https://link.zhihu.com/?target=http%3A//man7.org/linux/man-pages/man2/read.2.html)或[write](https://link.zhihu.com/?target=http%3A//man7.org/linux/man-pages/man2/write.2.html)。

### 非阻塞IO
在发起IO时，通过对文件描述符设置O_NONBLOCK flag来指定该文件描述符的IO操作为非阻塞。

非阻塞IO通常发生在一个for循环当中，因为每次进行IO操作时要么IO操作成功，要么当IO操作会阻塞时返回错误，然后再根据需要进行下一次的for循环操作。

### IO多路复用
IO多路复用在Linux下包括了三种，[select](https://link.zhihu.com/?target=http%3A//man7.org/linux/man-pages/man2/select.2.html)、[poll](https://link.zhihu.com/?target=http%3A//man7.org/linux/man-pages/man2/poll.2.html)、[epoll](https://link.zhihu.com/?target=http%3A//man7.org/linux/man-pages/man7/epoll.7.html)

功能是类似的，但具体细节各有不同：首先都会对一组文件描述符进行相关事件的注册，然后阻塞等待某些事件的发生或等待超时。
#### 信号驱动IO

[信号驱动IO](https://link.zhihu.com/?target=http%3A//man7.org/linux/man-pages/man7/signal.7.html)是利用信号机制，让内核告知应用程序文件描述符的相关事件。这里有一个信号驱动IO相关的[例子](https://link.zhihu.com/?target=https%3A//github.com/troydhanson/network/blob/master/tcp/server/sigio-server.c)。

#### 异步IO

异步IO和信号驱动IO差不多，但它比信号驱动IO可以多做一步：相比信号驱动IO需要在程序中完成数据从用户态到内核态(或反方向)的拷贝，异步IO可以把拷贝这一步也帮我们完成之后才通知应用程序。