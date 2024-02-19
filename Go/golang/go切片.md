> [Slice · Go语言圣经 (studygolang.com)](https://books.studygolang.com/gopl-zh/ch4/ch4-02.html)
> [Go 语言切片(Slice) | 菜鸟教程 (runoob.com)](https://www.runoob.com/go/go-slice.html)
## slice构成
一个slice由三个部分构成：**指针**、**长度和容量**。
- 指针指向第一个slice元素对应的底层数组元素的地址，要注意的是slice的第一个元素并不一定就是数组的第一个元素。
- 长度对应slice中元素的数目；
- 长度不能超过容量，容量一般是从slice的开始位置到底层数据的结尾位置。内置的len和cap函数分别返回slice的长度和容量。
## 示例
起因于如何使用`append`在一个切片中插入一个元素
最简单粗暴的办法就是再写一个
```
func insertX(plaintext []byte, x byte) []byte {
    result := make([]byte, 0, len(plaintext)*2)

    for i := 0; i < len(plaintext)-1; i++ {
        result = append(result, plaintext[i])

        if plaintext[i] == plaintext[i+1] {
            result = append(result, x)
        }
    }

    // 添加最后一个字母
    result = append(result, plaintext[len(plaintext)-1])

    return result
}

func main() {
    plaintext := []byte("balloon")
    x := byte('x')

    result := insertX(plaintext, x)
    fmt.Println(string(result))
}
```
这个方法很不错，但是比较是学习切片，要上点难度
```
func insertX(plaintext []byte, x byte) []byte {
    result := make([]byte, 0, len(plaintext))

    for i := 0; i < len(plaintext)-1; i++ {
        result = append(result, plaintext[i])

        if plaintext[i] == plaintext[i+1] {
            result = append(result, x)
        }
    }

    // 添加最后一个字母
    result = append(result, plaintext[len(plaintext)-1])

    return result
}

func main() {
    plaintext := []byte("balloon")
    x := byte('x')

    result := insertX(plaintext, x)
    fmt.Println(string(result))
}
```
append后面可以接受可变长切片`...`
这里我们使用`append`函数来动态地向`result`切片中添加元素。首先，我们创建了一个容量为`len(plaintext)`的空切片`result`，然后在循环中使用`append`函数将元素逐个添加到`result`中。最后，我们将最后一个字母添加到`result`中，并返回结果。
## 切片的底层原理
字符串的切片操作和[]byte字节类型切片的切片操作是类似的。都写作x[m:n]，并且都是返回一个原始字节序列的子序列，底层都是共享之前的底层数组，因此这种操作都是常量时间复杂度。x[m:n]切片操作对于字符串则生成一个新字符串，如果x是[]byte的话则生成一个新的[]byte。

因为slice值包含指向第一个slice元素的指针，因此向函数传递slice将允许在函数内部修改底层数组的元素。**换句话说，复制一个slice只是对底层的数组创建了一个新的slice别名**（§2.3.2）。

和数组不同的是，slice之间不能比较，因此我们不能使用 == 操作符来判断两个slice是否含有全部相等元素。

> [Slice底层实现 - 地鼠文档 (topgoer.cn)](https://topgoer.cn/docs/golang/chapter03-11)
### 切片的数据结构

**切片本身并不是动态数组或者数组指针。它内部实现的数据结构通过指针`引用底层数组`，设定相关属性将`数据读写操作限定在指定的区域`内。切片本身是一个只读对象，其工作机制类似`数组指针的一种封装`。**

切片（slice）是对数组一个连续片段的引用，所以切片是一个引用类型（因此更类似于 C/C++ 中的数组类型，或者 Python 中的 list 类型）。这个片段可以是整个数组，或者是由起始和终止索引标识的一些项的子集。需要注意的是，终止索引标识的项不包括在切片内。**切片提供了一个与指向数组的动态窗口。**

给定项的切片索引可能比相关数组的相同元素的索引小。和数组不同的是，切片的长度可以在运行时修改，最小为 0 最大为相关数组的长度：切片是一个长度可变的数组。

Slice 的数据结构定义如下:

```go
type slice struct {
    array unsafe.Pointer
    len   int
    cap   int
}
```

![](https://topgoer.cn/uploads/golang/images/m_b7bffbf0976c474809a222748dbc42ba_r.png "null")

切片的结构体由3部分构成，Pointer 是指向一个数组的指针，len 代表当前切片的长度，cap 是当前切片的容量。cap 总是大于等于 len 的。

![](https://topgoer.cn/uploads/golang/images/m_b3ed4a54677e9c668021c12d6cac6258_r.png "null")
### 切片扩容
**新的切片和之前的切片已经完全不同了**，因为新的切片更改了一个值，并没有影响到原来的数组，新切片指向的数组是一个全新的数组。并且 cap 容量也发生了变化。
Go 中切片扩容的策略是这样的：

- 切片的容量小于 1024 个元素，于是扩容的时候就翻倍增加容量。上面那个例子也验证了这一情况，总容量从原来的4个翻倍到现在的8个。

- 元素个数超过 1024 个元素，那么增长因子就变成 1.25 ，即每次增加原来容量的四分之一。

> 注意：扩容扩大的容量都是针对原来的容量而言的，而不是针对原来数组的长度而言的。
> 
> **数组的容量已经达到了最大值，再想扩容， Go 默认会先开一片内存区域，把原来的值拷贝过来，然后再执行 append() 操作。这种情况丝毫不影响原数组。**
> **如果原数组还有容量可以扩容，所以执行 append() 操作以后，会在原数组上直接操作，所以这种情况下，扩容以后的数组还是指向原来的数组。**