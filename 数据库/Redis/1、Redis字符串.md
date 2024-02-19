## 内部实现
String 类型的底层的数据结构实现主要是 SDS（简单动态字符串）。 SDS 和我们认识的 C 字符串不太一样，之所以没有使用 C 语言的字符串表示，因为 SDS 相比于 C 的原生字符串：

- **SDS 不仅可以保存文本数据，还可以保存二进制数据**。因为 SDS 使用 len 属性的值而不是空字符来判断字符串是否结束，并且 SDS 的所有 API 都会以处理二进制的方式来处理 SDS 存放在 buf[] 数组里的数据。所以 SDS 不光能存放文本数据，而且能保存图片、音频、视频、压缩文件这样的二进制数据。
- **SDS 获取字符串长度的时间复杂度是 O(1)**。因为 C 语言的字符串并不记录自身长度，所以获取长度的复杂度为 O(n)；而 SDS 结构里用 len 属性记录了字符串长度，所以复杂度为 O(1)。
- **Redis 的 SDS API 是安全的，拼接字符串不会造成缓冲区溢出**。因为 SDS 在拼接字符串之前会检查 SDS 空间是否满足要求，如果空间不够会自动扩容，所以不会导致缓冲区溢出的问题。
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
