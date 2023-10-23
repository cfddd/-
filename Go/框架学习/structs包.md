> [Go三方库——structs | Go 技术论坛 (learnku.com)](https://learnku.com/articles/62693)
> [structs package - github.com/gookit/goutil/structs - Go Packages](https://pkg.go.dev/github.com/gookit/goutil/structs#section-readme)
## 将 struct 转为 map
**Examples**:

```
	type User1 struct {
		Name string `json:"name"`
		Age  int    `json:"age"`
		city string
	}

	u1 := &User1{
		Name: "inhere",
		Age:  34,
		city: "somewhere",
	}

	mp := structs.ToMap(u1)
	dump.P(mp)
```

**Output**:

```
map[string]interface {} { #len=2
  "age": int(34),
  "name": string("inhere"), #len=6
},
```
可以把结构体转换为map
map的类型为`map[string]interface{}`

## tag
```
type ArticleModel struct {
	Title    string `json:"title" structs:"title"`                // 文章标题
	Keyword  string `json:"keyword,omit(list)" structs:"keyword"` // 关键字
	Abstract string `json:"abstract" structs:"abstract"`          // 文章简介
	Content  string `json:"content,omit(list)" structs:"content"` // 文章内容
}
```
`structs`后面key可以是转换后map的键
`ArticleModel`结构体的成员变量就是map的值
> structs后面可以使用`-`忽略掉转换为map中的值
