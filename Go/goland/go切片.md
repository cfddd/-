> [Slice · Go语言圣经 (studygolang.com)](https://books.studygolang.com/gopl-zh/ch4/ch4-02.html)
> [Go 语言切片(Slice) | 菜鸟教程 (runoob.com)](https://www.runoob.com/go/go-slice.html)
## slice构成
一个slice由三个部分构成：指针、长度和容量。
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
## 关于切片的底层原理
字符串的切片操作和[]byte字节类型切片的切片操作是类似的。都写作x[m:n]，并且都是返回一个原始字节序列的子序列，底层都是共享之前的底层数组，因此这种操作都是常量时间复杂度。x[m:n]切片操作对于字符串则生成一个新字符串，如果x是[]byte的话则生成一个新的[]byte。

因为slice值包含指向第一个slice元素的指针，因此向函数传递slice将允许在函数内部修改底层数组的元素。**换句话说，复制一个slice只是对底层的数组创建了一个新的slice别名**（§2.3.2）。

和数组不同的是，slice之间不能比较，因此我们不能使用 == 操作符来判断两个slice是否含有全部相等元素。
