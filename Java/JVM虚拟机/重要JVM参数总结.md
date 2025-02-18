## 堆内存相关
#### 1.显式指定堆内存
```
-Xms<heap size>[unit]
-Xmx<heap size>[unit]
```

**heap size** 表示要初始化内存的具体大小。

**unit** 表示要初始化内存的单位。单位为 **_“ g”_** (GB)、**_“ m”_**（MB）、**_“ k”_**（KB）。

`-Xms2G -Xmx5G`
#### 2.显式指定新生代内存
```
XX:NewSize=<young size>[unit]
XX:MaxNewSize=<young size>[unit]
```

```
-XX:NewSize=256m
-XX:MaxNewSize=1024m
```

#### 3.显式指定永久代/元空间的大小
**JDK 1.8 的时候，方法区（HotSpot 的永久代）被彻底移除了（JDK1.7 就已经开始了），取而代之是元空间，元空间使用的是本地内存。**

下面是一些常用参数：

```
-XX:MetaspaceSize=N #设置 Metaspace 的初始大小（是一个常见的误区，后面会解释）
-XX:MaxMetaspaceSize=N #设置 Metaspace 的最大大小
```

### 4.显示指定栈大小
栈空间是每个线程各自有的一块区域，如果栈空间太小，也会导致 StackOverFlow 异常。而要设置栈空间大小，只需要使用 -Xss 参数就可以。

```mipsasm
java -Xss2m GCDemo
```

上面的启动命令设置最大栈空间为 2M。

## 垃圾收集相关
1. 垃圾收集器
	1. 查看默认垃圾收集器`java -XX:+PrintCommandLineFlags -version`
	2. `jconsole`工具
2. GC 日志记录
3. 处理 OOM