## 上下文
利用Threadlocal在每个线程的内存空间隔离的性质，写一个静态类

```java
@Slf4j
public class Context {
    private static final TransmittableThreadLocal<long> timestamp = new TransmittableThreadLocal<>();
}
```

TransmittableThreadLocal 是阿里巴巴开源项目 "Transmittable ThreadLocal" 的一部分，它扩展了 java.lang.ThreadLocal，解决了在使用线程池等会复用线程的场景下，ThreadLocal 变量值传递问题。通过使用 TransmittableThreadLocal，OperateEnterContext 能够确保即使在异步任务和线程池的使用场景下，每个线程都能正确地获取到属于自己的上下文信息，从而保证了数据的隔离性和一致性。

使用Threadlocal+静态类可以很轻松的实现，只在一个线程内的上下文机制