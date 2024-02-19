## RPC介绍

远程过程调用（Remote Procedure Call，缩写为 RPC）是一个计算机通信协议。  
该协议允许运行于一台计算机的程序调用另一台计算机的子程序，而程序员无需额外地为这个交互作用编程。
![](https://www.topgoer.cn/uploads/rpc/images/m_bc6111f979252e585b78349207b67057_r.gif)
从上图可以看出, RPC 本身是 client-server模型,也是一种 request-response 协议。

有些实现扩展了远程调用的模型，实现了双向的服务调用，但是不管怎样，调用过程还是由一个客户端发起，服务器端提供响应，基本模型没有变化。

服务的调用过程为：

1. client调用client stub，这是一次本地过程调用
2. client stub将参数打包成一个消息，然后发送这个消息。打包过程也叫做 marshalling
3. client所在的系统将消息发送给server
4. server的的系统将收到的包传给server stub
5. server stub解包得到参数。 解包也被称作 unmarshalling
6. 最后server stub调用服务过程. 返回结果按照相反的步骤传给client
## RPC over TCP vs RESTful
RPC 的消息传输可以通过 TCP、UDP 或者 HTTP等，所以有时候我们称之为 RPC over TCP、 RPC over HTTP。RPC 通过 HTTP 传输消息的时候和 RESTful的架构是类似的，但是也有不同。

比较 RPC over HTTP 和 RESTful：
- 首先 **RPC 的客户端和服务器端是紧耦合的**，客户端需要知道调用的过程的名字，过程的参数以及它们的类型、顺序等。一旦服务器更改了过程的实现， 客户端的实现很容易出问题。**RESTful基于 http的语义操作资源**，参数的顺序一般没有关系，也很容易的通过代理转换链接和资源位置，从这一点上来说，**RESTful 更灵活**。

- 其次，它们操作的对象不一样。 RPC 操作的是方法和过程，它要操作的是方法对象。 RESTful 操作的是资源(resource)，而不是方法。

- 第三，RESTful执行的是对资源的操作，增加、查找、修改和删除等，主要是CURD，所以如果要实现一个特定目的的操作，在这种情况下，RPC的实现更有意义

比较一下 RPC over TCP 和 RESTful：
- 直接使用socket实现 RPC，除了上面的不同外，我们可以获得性能上的优势
- 通过长连接减少连接的建立所产生的花费，在调用次数非常巨大的时候(这是目前互联网公司经常遇到的情况,大并发的情况下)，这个花费影响是非常巨大的。
## 安装依赖
```go
go get -u -v -tags "reuseport quic kcp zookeeper etcd consul ping" github.com/smallnest/rpcx/...
```
**tags** 对应:

- **quic**: 支持 quic 协议
- **kcp**: 支持 kcp 协议
- **zookeeper**: 支持 zookeeper 注册中心
- **etcd**: 支持 etcd 注册中心
- **consul**: 支持 consul 注册中心
- **ping**: 支持 网络质量负载均衡
- **reuseport**: 支持 reuseport
## server简单实现

### Service
作为服务提供者，需要先定义服务。
```go
func (s *Server) Register(rcvr interface{}, metadata string) error
func (s *Server) RegisterName(name string, rcvr interface{}, metadata string) error
```

```go
import "context"

type Args struct {
    A int
    B int
}

type Reply struct {
    C int
}

type Arith int

func (t *Arith) Mul(ctx context.Context, args *Args, reply *Reply) error {
    reply.C = args.A * args.B
    return nil
}
```
### Server
定义完服务后，将它暴露出去来使用。通过启动一个TCP或UDP服务器来监听请求。
```go
package main

import (
    "flag"

    example "github.com/rpcx-ecosystem/rpcx-examples3"
    "github.com/smallnest/rpcx/server"
)

var (
    addr = flag.String("addr", "localhost:8972", "server address")
)

func main() {
    flag.Parse()

    s := server.NewServer()
    //s.RegisterName("Arith", new(example.Arith), "")
    s.Register(new(example.Arith), "")
    s.Serve("tcp", *addr)
}
```
服务器支持以如下这些方式启动，监听请求和关闭：

```go
    func NewServer(options ...OptionFn) *Server
    func (s *Server) Close() error
    func (s *Server) RegisterOnShutdown(f func())
    func (s *Server) Serve(network, address string) (err error)
    func (s *Server) ServeHTTP(w http.ResponseWriter, req *http.Request)
```

rpcx 提供了 3个 `OptionFn` 来设置启动选项：

```go

    func WithReadTimeout(readTimeout time.Duration) OptionFn
    func WithTLSConfig(cfg *tls.Config) OptionFn
    func WithWriteTimeout(writeTimeout time.Duration) OptionFn
```

可以设置 读超时、写超时和tls证书。

`ServeHTTP` 将服务通过HTTP暴露出去。

`Serve` 通过TCP或UDP协议与客户端通信。

rpcx 支持如下的网络类型：

- tcp: 推荐使用
- http: 通过劫持http连接实现
- unix: unix domain sockets
- reuseport: 要求 `SO_REUSEPORT` socket 选项, 仅支持 Linux kernel 3.9+
- quic: support [quic protocol](https://en.wikipedia.org/wiki/QUIC)
- kcp: sopport [kcp protocol](https://github.com/skywind3000/kcp)

## client简单实现
```go
  client := &Client{
        option: DefaultOption,
    }

    err := client.Connect("tcp", addr)
    if err != nil {
        t.Fatalf("failed to connect: %v", err)
    }
    defer client.Close()

    args := &Args{
        A: 10,
        B: 20,
    }

    reply := &Reply{}
    err = client.Call(context.Background(), "Arith", "Mul", args, reply)
    if err != nil {
        t.Fatalf("failed to call: %v", err)
    }

    if reply.C != 200 {
        t.Fatalf("expect 200 but got %d", reply.C)
    }
```
这些方法：

```go
    func (client *Client) Call(ctx context.Context, servicePath, serviceMethod string, args interface{}, reply interface{}) error
    func (client *Client) Close() error
    func (c *Client) Connect(network, address string) error
    func (client *Client) Go(ctx context.Context, servicePath, serviceMethod string, args interface{}, reply interface{}, done chan *Call) *Call
    func (client *Client) IsClosing() bool
    func (client *Client) IsShutdown() bool
```

`Call` 代表对服务同步调用。客户端在收到响应或错误前一直是阻塞的。 然而 `Go` 是异步调用。它返回一个指向 Call 的指针， 你可以检查 `*Call` 的值来获取返回的结果或错误。

`Close` 会关闭所有与服务的连接。他会立刻关闭连接，不会等待未完成的请求结束。

`IsClosing` 表示客户端是关闭着的并且不会接受新的调用。  

`IsShutdown` 表示客户端不会接受服务返回的响应。

`Client` 使用默认的 [CircuitBreaker (circuit.NewRateBreaker(0.95, 100))](https://godoc.org/github.com/rubyist/circuitbreaker#NewRateBreaker) 来处理错误。这是rpc处理错误的普遍做法。当出错率达到阈值， 这个服务就会在接下来的10秒内被标记为不可用。你也可以实现你自己的 CircuitBreaker。
### XClient
`XClient` 是对客户端的封装，增加了一些服务发现和服务治理的特性。

`Go` 代表异步调用， `Call` 代表同步调用。

XClient对于一个服务节点使用单一的连接，并且它会缓存这个连接直到失效或异常。
### 服务发现

rpcx 支持许多服务发现机制，你也可以实现自己的服务发现。

-  [Peer to Peer](https://www.topgoer.cn/docs/part2/registry.md#peer2peer): 客户端直连每个服务节点。
- [Peer to Multiple](https://www.topgoer.cn/docs/part2/registry.md#multiple): 客户端可以连接多个服务。服务可以被编程式配置。
- [Zookeeper](https://www.topgoer.cn/docs/part2/registry.md#zookeeper): 通过 zookeeper 寻找服务。
- [Etcd](https://www.topgoer.cn/docs/part2/registry.md#etcd): 通过 etcd 寻找服务。
- [Consul](https://www.topgoer.cn/docs/part2/registry.md#consul): 通过 consul 寻找服务。
- [mDNS](https://www.topgoer.cn/docs/part2/registry.md#mdns): 通过 mDNS 寻找服务（支持本地服务发现）。
- [In process](https://www.topgoer.cn/docs/part2/registry.md#inprocess): 在同一进程寻找服务。客户端通过进程调用服务，不走TCP或UDP，方便调试使用。
### 服务治理 (失败模式与负载均衡)
在一个大规模的rpc系统中，有许多服务节点提供同一个服务。客户端如何选择最合适的节点来调用呢？如果调用失败，客户端应该选择另一个节点或者立即返回错误？这里就有了故障模式和负载均衡的问题。

rpcx 支持 故障模式：

- [Failfast](https://www.topgoer.cn/docs/part3/failmode.md#failfast)：如果调用失败，立即返回错误
- [Failover](https://www.topgoer.cn/docs/part3/failmode.md#failover)：选择其他节点，直到达到最大重试次数
- [Failtry](https://www.topgoer.cn/docs/part3/failmode.md#failtry)：选择相同节点并重试，直到达到最大重试次数

对于负载均衡，rpcx 提供了许多选择器：

- [Random](https://www.topgoer.cn/docs/part3/selector.md#random_selector)： 随机选择节点
    - [Roundrobin](https://www.topgoer.cn/docs/part3/selector.md#roundrobin_selector)： 使用 [roundrobin](https://zh.wikipedia.org/wiki/%E5%BE%AA%E7%92%B0%E5%88%B6) 算法选择节点
    - [Consistent hashing](https://www.topgoer.cn/docs/part3/selector.md#hash_selector): 如果服务路径、方法和参数一致，就选择同一个节点。使用了非常快的[jump consistent hash](https://arxiv.org/abs/1406.2294)算法。
    - [Weighted](https://www.topgoer.cn/docs/part3/selector.md#weighted_selector): 根据元数据里配置好的权重(`weight=xxx`)来选择节点。类似于nginx里的实现(smooth weighted algorithm)
    - [Network quality](https://www.topgoer.cn/docs/part3/selector.md#ping_selector): 根据`ping`的结果来选择节点。网络质量越好，该节点被选择的几率越大。
    - [Geography](https://www.topgoer.cn/docs/part3/selector.md#geo_selector): 如果有多个数据中心，客户端趋向于连接同一个数据机房的节点。
    - [Customized Selector](https://www.topgoer.cn/docs/part3/selector.md#user_selector): 如果以上的选择器都不适合你，你可以自己定制选择器。例如一个rpcx用户写过它自己的选择器，他有2个数据中心，但是这些数据中心彼此有限制，不能使用 `Network quality` 来检测连接质量。

下面是一个异步的 rpcx 例子：
```go
package main

import (
    "context"
    "flag"
    "log"

    example "github.com/rpcx-ecosystem/rpcx-examples3"
    "github.com/smallnest/rpcx/client"
)

var (
    addr2 = flag.String("addr", "localhost:8972", "server address")
)

func main() {
    flag.Parse()

    d := client.NewPeer2PeerDiscovery("tcp@"+*addr2, "")
    xclient := client.NewXClient("Arith", client.Failtry, client.RandomSelect, d, client.DefaultOption)
    defer xclient.Close()

    args := &example.Args{
        A: 10,
        B: 20,
    }

    reply := &example.Reply{}
    call, err := xclient.Go(context.Background(), "Mul", args, reply, nil)
    if err != nil {
        log.Fatalf("failed to call: %v", err)
    }

    replyCall := <-call.Done
    if replyCall.Error != nil {
        log.Fatalf("failed to call: %v", replyCall.Error)
    } else {
        log.Printf("%d * %d = %d", args.A, args.B, reply.C)
    }
}
```