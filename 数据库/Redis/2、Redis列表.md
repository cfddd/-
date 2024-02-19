## 内部实现
List 类型的底层数据结构是由**双向链表或压缩列表**实现的：

- 如果列表的元素个数小于 512 个（默认值，可由 list-max-ziplist-entries 配置），列表每个元素的值都小于 64 字节（默认值，可由 list-max-ziplist-value 配置），Redis 会使用**压缩列表**作为 List 类型的底层数据结构；
- 如果列表的元素不满足上面的条件，Redis 会使用**双向链表**作为 List 类型的底层数据结构；
## 常用命令
|命令|介绍|
|---|---|
|RPUSH key value1 value2 ...|在指定列表的尾部（右边）添加一个或多个元素|
|LPUSH key value1 value2 ...|在指定列表的头部（左边）添加一个或多个元素|
|LSET key index value|将指定列表索引 index 位置的值设置为 value|
|LPOP key|移除并获取指定列表的第一个元素(最左边)|
|RPOP key|移除并获取指定列表的最后一个元素(最右边)|
|LLEN key|获取列表元素数量|
|LRANGE key start end|获取列表 start 和 end 之间 的元素|
## 用法
```
LPUSH letter a
(integer）1
LRANGE letter 0 -1
1) "a"
LPUSH letter b
(integer）2
LRANGE letter 0 -1
1) "b"
2) "a"
LPUSH letter c d e
(integer) 5
LRANGE letter 0 -1
1) "e"
2) "d" 
3) "c"
4) "b"
5) "a"

RPUSH letter f
(integer) 6
LRANGE letter 0 -1
1) "e"
2) "d"
3) "c"
4）"b"
5) "a"
6) "f"
RPOP letter
"f"
LRANGE letter 0 -1
1) "e"
2)" d"
3) "c"
4) "b" 
5) "a"

LPOP letter 2
1) "e"
2) "d"
LRANGE letter 0 -1
1) "c"
2) "b"
3) "a"
LLEN letter
(integer) 3

```

## trim修剪
```
LRANGE letter 0 -1
1) "e"
2) "d" 
3) "c" 
4) "b" 
5) "a"
LTRIM letter 1 3
"OK"
LRANGE letter 0 -1
1) "d"
2) "c"
3) "b"

```