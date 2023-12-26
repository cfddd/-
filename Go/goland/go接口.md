
接口类型是对**其它类型行为的抽象和概括**；因为接口类型不会和特定的实现细节绑定在一起，通过这种抽象的方式我们可以让我们的函数更加灵活和更具有适应能力。

Go语言中接口类型的独特之处在于**它是满足隐式实现**的。也就是说，如果**一个类型实现了一个接口定义的所有方法，那么它就自动地实现了该接口**。因此，我们可以通过将接口作为参数来实现对不同类型的调用，从而实现多态。

## 接口约定
当你有看到一个接口类型的值时，你不知道它是什么，唯一知道的就是可以通过它的方法来做什么。
接口可以让我们将不同的类型绑定到一组公共的方法上，从而实现多态和灵活的设计。
```go
type ByteCounter int

func (c *ByteCounter) Write(p []byte) (int, error) {
	*c += ByteCounter(len(p)) // convert int to ByteCounter
	return len(p), nil
}
func main() {
	var c ByteCounter
	c.Write([]byte("hello"))
	fmt.Println(c) // "5", = len("hello")
	c = 0          // reset the counter
	var name = "Dolly"
	fmt.Fprintf(&c, "hello, %s", name)
	fmt.Println(c) // "12", = len("hello, Dolly")
}
```
![[Pasted image 20231127204316.png]]
Fprintf 函数接受一个 io.Writer 的接口类型
要让我们的 ByteCounter 类型可以被 Fprintf 函数接受，就需要我们实现 Fprintf 函数的方法（唯一的一个 Write 方法）

## 接口类型
**接口类型具体描述了一系列方法的集合**，一个实现了这些方法的具体类型是这个接口类型的实例。
```
import "fmt"

type person interface {
	Eat() (msg string)
	Touch() (msg string)
}

type user struct{}

func (p *user) Eat() (msg string) {
	return "full"
}
func (p *user) Touch() (msg string) {
	return "hurt"
}
func main() {
	var cfd user
	fmt.Println(cfd.Eat())   //"full"
	fmt.Println(cfd.Touch()) //"hurt"
}
```

## 实现接口的条件
**一个类型如果拥有一个接口需要的所有方法，那么这个类型就实现了这个接口。**
接口指定的规则非常简单：表达一个类型属于某个接口只要这个类型实现这个接口。所以：

```go
var w io.Writer         // 只用实现 Write 方法
w = os.Stdout           // OK: *os.File 有 Write 方法
w = new(bytes.Buffer)   // OK: *bytes.Buffer 有 Write 方法
w = time.Second         // compile error: time.Duration 没有 Write 方法

var rwc io.ReadWriteCloser // 需要实现 Read, Write, Close 方法
rwc = os.Stdout         // OK: *os.File 有 Read, Write, Close 方法
rwc = new(bytes.Buffer) // compile error: *bytes.Buffer 没有 Close 方法
```

这个规则甚至适用于等式右边本身也是一个接口类型

```go 
w = rwc                 // OK: io.ReadWriteCloser 有 Write 方法
rwc = w                 // compile error: io.Writer 没有 Close 方法
```

```go
type IntSet struct {
}

func (*IntSet) String() string {
	return ""
}

var _ = IntSet{}.String()    // compile error: String 需要 *IntSet 接收
var _ = (&IntSet{}).String() // OK: s 是一个变量而 &s 有 a String 方法
```

```go
var s IntSet
var _ fmt.Stringer = &s // OK
var _ fmt.Stringer = s  // compile error: IntSet 没有 String 方法

```
由于只有 `*IntSet` 类型有 `String` 方法，所以也只有 `*IntSet` 类型实现了 `fmt.Stringer `接口：
### 空接口
```go
var any interface{}
any = true
any = 12.34
any = "hello"
any = map[string]int{"one": 1}
any = new(bytes.Buffer)
```
interface{}被称为空接口类型是不可或缺的。
因为空接口类型对实现它的类型没有要求，所以我们可以将任意一个值赋给空接口类型。

## 接口值
接口值，由两个部分组成，**一个具体的类型**和**那个类型的值**。它们被称为**接口的动态类型**和**动态值**。
> [接口值 · Go语言圣经 (studygolang.com)](https://books.studygolang.com/gopl-zh/ch7/ch7-05.html)

### 一个包含nil指针的接口不是nil接口
一个不包含任何值的nil接口值和一个刚好包含nil指针的接口值是不同的。这个细微区别产生了一个容易绊倒每个Go程序员的陷阱。
```go
package main

import (
	"bytes"
	"fmt"
	"io"
)

const debug = false

func main() {
	var buf *bytes.Buffer = nil
	if debug {
		buf = new(bytes.Buffer) // enable collection of output
	}
	//buf.Write([]byte("hello"))
	f(buf) // NOTE: subtly incorrect!
	if debug {
		// ...use buf...
	}
}

// If out is non-nil, output will be written to it.
func f(out io.Writer) {
	fmt.Printf("%T", out)
	// ...do something...
	if out != nil {
		out.Write([]byte("done!\n"))
	}
}
```
当`main`函数调用函数f时，它给`f`函数的`out`参数赋了一个`*bytes.Buffer`的空指针，所以`out`的动态值是`nil`。然而，它的动态类型是`*bytes.Buffer`，意思就是`out`变量是一个包含空指针值的非空接口，所以防御性检查`out!=nil`的结果依然是`true`。

问题在于尽管一个nil的 `*bytes.Buffer` 指针有实现这个接口的方法，它也不满足这个接口具体的行为上的要求。

解决方案就是避免一开始就将一个不完整的值赋值给这个接口：
```go
var buf io.Writer
if debug {
    buf = new(bytes.Buffer) // enable collection of output
}
f(buf) // OK
```