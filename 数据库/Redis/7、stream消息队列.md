```sh
XADD geekhour * course redis
"1686495710450-0"
XADD geekhour * course git
"1686495727844-0"
XADD geekhour * course docker
"1686495730661-0"
XLEN geek
(integer) 3

XRANGE geekhour - + 
1) 1) "1686495710450-0"
	2)  1)"course"
		2) "redis"
2) 1) "1686495727844-0"
	2)  1)"course"
		2) "git"
3) 1）"1686495730661-0"
	2)  1) "course"
		2)"docker"

XDEL geekhour 1686495710450-0
XTRIM geekhour MAXLEN 0
(integer) 2

XREAD COUNT 2 BLOCK 1000 STREAMS geekhour 0

XREAD COUNT 2 BLOCK 1000 STREAMS geekhour 1

```
## 消费者组
```
XGROUP CREATE geekhour group1 0
"oK"
XINFO GROUPs geekhour
1) 1) "name"
   2) "group1" 
   3) "consumers" 
   4) "0"
   5) "pending " 
   6) "0"
   7) "last-delivered-id"
   8) "0-0"
   9) "entries-read"
   10) "null"
   11) "lag"
   12) "4"
XGROUP CREATECONSUMER geekhour group1 consumer1
(integer) 1
XGROUP CREATECONSUMER geekhour group1 consumer2
(integer) 1
XGROUP CREATECONSUMER geekhour group1 consumer3
(integer) 1
XINFO GROUPS geekhour
```