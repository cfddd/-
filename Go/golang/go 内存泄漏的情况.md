## groutine 
1. groutine 使用完未关闭
	groutine一直没使用，但是因为还有引用所以不会被gc掉
2. groutine 关闭后缓冲区仍有数据
	groutine还有残留数据，所以也没被gc

### 切片
切片的底层原理是对数组的引用和长度和容量

既然是引用，如果一个切片引用的底层数组非常大，但是切片实际的长度和容量非常小，完全不在一个数量级，就会导致剩余的数组泄露，而且不会被GC回收

```go
var a []int = make([]int,100000,0)

func test(b []int) {
    a = b[:3]        
    return
}
```