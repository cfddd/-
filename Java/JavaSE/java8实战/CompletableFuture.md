## Future 接口
设计初衷是对将来某个时刻会发生的结果进行建模。它建模了一种异步计算，返回一个执行运算结果的引用，当运算结束后，这个引用被返回给调用方。

- isDone() 判断是否结束
- get() 等待结束返回结果

要使用Future，通常你只需要将耗时的操作封装在一个Callable对象中，再将它提交给ExecutorService，就万事大吉了。
```java
ExecutorService executor = Executors.newCachedThreadPool();  
Future<Double> future = executor.submit(new Callable<Double>() {  
    public Double call() {  
        return doSomeLongComputation();  
    }});  
doSomethingElse();  
try {  
    // 1后超时退出  
    Double result = future.get(1, TimeUnit.SECONDS);  
} catch (ExecutionException ee) {  
    // 计算抛出一个异常  
} catch (InterruptedException ie) {  
    // 当前线程在等待过程中被中断  
} catch (TimeoutException te) {  
    // 在Future对象完成之前超过已过期  
}
```
### 局限性
- `Future` 只能通过 `get()` 阻塞式获取结果，没法非阻塞查询。
- 异常传播不直观，需要在 `get()` 时才能感知。
- 多个任务之间缺乏组合和依赖机制（没法写链式逻辑）。
- 没有更细粒度的超时/回调机制。
## CompletableFuture

- 实现了 **Future** 和 **CompletionStage** 接口。
- 提供了一种 **声明式、链式的异步编程模型**。
- 支持：异步执行、任务组合、异常处理。
### 创建 `CompletableFuture`

**同步执行**：
```
CompletableFuture<String> future = CompletableFuture.completedFuture("Hello");
```
**异步执行**：
```
CompletableFuture.supplyAsync(() -> "Hello");   // 有返回值 

CompletableFuture.runAsync(() -> doSomething()); // 无返回值
```
**可传入自定义线程池**：

`CompletableFuture.supplyAsync(() -> "Hello", executor);`

### 转换与组合

- **thenApply**：转换结果（同步/异步）`future.thenApply(String::toUpperCase);`
- **thenCompose**：依赖另一个 Future，扁平化嵌套`future.thenCompose(s -> CompletableFuture.supplyAsync(() -> s + " World"));`
- **thenCombine**：两个 Future 都完成后组合结果`f1.thenCombine(f2, (a, b) -> a + " " + b);`
- **allOf / anyOf**：多个任务并行
```
CompletableFuture.allOf(f1, f2, f3).join(); // 全部完成 CompletableFuture.anyOf(f1, f2, f3).join(); // 任意一个完成
```

### 异常处理

- **exceptionally**：捕获并返回替代值`future.exceptionally(ex -> "fallback");`
- **handle**：同时处理正常结果和异常`future.handle((res, ex) -> ex != null ? "error" : res);`
### Async 变体

大部分方法都有 `xxxAsync` 版本：

- `thenApply` vs `thenApplyAsync`
- `thenCompose` vs `thenComposeAsync`
- 区别：是否在 **相同线程**（调用线程/前一个 stage 的线程）执行，还是交给 **ForkJoinPool.commonPool** 或指定线程池。

## 使用定制的执行器
明智的选择似乎是创建一个配有线程池的执行器，线程池中线程的数目取决于你预计你的应用需要处理的负荷，但是你该如何选择合适的线程数目呢？
![](../../../addition/Pasted%20image%2020250903153311.png)
> 守护线程相关的概念：java中创建**用户线程**和**守护线程**，这二者之间没有性能上的差异。
> - **守护线程**：是 Java 中线程的一种类型，当所有用户线程结束时，守护线程会被强制终止。
> 	- **用户线程**：需要等待线程池中的任务完成，才能退出程序。即使程序中的其他线程已经完成，用户线程仍然会运行，直到任务完成后才能退出。
> 	- **守护线程**：不需要等待守护线程完成，程序就可以退出。如果所有的用户线程都结束，守护线程会被强制终止。
> - **守护进程**：是操作系统中运行的后台进程，不会阻止系统退出，通常用于执行系统级别的服务和任务，如日志记录、数据库服务等。

**使用parallelStream还是completableFuture？**：
- 如果你进行的是计算密集型的操作，并且没有I/O，那么推荐使用Stream接口，因为实现简单，同时效率也可能是最高的（如果所有的线程都是计算密集型的，那就没有必要创建比处理器核数更多的线程）。
- 如果你并行的工作单元还涉及等待I/O的操作（包括网络连接等待），那么使用CompletableFuture灵活性更好，因为可以依据等待/计算，或者W/C的比率设定需要使用的线程数。不用并行流的另一个原因是，处理流的流水线中如果发生I/O等待，流的延迟特性会让我们很难判断到底什么时候触发了等待。
## 例子：电商比价服务
### 异步获取价格
```java
List<CompletableFuture<String>> priceFutures =
    shops.stream()
         .map(shop -> CompletableFuture.supplyAsync(
             () -> shop.getPrice(product), executor)) // 自定义线程池
         .collect(Collectors.toList());
```
这里每个 supplyAsync 就是一个任务，去调用不同商店的 API。
### 解析并应用折扣
每个商店返回的结果不是简单价格，而是带折扣信息的字符串，例如：`BestPrice:123.26:GOLD`

需要解析 → 应用折扣。这里用 **thenApply + thenCompose** 来链式处理：
```java
CompletableFuture.supplyAsync(() -> shop.getPrice(product), executor)
    .thenApply(Quote::parse)  // 转成 Quote 对象
    .thenCompose(quote -> CompletableFuture.supplyAsync(
        () -> Discount.applyDiscount(quote), executor));//依赖另一个异步任务，展开成一个新的 CompletableFuture。
```
### 汇总所有结果
用 CompletableFuture.allOf() 等待所有任务完成，然后收集结果：
```java
CompletableFuture[] futures = findPricesStream("myPhone") 
 .map(f -> f.thenAccept(System.out::println)) 
 .toArray(size -> new CompletableFuture[size]); 

CompletableFuture.allOf(futures).join(); //allOf工厂方法接收一个由CompletableFuture构成的数组，数组中的所有CompletableFuture对象执行完成之后，它返回一个CompletableFuture<Void>对象
```

## 总结
1. 用 CompletableFuture.supplyAsync 并行发起任务
2. 用 thenApply/thenCompose 处理结果和依赖关系
3. 用 allOf 聚合多个任务结果
4. 用自定义线程池解决 IO 阻塞问题