## 主函数中使用
```
	// 命令行参数绑定
	option := flag.Parse()
```
## Parse 解析命令行参数
```
func Parse() Option {
	db := sys_flag.Bool("db", false, "初始化数据库")
	user := sys_flag.String("u", "", "创建用户")
	es := sys_flag.Bool("es", false, "es操作")
	dump := sys_flag.String("dump", "", "将索引下的数据导出为json文件")
	load := sys_flag.String("load", "", "导入json数据，并创建索引")
	// 解析命令行参数写入注册的flag里
	sys_flag.Parse()
	return Option{
		DB:   *db,
		User: *user,
		ES:   *es,
		Dump: *dump,
		Load: *load,
	}
}
```
## SwitchOption 根据命令执行不同的函数
```
func SwitchOption(option Option) {
	if option.DB {
		Makemigrations()
		return
	}
	if option.User == "admin" || option.User == "user" {
		CreateUser(option.User)
		return
	}
	if option.ES {
		global.ESClient = core.EsConnect()
		if option.Dump != "" {
			DumpIndex(option.Dump)
		}
		if option.Load != "" {
			LoadIndex(option.Load)
		}
	}
}
```