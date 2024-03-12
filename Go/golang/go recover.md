> [一文初探Go的defer、panic和recover - 掘金 (juejin.cn)](https://juejin.cn/post/7197361396220100663)
> [【Go 基础篇】Go语言中的defer和recover：优雅处理错误_go defer recover-CSDN博客](https://blog.csdn.net/qq_21484461/article/details/132515825)

## 作用

`recover`是Go语言的内置函数，用于从 `panic` 中恢复并返回一个错误值。

它只能在延迟函数（`defer`语句）内部调用，用于捕获并处理由`panic`引起的恐慌。

如果没有发生恐慌，或者`recover`不在延迟函数中调用，它会返回`nil`。

##  实例
前
```go
func main() {
  AAA()
  BBB()
  CCC()
}

func AAA() {
  fmt.Println("func AAA")
}
func BBB() {
  panic("func BBB")
}
func CCC() {
  fmt.Println("func CCC")
}

//输出结果：
//func AAA
//panic: func BBB
//
//goroutine 1 [running]:
//main.BBB(...)
//    .../09_func_panic_recover.go:27
//main.main()
//    .../09_func_panic_recover.go:7 +0x96
//exit status 2


```
后
```go
func main() {
  AAA()
  BBB()
  CCC()
}

func AAA() {
  fmt.Println("func AAA")
}

func BBB() {
  defer func() {
    if err := recover(); err != nil {
      fmt.Println("recover in func BBB")
      fmt.Println(fmt.Sprintf("%T %v", err, err))
      //...handle  打日志等
    }
  }()

  panic("func BBB") //panic之前，如果没有recover，则程序会崩溃，异常退出。有则继续执行
}

func CCC() {
  fmt.Println("func CCC")
}

//输出结果：
//func AAA
//recover in func BBB
//string func BBB
//func CCC

```