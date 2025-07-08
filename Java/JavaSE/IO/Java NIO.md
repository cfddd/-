提供了非阻塞、基于缓冲区（Buffer）和通道（Channel）的 IO 操作方式，显著提升了高并发场景下的性能。
## 基础知识
### 通道（Channel）
通道 Channel 是对原 I/O 包中的流的模拟，可以通过它读取和写入数据。

通道与流的不同之处在于，流只能在一个方向上移动(一个流必须是 InputStream 或者 OutputStream 的子类)，而通道是双向的，**可以用于读、写或者同时用于读写**。

**核心实现类**：
- `FileChannel`：文件 IO。
- `SocketChannel`/`ServerSocketChannel`：TCP 网络 IO。
- `DatagramChannel`：UDP 网络 IO。
### 缓冲区（Buffer）
缓冲区实质上是一个数组，但它不仅仅是一个数组。缓冲区提供了对数据的结构化访问，而且还可以跟踪系统的读/写进程。

**核心属性**：
- `capacity`：容量（固定值）。
- `position`：当前读写位置。
- `limit`：读写限制位置（写模式下等于 capacity，读模式下为实际写入的数据量）。

缓冲区包含各种基础类型包装类的类，例如ByteBuffer、CharBuffer等等
### 选择器（Selector）
