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