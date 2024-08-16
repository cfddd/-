  [理解 ReentrantLock 这一篇就够了（同步锁，重入锁，独占锁，公平锁）_为什么reentrantlock的锁在没有获取到锁之后可以不进入同步队列吗-CSDN博客](https://blog.csdn.net/qq_36537546/article/details/121542098)
[ReentrantLock-公平锁、非公平锁、互斥锁、自旋锁 - 光何 - 博客园 (cnblogs.com)](https://www.cnblogs.com/guanghe/p/13469924.html)

> 什么是再入锁？

**再入性**：当一个线程试图获得一个他已经获得过的锁，这个动作是成功的

这是对锁获取粒度的一个概念，即，锁的持有是**以线程为单位而不是基于调用次数**

> 再入锁是互斥锁还是自旋锁？

互斥锁

> Java 中再入锁 ReentrantLock 和 synchronized 的区别 ?

- 再入锁 ReentrantLock 可以实现 synchronized 所有功能，可以替代它
- **可重入性**
- 可设置为**公平锁**和**不公平锁**，默认是**不公平锁**
- 功能更加强大，例如：带超时的获取锁尝试、判断是否有某个线程在排队等待获取锁

> 什么是公平锁和不公平锁？有什么区别

**不公平锁（synchronized）**

线程的分配机制不会保证不出现“饥饿（有线程始终得不到机会运行）”现象，线程之间存在竞争

**公平锁（ReentrantLock可设置）**

谁等待的时间长谁就先执行，保证不出现饥饿现象，可以通过INFO队列实现

ReentrantLock 默认是不公平锁，可以在创建时选择开启公平锁
