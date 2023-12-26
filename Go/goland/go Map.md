> [Go 语言 Map(集合) | 菜鸟教程 (runoob.com)](https://www.runoob.com/go/go-map.html)
> [Map · Go语言圣经 (studygolang.com)](https://books.studygolang.com/gopl-zh/ch4/ch4-03.html)


Map 是一种无序的键值对的集合。

Map 最重要的一点是通过 key 来快速检索数据，key 类似于索引，指向数据的值。

Map 是一种集合，所以我们可以像迭代数组和切片那样迭代它。**不过，Map 是无序的，遍历 Map 时返回的键值对的顺序是不确定的。**

**在获取 Map 的值时，如果键不存在，返回该类型的零值**，例如 int 类型的零值是 0，string 类型的零值是 ""。

**Map 是引用类型，如果将一个 Map 传递给一个函数或赋值给另一个变量，它们都指向同一个底层数据结构，因此对 Map 的修改会影响到所有引用它的变量。**

Map 类型可以写为`map[K]V`。**其中K对应的key必须是支持 == 比较运算符的数据类型**，所以 Map 可以通过测试key是否相等来判断是否已经存在。

map类型的零值是nil，也就是没有引用任何哈希表。

和slice一样，**map之间也不能进行相等比较**；唯一的例外是和nil进行比较。要判断两个map是否包含相同的key和value，我们必须通过一个循环实现：

```go
func equal(x, y map[string]int) bool {
    if len(x) != len(y) {
        return false
    }
    for k, xv := range x {
        if yv, ok := y[k]; !ok || yv != xv {
            return false
        }
    }
    return true
}
```
### **Map 的并发安全性**

在多个 Goroutine 中使用 Map 时，我们需要注意 Map 的并发安全性。多个 Goroutine 对同一个 Map 进行读写操作时，可能会导致竞争条件和数据竞争等问题。为了解决这些问题，Go 语言提供了 sync 包中的 Map 类型。sync.Map 类型可以安全地在多个 Goroutine 中使用。

一般使用读写锁 sync.RWMutex 来保证安全。