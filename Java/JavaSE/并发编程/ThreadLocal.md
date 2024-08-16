## 什么是ThreadLocal
每个线程隔离的变量副本

ThreadLocal对象可以提供线程局部变量，每个线程Thread拥有一份自己的副本变量，多个线程互不干扰。

ThreadLocal 在每个线程的 Thread 对象实例数据中分配独立的内存区域，当我们访问 ThreadLocal 时，本质上是在访问当前线程的 Thread 对象上的实例数据，不同线程访问的是不同的实例数据，因此实现线程隔离。

## ThreadLocal 数据结构

每个Thread线程有一个自己的ThreadLocalMap

ThreadLocalMap有自己的独立实现，可以简单地将它的key视作ThreadLocal，value为代码中放入的值（实际上key并不是ThreadLocal本身，而是它的一个**弱引用**）。

每个线程在往ThreadLocal里放值的时候，都会往自己的ThreadLocalMap里存，读也是以ThreadLocal作为引用，在自己的map里找对应的key，从而实现了线程隔离。

ThreadLocal 提供具有自动清理数据的能力，具体分为 2 个颗粒度：
1. 自动清理散列表： ThreadLocal 数据是 Thread 对象的实例数据，当线程执行结束后，就会跟随 Thread 对象 GC 而被清理；
2. 自动清理无效键值对： ThreadLocal 是使用弱键的动态散列表，当 Key 对象不再被持有强引用时，垃圾收集器会按照弱引用策略自动回收 Key 对象，并在下次访问 ThreadLocal 时清理无效键值对。

自动清理无效键值对会存在 “滞后性”，在滞后的这段时间内，无效的键值对数据没有及时回收，就发生内存泄漏。为了避免内存泄漏，在业务开发中应该及时调用 ThreadLocal#remove 清理无效的局部存储。

> [Threadlocal为什么是弱引用?](https://blog.csdn.net/foxException/article/details/123496254)
### 既然弱引用在gc后会被回收，那么gc后key是否是null呢？
[linker](https://javaguide.cn/java/concurrent/threadlocal.html#gc-%E4%B9%8B%E5%90%8E-key-%E6%98%AF%E5%90%A6%E4%B8%BA-null)
## ThreadLocalMap Hash 算法
- 使用黄金分割数来作为hash计算因子
- 使用渐进式hash（hash冲突则循环向后寻找为冲突的位置存放）
- 遇到虚引用的冲突键，首先会向后查找有没有相同的value，直到找到null的位置，如果没有则替换该虚引用，有则替换该虚引用