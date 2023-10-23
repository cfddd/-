## 常用命令
|命令|介绍|
|---|---|
|SET key value|设置指定 key 的值|
|SETNX key value|只有在 key 不存在时设置 key 的值|
|GET key|获取指定 key 的值|
|MSET key1 value1 key2 value2 …|设置一个或多个指定 key 的值|
|MGET key1 key2 ...|获取一个或多个指定 key 的值|
|STRLEN key|返回 key 所储存的字符串值的长度|
|INCR key|将 key 中储存的数字值增一|
|DECR key|将 key 中储存的数字值减一|
|EXISTS key|判断指定 key 是否存在|
|DEL key（通用）|删除指定的 key|
|EXPIRE key seconds（通用）|给指定 key 设置过期时间|
## 增删改查
```
127.0.0.1:6379> SET name geekhour
0K
127.0.0.1:6379> GET name"geekhour"
127.0.0.1:6379>SET Name GeekHour
0K
127.0.0.1:6379> GET Name"GeekHour"
127.0.0.1:6379> SET age 25
0K
127.0.0.1:6379> GET age"25"
127.0.0.1:6379> DEL name
(integer) 1
127.0.0.1:6379> GET name
(nil)
127.0.0.1:6379> EXISTS name
(integer)0
127.0.0.1:6379> EXISTS age
(integer)1
127.0.0.1:6379> KEYS *
1) "age"
2）"Name"

127.0.0.1:6379> FLUSHALL   //清空
127.0.0.1:6379> KEYS *
(empty array)
127.0.0.1:6379>1

```
## 设置中文
```
127.0.0.1:6379>SET name 一键三连
OK
127.0.0.1:6379> GET name
"\xe4\xb8\x80\xe9\x94\xae\xe4\xb8\×89\xe8\xbf\x9e"
127.0.0.1:6379>quit
～
# 中文原始模式进入
redis-cli --raw     
127.0.0.1:6379> GET name
一键三连

# 清屏
clear
```
## 设置过期时间
```
127.0.0.1:6379> TTL name
-1
127.0.0.1:6379> EXPIRE name 10  # 设置过期时间
1
127.0.0.1:6379> TTL name
9
127.0.0.1:6379> TTL name
7
127.0.0.1:6379> TTL name
6
127.0.0.1:6379> TTL name
5
127.0.0.1:6379> GET name 
一键三连
127.0.0.1:6379> TTL name
1
127.0.0.1:6379> TTL name
-2
127.0.0.1:6379> GET name
    # 没有输出，过期，也不显示有这个key
```
同时设置
```
127.0.0.1:6379> SETEX name 5 一键三连
0K
127.0.0.1:6379> TTL name
2
127.0.0.1:6379> TTL name
1
127.0.0.1:6379> TTL name
0
127.0.0.1:6379> TTL name
-2
# SETNX表示，只有在这个键不存在时，才会创建
127.0.0.1:6379> SETNX name 

```