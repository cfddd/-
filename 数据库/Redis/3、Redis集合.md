## 常用命令
|命令|介绍|
|---|---|
|SADD key member1 member2 ...|向指定集合添加一个或多个元素|
|SMEMBERS key|获取指定集合中的所有元素|
|SCARD key|获取指定集合的元素数量|
|SISMEMBER key member|判断指定元素是否在指定集合中|
|SINTER key1 key2 ...|获取给定所有集合的交集|
|SINTERSTORE destination key1 key2 ...|将给定所有集合的交集存储在 destination 中|
|SUNION key1 key2 ...|获取给定所有集合的并集|
|SUNIONSTORE destination key1 key2 ...|将给定所有集合的并集存储在 destination 中|
|SDIFF key1 key2 ...|获取给定所有集合的差集|
|SDIFFSTORE destination key1 key2 ...|将给定所有集合的差集存储在 destination 中|
|SPOP key count|随机移除并获取指定集合中一个或多个元素|
|SRANDMEMBER key count|随机获取指定集合中指定数量的元素|
## 用法
```
SADD course Redis
(integer) 1
SMEMBERS course
1) "Redis"
SADD course Redis
(integer）0
SMEMBERS course
1) "Redis"
SISMEMBER course Redis
(integer) 1
SISMEMBER course Python
(integer) 0
SREM course Redis
(integer) 1
SMEMBERS course
# 查看集合中的元素
```