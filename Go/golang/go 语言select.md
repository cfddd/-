> [条件语句select - 地鼠文档 (topgoer.cn)](https://www.topgoer.cn/docs/golang//977)

select 是 Go 中的一个控制结构，类似于 switch 语句。

select 语句只能用于通道操作，每个 case 必须是一个通道操作，要么是发送要么是接收。

select 语句会监听所有指定的通道上的操作，一旦其中一个通道准备好就会执行相应的代码块。

如果多个通道都准备好，那么 select 语句会随机选择一个通道执行。如果所有通道都没有准备好，那么执行 default 块中的代码。

```go
select {
  case <- channel1:
    // 执行的代码
  case value := <- channel2:
    // 执行的代码
  case channel3 <- value:
    // 执行的代码

    // 你可以定义任意数量的 case

  default:
    // 所有通道都没有准备好，执行的代码
}
```