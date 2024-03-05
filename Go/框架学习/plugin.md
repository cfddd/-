> [一文搞懂Go语言的plugin - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/391197032)
> [一文详解搞懂Go语言的plugin - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/670127699)
> [6.5840 Lab 1: MapReduce (mit.edu)](https://pdos.csail.mit.edu/6.824/labs/lab-mr.html)

### **1.** **引言**

Go plugin系统自Go 1.8版本引入，它允许Go应用动态加载预编译的代码模块，从而实现了模块的热插拔功能。

### **2.** **Go** **Plugin包的基础**

Go的plugin包提供了动态加载共享库（.so文件）的能力。这些文件是Go代码编译的结果，可以在运行时被加载。

### **3.** **创建和使用Go** **Plugins**

创建一个plugin，首先需要编写一个普通的Go包，例如：

```go
package main

import "fmt"

func Greet(name string) {
    fmt.Println("Hello", name)
}
```

然后使用`go` `build` `-buildmode=plugin`命令编译成.so文件。

加载和使用这个plugin：

```go
package main

import (
    "plugin"
    "log"
)

func main() {
    p, err := plugin.Open("path_to_plugin.so")
    if err != nil {
        log.Fatal(err)
    }
    greet, err := p.Lookup("Greet")
    if err != nil {
        log.Fatal(err)
    }
    greet.(func(string))("Go")
}
```
### 4. 6824中的使用解读

```sh
# 编译插件
go build -buildmode=plugin ../mrapps/wc.go
# 在命令行中使用插件
go run mrsequential.go wc.so pg*.txt
```

```go
// load the application Map and Reduce functions
// from a plugin file, e.g. ../mrapps/wc.so
func loadPlugin(filename string) (func(string, string) []mr.KeyValue, func(string, []string) string) {
	p, err := plugin.Open(filename)
	if err != nil {
		log.Fatalf("cannot load plugin %v", filename)
	}
	xmapf, err := p.Lookup("Map")
	if err != nil {
		log.Fatalf("cannot find Map in %v", filename)
	}
	mapf := xmapf.(func(string, string) []mr.KeyValue)
	xreducef, err := p.Lookup("Reduce")
	if err != nil {
		log.Fatalf("cannot find Reduce in %v", filename)
	}
	reducef := xreducef.(func(string, []string) string)

	return mapf, reducef
}
```
简单来说，就是以下几个步骤
- plugin.Open 通过文件名，导入编译好的插件
- Lookup 从插件中得到名为参数的函数调用
- 通过函数的正确类型，**断言**出需要调用的函数
```go
func main() {
	if len(os.Args) < 3 {
		fmt.Fprintf(os.Stderr, "Usage: mrsequential xxx.so inputfiles...\n")
		os.Exit(1)
	}

	mapf, reducef := loadPlugin(os.Args[1])
}
```