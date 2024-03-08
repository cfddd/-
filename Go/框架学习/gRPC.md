> https://www.bookstack.cn/read/learning-grpc/introduction-information.md
> [【狂神说】gRPC最新超详细版教程通俗易懂 | Go语言全栈教程_哔哩哔哩_bilibili](https://www.bilibili.com/video/BV1S24y1U7Xp/?spm_id_from=333.337.search-card.all.click)
## 安装
### 1. Proto buffers

> [安装链接 Releases · protocolbuffers/protobuf (github.com)](https://github.com/protocolbuffers/protobuf/releases)

然后将解压出来的 bin 目录添加到环境变量，使用 ` protoc --version` 命令检测是否安装成功

### 2. gRPC库
```sh
go get google.golang.org/grpc
```
### 3. go相关代码生成器
protoc-gen-go
```sh
go install google.golang.org/protobuf/cmd/protoc-gen-go@v1.28
go install google.golang.org/grpc/cmd/protoc-gen-go-grpc@v1.2
```

```protobuf
protoc --go_out=. hello.proto
protoc --go-grpc_out=. hello.proto
```
### 简单RPC Server+Client
```protobuf
// 使用 proto3 语法  
syntax = "proto3";  
// 生成最后的go文件路径，'.'代表当前目录，service代表包名  
option go_package = ".;service";  
  
package helloworld;  
  
// 定义一个服务，服务中用一个方法，接受客户端 参数，再返回服务端 响应  
service Greeter {  
  // RPC方法 SayHello  rpc SayHello (HelloRequest) returns (HelloReply) {}  
}  
  
// message 类似结构体  
// 注意后面的并不是赋值，是定义这个变量在message中的位置（顺序号）  
message HelloRequest {  
  string name = 1;  
//  int64 age = 2;  
}  
  
// 同理，返回类型  
message HelloReply {  
  string message = 1;  
}
```
再运行上面生成代码的指令即可
## proto 文件字段介绍
> **message**

类似一个结构体
```proto
message Point {
  int32 latitude = 1;
  int32 longitude = 2;
}
```
> **字段规则**

- optional
	可选字段，默认
- repeated
    可重复字段，go中定义是切片

> message 消息是可以嵌套的

> **rpc 服务定义**

```proto
// 定义一个服务，服务中用一个方法，接受客户端 参数，再返回服务端 响应  
service Greeter {  
  // RPC方法 SayHello  rpc SayHello (HelloRequest) returns (HelloReply) {}  
}
```
## 服务端编写流程
```go
package main  
  
type Server struct {  
    pb.GreeterServer  
}  
  
func (*Server) SayHello(ctx context.Context, req *pb.HelloRequest) (*pb.HelloReply, error) {  
    return &pb.HelloReply{Message: "hello " + req.Name}, nil  
}  
  
func (*Server) mustEmbedUnimplementedGreeterServer() {}  
  
func main() {  
    // 开启端口  
    listener, _ := net.Listen("tcp", ":9090")  
    // 启动grpc服务器  
    grpcServer := grpc.NewServer()  
    // 注册服务  
    pb.RegisterGreeterServer(grpcServer, &Server{})  
    // 启动服务  
    grpcServer.Serve(listener)  
  
}
```

1. 创建 Server 对象，实现对proto文件生成 go 代码中接口类型的实现
2. 将Server 注册到gRPC内部的注册中心
3. 创建Listen监听TCP端口
4. gRPC 服务启动
## 客户端编写

- 创建接口类型的实现
- 创建server客户端链接对象
- 发送 RPC 请求，等待响应
- 接收响应

```go
func main() {
	conn, err := grpc.Dial("127.0.0.1:9090", grpc.WithTransportCredentials(insecure.NewCredentials()))
	if err != nil {
	}
	defer conn.Close()

	client := pb.NewGreeterClient(conn)

	resp, _ := client.SayHello(context.Background(), &pb.HelloRequest{Name: "cfd"})

	fmt.Println(resp)
}
```
## SSL 认证方式
1. 安装 openSSL [Win32/Win64 OpenSSL Installer for Windows - Shining Light Productions (slproweb.com)](https://slproweb.com/products/Win32OpenSSL.html)安装后添加 bin目录到环境变量
2. 再运行下面的命令
```sh
openssl genrsa -out server.key 2048
openssl req -new -x509 -key server.key -out server.crt -days 36500
openssl req -new -key server.key -out server.csr
```
![](http://douyin.cfddfc.online/myPicture/20240226111959.png)

`openssl req -new -nodes -key test.key -out test.csr -days 3650 -subj "/C=cn/OU=myorg/O=mycomp/CN=myname" -config ./openssl.cfg -extensions v3_req`

`openssl x509 -req -days 365 -in test.csr -out test.pem -CA server.crt -CAkey server.key -CAcreateserial -extfile ./openssl.cfg -extensions v3_req`
## Token 认证方式
...