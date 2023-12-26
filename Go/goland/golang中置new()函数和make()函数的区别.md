[【go】golang中置new()函数和make()函数的区别 - 踏雪无痕SS - 博客园 (cnblogs.com)](https://www.cnblogs.com/chenpingzhao/p/9918062.html)
## new
内建函数 new 用来分配内存，第一个参数是一个类型，不是一个值，返回值是一个指向分配零值的指针

new和其他语言中的同名函数一样，new(t)分配了零值填充的类型为T内存空间，并且返回其地址，即一个*t类型的值。**返回的永远是类型的指针，指向分配类型的内存地址**

它并不初始化内存，只是将其置零。*t指向的内容的值为零（zero value）。注意并不是指针为零。
## make
内建函数 make 用来为 slice，map 或 chan 类型分配内存和初始化一个对象(注意：只能用在这三种类型上)
## 总结

new和make都在堆上分配内存，但是它们的行为不同，适用于不同的类型。

new(T) 为每个新的类型T分配一片内存，初始化为 0 并且返回类型为*T的内存地址：这种方法 **返回一个指向类型为 T，值为 0 的地址的指针**，它适用于值类型如数组和结构体；它相当于 &T{}。

make(T) **返回一个类型为 T 的初始值**，是三个引用类型本身，它只适用于3种内建的引用类型：slice、map 和 channel。

换言之，new 函数分配内存，make 函数初始化；

#### 其实new不常用

所以有new这个内置函数，可以给我们分配一块内存让我们使用，但是现实的编码中，它是不常用的。我们通常都是采用短语句声明以及结构体的字面量达到我们的目的，比如：

```go
i:=0
u:=user{}
```

这样更简洁方便，而且不会涉及到指针这种比麻烦的操作。

make函数是无可替代的，我们在使用slice、map以及channel的时候，还是要使用make进行初始化，然后才可以对他们进行操作。
```go
package main
 
import (
    "fmt"
)
 
func main() {
    p := new([]int) //p == nil; with len and cap 0
    fmt.Println(p)
 
    v := make([]int, 10, 50) // v is initialed with len 10, cap 50
    fmt.Println(v)
 
    /*********Output****************
        &[]
        [0 0 0 0 0 0 0 0 0 0]
    *********************************/
 
    (*p)[0] = 18        // panic: runtime error: index out of range
                        // because p is a nil pointer, with len and cap 0
    v[1] = 18           // ok
     
}
```