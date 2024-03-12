
>[**Lei Qi**](https://leiqicn.gitee.io/)
>[go语言-回调函数（钩子） - Lei Qi's Blog (gitee.io)](https://leiqicn.gitee.io/2023-05-25-2cbe3a05ec00.html#%E5%9B%9E%E8%B0%83%E5%87%BD%E6%95%B0%E7%9A%84%E4%B8%8D%E5%90%8C)

## 基础用法
在Go语言中，回调函数和钩子函数通常是**使用函数类型作为参数传递给函数或方法**，以便在特定事件发生时被调用。这种机制非常灵活，可以让你编写出高效的、可复用的代码。

以下是一个简单的例子，展示了如何使用回调函数来实现一个函数，当输出文本时会同时调用传入的回调函数：
```
package main

import (
    "fmt"
)

func printWithCallback(callback func(string)) {
    text := "Hello, world!"
    fmt.Println(text)
    callback(text)
}

func main() {
    callback := func(text string) {
        fmt.Printf("Printed: %s\n", text)
    }
    printWithCallback(callback)
}
```

以上代码定义了一个名为 printWithCallback 的函数，该函数接受一个函数类型参数 callback，其中这个 callback 会在输出文本后被调用。然后在主函数中定义了一个回调函数并将其作为参数传递给 printWithCallback 函数。

当程序运行起来后，会输出如下内容：

```
Hello, world!
Printed: Hello, world!
```
这表示 printWithCallback 函数被成功调用，并且在输出文本之后 callback 被调用了一次。

对于钩子函数的例子，假设我们正在编写一段需要进行时间测量的代码。我们可以先定义一个钩子函数 TimeElapsedCallback，然后将它作为参数传递给要测量的代码：

```
package main

import (
    "fmt"
    "time"
)

func TimeElapsedCallback(start time.Time) {
    elapsed := time.Now().Sub(start)
    fmt.Printf("Time elapsed: %v\n", elapsed)
}

func heavyCalculation(callback func(time.Time)) {
    start := time.Now()
    // 这里进行任何需要测量时间的操作
    callback(start)
}

func main() {
    heavyCalculation(TimeElapsedCallback)
}
```
在以上示例中，当我们调用 heavyCalculation 函数时，会传入一个名为 TimeElapsedCallback 的钩子函数。这个钩子函数被定义为一个打印程序执行时长的简单函数，它接受一个 time.Time 类型参数来计算程序耗时。

在执行完 heavyCalculation 函数后，TimeElapsedCallback 钩子函数就会被自动调用，然后打印程序运行时长，输出如下内容：

Time elapsed: 1.001201ms
以上演示了如何使用回调函数和钩子函数来处理事件。在Go语言中，这种技术经常用于实现异步操作、协程等方面。

回调函数的不同
以下是不使用回调函数的方式实现计算程序执行时间的示例代码：
```
package main

import (
    "fmt"
    "time"
)

func heavyCalculation() {
    start := time.Now()
    // 这里进行任何需要测量时间的操作
    elapsed := time.Since(start)
    fmt.Printf("Time elapsed: %v\n", elapsed)
}

func main() {
    heavyCalculation()
}
```
和之前使用回调函数的示例代码相比，主要区别在于重构了 heavyCalculation 函数代码。

在这个更改后的函数中，我们将钩子函数的功能直接集成到代码流程中，通过调用 time.Since(start) 来得到程序的执行时间。这样做的缺点在于，在需要使用程序执行时间的其他场合还需要重新编写和复制此段逻辑, 这样就会限制程序的可重用性和可扩展性。

当然，在一些简单的场合下该方法也能够正常工作，不过如果需要在多处使用计算执行时间的逻辑或者需要更加细致的精度控制，建议使用钩子函数来实现。

使用钩子函数和不使用钩子函数的主要区别在于代码结构和灵活性。

而使用钩子函数，可以将打印程序执行时间的功能单独提出来作为一个函数。这使得我们可以像 heavyCalculation 函数那样封装其他计算逻辑并复用 TimeElapsedCallback 钩子函数。

钩子函数的使用场景非常广泛，在几乎所有需要在特定事件发生时自动执行一些附加逻辑的场景中都可以使用。

### 以下是使用回调函数的优点：
灵活性：可以轻松地将自定义代码插入到已有的代码流程中。
可重用性：可以将钩子函数单独进行封装，以供不同的代码文件或项目中使用。
易于维护：通过修改单个钩子函数即可更改所有使用该钩子函数的代码的行为。
总之，使用钩子函数可以帮助我们让代码变得更加简洁、灵活、模块化和可重用。

## gorm中的钩子函数用法

```

type BannerModel struct {
	MODEL
	Path      string          `json:"path"`                        // 图片路径
	Hash      string          `json:"hash"`                        // 图片的hash值，用于判断重复图片
	Name      string          `gorm:"size:38" json:"name"`         // 图片名称
	ImageType ctype.ImageType `gorm:"default:1" json:"image_type"` // 图片的类型， 本地还是七牛，默认本地
}

func (b *BannerModel) BeforeDelete(tx *gorm.DB) (err error) {
	if b.ImageType == ctype.Local {
		// 本地图片，删除，还要删除本地的存储
		err = os.Remove(b.Path[1:])
		if err != nil {
			global.Log.Error(err)
			return err
		}
	}
	// 云端的图片不用删除
	return nil
}

```

在插入一条这样的结构体记录到数据库的时候，我希望执行一些操作
`BeforeDelete`是`BannerModel`中的一个钩子函数，在从数据库删除数据时，都会先调用这个函数