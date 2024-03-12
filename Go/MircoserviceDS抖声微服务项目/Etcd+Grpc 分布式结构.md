## 服务注册与发现基本概念
**服务注册**，就是**将提供某个服务的模块信息**(通常是这个服务的ip和端口)**注册到1个公共的组件**上去（比如: zookeeper\consul）。

**服务发现**，就是**新注册的这个服务模块能够及时的被其他调用者发现。不管是服务新增和服务删减都能实现自动发现。**
## 微服务时代的服务管理
所有的服务都在在1个server里，按照功能或者对象拆分成多个服务模块

- 好处：**各个微服务相互独立**，**深度解耦**，能够实现快速的迭代更新。
- 坏处：**服务的管理和控制变得异常的复杂和繁琐**、**人工维护难度变大**和**排查问题和性能变差（服务调用时的网络开销）**
### 服务注册
每一个服务对应的机器或者实例在启动运行的时候，都去向名字服务集群注册自己。

**启动完成后，注册的服务将开启自己的端口，等待远程调用连接。同时，还会设置一个租约时间，并持续续约，当单个服务出问题后，当时间结束就会自动解除注册。**

注册的方式有很多的形式，有HTTP接口形式，有RPC的方式，也有使用JSON格式的配置表的形式的。方式虽然不同，但是结果都是一样。

实例注册到名字服务上之后，接下来就是服务发现了。
### 服务发现
通过服务发现，就获得了一个**键（服务名称）下的所有的ip列表**，然后再用一定的负载均衡算法，或者干脆随机取1个ip，进行调用。

当然，也有些注册服务软件也提供了DNS解析功能或者负载均衡功能，它会直接返回给你一个可用的ip，直接调用即可。

和服务注册的方式一样，服务发现的方式，不同的名字服务软件的方式也会不一样，有的是得自己发送HTTP接口去**轮训调用**，如果发现有更新，就更新自己本地的配置文件。有的是可以通过实时的**sub/pub**的方式实现的**自动发现服务**，当我订阅的这个服务内容发生了更新，就实时更新自己的配置文件。也有的是通过RPC的方式。方式虽然不同，但是结果都是一样。

**这样一来，我们就可以通过服务注册和发现的方式，维护各个服务IP列表的更新，各个模块只需要向名字服务中心去获取某个服务的IP就可以了，不用再写死IP，整个服务的维护也变得轻松了很多**。
### 健康检查
注册服务的这一组机器，当这个服务组的某台机器，如果出现宕机或者服务死掉的时候，就会标记这个实例的状态为故障，或者干脆剔除掉这台机器。这样一来，就实现了自动监控和管理。这就是**健康检查**。

> 健康检查有多重实现方式
> 
> 比如有几秒就发一次健康检查心跳，如果返回的HTTP状态不是200，那么就判断这台服务不可用，对它进行标记。
> 
> 也可以执行一个shell脚本，看执行的返回结果，来标记状态等等。

当服务不可用的时候，有些名字服务软件也会提供发送邮件或者消息功能，及时的提示你服务出现故障。这样一来，我们就通过健康检查功能，来帮我们及时的去规避问题，降低影响。

当出现故障的服务被修复后，服务重新启动后，健康检查会检查通过，然后这台机器就会被标记为健康，这样，服务发现，就又可以发现这台机器了。

这样，整个服务注册和服务发现，就实现了闭环。
## 服务注册和发现的特点总结

1. **集群：** 得组成集群，这样单台出现故障，不至于服务宕机
2. **数据同步：** 组成了集群，得要数据同步，注册的信息，在1台注册了，在其他机器上也能看到，不然的话，1台挂了，他这台的数据都丢失了。
3. **强一致性：** 数据同步，在多台要有一致性的要求，保证数据不会出现不一致的情况。
4. **高并发高可用：** 要能保证请求量比较大的情况下，服务还能保持高可用。
5. **选举机制：** 在有集群和数据同步以及一致性要求的情况下，得有一个master来主持整个运作，那就要有选取机制，确保选举公平和稳定。
6. **分布式:** 随着微服务上云，各个机器可能近在眼前，却远在天边，如何支撑分布式上的不同环境的机器互联，这也是一个很大的问题。
7. **安装简单：** 一个软件好不好用，是否亲民，安装的易用性是一个很大的因素，如果一个软件安装简单，调试方便，那么就会很受欢迎。
## Grpc 示例
### 服务注册
```go
// ServiceRegister 服务注册。expire表示过期时间,serviceName和serviceAddr分别是服务名与服务地址
func (e *EtcdRegister) ServiceRegister(serviceName, serviceAddr string, expire int64) (err error) {

	// 创建租约CreateLease 创建租约 expire表示有效期(s)
	err = e.CreateLease(expire)
	if err != nil {
		return err
	}

	// 将租约与k-v绑定BindLease 
	err = e.BindLease(serviceName, serviceAddr)
	if err != nil {
		return err
	}

	// 持续续租KeepAlive 获取续约通道 并 持续续租
	err = e.KeepAlive()
	return err
}
```
CreateLease 和 BindLease 为服务设置了有效时间

接下来需要持续续约，以保持当前服务可被发现，算是有**高可用**的思想
```go
// KeepAlive 获取续约通道 并 持续续租
func (e *EtcdRegister) KeepAlive() error {
	keepAliveChan, err := e.etcdCli.KeepAlive(context.Background(), e.leaseId)

	if err != nil {
		log.Println(err.Error())
		return err
	}
	// 只读的通道，通过自旋锁持久续约，并打印续约消息
	go func(keepAliveChan <-chan *clientv3.LeaseKeepAliveResponse) {
		for {
			select {
			case resp := <-keepAliveChan:
				log.Printf("续约成功...leaseID=%d", resp.ID)
			}
		}
	}(keepAliveChan)

	return nil
}
```

其实这个功能，类似微服务的**心跳检测**，确保了该服务实例是有效的
### 服务发现
读取etcd的服务并开启协程监听kv变化，自动更新服务列表。

> 为什么这个`ServiceDiscovery`只在启用时调用了一次，之后如果出现了问题不需要调用吗？
> 
> 这是因为如果出现了服务故障导致当前xx服务不可用，说明从 grpc 从 Etcd 得到的服务信息是有误的，必须要更新。所以在调用时，grpc 框架仍然会根据已有的服务键获取xx服务的信息列表并通过一些选举方法返回一个有效的信息，并不需要手动调用该函数修改。
> 
> 注册服务时，因为已经有了一个类似心跳检测的续约，可以自动激活某个服务实例。让grpc发现这个服务配置信息是可用的
> 
> 但是需要注意的是，在列表中**可用的服务数量可能是不断减少的**，他只能从已有的可用服务中选举一个来接收 RPC 调用，如果所有的可用服务都坏掉了（不太可能hh了，但是也可以讨论一下），才会真的无法调用。
> 
> 
> 注意：**代码中`GetService`的功能只是更新了serviceMap中关于服务名称的Map，只能返回一个服务的名称，所以负载均衡暂未实现**


```go
// ServiceDiscovery 读取etcd的服务并开启协程监听kv变化
func (e *EtcdDiscovery) ServiceDiscovery(prefix string) error {
	// 根据服务名称的前缀，获取所有的注册服务
	resp, err := e.cli.Get(context.Background(), prefix, clientv3.WithPrefix())
	if err != nil {
		return err
	}

	// 遍历key-value存储到本地map
	for _, kv := range resp.Kvs {
		e.putService(string(kv.Key), string(kv.Value))
	}

	// 开启监听协程，监听prefix的变化
	go func() {
		watchRespChan := e.cli.Watch(context.Background(), prefix, clientv3.WithPrefix())
		log.Printf("watching prefix:%s now...", prefix)
		for watchResp := range watchRespChan {
			for _, event := range watchResp.Events {
				switch event.Type {
				case mvccpb.PUT: // 发生了修改或者新增
					e.putService(string(event.Kv.Key), string(event.Kv.Value)) // ServiceMap中进行相应的修改或新增
				case mvccpb.DELETE: //发生了删除
					e.delService(string(event.Kv.Key)) // ServiceMap中进行相应的删除
				}
			}
		}
	}()

	return nil
}
```

获取服务对象的**键值对RPC调用**
```go
	// 服务的RPC调用实例，一个map
	instance := make(map[string]interface{})
	// 创建etcd连接实例
	etcdAddress := viper.GetString("etcd.address")
	serviceDiscovery, err := etcd.NewServiceDiscovery([]string{etcdAddress})

	if err != nil {
		logger.Log.Fatal(err)
	}
	defer serviceDiscovery.Close()
	
	// 获取用户服务实例
	err = serviceDiscovery.ServiceDiscovery("user_service")
	if err != nil {
		logger.Log.Fatal(err)
	}
	userServiceAddr, _ := serviceDiscovery.GetService("user_service")
	userConn, err := grpc.Dial(userServiceAddr, grpc.WithTransportCredentials(insecure.NewCredentials()))
	if err != nil {
		fmt.Println("获取视频服务实例出错")
		logger.Log.Fatal(err)
	}
	userClient := service.NewUserServiceClient(userConn)
	logger.Log.Info("获取用户服务实例--成功--")
	instance["user_service"] = userClient

```
