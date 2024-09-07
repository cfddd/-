> [并发编程系列之Callable和Runnable的不同？-腾讯云开发者社区-腾讯云 (tencent.com)](https://cloud.tencent.com/developer/article/1874901)

Callable是可以组合线程池或者FutureTask一起使用，同时可以将结果返回给Future，通过 Future 可以了解任务执行情况，或者取消任务的执行
## 相同点
1. 都是接口，实现这些接口的类可以调用多线程
2. 都可以用`Thread.start()`来启动线程
## 不同点
Runnable 没有返回值；Callable 可以返回执行结果，是个泛型，和Futrue，FutureTask配合可以用来获取异步线程的执行结果

>Callable是可以组合线程池或者FutureTask一起使用，同时可以将结果返回给Future，通过 Future 可以了解任务执行情况，或者取消任务的执行

Runnable 方法异常只能在内部处理，不可以往外抛；Callable方法的call方法允许抛出异常

