## 前期准备

### 安装 Go 编程环境

首先，确保你已经安装了 Go 编程语言。你可以从 [Go 官方网站](https://golang.org/) 下载并安装适用于 Windows 的 Go 安装程序。安装完成后，你需要设置 Go 的环境变量，包括 `GOPATH` 和 `PATH`。

### 安装 Redis 数据库

你需要安装 Redis 数据库，它是一个开源的高性能键值存储数据库。在 Windows 上，你可以使用 Microsoft 提供的 Windows 版本 Redis。以下是安装步骤：

1. 前往 [Microsoft 官方的 Windows 版 Redis](https://github.com/microsoftarchive/redis/releases) 页面。

2. 下载适用于 Windows 的 Redis 安装包，根据你的系统选择 32 位或 64 位版本。

3. 解压下载的文件到一个你喜欢的目录，例如 `C:\Redis`。

4. 打开命令提示符，进入解压后的 Redis 目录，运行 `redis-server.exe` 启动 Redis 服务器。

5. 打开另一个命令提示符，同样进入 Redis 目录，运行 `redis-cli.exe` 来连接到 Redis 服务器。

6. 在 `redis-cli.exe` 中，你可以运行各种 Redis 命令，比如 `set mykey "Hello"` 来设置键值对。

### 安装 go-redis 框架

使用第三方库 [go-redis](https://github.com/go-redis/redis) 可以方便地连接到 Redis 数据库并进行操作。你可以通过以下命令来下载并安装这个库：

```shell
go get github.com/go-redis/redis/v8
```

## 示例代码和操作步骤

### 运行 Redis 服务器和客户端

在你开始使用 go-redis 框架之前，确保你已经启动了 Redis 服务器和客户端。在命令提示符中，进入你的 Redis 目录，然后运行以下命令：

```shell
redis-server.exe
```

在另一个命令提示符中，同样进入 Redis 目录，运行以下命令来启动 Redis 客户端：

```shell
redis-cli.exe
```

你可以使用 `monitor` 命令来监控 Redis 操作。

### 编写示例代码

在你喜欢的代码编辑器中创建一个名为 `main.go` 的文件，然后复制以下示例代码到文件中：

![](addition/Pasted%20image%2020230827214048.png)

输出
```shell
name: Alice
message exists


```

## ## **go-redis 常用功能**

### **发布订阅**

```text
package main

import (
    "fmt"

    "github.com/go-redis/redis"
)

func subscriber(client *redis.Client) {
    pubsub := client.Subscribe("mychannel")
    defer pubsub.Close()

    // 处理订阅接收到的消息
    for {
        msg, err := pubsub.ReceiveMessage()
        if err != nil {
            return
        }

        fmt.Println(msg.Channel, msg.Payload)
    }
}

func publisher(client *redis.Client) {
    for {
        // 发布消息到频道
        err := client.Publish("mychannel", "hello").Err()
        if err != nil {
            panic(err)
        }
    }
}

func main() {
    client := redis.NewClient(&redis.Options{
        Addr: "localhost:6379",
    })

    go subscriber(client)
    go publisher(client)

    <-make(chan struct{})
}
```

- 通过client.Subscribe订阅了一个频道,然后在循环里接收消息。
- 另开一个goroutine发布消息。使用go-redis可以方便地实现发布订阅模型。

### **消息队列**

```text
package main

import (
    "fmt"
    "math/rand"
    "time"

    "github.com/go-redis/redis"
)

var client *redis.Client 

// 初始化连接
func initClient() {
    client = redis.NewClient(&redis.Options{
        Addr:     "localhost:6379",
        Password: "", 
        DB:       0, 
    })
}

// 生产者 - 发布消息
func producer() {
    for {
        message := rand.Intn(1000)
        err := client.LPush("queue", message).Err()
        if err != nil {
            panic(err)
        }
        fmt.Println("pushed", message)
        time.Sleep(1 * time.Second)
    }
}

// 消费者 - 处理消息
func consumer(id int) {
    for {
        message, err := client.BRPop(0, "queue").Result()
        if err != nil {
            panic(err)
        }
        fmt.Printf("consumer%d popped %s \n", id, message[1])
        time.Sleep(500 * time.Millisecond)
    }
}

func main() {
    // 初始化
    initClient()

    // 生产者goroutine
    go producer()

    // 3个消费者goroutine
    for i := 0; i < 3; i++ {
        go consumer(i)
    }

    // 阻塞主goroutine
    <-make(chan struct{}) 
}
```

- 使用BRPop实现阻塞式出队,LPush入队,可以构建基于Redis的消息队列。
- 多个消费者可以共享队列实现负载均衡

### **pipeline访问**

Redis Pipeline实现了一种批量发送请求和响应的模式,它允许客户端在单次网络交互中缓冲并发送多个命令,然后再次单次接收所有响应。这种方式极大地减少了客户端和服务器之间的网络往返次数,优化了网络传输开销，显著提升Redis的总体吞吐量和命令处理性能。测试结果表明,Pipeline模式可以使Redis命令处理的TPS吞吐量提升数倍。同时它也减轻了客户端等待响应的时间,有效隐藏了网络通信时延。

此外,Pipeline模式下可轻松传输更大的数据包,避免小包命令多次网络传输的资源消耗。它还可以减少客户端和服务器端的CPU和内存占用。

总之,Redis Pipeline通过优化网络传输、批量命令执行等手段,极大地提升了Redis的性能,是非常重要的客户端访问优化方式。它尤其适合用于高负载、低延迟的Redis访问场景。

```text
func pipelineDemo(client *redis.Client) {
    // 创建pipeline
    pipe := client.Pipeline()

    // 添加多个命令到pipeline
    setCmd := pipe.Set("key1", "value1", 0)
    getCmd := pipe.Get("key1")
    incrCmd := pipe.Incr("counter")
    pipe.Expire("key1", time.Hour)

    // 执行pipeline
    _, err := pipe.Exec()
    if err != nil {
        panic(err)
    }

    fmt.Println("setCmd:", setCmd.Val())
    fmt.Println("getCmd:", getCmd.Val())
    fmt.Println("incrCmd:", incrCmd.Val())
}
```

- pipeline通过redis.Pipeline() 创建,使用管道对象添加命令,最后调用Exec() 执行。
- 返回的cmds包含每个命令的响应,可以依次处理。
- 这样就可以批量的发送多个命令,优化访问Redis服务器。

### **访问Redis集群**

```text
func clusterDemo() {

    client := redis.NewClusterClient(&redis.ClusterOptions{
        Addrs: []string{
            "127.0.0.1:7000",
            "127.0.0.1:7001", 
            "127.0.0.1:7002",
        },
    })

    err := client.Set("key", "value", 0).Err()
    if err != nil {
        panic(err)
    }

    val, err := client.Get("key").Result()
    if err != nil {
        panic(err)
    }
    fmt.Println("key", val)

    err = client.Close()
    if err != nil {
        panic(err)
    }
}
```

- 通过NewClusterClient创建客户端,传入集群节点地址。
- 然后就可以像使用单机客户端一样,直接操作集群了。
- go-redis会自动将请求路由到正确的节点上。
- 因此go-redis可以非常容易地访问Redis集群

### **事务(transaction)**

```text
func transactionDemo(client *redis.Client) {

  pipe := client.TxPipeline()

  pipe.Set("key1", "value1", 0)
  pipe.Set("key2", "value2", 0)

  _, err := pipe.Exec()
  if err != nil {
    panic(err)
  }

}
```

- 使用TxPipeline() 创建一个事务管道。
- 然后将命令添加到管道中,最后调用Exec() 执行。
- 这会将所有命令作为一个事务(transaction)发送到Redis服务器。

需要注意的是，Redis中的事务(transaction)不是一个原子操作。它只是一种将多个命令打包然后顺序执行的机制。与关系型数据库的事务不同,Redis事务中若某个命令执行失败, 后续的命令将不会继续执行, 但是不会回滚整个事务。
## 参考文章
[go-redis开发手册 - Go语言玩转Redis的正确姿势 - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/645669818)
[Go Redis [快速入门] (uptrace.dev)](https://redis.uptrace.dev/zh/guide/go-redis.html#dial-tcp-i-o-timeout)
[go-redis使用入门 - 掘金 (juejin.cn)](https://juejin.cn/post/7202521955366879288)
https://www.cnblogs.com/itbsl/p/14198111.html