> [一篇带你全面掌握go反射的用法 - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/411313885)
> [反射 | Golang中文学习文档 (halfiisland.com)](https://golang.halfiisland.com/essential/senior/105.reflect.html)
> [Go reflection 三定律与最佳实践 (halfrost.com)](https://halfrost.com/go_reflection/)
## 为什么用反射
强调一下反射的2个弊端：

1. 代码不易阅读，不易维护，容易发生线上panic
2. 性能很差，比正常代码慢一到两个数量级

go语言反射里最重要的两个概念是Type和Value，Type用于获取类型相关的信息（比如Slice的长度，struct的成员，函数的参数个数），Value用于获取和修改原始数据的值（比如修改slice和map中的元素，修改struct的成员变量）。

它们和go原始数据类型（比如int、float、map、struct等）的转换方式如下图：

![](https://img.halfrost.com/Blog/ArticleImage/148_6_0.png)
## reflect.Type
```go
package tests

import (
	"fmt"
	"reflect"
	"testing"

	"github.com/stretchr/testify/assert"
)

// 如何得到Type
func TestReflect1(t *testing.T) {
	typeI := reflect.TypeOf(1)
	typeS := reflect.TypeOf("hello")
	fmt.Println(typeI) //int
	fmt.Println(typeS) //string

	typeUser := reflect.TypeOf(&User{})
	fmt.Println(typeUser)               //*tests.User
	fmt.Println(typeUser.Kind())        //ptr
	fmt.Println(typeUser.Elem().Kind()) //struct
}

// 指针Type转为非指针Type
func TestReflect2(t *testing.T) {
	typeUser := reflect.TypeOf(&User{})
	typeUser2 := reflect.TypeOf(User{})
	assert.Equal(t, typeUser.Elem(), typeUser2) // 相等
}

// 获取struct成员变量的信息
func TestReflect3(t *testing.T) {
	typeUser := reflect.TypeOf(User{}) //需要用struct的Type，不能用指针的Type
	fieldNum := typeUser.NumField()    //成员变量的个数
	for i := 0; i < fieldNum; i++ {
		field := typeUser.Field(i)
		fmt.Printf("%d %s offset %d anonymous %t type %s exported %t json tag %s\n", i,
			field.Name,            //变量名称
			field.Offset,          //相对于结构体首地址的内存偏移量，string类型会占据16个字节
			field.Anonymous,       //是否为匿名成员
			field.Type,            //数据类型，reflect.Type类型
			field.IsExported(),    //包外是否可见（即是否以大写字母开头）
			field.Tag.Get("json")) //获取成员变量后面``里面定义的tag
	}
	fmt.Println()

	//可以通过FieldByName获取Field
	if nameField, ok := typeUser.FieldByName("Name"); ok {
		fmt.Printf("Name is exported %t\n", nameField.IsExported())
	}

	//也可以根据FieldByIndex获取Field
	Field := typeUser.FieldByIndex([]int{1})                          //参数是个slice，因为有struct嵌套的情况
	FieldField := reflect.TypeOf(VIPUser{}).FieldByIndex([]int{0, 1}) // 嵌套的切片，下标对应struct嵌套的层数
	fmt.Printf("field name %s\n fieldfield name %s\n", Field.Name, FieldField.Name)
}

// 获取struct成员方法的信息
func TestReflect4(t *testing.T) {
	typeUser := reflect.TypeOf(User{})
	methodNum := typeUser.NumMethod() // 成员方法的个数。接收者为指针的方法【不】包含在内
	for i := 0; i < methodNum; i++ {
		method := typeUser.Method(i)
		fmt.Printf("method name:%s ,type:%s, exported:%t\n", method.Name, method.Type, method.IsExported())
	}
	fmt.Println()

	typeUser2 := reflect.TypeOf(&User{})
	methodNum = typeUser2.NumMethod() // 成员方法的个数。接收者为指针或值的方法【都】包含在内，也就是说值实现的方法指针也实现了（反之不成立）
	for i := 0; i < methodNum; i++ {  // 根据Go语言的官方文档，不推荐在同一个结构体上同时定义值接收器（value receiver）和指针接收器（pointer receiver）的方法
		method := typeUser2.Method(i)
		fmt.Printf("method name:%s ,type:%s, exported:%t\n", method.Name, method.Type, method.IsExported())
	}
}

// 获取函数的信息
func TestReflect5(t *testing.T) {
	Add := func(a, b int) int {
		return a + b
	}
	typeFunc := reflect.TypeOf(Add) //获取函数类型
	fmt.Printf("is function type %t\n", typeFunc.Kind() == reflect.Func)
	argInNum := typeFunc.NumIn()   //输入参数的个数
	argOutNum := typeFunc.NumOut() //输出参数的个数
	for i := 0; i < argInNum; i++ {
		argTyp := typeFunc.In(i)
		fmt.Printf("第%d个输入参数的类型%s\n", i, argTyp)
	}
	for i := 0; i < argOutNum; i++ {
		argTyp := typeFunc.Out(i)
		fmt.Printf("第%d个输出参数的类型%s\n", i, argTyp)
	}
}

// 判断类型是否实现了某接口
func TestReflect6(t *testing.T) {
	//通过reflect.TypeOf((*<interface>)(nil)).Elem()获得接口类型。因为People是个接口不能创建实例，所以把nil强制转为*tests.People类型
	typeOfPeople := reflect.TypeOf((*People)(nil)).Elem()
	fmt.Printf("typeOfPeople kind is interface %t\n", typeOfPeople.Kind() == reflect.Interface)
	t1 := reflect.TypeOf(User{})
	t2 := reflect.TypeOf(&User{})
	//如果值类型实现了接口，则指针类型也实现了接口
	//反之不成立
	fmt.Printf("t1 implements People interface %t\n", t1.Implements(typeOfPeople))
	fmt.Printf("t1 implements People interface %t\n", t2.Implements(typeOfPeople))
}

```

## reflect.Value
```go
package tests

import (
	"fmt"
	"reflect"
	"testing"

	"github.com/stretchr/testify/assert"
)

// 通过ValueOf()得到Value
func TestReflect7(t *testing.T) {

	fmt.Println(iValue)       //1
	fmt.Println(sValue)       //hello
	fmt.Println(userPtrValue) //&{7 杰克逊  65 1.68}
}

// Value转为Type
func TestReflect8(t *testing.T) {
	iType := iValue.Type()
	sType := sValue.Type()
	userType := userPtrValue.Type()
	//在Type和相应Value上调用Kind()结果一样的
	fmt.Println(iType.Kind() == reflect.Int, iValue.Kind() == reflect.Int, iType.Kind() == iValue.Kind())
	fmt.Println(sType.Kind() == reflect.String, sValue.Kind() == reflect.String, sType.Kind() == sValue.Kind())
	fmt.Println(userType.Kind() == reflect.Ptr, userPtrValue.Kind() == reflect.Ptr, userType.Kind() == userPtrValue.Kind())
}

// 指针Value和非指针Value互相转换
func TestReflect9(t *testing.T) {
	userValue := userPtrValue.Elem()                    //Elem() 指针Value转为非指针Value
	fmt.Println(userValue.Kind(), userPtrValue.Kind())  //struct ptr
	userPtrValue3 := userValue.Addr()                   //Addr() 非指针Value转为指针Value
	fmt.Println(userValue.Kind(), userPtrValue3.Kind()) //struct ptr
}

// 得到Value对应的原始数据
// 通过Interface()函数把Value转为interface{}，再从interface{}强制类型转换，转为原始数据类型。或者在Value上直接调用Int()、String()等一步到位。
func TestReflect10(t *testing.T) {
	userValue := userPtrValue.Elem()
	fmt.Printf("origin value iValue is %d %d\n", iValue.Interface().(int), iValue.Int())
	fmt.Printf("origin value sValue is %s %s\n", sValue.Interface().(string), sValue.String())
	user := userValue.Interface().(User)
	fmt.Printf("id=%d name=%s weight=%.2f height=%.2f\n", user.Id, user.Name, user.Weight, user.Height)
	user2 := userPtrValue.Interface().(*User)
	fmt.Printf("id=%d name=%s weight=%.2f height=%.2f\n", user2.Id, user2.Name, user2.Weight, user2.Height)
}

// 空Value的判断
func TestReflect11(t *testing.T) {
	var i interface{} //接口没有指向具体的值
	v := reflect.ValueOf(i)
	fmt.Printf("v持有值 %t, type of v is Invalid %t\n", v.IsValid(), v.Kind() == reflect.Invalid)

	var user *User = nil
	v = reflect.ValueOf(user) //Value指向一个nil
	if v.IsValid() {
		fmt.Printf("v持有的值是nil %t\n", v.IsNil()) //调用IsNil()前先确保IsValid()，否则会panic
	}

	var u User //只声明，里面的值都是0值nil
	v = reflect.ValueOf(u)
	if v.IsValid() {
		fmt.Printf("v持有的值是对应类型的0值 %t\n", v.IsZero()) //调用IsZero()前先确保IsValid()，否则会panic
	}
}

// 通过Value修改原始数据的值
func TestReflect12(t *testing.T) {

	valueI.Elem().SetInt(8) //由于valueI对应的原始对象是指针，通过Elem()返回指针指向的对象
	valueS.Elem().SetString("golang")
	valueUser.Elem().FieldByName("Weight").SetFloat(68.0) //FieldByName()通过Name返回类的成员变量
	assert.Equal(t, valueI.Elem().Int(), int64(8))
	assert.Equal(t, valueS.Elem().String(), "golang")
	assert.Equal(t, valueUser.Elem().Interface().(User), user)

	//强调一下，要想修改原始数据的值，给ValueOf传的必须是指针，
	//指针Value不能调用Set和FieldByName方法，所以得先通过Elem()转为非指针Value。

}

// 未导出成员的值不能通过反射进行修改。
func TestReflect13(t *testing.T) {
	addrValue := valueUser.Elem().FieldByName("address")
	if addrValue.CanSet() {
		addrValue.SetString("北京")
	} else {
		fmt.Println("addr是未导出成员，不可Set") //以小写字母开头的成员相当于是私有成员
	}
}

// 通过Value修改Slice
func TestReflect14(t *testing.T) {
	users := make([]*User, 1, 5) //len=1，cap=5
	users[0] = &User{
		Id:     7,
		Name:   "杰克逊",
		Weight: 65.5,
		Height: 1.68,
	}

	sliceValue := reflect.ValueOf(&users) //准备通过Value修改users，所以传users的地址
	if sliceValue.Elem().Len() > 0 {      //取得slice的长度
		sliceValue.Elem().Index(0).Elem().FieldByName("Name").SetString("令狐一刀")
		fmt.Printf("1st user name change to %s\n", users[0].Name)
	}

	//甚至可以修改slice的cap，新的cap必须位于原始的len到cap之间，即只能把cap改小。

	sliceValue.Elem().SetCap(3)
	//通过把len改大，可以实现向slice中追加元素的功能。

	sliceValue.Elem().SetLen(2)
	//调用reflect.Value的Set()函数修改其底层指向的原始数据
	sliceValue.Elem().Index(1).Set(reflect.ValueOf(&User{
		Id:     8,
		Name:   "李达",
		Weight: 80,
		Height: 180,
	}))
	fmt.Printf("2nd user name %s\n", users[1].Name)
}

// Value.SetMapIndex()函数：往map里添加一个key-value对
//
// Value.MapIndex()函数： 根据Key取出对应的map
func TestReflect15(t *testing.T) {
	u1 := &User{
		Id:     7,
		Name:   "杰克逊",
		Weight: 65.5,
		Height: 1.68,
	}
	u2 := &User{
		Id:     8,
		Name:   "杰克逊",
		Weight: 65.5,
		Height: 1.68,
	}
	userMap := make(map[int]*User, 5)
	userMap[u1.Id] = u1

	mapValue := reflect.ValueOf(&userMap)                                                         //准备通过Value修改userMap，所以传userMap的地址
	mapValue.Elem().SetMapIndex(reflect.ValueOf(u2.Id), reflect.ValueOf(u2))                      //SetMapIndex 往map里添加一个key-value对
	mapValue.Elem().MapIndex(reflect.ValueOf(u1.Id)).Elem().FieldByName("Name").SetString("令狐一刀") //MapIndex 根据Key取出对应的map
	for k, user := range userMap {
		fmt.Printf("key %d name %s\n", k, user.Name)
	}
}

// 调用函数
func TestReflect16(t *testing.T) {
	Add := func(a int, b int) int {
		return a + b
	}
	valueFunc := reflect.ValueOf(Add) //函数也是一种数据类型
	typeFunc := reflect.TypeOf(Add)
	argNum := typeFunc.NumIn()            //函数输入参数的个数
	args := make([]reflect.Value, argNum) //准备函数的输入参数
	for i := 0; i < argNum; i++ {
		if typeFunc.In(i).Kind() == reflect.Int {
			args[i] = reflect.ValueOf(3) //给每一个参数都赋3
		}
	}
	sumValue := valueFunc.Call(args) //返回[]reflect.Value，因为go语言的函数返回可能是一个列表
	if typeFunc.Out(0).Kind() == reflect.Int {
		sum := sumValue[0].Interface().(int) //从Value转为原始数据类型
		fmt.Printf("sum=%d\n", sum)
	}
}

// 调用成员方法
func TestReflect17(t *testing.T) {
	valueUser := reflect.ValueOf(&User{
		Id:     7,
		Name:   "杰克逊",
		Weight: 65.5,
		Height: 1.68,
	}) //必须传指针，因为UpdateName()在定义的时候它是指针的方法
	UpdateNameMethod := valueUser.MethodByName("UpdateName") //MethodByName()通过Name返回类的成员变量
	args := make([]reflect.Value, 1)                         //准备函数的输入参数
	args[0] = reflect.ValueOf("handsome")
	UpdateNameMethod.Call(args) //无参数时传一个空的切片
	fmt.Println(valueUser.Elem().Interface().(User).Name)

	//GetName()在定义的时候用的不是指针，valueUser可以用指针也可以不用指针
	thinkMethod := valueUser.MethodByName("GetName")
	result := thinkMethod.Call([]reflect.Value{})
	fmt.Println("Name is:" + result[0].String())

	valueUser2 := reflect.ValueOf(user)
	thinkMethod = valueUser2.MethodByName("GetName")
	thinkMethod.Call([]reflect.Value{})
	result = thinkMethod.Call([]reflect.Value{})
	fmt.Println("Name is:" + result[0].String())
}

```
## 创建对象
```go
package tests

import (
	"fmt"
	"reflect"
	"testing"
)

// 创建struct
func TestReflect18(t *testing.T) {
	userT := reflect.TypeOf(User{})
	value := reflect.New(userT) //根据reflect.Type创建一个对象，得到该对象的指针，再根据指针提到reflect.Value
	value.Elem().FieldByName("Id").SetInt(10)
	user := value.Interface().(*User) //把反射类型转成go原始数据类型Call([]reflect.Value{})
	fmt.Println(user.GetName())
}

// 创建map
func TestReflect19(t *testing.T) {
	var slice []User
	sliceType := reflect.TypeOf(slice)
	sliceValue := reflect.MakeSlice(sliceType, 1, 3)
	sliceValue.Index(0).Set(reflect.ValueOf(User{
		Id:     8,
		Name:   "李达",
		Weight: 80,
		Height: 180,
	}))
	users := sliceValue.Interface().([]User)
	fmt.Printf("1st user name %s\n", users[0].Name)
	fmt.Println(sliceValue.Interface().([]User))
}

// 创建slice
func TestReflect20(t *testing.T) {
	var userMap map[int]*User
	mapType := reflect.TypeOf(userMap)
	//mapValue := reflect.MakeMap(mapType)
	mapValue := reflect.MakeMapWithSize(mapType, 10)

	user := &User{
		Id:     7,
		Name:   "杰克逊",
		Weight: 65.5,
		Height: 1.68,
	}
	key := reflect.ValueOf(user.Id)
	mapValue.SetMapIndex(key, reflect.ValueOf(user))                    //SetMapIndex 往map里添加一个key-value对
	mapValue.MapIndex(key).Elem().FieldByName("Name").SetString("令狐一刀") //MapIndex 根据Key取出对应的map
	userMap = mapValue.Interface().(map[int]*User)
	fmt.Printf("user name %s %s\n", userMap[7].Name, user.Name)
}

```

## common
```go
package tests

import (
	"reflect"
)

type User struct {
	Id      int
	Name    string
	Weight  float64
	Height  float64
	address string
}

func (Copy User) GetName() string {
	return Copy.Name
}
func (Ptr *User) UpdateName(name string) {
	Ptr.Name = name
}

type VIPUser struct {
	User
	level int
}
type People interface {
	Body() string
}

func (Copy User) Body() string {
	return ""
}

var (
	iValue       = reflect.ValueOf(1)
	sValue       = reflect.ValueOf("hello")
	userPtrValue = reflect.ValueOf(&User{
		Id:     7,
		Name:   "Handsome",
		Weight: 65,
		Height: 1.68,
	})

	i    = 10
	s    = "hello"
	user = User{
		Id:     7,
		Name:   "杰克逊",
		Weight: 65.5,
		Height: 1.68,
	}

	valueI    = reflect.ValueOf(&i) //由于go语言所有函数传的都是值，所以要想修改原来的值就需要传指针
	valueS    = reflect.ValueOf(&s)
	valueUser = reflect.ValueOf(&user)
)
```

## 实例讲解
```go
// List 查询列表
func List[T any]() {
	indexName := reflect.ValueOf(new(T)).MethodByName("IndexName").Call([]reflect.Value{})[0].String()
}
```

以上代码可以拆解为

1. new(T) 根据 T 的类型，创建一个 `*Value` 类型的值
2. ValueOf() 获得它的值
3. MethodByName() 获得他的成员函数，通过"IndexName"
4. Call([]Value{}) 调用成员函数，入参为空
5. [0].string() 拿到函数调用结果返回的第一个值，并拿到他的string的值

路线为 Type -> `*Value` -> `(*Value).func`