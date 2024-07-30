## AtomicIntger是什么？
AtomicIntger是对int类型的一个封装，提供原子性的访问和更新操作

其原子性操作的实现是基于CAS

> CAS:
>
> 如果当前数值未变，代表没有其他线程进行并发修改，则成功更新。否则，可能出现不同的选择，要么进行重试，要么就返回一个成功或者失败的结果。

CAS是Java并发中所谓lock-free机制的基础

下面介绍一下AbstractQueuedSynchronizer（AQS），其是Java并发包中，实现各种同步结构和部分其他组成单元（如线程池中的Worker）的基础。

## AbstractQueuedSynchronizer（AQS）
这是一个比较复杂的模块，我们尽量简化一下，理解为什么需要AQS，如何使用AQS，至少要做什么
### AQS 核心思想
- 如果被请求的共享资源空闲，则将当前请求资源的线程设置为有效的工作线程，并且将共享资源设置为锁定状态
- 如果被请求的共享资源被占用，那么就需要一套线程阻塞等待以及被唤醒时锁分配的机制，这个机制 AQS 是基于 CLH 锁 （Craig, Landin, and Hagersten locks） 实现的。

> **CLH 锁：对自旋锁的一种改进**
>
> 一个虚拟的双向队列（虚拟的双向队列即不存在队列实例，仅存在结点之间的关联关系）
>
> 将暂时获取不到锁的线程加入到该队列中，通过内置的 FIFO 线程等待/等待队列 来完成获取资源线程的排队工作
>
> 每条请求共享资源的线程，是 CLH 队列锁的一个结点，有线程的引用（thread）、 当前节点在队列中的状态（waitStatus）、前驱节点（prev）、后继节点（next）。
>
> 用 volatile 修饰 int 成员变量 state 表示同步状态
>
> 以及各种基于CAS的基础操作方法，以及各种期望具体同步结构去实现的acquire/release方法。

利用AQS实现一个同步结构，至少要实现两个基本类型的方法，分别是：

acquire操作，获取资源的独占权；还有release操作，释放对某个资源的独占。

### 举例：ReentrantLock
它内部通过扩展AQS实现了Sync类型，以AQS的state来反映锁的持有情况。

下面是ReentrantLock对应acquire和release操作
```java
public void lock() {
    sync.acquire(1);
}
public void unlock() {
    sync.release(1);
}
```
关于ReentrantLock公平和非公平的实现，是建立在tryAcquire方法上的，配合CAS实现的

例如，锁无人占有时，并不检查是否有其他等待者，体现了非公平的语义

关于ReentrantLock可再入的实现，是通过state的值来确定的。

初始值为 0，本线程再次获取锁，会让state + 1；对应的，每次释放锁，会让state - 1，直到state回到零0才会释放共享资源给其他线程使用
### AQS 资源共享方式
1. Exclusive（独占，只有一个线程能执行，如ReentrantLock）
2. Share（共享，多个线程可同时执行，如Semaphore/CountDownLatch）