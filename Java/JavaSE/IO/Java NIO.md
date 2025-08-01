> https://pdai.tech/md/java/io/java-io-nio.html

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
NIO 实现了 IO 多路复用中的 Reactor 模型，一个线程 Thread 使用一个选择器 Selector 通过轮询的方式去监听多个通道 Channel 上的事件，从而让一个线程就可以处理多个事件。线程可以非阻塞地等待一个或多个通道就绪，然后进行相应的I/O操作，避免了传统I/O模型中线程阻塞的问题。

通过配置监听的通道 Channel 为非阻塞，那么当 Channel 上的 IO 事件还未到达时，就不会进入阻塞状态一直等待，而是继续轮询其它 Channel，找到 IO 事件已经到达的 Channel 执行。


因为创建和切换线程的开销很大，因此使用一个线程来处理多个事件而不是一个线程处理一个事件具有更好的性能。

### 使用`FileChannel`进行文件的NIO读取

实现一个非阻塞的网络客户端，可以用`SocketChannel`配合`Selector`来处理网络I/O，避免一个线程处理一个请求的方式。

```java
import java.io.IOException;
import java.nio.channels.*;
import java.nio.*;
import java.net.*;

public class NonBlockingClient {
    public static void main(String[] args) {
        try {
            // 创建一个非阻塞的SocketChannel
            SocketChannel client = SocketChannel.open();
            client.configureBlocking(false);

            // 连接到服务器
            client.connect(new InetSocketAddress("localhost", 8080));

            // 创建一个Selector来监听通道的事件
            Selector selector = Selector.open();

            // 注册连接事件
            client.register(selector, SelectionKey.OP_CONNECT);

            while (true) {
                // 阻塞直到有事件发生
                selector.select();

                // 获取所有已就绪的事件
                for (SelectionKey key : selector.selectedKeys()) {
                    if (key.isConnectable()) {
                        // 连接已就绪，完成连接过程
                        client.finishConnect();
                        System.out.println("连接已建立");

                        // 注册读取事件
                        client.register(selector, SelectionKey.OP_READ);
                    } else if (key.isReadable()) {
                        // 读取数据
                        ByteBuffer buffer = ByteBuffer.allocate(1024);
                        client.read(buffer);
                        buffer.flip();
                        System.out.println("收到服务器数据: " + new String(buffer.array()));
                    }
                }

                // 清除已处理的事件
                selector.selectedKeys().clear();
            }
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```