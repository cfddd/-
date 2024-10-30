# 工具类举例
## 原子类（Atomic Classes）
原子类提供了对基本数据类型和对象引用的原子性操作，确保在多线程环境下对这些变量的操作是不可分割的，避免了并发访问时可能出现的数据不一致问题。

- `AtomicInteger`：用于对整数进行原子操作，如`get`（获取当前值）、`set`（设置值）、`incrementAndGet`（自增并返回新值）、`decrementAndGet`（自减并返回新值）等方法。
- `AtomicLong`：类似于`AtomicInteger`，但用于长整型数据。
- `AtomicBoolean`：用于对布尔值进行原子操作，常用于多线程环境下的标志位控制。
- `AtomicReference`：提供对对象引用的原子操作，可以确保在多线程环境下对对象引用的赋值、比较和交换等操作的原子性。

## 锁机制
### ReentrantLock（可重入锁）
**重入**：对于已经拿到锁的线程，可以再次拿锁

一种可重入的互斥锁，与`synchronized`关键字类似，但提供了更灵活的锁控制功能。

它支持公平锁和非公平锁两种模式，默认是非公平锁。

公平锁会按照线程请求锁的顺序来分配锁，而非公平锁则允许插队获取锁，可能会提高并发性能，但可能导致某些线程长时间等待。
```java

    private static final ReentrantLock lock = new ReentrantLock();
    @Test
    public void reentrantLock(){
        Thread thread1 = new Thread(() -> {
            lock.lock();
            lock.lock();    // 可重入

                // 临界区代码
                System.out.println("Thread 1 acquired the lock");

                lock.unlock();
                lock.unlock();
        });

        Thread thread2 = new Thread(() -> {
            lock.lock();
                // 临界区代码
                System.out.println("Thread 2 acquired the lock");
                lock.unlock();
        });

        thread1.start();
        thread2.start();
    }
```
### ReadWriteLock（读写锁）
- 读-读不互斥
- 读-写、写-写互斥
- 适用于读多写少的场景。
```java

    ReadWriteLock readWriteLock = new ReentrantReadWriteLock();
    private static final int[] sharedData = {1, 2, 3, 4, 5};
    @Test
    public void testRWLock(){
        // 多个线程同时读取数据
        Thread reader1 = new Thread(() -> {
            readWriteLock.readLock().lock();
            try {
                System.out.println("Reader 1 is reading data: " + sharedData[0]);
            } finally {
                readWriteLock.readLock().unlock();
            }
        });

        Thread reader2 = new Thread(() -> {
            readWriteLock.readLock().lock();
            try {
                System.out.println("Reader 2 is reading data: " + sharedData[1]);
            } finally {
                readWriteLock.readLock().unlock();
            }
        });

        // 一个线程写入数据
        Thread writer = new Thread(() -> {
            readWriteLock.writeLock().lock();
            try {
                sharedData[0] = 10;
                System.out.println("Writer is writing data");
            } finally {
                readWriteLock.writeLock().unlock();
            }
        });

        reader1.start();
        reader2.start();
        writer.start();
    }
```
## 并发容器
### ConcurrentHashMap
它是一个线程安全的哈希表实现，在多线程环境下可以高效地进行插入、删除和查找操作。
### CopyOnWriteArrayList 和 CopyOnWriteArraySet
这两个集合类采用了写时复制的策略来实现线程安全。

**写时复制机制**：
- 当对集合进行写操作（如添加、删除元素）时，会复制一份原集合，在新的副本上进行修改，然后将原集合的引用指向新的副本。
- 这种方式在写操作时会有一定的性能开销，但读操作可以在原集合上进行，无需加锁，因此在多读少写的场景下具有较好的并发性能。
```java
    @Test
    public void copyOnWrite() throws InterruptedException {
        CopyOnWriteArrayList<Integer> list = new CopyOnWriteArrayList<>();
        for (int i = 0; i < 10; i++) {
            list.add(i);
        }
        // 多个线程可以并发地读取 list
        Thread reader1 = new Thread(() -> {
            for (int i = 0; i < 10; i++) {
                System.out.println("reads: " + list.get(i));
            }
        });

        Thread reader2 = new Thread(() -> {
            for (int i = 0; i < 10; i++) {
                try {
                    Thread.sleep(1);
                } catch (InterruptedException e) {
                    throw new RuntimeException(e);
                }
                list.set(i,-i);
            }
        });
        reader1.start();
        reader2.start();
        Thread.sleep(1000);
    }
```
#### 缺点
- 内存占用：每次写操作都需要复制一份原始数据，会占用额外的内存空间，在数据量比较大的情况下，可能会导致内存资源不足。
- 写操作开销：每一次写操作都需要复制一份原始数据，然后再进行修改和替换，所以写操作的开销相对较大，在写入比较频繁的场景下，性能可能会受到影响。
- 数据一致性问题：修改操作不会立即反映到最终结果中，还需要等待复制完成，这可能会导致一定的数据一致性问题。
## 线程池
线程池是一种用于管理和复用线程的机制，它可以避免频繁地创建和销毁线程所带来的性能开销。线程池维护了一个固定数量或可动态调整数量的线程集合，当有任务需要执行时，从线程池中获取一个空闲线程来执行任务，任务执行完成后，线程并不会立即销毁，而是回到线程池中等待下一次任务分配。
### 线程池核心参数
- **corePoolSize** : 任务队列未达到队列容量时，最大可以同时运行的线程数量。
- **maximumPoolSize** : 任务队列中存放的任务达到队列容量的时候，当前可以同时运行的线程数量变为最大线程数。
- **workQueue**: 新任务来的时候会先判断当前运行的线程数量是否达到核心线程数，如果达到的话，新任务就会被存放在队列中。
- **keepAliveTime**:线程池中的线程数量大于 corePoolSize 的时候，如果这时没有新的任务提交，核心线程外的线程不会立即销毁，而是会等待，直到等待的时间超过了 keepAliveTime才会被回收销毁。
- **unit** : keepAliveTime 参数的时间单位。
- **threadFactory** :executor 创建新线程的时候会用到。
- **handler** :拒绝策略，程池暂时无法处理的任务会先被放在阻塞队列中，阻塞队列满了才会触发拒绝策略。
## 信号量（Semaphore）
信号量用于**控制对有限资源的访问数量**，它维护了一个许可数量，线程在访问资源前需要获取许可，当许可数量为 0 时，线程会被阻塞，直到有其他线程释放许可。信号量可以用于实现资源的限流、控制并发访问的数量等场景。

```java
    @Test
    public void testSemaphore(){
        // 定义一个信号量，初始许可数量为3
        Semaphore semaphore = new Semaphore(3);

        // 创建10个线程尝试获取许可
        for (int i = 0; i < 10; i++) {
            final int threadId = i;
            Thread thread = new Thread(() -> {
                try {
                    semaphore.acquire();
                    System.out.println("Thread " + threadId + " acquired a permit");
                    Thread.sleep(2000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                } finally {
                    semaphore.release();
                    System.out.println("Thread " + threadId + " released a permit");
                }
            });
            thread.start();
        }
    }
```
## 倒计时器（CountDownLatch）
`CountDownLatch` 允许 `count` 个线程阻塞在一个地方，直至所有线程的任务都执行完毕。

`CountDownLatch` 是一次性的，计数器的值只能在构造方法中初始化一次，之后没有任何机制再次对其设置值，当 `CountDownLatch` 使用完毕后，它不能再次被使用。
```java
    @Test
    public void testCountDownLatch(){
        // 创建一个倒计时器，初始计数器值为5
        CountDownLatch countDownLatch = new CountDownLatch(5);

        // 创建5个子线程
        for (int i = 0; i < 5; i++) {
            final int threadId = i;
            Thread thread = new Thread(() -> {
                try {
                    System.out.println("Thread " + threadId + " is working");
                    Thread.sleep(2000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                } finally {
                    countDownLatch.countDown();
                    System.out.println("Thread " + threadId + " completed its task");
                }
            });
            thread.start();
        }

        try {
            // 主线程等待所有子线程完成任务
            countDownLatch.await();
            System.out.println("All threads have completed their tasks");
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        /*
        Thread 0 is working
Thread 2 is working
Thread 1 is working
Thread 4 is working
Thread 3 is working
Thread 4 completed its task
All threads have completed their tasks
Thread 2 completed its task
Thread 3 completed its task
Thread 0 completed its task
Thread 1 completed its task

Process finished with exit code 0

         */
    }
```

### CyclicBarrier
`CyclicBarrier` 的字面意思是可循环使用（Cyclic）的屏障（Barrier）。它要做的事情是：让一组线程到达一个屏障（也可以叫同步点）时被阻塞，直到最后一个线程到达屏障时，屏障才会开门，所有被屏障拦截的线程才会继续干活。
```java
public class CyclicBarrierExample1 {
  // 请求的数量
  private static final int threadCount = 550;
  // 需要同步的线程数量
  private static final CyclicBarrier cyclicBarrier = new CyclicBarrier(2);

  public static void main(String[] args) throws InterruptedException {
    // 创建线程池
    ExecutorService threadPool = Executors.newFixedThreadPool(10);

    for (int i = 0; i < threadCount; i++) {
      final int threadNum = i;
      Thread.sleep(1000);
      threadPool.execute(() -> {
        try {
          test(threadNum);
        } catch (InterruptedException e) {
          // TODO Auto-generated catch block
          e.printStackTrace();
        } catch (BrokenBarrierException e) {
          // TODO Auto-generated catch block
          e.printStackTrace();
        }
      });
    }
    threadPool.shutdown();
  }

  public static void test(int threadnum) throws InterruptedException, BrokenBarrierException {
    System.out.println("threadnum:" + threadnum + "is ready");
    try {
      /**等待60秒，保证子线程完全执行结束*/
      cyclicBarrier.await(60, TimeUnit.SECONDS);
    } catch (Exception e) {
      System.out.println("-----CyclicBarrierException------");
    }
    System.out.println("threadnum:" + threadnum + "is finish");
  }

}

```

运行结果，例如：
```sh
threadnum:0is ready
threadnum:1is ready
threadnum:0is finish
threadnum:1is finish
threadnum:2is ready
threadnum:3is ready
threadnum:3is finish
threadnum:2is finish
...
```
可以看见，每次都是两个两个线程在执行，执行完了再放后面两个任务去执行

**使用场景：**

- 首先，CyclicBarrier的功能是CountDownLatch的父集，所有他能做的，CyclicBarrier都能做
- **多线程模拟测试中的同步**：比如，要模拟 100 个用户同时访问一个服务器资源的场景，每个线程代表一个用户。可以使用`CyclicBarrier`让这 100 个线程都准备好后，再同时向服务器发送请求，这样可以更真实地模拟并发访问的情况，便于准确评估服务器在高并发下的性能和稳定性。

# 相关原理
## CAS
CAS（Compare-And-Swap, 比较并交换）

CAS 操作包含三个操作数：内存位置（V）、预期原值（A）和新值（B）。

在执行 CAS 操作时，处理器会比较内存位置 V 的值与预期原值 A 是否相等，如果相等，则将内存位置 V 的值更新为新值 B；如果不相等，则说明内存位置 V 的值已被其他线程修改，本次 CAS 操作失败，不会执行更新操作。

`Unsafe`类中的 CAS 方法是`native`方法。`native`关键字表明这些方法是用本地代码（通常是 C 或 C++）实现的，而不是用 Java 实现的。这些方法直接调用底层的硬件指令来实现原子操作。也就是说，Java 语言并没有直接用 Java 实现 CAS。

**CAS优点**：
- **原子性操作保证**：CAS 操作是原子性的，它在硬件层面保证了比较和交换操作的不可分割性，避免了多线程并发访问共享资源时可能出现的数据不一致问题。
- **高效的并发性能**：由于 CAS 操作不需要像传统锁那样进行线程的阻塞和唤醒操作，当多个线程同时尝试更新一个共享变量时，只有少数线程的 CAS 操作会失败，大多数情况下线程可以无阻塞地执行，因此在高并发场景下具有较高的性能。特别是在读写操作比例较高的情况下，CAS 操作能够提供比锁机制更好的并发性能。
- **乐观锁策略体现**：CAS 操作体现了一种乐观锁的思想，即假设在数据更新期间没有其他线程对数据进行修改，只有在实际比较时发现数据已被修改，才会进行重试或采取其他相应的策略。这种乐观的并发控制策略在多读少写的场景中能够有效地提高系统的吞吐量和响应速度。

**缺点：**
- **ABA 问题**：CAS 操作在比较内存位置的值与预期原值时，只关注值是否相等，而不关心值的变化过程。如果一个变量的值从 A 变为 B，再从 B 变回 A，那么在使用 CAS 操作时可能会认为该变量的值没有发生变化，从而导致一些潜在的问题。
	- 为了解决 ABA 问题，可以使用带有版本号或时间戳的原子引用类，如`AtomicStampedReference`或`AtomicMarkableReference`，在比较值的同时也比较版本号或时间戳，确保数据的完整性。
- **循环开销**：当多个线程同时竞争一个共享资源时，CAS 操作可能会频繁失败，导致线程需要不断地重试，从而增加了 CPU 的开销。
## AQS
**基本原理**：AQS 通过**维护一个同步状态和一个阻塞队列来实现同步功能**。

**同步状态**用于表示资源的当前状态，不同的同步器可以根据具体需求定义同步状态的含义。

**阻塞队列**则用于存储等待获取资源的线程，这些线程会在队列中等待，直到有机会获取资源。

### 同步状态
AQS 使用一个`volatile`修饰的`int`类型变量`state`来表示同步状态。

AQS 在对同步状态`state`进行操作时，大量**使用了 CAS（Compare and Swap）操作来保证操作的原子性**。

通过 CAS 操作，线程可以在不被其他线程干扰的情况下尝试更新同步状态，如果更新成功，则表示获取或释放资源成功；如果更新失败，则说明资源已被其他线程占用或状态已发生变化，需要采取相应的处理策略，如重试或进入阻塞队列等待。

例如，在可重入锁中，`state`可以用来表示锁的持有次数；在读写锁中，`state`的不同位可以分别表示读锁和写锁的持有情况。

### 阻塞队列
AQS 内部维护了一个**双向链表实现的阻塞队列**，队列中的**节点代表等待获取资源的线程**。

每个节点包含了线程的引用以及对前后节点的引用，通过这些引用可以方便地实现队列的操作，如入队、出队等。

在等待队列中，线程处于阻塞状态，等待前驱节点的唤醒或者被中断。当资源可用时，AQS 会根据具体的同步器策略，从等待队列中选择一个或多个合适的线程进行唤醒，让它们有机会再次尝试获取资源。

当一个线程尝试获取资源但资源不可用时，该线程会被封装成一个节点并加入到阻塞队列中等待，直到资源可用并轮到它获取资源。
### 获取资源方法
**`acquire`方法**：它首先尝试通过调用`tryAcquire`方法来获取资源，如果获取成功，则线程可以继续执行后续操作；如果获取失败，则线程会被阻塞并加入到阻塞队列中等待。
### 释放资源方法
当线程完成对资源的使用后，需要调用`release`方法来释放资源。
**`release`方法**：`release`方法首先会调用`tryRelease`方法来尝试释放资源，如果释放成功，并且阻塞队列中还有等待的线程，那么会唤醒队列中的一个或多个线程，让它们有机会获取资源。

> 在`ReentrantLock`中是怎么利用这些方法实现可重入的？

`ReentrantLock`重写了 `tryAcquire`和`tryRelease`方法，来实现具体的**资源获取**和**资源释放**逻辑

- `tryAcquire`方法会先尝试通过 CAS 操作直接获取锁，如果获取失败，再判断当前线程是否已经持有锁，如果是则增加锁的持有次数，否则返回获取失败。
- `tryRelease`方法会将锁的持有次数减 1，如果减到 0，则表示锁已完全释放，可以唤醒其他等待的线程；如果不是0，说明当前线程任然持有这个锁没释放完，需要其他线程等待
