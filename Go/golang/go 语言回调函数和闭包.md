> [Go基础系列：函数(2)——回调函数和闭包 - 骏马金龙 - 博客园 (cnblogs.com)](https://www.cnblogs.com/f-ck-need-u/p/9878898.html)

当函数具备以下两种特性的时候，就可以称之为高阶函数(high order functions)：

1. 函数可以作为另一个函数的参数(典型用法是回调函数)
2. 函数可以返回另一个函数，即让另一个函数作为这个函数的返回值(典型用法是闭包)

一般来说，附带的还具备一个特性：函数可以作为一个值赋值给变量。

**由于Go中函数不能嵌套命名函数，所以函数返回函数的时候，只能返回匿名函数。**

在Go语言中，回调函数和闭包都是函数式编程的重要概念，它们都可以用于在函数间传递和操作函数。下面分别介绍一下回调函数和闭包在Go语言中的概念和用法：

### 回调函数（Callback Functions）：

回调函数是指作为参数传递给其他函数的函数。在Go语言中，函数和变量的地位相同，是一等公民，因此可以像其他数据类型一样作为参数传递给函数。通过使用回调函数，可以在函数内部调用外部定义的函数，实现更灵活和可复用的代码结构。

示例：

```go
package main

import "fmt"

// 回调函数，接收一个int类型参数并返回一个int类型结果
func callbackFunc(x int) int {
    return x * x
}

// 函数f接收一个int类型参数和一个函数类型参数，函数类型参数的类型是func(int) int
func f(y int, callback func(int) int) int {
    return callback(y) // 调用传入的回调函数
}

func main() {
    result := f(3, callbackFunc) // 将callbackFunc作为回调函数传递给函数f
    fmt.Println(result) // 输出结果：9
}
```
### 使用回调函数实现类似函数重载的效果
在Go的sort包中有一个很强大的Slice排序工具SliceStable()，它就是将排序函数作为参数的：

```go
func SliceStable(slice interface{}, less func(i, j int) bool)
```

`less` 作为一个回调函数，把 i，j 作为slice中的下标，返回比较结果，和 C++ 中的 `sort` 类似
### 闭包（Closure）：

闭包是指在函数内部定义的函数，它可以访问其外部函数中的变量。在Go语言中，闭包可以帮助我们实现类似于函数式编程的一些特性，如函数柯里化、延迟执行等。

**闭包中的变量被分配到堆上。**

示例：

```go
package main

import "fmt"

// 函数返回一个函数，内部的函数形成了闭包，可以访问外部函数的变量x
func adder(x int) func(int) int {
    return func(y int) int {
        return x + y
    }
}

func main() {
    // 使用闭包创建一个加法器函数
    addTwo := adder(2)
    // 使用闭包创建一个减法器函数
    subtractFive := adder(-5)

    // 调用加法器函数
    fmt.Println(addTwo(3)) // 输出结果：5 （2+3）
    // 调用减法器函数
    fmt.Println(subtractFive(10)) // 输出结果：5 （-5+10）
}
```

在这个示例中，`adder`函数返回一个闭包，闭包可以访问外部函数中的变量`x`，从而实现了对外部变量的“捕获”。闭包`addTwo`和`subtractFive`分别是加法器和减法器函数，它们可以分别执行加法和减法操作。

总的来说，**回调函数和闭包是Go语言中非常常用的函数式编程的概念，它们都能够实现函数间的灵活传递和操作，使得代码更加灵活和可复用**。
