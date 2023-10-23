> [golang-guide/Redis/面经/Redis.md at main · mao888/golang-guide (github.com)](https://github.com/mao888/golang-guide/blob/main/Redis/%E9%9D%A2%E7%BB%8F/Redis.md)
## 简介
Redis（Remote Dictionary Server）是一个快速、开源、内存数据存储系统，可用作数据库、缓存和消息中间件。它支持多种数据结构，适用于多种应用场景。
### 为什么需要Redis
- 数据从单表，演进出了分库分表 
- MySQL从单机演进出了集群 
	- 数据量增长
	- 读写数据压力的不断增加
- 数据分冷热 
	 - 热数据：经常被访问到的数据
	 - 冷数据：不经常被访问到的数据
- 将热数据存储到内存中
### 区别于其他 key-value 存储 
- 复杂数据结构：提供复杂的数据结构和对应的原子性操作。
- 内存与持久化：运行在内存中，但可以持久化到磁盘。 
- 简单操作：在内存中操作简单，磁盘格式紧凑，追加写入方式。 
## 适用场景
- 缓存：提高数据访问速度，减轻数据库压力。
- 实时统计：用于计数、排名等实时统计场景。
- 会话存储：用于跨多台服务器的状态共享。
- 排行榜：通过有序集合实现游戏中的玩家排名。
- 任务队列：使用列表结构实现异步任务处理。
- 地理位置存储：存储和查询位置信息。
## 基本概念
- **键值存储**: Redis 是键值存储系统，每个值可以是字符串、哈希、列表、集合、有序集合等。
- **内存存储**: 数据存储在内存中，因此读写操作快速，但数据会在适当时候持久化到磁盘。
- **数据结构**: Redis 提供字符串、哈希、列表、集合、有序集合等多种数据结构。
- **持久化**: Redis 支持将数据持久化到磁盘，以便在重启后恢复数据。
- **发布-订阅模式**: Redis 支持发布消息和订阅消息，用于实现实时消息传递。

## 常用命令示例

### 字符串操作
string 类型是 Redis 最基本的数据类型，string 类型的值最大能存储 512MB。
```redis
SET username "john"
GET username
```

### 哈希操作
Redis hash 是一个键值(key=>value)对集合。

Redis hash 是一个 string 类型的 field 和 value 的映射表，hash 特别适合用于存储对象。
```redis
HSET user:1 name "Alice"
HSET user:1 age 25
HGETALL user:1
```

### 列表操作
Redis 列表是简单的字符串列表，按照插入顺序排序。你可以添加一个元素到列表的头部（左边）或者尾部（右边）。
```redis
RPUSH tasks "task1"
RPUSH tasks "task2"
LRANGE tasks 0 -1
```

### 集合操作
Redis 的 Set 是 string 类型的无序集合。

集合是通过哈希表实现的，所以添加，删除，查找的复杂度都是 O(1)。

- sadd 命令
	添加一个 string 元素到 key 对应的 set 集合中，成功返回 1，如果元素已经在集合中返回 0。
```redis
SADD tags "tag1"
SADD tags "tag2"
SMEMBERS tags
```

### 有序集合操作
- zadd 命令
	添加元素到集合，元素在集合中存在则更新对应score
```redis
ZADD scores 100 "player1"
ZADD scores 200 "player2"
ZREVRANGE scores 0 -1 WITHSCORES
```
### 原子性操作

Redis 所有操作都是原子性的，意味着每个操作要么完全执行成功，要么完全不执行，保证了数据的一致性和可靠性。此外，Redis 还支持事务操作，即将多个操作封装成一个事务，在 EXEC 指令时一次性执行，保证了多个操作的原子性。

### 丰富的特性

Redis 还提供了许多其他特性，包括：

- **发布-订阅模式**：支持消息的发布和订阅，用于实现实时消息传递。
    ```redis
    SUBSCRIBE channel-name
	PUBLISH channel-name "Hello, Redis!"
	```
- **通知**：支持键的事件通知，如键过期、删除等。
    
- **过期键**：可以设置键在一定时间后自动过期，从而释放内存空间。
## 总结
Redis 是一个多功能的内存数据存储系统，适用于多种应用场景。了解基本概念和常用命令，可以在你的项目中充分利用 Redis 的优势，提高应用性能和灵活性。

## 在GO中使用redis
- [go-redis开发手册 - Go语言玩转Redis的正确姿势 - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/645669818)
- [go-redis使用入门 - 掘金 (juejin.cn)](https://juejin.cn/post/7202521955366879288)

## 参考
- [Redis 教程 | 菜鸟教程 (runoob.com)](https://www.runoob.com/redis/redis-tutorial.html)
- ﻿[​﻿‌‬​⁢⁤​⁢‍⁢‌⁣‌‍⁣​⁤⁡⁡⁡⁣​⁣‌⁣﻿​⁡​﻿‍‬‬⁤‌‌⁡⁡‬⁣‌‌‌⁣​﻿‌Redis-大厂程序员是怎么用的.pptx - 飞书云文档 (feishu.cn)](https://bytedance.feishu.cn/file/TcbGb6isWoTKbLxCaqfc77Qqnee)
- [Redis【入门】就这一篇！ - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/37982685)