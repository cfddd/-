## 什么是ThreadLocal
每个线程隔离的变量副本

ThreadLocal对象可以提供线程局部变量，每个线程Thread拥有一份自己的副本变量，多个线程互不干扰。

## ThreadLocal 数据结构

每个Thread线程有一个自己的ThreadLocalMap

ThreadLocalMap有自己的独立实现，可以简单地将它的key视作ThreadLocal，value为代码中放入的值（实际上key并不是ThreadLocal本身，而是它的一个**弱引用**）。

每个线程在往ThreadLocal里放值的时候，都会往自己的ThreadLocalMap里存，读也是以ThreadLocal作为引用，在自己的map里找对应的key，从而实现了线程隔离。

这个map里面有Entry，它的key是ThreadLocal<?> k ，继承自WeakReference， 也就是我们常说的弱引用类型；它的value是一个Object

> 1. 强引用：我们常常 new 出来的对象就是强引用类型，只要强引用存在，垃圾回收器将永远不会回收被引用的对象，哪怕内存不足的时候
> 2. 软引用：使用 SoftReference 修饰的对象被称为软引用，软引用指向的对象在内存要溢出的时候被回收
> 3. 弱引用：使用 WeakReference 修饰的对象被称为弱引用，只要发生垃圾回收，若这个对象只被弱引用指向，那么就会被回收
> 4. 虚引用：虚引用是最弱的引用，在 Java 中使用 PhantomReference 进行定义。虚引用中唯一的作用就是用队列接收对象即将死亡的通知

### 既然弱引用在gc后会被回收，那么gc后key是否是null呢？
[linker](https://javaguide.cn/java/concurrent/threadlocal.html#gc-%E4%B9%8B%E5%90%8E-key-%E6%98%AF%E5%90%A6%E4%B8%BA-null)
## ThreadLocalMap Hash 算法
- 使用渐进式hash