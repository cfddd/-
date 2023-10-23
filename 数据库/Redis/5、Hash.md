## 常用命令
|命令|介绍|
|---|---|
|HSET key field value|设置指定哈希表中指定字段的值|
|HSETNX key field value|只有指定字段不存在时设置指定字段的值|
|HMSET key field1 value1 field2 value2 ...|同时将一个或多个 field-value (域-值)对设置到指定哈希表中|
|HGET key field|获取指定哈希表中指定字段的值|
|HMGET key field1 field2 ...|获取指定哈希表中一个或者多个指定字段的值|
|HGETALL key|获取指定哈希表中所有的键值对|
|HEXISTS key field|查看指定哈希表中指定的字段是否存在|
|HDEL key field1 field2 ...|删除一个或多个哈希表字段|
|HLEN key|获取指定哈希表中字段的数量|
|HINCRBY key field increment|对指定哈希中的指定字段做运算操作（正数为加，负数为减）|
## 用法
```sh
127.0.0.1:6379> HSET person name laoyang
1
127.0.0.1:6379> HSET person age 100
1
127.0.0.1:6379> HGET person name
laoyang
127.0.0.1:6379> HGET person age
100
127.0.0.1:6379> HGETALL person name
laoyangage
100
127.0.0.1:6379> HDEL person age
1
127.0.0.1:6379> HGETALL person name
laoyang
127.0.0.1:6379> HEXISTS person name
1
127.0.0.1:6379> HEXISTS person age
0

```
