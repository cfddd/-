- [`reflect.TypeOf`](https://draveness.me/golang/tree/reflect.TypeOf) 能获取类型信息；
- [`reflect.ValueOf`](https://draveness.me/golang/tree/reflect.ValueOf) 能获取数据的运行时表示；
两个类型是 [`reflect.Type`](https://draveness.me/golang/tree/reflect.Type) 和 [`reflect.Value`](https://draveness.me/golang/tree/reflect.Value)，它们与函数是一一对应的关系：
```go
typeI := reflect.TypeOf(1)  
typeS := reflect.TypeOf("hello")  
fmt.Println(typeI) //int  
fmt.Println(typeS) //string  
  
valueI := reflect.ValueOf(1)  
valueS := reflect.ValueOf("hello")  
fmt.Println(valueI) //1  
fmt.Println(valueS) //hello
```

 [`reflect.Type`](https://draveness.me/golang/tree/reflect.Type) 是一个接口
 [`reflect.Value`](https://draveness.me/golang/tree/reflect.Value) 是一个结构体

--- 
> [Go 语言反射的实现原理 | Go 语言设计与实现 (draveness.me)](https://draveness.me/golang/docs/part2-foundation/ch04-basic/golang-reflect/)