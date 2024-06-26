## 内网网络配置
一个简单的、独立的需求

从数据库到缓存都有试水，了解了代码的基本框架

了解了专业开发流程中的协作方法，比如git、gitlab等等

期间还写了大量的单元测试，当苦力
## 节点日志
独立分析团队的业务需求，和产品经理对过，也和测试打过交道，甚至还和前端对过

先是写代码，完成节点日志的需求，需要写数据库，需要整合比较多的数据

写完后push上传并创建合并请求，mt审核没过，修改几次成功合并

到了一次项目短期目标后，需要配合测试部署环境，然后修改bug

我的做的需求只有一个bug，逻辑方面没有问题，主要还是没对上需求，和前端少要了一个参数

除了自己的bug，我还分配到了许多别人的bug，被迫去看别人的代码，发现更复杂的需求

- **bug 1** 
	
	es 查询时，使用到了反射。封装的查询函数，通过 **参数限制** 读到对应模型的配置方法，然后在通过反射赋值给传递进来的 对应模型 的切片更新创建时间，再更新es中的数据。
	
	这里的bug很小，但是panic很难发现。经过检查发现，是反射时传递的形参结构体里面创建日期成员是指针类型，但是反射赋值却是值类型产生报错。
	
- **bug 2**
	
	获取bk_cmdb资产数据分页的问题，排查到这里非常麻烦，要和数据库里面的多张表结合，有非常多的可能问题。
	
	最后排查出问题是分页大小每次都是固定的，但是最后一页返回逻辑是最后页大小的数据数的数据
	
	发现问题后就很好解决了
- **bug 3**
	定时任务，同样是封装了各种功能，从零开始读有较大难度
	
	好在最后理解下来了代码的主干逻辑，画了一张简单的逻辑草图，得到mt的认可
	
	但是逻辑并没有问题，猜测可能的原因是：
	
	一个任务每分钟执行一次，检测数据库中存的任务配置信息，读出来并和现有的比较，如果有不同就更新，没有就不执行。这是一个发配任务的任务，再代码启动时就会启动，还用来加载数据库里已有的任务
	
	这个小bug是，如果操作时间是当前时间的+1分钟，那么任务检测会在**下一分钟**更新当前任务，更新后加载的修改后的任务发现时间已经过了，所以不执行。
	
	非常难以发现
## es
### mapping
mapping使用Init，将初始化函数添加到各个包中，自动migrate

但是需要注意，包含init的包必须被导入后才能执行init()

```go
import _ fobrain/xxx
```

如果包里面没有其他的函数，可以这样子来执行init

## time 的封装
如果我想让一些json通过Marshall或者UnMarshall可以匹配 "2006-01-02 15:04:05" 这样的localtime，而不是time.time 里面 复杂且详细的时间，就需要我们自己封装一个 localtime

```go
package localtime

import (
	"database/sql/driver"
	"fmt"
	"time"
)

// Time 全局定义
type Time time.Time

const timeFormat = "2006-01-02 15:04:05"
const DateTimeLayout = "20060102150405"

// MarshalJSON json
func (t Time) MarshalJSON() ([]byte, error) {
	b := make([]byte, 0, len(timeFormat)+2)
	b = append(b, '"')
	b = time.Time(t).AppendFormat(b, timeFormat)
	b = append(b, '"')
	return b, nil
}

// UnmarshalJSON 解析json
func (t *Time) UnmarshalJSON(data []byte) (err error) {
	now, err := time.ParseInLocation(`"`+timeFormat+`"`, string(data), time.Local)
	*t = Time(now)
	return
}

// String 字符串
func (t Time) String() string {
	return time.Time(t).Format(timeFormat)
}

// local 获取时间
func (t Time) local() time.Time {
	return time.Time(t)
}

func NewLocalTime(t time.Time) *Time {
	ts := Time(t)
	return &ts
}

// Value 获取时间
func (t Time) Value() (driver.Value, error) {
	var zeroTime time.Time
	var ti = time.Time(t)
	if ti.UnixNano() == zeroTime.UnixNano() {
		return nil, nil
	}
	return ti, nil
}

// Scan 获取时间
func (t *Time) Scan(v interface{}) error {
	value, ok := v.(time.Time)
	if ok {
		*t = Time(value)
		return nil
	}
	return fmt.Errorf("can not convert %v to timestamp", v)
}

```

这些方法在time.time里面都有对应的