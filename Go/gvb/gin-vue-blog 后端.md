
[fast_gin/core/logrus.go at main · fengfengzhidao/fast_gin (github.com)](https://github.com/fengfengzhidao/fast_gin/blob/main/core/logrus.go) 博客链接

# 后端项目搭建
建了目录结构：
![[Pasted image 20230918110632.png]]

**.gitignore** ： 
- git 为我们提供了一个.gitignore 文件，只要在这个文件中声明哪些文件你不希望添加到 git 中去，这样当你使用 git add .的时候这些文件就会被自动忽略掉。


在 settings. yaml 中添加数据库 mysql，日志 logger，系统设置相关的 system 东西:
```go
mysql:  
host: 127.0.0.1  
port: 3306  
db: gvb_db  
user: root  
password: 123456  
log_level: dev # 开发环境  
  
# 日志  
logger:  
level: info # 等级  
prefix: '[gvb]' # 前缀  
director: log # 存放的目录  
show_line: true # 是否显示行号  
log_in_console: true # 是否显示文件路径  
  
# 系统相关的(运行在哪个ip上，哪个端口上)  
system:  
host: "0.0.0.0" # 我所有的ip，所有的网卡  
port: 8080 # 程序的端口上  
env: dev # 环境：开发环境
```

在 config 目录中创建相应结构体：
![[Pasted image 20230918110851.png]]


```go
// enter.go
// 第一件事情就是执行这里面的事情  
type Config struct {  
	Mysql Mysql `yaml:"mysql"`  
	Logger Logger `yaml:"logger"`  
	System System `yaml:"system"`  
}

// conf_mysql.go
type Mysql struct {  
	Host string `yaml:"host"`  
	Port int `yaml:"port"`  
	DB string `yaml:"db"`  
	User string `yaml:"user"`  
	Password string `yaml:"password"`  
	LogLevel string `yaml:"logLevel"` // 日志等级，debug就是输出全部sql，dev，release  
}
```

## 配置文件的读取：

读取 yaml 文件的配置: 
在 core 中创建 conf. go ...
```go
package core

import (
	"fmt"
	"gopkg.in/yaml.v2"
	"gvb_server/config"
	"gvb_server/global"
	"io/ioutil"
	"log"
)

// InitConf 读取yaml文件的配置
func InitConf() {
	const ConfigFile = "settings.yaml"
	c := &config.Config{} // 读取的配置文件要存储在一个实体中
	yamlConf, err := ioutil.ReadFile(ConfigFile)
	if err != nil {
		panic(fmt.Errorf("get yamlconf error: %S", err))
	}
	err = yaml.Unmarshal(yamlConf, c)
	if err != nil {
		log.Fatalf("config Init Unmarshal: %V", err)
	}
	log.Println("config yamlFile Load Init success.")
	global.Config = c // 存到一个实体中
}
```
**需要一个全局变量，用于保存配置文件，存在 global 目录下**

加入 yaml：
```shell
 go get gopkg.in/yaml.v2
```
在主函数 main. go 中读取运行： 
```go
package main

import (
	"fmt"
	"gvb_server/core"
	"gvb_server/global"
)

func main() {
	// 读取配置文件
	core.InitConf()
	// 打印
	fmt.Println(global.Config)
}

```

## GORM 配置

gorm 初始化
```go
package core

import (
	"fmt"
	"gorm.io/driver/mysql"
	"gorm.io/gorm"
	"gorm.io/gorm/logger"
	"gvb_server/global"
	"log"
	"time"
)

func InitGorm() *gorm.DB {
	if global.Config.Mysql.Host == "" {
		log.Println("未配置mysql，取消gorm连接")
		return nil
	}
	dsn := global.Config.Mysql.Dsn()

	var mysqlLogger logger.Interface
	// 判断环境
	if global.Config.System.Env == "debug" {
		// 开发环境显示所有的sql
		mysqlLogger = logger.Default.LogMode(logger.Info) // 打印所有日志
	} else {
		mysqlLogger = logger.Default.LogMode(logger.Error) // 打印错误日志
	}

	db, err := gorm.Open(mysql.Open(dsn), &gorm.Config{
		// Logger的配置
		Logger: mysqlLogger,
	})

	// 连接失败就可以退出啦
	if err != nil {
		log.Fatal(fmt.Sprintf("[%s] mysql连接失败"), dsn)
	}

	sqlDB, _ := db.DB()
	sqlDB.SetMaxIdleConns(10)               // 最大空闲连接数
	sqlDB.SetMaxOpenConns(100)              // 最多可容纳
	sqlDB.SetConnMaxLifetime(time.Hour * 4) // 连接最大复用时间，不能超过mysql的wait_timeout
	return db
}
```

main. go 中，连接数据库
```go
// 连接数据库  
global.DB = core.InitGorm()  
fmt.Println(global.DB)
```

**go get 下载那些配置**

## 日志配置

```go
package core

import (
	"bytes"
	"fmt"
	"github.com/sirupsen/logrus"
	"gvb_server/global"
	"os"
	"path"
)

// 颜色的常量
const (
	red    = 31
	yellow = 33
	blue   = 36
	gray   = 37
)

type LogFormatter struct{}

// Format 实现Formatter(entry *logrus.Entry) ([]byte, error)接口
func (t *LogFormatter) Format(entry *logrus.Entry) ([]byte, error) {
	//根据不同的level去展示颜色
	var levelColor int
	switch entry.Level {
	case logrus.DebugLevel, logrus.TraceLevel:
		levelColor = gray
	case logrus.WarnLevel:
		levelColor = yellow
	case logrus.ErrorLevel, logrus.FatalLevel, logrus.PanicLevel:
		levelColor = red
	default:
		levelColor = blue
	}

	var b *bytes.Buffer
	if entry.Buffer != nil {
		b = entry.Buffer
	} else {
		b = &bytes.Buffer{}
	}

	log := global.Config.Logger // (log.Prefix是前缀)
	//自定义日期格式
	timestamp := entry.Time.Format("2006-01-02 15:04:05")
	if entry.HasCaller() {
		//自定义文件路径
		funcVal := entry.Caller.Function
		fileVal := fmt.Sprintf("%s:%d", path.Base(entry.Caller.File), entry.Caller.Line)
		//自定义输出格式
		fmt.Fprintf(b, "%s[%s] \x1b[%dm[%s]\x1b[0m %s %s %s\n", log.Prefix, timestamp, levelColor, entry.Level, fileVal, funcVal, entry.Message)
	} else {
		fmt.Fprintf(b, "%s[%s] \x1b[%dm[%s]\x1b[0m %s\n", log.Prefix, timestamp, levelColor, entry.Level, entry.Message)
	}
	return b.Bytes(), nil
}

// 初始化
func InitLogger() *logrus.Logger {
	mLog := logrus.New()                                //新建一个实例
	mLog.SetOutput(os.Stdout)                           //设置输出类型
	mLog.SetReportCaller(global.Config.Logger.ShowLine) //开启返回函数名和行号
	mLog.SetFormatter(&LogFormatter{})                  //设置自己定义的Formatter
	level, err := logrus.ParseLevel(global.Config.Logger.Level)
	if err != nil {
		level = logrus.InfoLevel
	}
	mLog.SetLevel(level)
	//mLog.SetLevel(logrus.DebugLevel)   //设置最低的Level
	InitDefaultLogger()
	return mLog
}

// 修改的全局log的使用（在InitLogger中调用一下就行）
// 修改之后调用全局logrus就会很好看(logrus.Warnln("嘻嘻嘻")为例就会和全局global.Log.Warnln("嘻嘻嘻")一样好看)
func InitDefaultLogger() {
	// 全局log
	logrus.SetOutput(os.Stdout)                           //设置输出类型
	logrus.SetReportCaller(global.Config.Logger.ShowLine) //开启返回函数名和行号
	logrus.SetFormatter(&LogFormatter{})                  //设置自己定义的Formatter
	level, err := logrus.ParseLevel(global.Config.Logger.Level)
	if err != nil {
		level = logrus.InfoLevel
	}
	logrus.SetLevel(level)
}

```


main. go 中，日志的初始化：
```go
// 初始化日志  
global.Log = core.InitLogger()  
global.Log.Warnln("嘻嘻嘻")  
global.Log.Error("嘻嘻嘻")  
global.Log.Infof("嘻嘻嘻")
```

## 路由配置

```go
package routers

import (
	"github.com/gin-gonic/gin"
	"gvb_server/global"
)

// 初始化路由
func InitRouter() *gin.Engine {
	/*
		简洁日志部分
		[GIN-debug] [WARNING] Running in "debug" mode. Switch to "release" mode in production.
		 - using env:   export GIN_MODE=release （在.yaml文件中）
		 - using code:  gin.SetMode(gin.ReleaseMode) （在路由初始化处）
	*/
	gin.SetMode(global.Config.System.Env)
	router := gin.Default()
	router.GET("", func(c *gin.Context) {
		c.String(200, "xxx")
	})
	return router
}
```

拼接地址部分：
```go
// 拼接地址  (在conf_system.go中)
func (s System) Addr() string {  
	return fmt.Sprintf("%s:%d", s.Host, s.Port)  
}
```

mian. go 中，路由初始化部分：
```go
// 初始化路由  
router := routers.InitRouter()  
addr := global.Config.System.Addr()  
global.Log.Infof("gvb_server运行在：%s", addr)  
  
// func (engine *Engine) Run(addr ...string) (err error) 所以要弄地址  
router.Run(addr)
```

![[Pasted image 20230920094909.png]]

路由分组，分层
```go
package routers

import (
	"github.com/gin-gonic/gin"
	"gvb_server/global"
)

// 初始化路由
func InitRouter() *gin.Engine {
	gin.SetMode(global.Config.System.Env)

	router := gin.Default()

	// 传入路由分组
	apiRouterGroup := router.Group("api")

	// 路由分层
	// 系统配置api
	SettingsRouter(apiRouterGroup)
	return router
}

```
具体的路由文件
settings_router. go
```go
package routers

import (
	"github.com/gin-gonic/gin"
	"gvb_server/api"
)

// 系统路由信息
func SettingsRouter(router *gin.RouterGroup) {
	settingsApi := api.ApiGroupApp.SettingsApi
	router.GET("settings", settingsApi.SettingsInfoView)
}
```
## 封装统一响应

```go
package res

import (
	"github.com/gin-gonic/gin"
	"net/http"
)

// 封装返回的数据类型
type Response struct {
	Code int    `json:"code"` // 0：成功的；非0：失败的
	Data any    `json:"data"`
	Msg  string `json:"msg"`
}

// 定义一个常量，(万一前端觉得0不是成功的)
const (
	Success = 0 // code为0：成功
	Error   = 7 // 其他错误
)

// 一些方法
func Result(code int, data any, msg string, c *gin.Context) {
	c.JSON(http.StatusOK, Response{
		Code: code,
		Data: data,
		Msg:  msg,
	})
}

// 封装一些相应成功的
func OK(data any, msg string, c *gin.Context) {
	Result(Success, data, msg, c)
}

func OKWithData(data any, c *gin.Context) {
	Result(Success, data, "成功", c)
}

func OKWithMessage(msg string, c *gin.Context) {
	Result(Success, map[string]any{}, msg, c)
}

// 封装一些相应失败的
func Fail(data any, msg string, c *gin.Context) {
	Result(Error, data, msg, c)
}
func FailWithMessage(msg string, c *gin.Context) {
	Result(Error, map[string]any{}, msg, c)
}
func FailWitheCode(code ErrorCode, c *gin.Context) {
	msg, ok := ErrorMap[code]
	if ok {
		Result(int(code), map[string]any{}, msg, c)
		return
	}
	Result(Error, map[string]any{}, "未知错误", c)

}
```

## 错误状态码

错误状态码：
err_code. go
```go
package res

type ErrorCode int // type ErrorCode int是一种类型声明。这行代码定义了一个新的类型ErrorCode，它是int类型的别名。

// 定义
const (
	SettingsError ErrorCode = 1001 // 系统错误
)

var (
	ErrorMap = map[ErrorCode]string{
		SettingsError: "系统错误",
	}
)
```



如果是 JSON 类型数据错误：
err_code. json
```json
{  
	"1001": "系统错误"  
}
```

测试函数：

1. 错误码读取. go
```go
package main

import (
	"encoding/json"
	"fmt"
	"github.com/sirupsen/logrus"
	"os"
)

const file = "models/res/err_code.json"

type ErrMap map[int]string

func main() {
	// 读取文件
	byteData, err := os.ReadFile(file)
	if err != nil {
		logrus.Error(err)
		return
	}

	var errMap = ErrMap{}
	// 将json数据转化成type ErrMap map[string]string类型
	err = json.Unmarshal(byteData, &errMap)
	if err != nil {
		logrus.Error(err)
		return
	}

	fmt.Println(errMap)
}

```

# 表结构搭建

gorm.Model 包含了逻辑删除，如果需要逻辑删除功能，就把 MODEL 改成 gorm.Model 即可

用户表：
```go
// 用户表UserModel
type UserModel struct {
	gorm.Model
	MODEL
	NickName       string           `gorm:"size:36" json:"nick_name"`                                                         
	// 昵称
	UserName       string           `gorm:"size:36" json:"user_name"`                                                         
	// 用户名
	Password       string           `gorm:"size:128" json:"-"`                                                               
	// 密码
	Avatar         string           `gorm:"size:256" json:"avatar"`                                                           
	// 头像
	Email          string           `gorm:"size:128" json:"email"`                                                            
	// 邮箱
	Tel            string           `gorm:"size:18" json:"tel"`                                                               
	// 手机号
	Addr           string           `gorm:"size:64" json:"addr"`                                                              
	// 地址
	Token          string           `gorm:"size:64" json:"token"`                                                             
	// 其他平台的唯一id
	IP             string           `gorm:"size:20" json:"ip"`                                                               
	// ip地址
	Role           ctype.Role       `gorm:"size:4;default:2" json:"role"`                                                     
	// 权限  1 管理员  2 普通用户  3 游客
	SignStatus     ctype.SignStatus `gorm:"type=smallint(6)" json:"sign_status"`                                              // 注册来源
	ArticleModels  []ArticleModel   `gorm:"foreignKey:AuthID" json:"-"`                                                       
	// 发布的文章列表（一对多）
	CollectsModels []ArticleModel   `gorm:"many2many:auth2_collects;joinForeignKey:AuthID;JoinReferences:ArticleID" json:"-"` 
	// 收藏的文章列表（多对多：自定义多对多）
}
```
**本项目大量使用到了枚举**

### 表结构说明

```go
user_model.go                         用户表
banner_model.go                     banner 表
article_model.go                      文章表
advert_model.go                      广告表
comment_model.go                 评论表
enter.go
fade_back_model.go                用户反馈表
login_data_model.go                登录信息表
menu_banner_model.go          菜单 banner 表
menu_model.go                        菜单表
message_model.go                  消息表
tag_model.go                            标签表
user_collect_model.go             用户收藏文章表
```


### 绑定命令行参数

```go
package flag

import (
	sys_flag "flag"
)

type Option struct {
	DB bool
}

// Parse 解析命令行参数
func Parse() Option {
	db := sys_flag.Bool("db", false, "初始化数据库")
	// 解析命令行参数写入注册的flag里
	sys_flag.Parse()
	return Option{
		DB: *db,
	}
}

// IsWebStop 是否停止web项目
func IsWebStop(option Option) bool {
	if option.DB {
		return true
	}
	return false
}

// SwitchOption 根据命令执行不同的函数
func SwitchOption(option Option) {
	if option.DB {
		Makemigrations()
	}
}

```

### 表结构迁移

```go
package flag

import (
	"gvb_server/global"
	"gvb_server/models"
)

// 迁移表结构
// Makemigrations 初始化数据库表，数据库迁移等工作
func Makemigrations() {
	var err error
	global.DB.SetupJoinTable(&models.UserModel{}, "CollectsModels", &models.UserCollectModel{})
	global.DB.SetupJoinTable(&models.MenuModel{}, "Banners", &models.MenuBannerModel{})
	// 生成四张表的表结构
	err = global.DB.Set("gorm:table_options", "ENGINE=InnoDB").
		AutoMigrate(
			&models.BannerModel{},
			&models.TagModel{},
			&models.MessageModel{},
			&models.AdvertModel{},
			&models.UserModel{},
			&models.CommentModel{},
			&models.ArticleModel{},
			&models.MenuModel{},
			&models.MenuBannerModel{},
			&models.FadeBackModel{},
			&models.LoginDataModel{},
		)

	if err != nil {
		global.Log.Error("[ error ] 生成数据库表结构失败")
	}
	global.Log.Infof("[ success ] 生成数据库表结构成功")
}

```

执行代码：
```shell
go run main.go -db
```

# 系统配置 API


## 主要的配置

### site_info
### qq
### email
### qiniu

- 七牛云是一家位于中国的**云服务提供商**，成立于 2011 年，总部位于上海 。七牛云提供**一站式的云计算服务**，包括**对象存储**、CDN 加速、直播云、容器云、大数据、深度学习平台等产品 23。他们的服务覆盖音视频生产、处理、传输、消费全流程，集直播、点播、实时音视频、摄像头智能分析为一体，满足不同场景需求 。
- 七牛云的目标是**以数据科技全面驱动数字化未来**，赋能各行各业全面进入**数据时代** 。他们的客户包括 OPPO、爱奇艺、平安银行、招商银行、上汽集团、芒果 TV 等知名企业。

官网：
[七牛云 - 国内领先的企业级云服务商 (qiniu.com)](https://www.qiniu.com/.env#:~:text=%E4%B8%93%E6%B3%A8%E4%BA%8E%E4%BB%A5%E6%95%B0%E6%8D%AE%E4%B8%BA%E6%A0%B8,%E8%A7%86%E9%A2%91%E4%BA%91%E8%A7%A3%E5%86%B3%E6%96%B9%E6%A1%88%E3%80%82) 

#### 对象存储

基本概念：
[基本概念_产品简介_对象存储 - 七牛开发者中心 (qiniu.com)](https://developer.qiniu.com/kodo/3978/the-basic-concept) 

- 七牛云海量存储系统（Kodo）是自主研发的**非结构化数据存储管理平台**，支持中心和边缘存储。平台经过多年大规模用户验证已跻身先进技术行列，并广泛应用于海量数据管理的各类场景。

- 七牛对象存储**将数据文件以资源的形式上传到空间中**。您可以创建一个或者多个空间，然后向每个空间中上传一个或多个文件。通过**获取**已上传文件的**地址**进行文件的分享和下载。您还可以通过修改存储空间或文件的属性或元信息来设置相应的**访问权限**。

快速入门文档：
[快速入门文档_快速入门_对象存储 - 七牛开发者中心 (qiniu.com)](https://developer.qiniu.com/kodo/1233/console-quickstart) 

官网：
[对象存储 Kodo_云存储_海量安全高可靠云存储_oss - 七牛云 (qiniu.com)](https://www.qiniu.com/products/kodo) 


### jwt

**在代码运行的过程中可以动态修改的**

## 将多个配置浓缩为一个配置接口

好处：只有一个 api 接口，后端代码清晰简介
坏处：接口的入参和出参不统一

### 查询 api

```go
// api/settings_api/settings_info.go
package settings_api

import (
	"github.com/gin-gonic/gin"
	"gvb_server/global"
	"gvb_server/models/res"
)

type SettingsUri struct {
	Name string `uri:"name"`
}

// 查看某一些配置信息（系统数据）
func (SettingsApi) SettingsInfoView(c *gin.Context) {

	var cr SettingsUri
	err := c.ShouldBindUri(&cr)
	if err != nil {
		res.FailWitheCode(res.ArgumentError, c)
		return
	}
	switch cr.Name {
	case "site":
		res.OKWithData(global.Config.SiteInfo, c)
	case "email":
		res.OKWithData(global.Config.Email, c)
	case "qq":
		res.OKWithData(global.Config.QQ, c)
	case "qiniu":
		res.OKWithData(global.Config.QiNiu, c)
	case "jwt":
		res.OKWithData(global.Config.Jwy, c)
	default:
		res.FailWithMessage("没有对应的配置信息", c)
	}

}

```

### 更新 api

```go
// api/settings_api/settings_update.go
package settings_api

import (
	"github.com/gin-gonic/gin"
	"gvb_server/config"
	"gvb_server/core"
	"gvb_server/global"
	"gvb_server/models/res"
)

// 网站的相关信息，但是这个只能在配置文件修改，不太方便，后续最好优化成可以直接在前端页面修改的那种
// 配置文件的修改（这里只改了SiteInfo部分）
// 修改某一项的配置信息
func (SettingsApi) SettingsInfoUpdateView(c *gin.Context) {
	var cr SettingsUri
	err := c.ShouldBindUri(&cr)
	if err != nil {
		res.FailWitheCode(res.ArgumentError, c)
		return
	}

	switch cr.Name {
	case "site":
		var info config.SiteInfo
		err = c.ShouldBindJSON(&info)
		if err != nil {
			res.FailWitheCode(res.ArgumentError, c)
			return
		}
		global.Config.SiteInfo = info
	case "email":
		var info config.Email
		err = c.ShouldBindJSON(&info)
		if err != nil {
			res.FailWitheCode(res.ArgumentError, c)
			return
		}
		global.Config.Email = info
	case "qq":
		var info config.QQ
		err = c.ShouldBindJSON(&info)
		if err != nil {
			res.FailWitheCode(res.ArgumentError, c)
			return
		}
		global.Config.QQ = info
	case "qiniu":
		var info config.QiNiu
		err = c.ShouldBindJSON(&info)
		if err != nil {
			res.FailWitheCode(res.ArgumentError, c)
			return
		}
		global.Config.QiNiu = info
	case "jwt":
		var info config.Jwy
		err = c.ShouldBindJSON(&info)
		if err != nil {
			res.FailWitheCode(res.ArgumentError, c)
			return
		}
		global.Config.Jwy = info
	default:
		res.FailWithMessage("没有对应的配置信息", c)
		return
	}

	// 修改配置信息
	core.SetYaml()
	res.OKWith(c)
}
```

**修改配置信息**： 

```go
// core/conf.go
package core

import (
	"fmt"
	"gopkg.in/yaml.v2"
	"gvb_server/config"
	"gvb_server/global"
	"io/fs"
	"io/ioutil"
	"log"
)

const ConfigFile = "settings.yaml"

// 对conf的初始化
// 读.yaml文件
// InitConf 读取yaml文件的配置
func InitConf() {

	c := &config.Config{} // 读取的配置文件要存储在一个实体中
	yamlConf, err := ioutil.ReadFile(ConfigFile)
	if err != nil {
		panic(fmt.Errorf("get yamlconf error: %S", err))
	}
	err = yaml.Unmarshal(yamlConf, c)
	if err != nil {
		log.Fatalf("config Init Unmarshal: %V", err)
	}
	log.Println("config yamlFile Load Init success.")
	global.Config = c // 存到一个实体中
}

// 修改配置文件!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!这里这里这里
func SetYaml() error {
	// 这个函数接受一个任意类型的输入参数in，并返回一个字节切片out和一个错误err
	// 这里是将整个配置文件转换成字节切片
	byteData, err := yaml.Marshal(global.Config)
	if err != nil {
		return err
	}

	// func WriteFile(filename string, data []byte, perm fs.FileMode) error
	err = ioutil.WriteFile(ConfigFile, byteData, fs.ModePerm)
	if err != nil {
		return err
	}

	global.Log.Info("配置文件修改成功")
	return nil
}

```

# 图片管理
## 图片上传

### 上传多个图片

这部分的图片都是本地的图片，所以重新定义了一个本地图片的存储变量：
```go
package config

// 本地图片
type Upload struct {
	Size int    `yaml:"size" json:"size"` // 图片上传的大小，MB
	Path string `yaml:"path" json:"path"` // 图片上传的目录
}
```
在配置文件中同样加入对应的信息啦...

```go
package images_api

import (
	"fmt"
	"github.com/gin-gonic/gin"
	"gvb_server/global"
	"gvb_server/models/res"
	"os"
	"path"
)

// 图片上传的响应
type FileUploadResponse struct {
	FileName  string `json:"file_name"`  // 文件名
	IsSuccess bool   `json:"is_success"` // 是否上传成功
	Msg       string `json:"msg"`        // 消息
}

// 上传图片，返回图片的URL
func (ImagesApi) ImageUploadView(c *gin.Context) {
	// 上传多个图片文件
	form, err := c.MultipartForm()
	if err != nil {
		res.FailWithMessage(err.Error(), c)
		return
	}

	fileList, ok := form.File["images"]
	if !ok {
		res.FailWithMessage("不存在的文件", c)
		return
	}

	// 判断路径是否存在
	// 截断（逐级判断）
	basePath := global.Config.Upload.Path
	// 在Go语言中，os.ReadDir(basePath)函数用于读取指定路径basePath下的所有目录条目，并将它们以文件名排序后返回。
	// 这个函数返回一个FileInfo结构的数组，每个元素代表一个目录条目。
	// 这个函数可以用来列出一个目录下的所有文件和子目录。
	_, err = os.ReadDir(basePath)
	if err != nil {
		// 在Go语言中，os.MkdirAll(basePath, os.ModePerm)函数用于创建指定路径basePath下的所有目录，包括任何必要的父目录。
		// 这个函数返回nil，或者在出错时返回一个错误。
		// 第二个参数os.ModePerm是一个权限位，用于设置所有被该函数创建的目录的读/写/执行权限。
		// 在Unix-like系统中，os.ModePerm等价于07772，意味着用户有权列出、修改和搜索目录中的文件。
		// 如果目录已经存在，os.MkdirAll()函数不会做任何事情，而是返回nil。
		// 递归创建
		err = os.MkdirAll(basePath, os.ModePerm)
		if err != nil {
			global.Log.Error(err)
		}
	}

	// 不存在就创建
	var resList []FileUploadResponse

	for _, file := range fileList {
		// 存数据库。。。。

		// 图片存储
		filePath := path.Join(basePath, file.Filename) // 存储路径

		// 判断大小
		// 加float64是因为要进行浮点数除法
		size := float64(file.Size) / float64(1024*1024)
		if size > float64(global.Config.Upload.Size) {
			resList = append(resList, FileUploadResponse{
				FileName:  file.Filename,
				IsSuccess: false,
				Msg:       fmt.Sprintf("图片大小超过设定大小，当前大小为：%.2fMB，设定大小为：%dMB", size, global.Config.Upload.Size),
			})
			continue
		}

		// 上传
		err := c.SaveUploadedFile(file, filePath)

		// 上传失败
		if err != nil {
			global.Log.Error(err)
			resList = append(resList, FileUploadResponse{
				FileName:  file.Filename,
				IsSuccess: false,
				Msg:       err.Error(),
			})
			continue
		}

		// 上传成功
		resList = append(resList, FileUploadResponse{
			FileName:  filePath,
			IsSuccess: true,
			Msg:       "上传成功",
		})

	}

	res.OKWithData(resList, c)
}

```

#### 递归创建文件夹
```go
// 在Go语言中，os.MkdirAll(basePath, os.ModePerm)函数用于创建指定路径basePath下的所有目录，包括任何必要的父目录。  
// 这个函数返回nil，或者在出错时返回一个错误。  
// 第二个参数os.ModePerm是一个权限位，用于设置所有被该函数创建的目录的读/写/执行权限。  
// 在Unix-like系统中，os.ModePerm等价于07772，意味着用户有权列出、修改和搜索目录中的文件。  
// 如果目录已经存在，os.MkdirAll()函数不会做任何事情，而是返回nil。  
// 递归创建  
err = os.MkdirAll(basePath, os.ModePerm)
```

#### 黑名单，白名单

黑名单
判断文件名后缀，如果与黑名单中的后缀符合，那就拒绝上传

白名单
只能上传在白名单中出现的文件后缀

```go
// api/images_api/image_upload.go
// 白名单
var (
	// WhiteImageList 图片上传的白名单
	WhiteImageList = []string{
		"jpg",
		"png",
		"jpeg",
		"gif",
		"ico",
		"tiff",
		"svg",
		"webp",
	}
)

// 具体判断部分
// 判断图片是否存在在白名单中
	fileName := file.Filename
	nameList := strings.Split(fileName, ".") // 图片名字切片
	suffix := nameList[len(nameList)-1]      // 获取后缀名
	if !utils.Inlist(suffix, WhiteImageList) {
		resList = append(resList, FileUploadResponse{
			FileName:  file.Filename,
			IsSuccess: false,
			Msg:       "非法文件",
		})
		continue
	}
```

判断字符串是否存在的自定义函数：
```go
// utils/utils.go
package utils

// Inlist 判断某字符串是否存在在字符串列表里面
func Inlist(key string, list []string) bool {
	for _, s := range list {
		if s == key {
			return true
		}
	}
	return false
}
```

### 图片存入数据库

#### MD5 加密

- MD5**信息摘要算法**，一种被广泛使用的密码散列函数，可以产生出一个 128 位（16 字节）的散列值（hash value），用于确保信息传输完整一致。
- MD5 算法因其普遍、稳定、快速的特点，仍广泛应用于普通数据的加密保护领域。

代码实用：
```go
package utils

import (
	"crypto/md5"
	"encoding/hex"
)

// Md5 md5
func Md5(src []byte) string {
	m := md5.New()
	m.Write(src)
	res := hex.EncodeToString(m.Sum(nil))
	return res
}

```

### 图片上传至七牛云

```go
// plugins/qiniu/enter.go
package qiniu

import (
	"bytes"
	"errors"
	"fmt"
	"github.com/qiniu/go-sdk/v7/auth/qbox"
	"github.com/qiniu/go-sdk/v7/storage"
	"golang.org/x/net/context"
	"gvb_server/config"
	"gvb_server/global"
	"time"
)

// 获取token
func getToken(q config.QiNiu) string {
	accessKey := q.AccessKey
	secretKey := q.SecretKey
	bucket := q.Bucket
	putPolicy := storage.PutPolicy{
		Scope: bucket,
	}
	mac := qbox.NewMac(accessKey, secretKey)
	upToken := putPolicy.UploadToken(mac)
	return upToken
}

// 获取上传的配置
func getCfg(q config.QiNiu) storage.Config {
	cfg := storage.Config{}
	// 空间对应的机房
	zone, _ := storage.GetRegionByID(storage.RegionID(q.Zone))
	cfg.Zone = &zone
	// 是否使用https域名
	cfg.UseHTTPS = false
	// 上传是否使用CDN上传加速
	cfg.UseCdnDomains = false
	return cfg
}

// 上传图片，文件数组，前缀
func UploadImage(data []byte, imageName string, prefix string) (filePath string, err error) {
	if !global.Config.QiNiu.Enable {
		return "", errors.New("请启用七牛云上传")
	}

	q := global.Config.QiNiu
	if q.AccessKey == "" || q.SecretKey == "" {
		return "", errors.New("请配置七牛云的AccessKey和SecretKey")
	}

	if float64(len(data))/1024/1024 > q.Size {
		return "", errors.New("文件超过长度")
	}

	upToken := getToken(q)
	cfg := getCfg(q)

	formUploader := storage.NewFormUploader(&cfg)
	ret := storage.PutRet{}
	putExtra := storage.PutExtra{
		Params: map[string]string{},
	}
	dataLen := int64(len(data))

	// 获取当前时间
	now := time.Now().Format("20060102150405")
	key := fmt.Sprintf("%s/%s__%s", prefix, now, imageName)

	err = formUploader.Put(context.Background(), &ret, upToken, key, bytes.NewReader(data), dataLen, &putExtra)
	if err != nil {
		return "", err
	}

	return fmt.Sprintf("%s%s", q.CDN, ret.Key), nil
}

```

配置文件含义教程：
[33.图片管理知识回顾_哔哩哔哩_bilibili](https://www.bilibili.com/video/BV1f24y1G72C/?p=33&spm_id_from=pageDriver&vd_source=5fd2c97b55c9ca08b4bb2e9f07fbea9a) 

### 图片上传优化

这个模块，主要负责处理图片的上传逻辑
- 白名单
- 文件大小限制
- 文件 hash
- 是否上传七牛
```go
// service/image_ser/image_upload_service.go
package image_ser

import (
	"fmt"
	"gvb_server/global"
	"gvb_server/models"
	"gvb_server/models/ctype"
	"gvb_server/plugins/qiniu"
	"gvb_server/utils"
	"io"
	"mime/multipart"
	"path"
	"strings"
)

var (
	// WhiteImageList 图片上传的白名单
	WhiteImageList = []string{
		"jpg",
		"png",
		"jpeg",
		"gif",
		"ico",
		"tiff",
		"svg",
		"webp",
	}
)

type FileUploadResponse struct {
	FileName  string `json:"file_name"`  // 文件名
	IsSuccess bool   `json:"is_success"` // 是否上传成功
	Msg       string `json:"msg"`        // 消息
}

// 向数据库添加数据，如果重复就不要添加啦
// ImageUploadService 文件上传方法
func (ImageService) ImageUploadService(file *multipart.FileHeader) (res FileUploadResponse) {
	// 判断图片是否存在在白名单中
	fileName := file.Filename
	basePath := global.Config.Upload.Path
	filePath := path.Join(basePath, file.Filename) // 存储路径

	res.FileName = filePath

	// 判断文件白名单
	nameList := strings.Split(fileName, ".") // 图片名字切片
	suffix := nameList[len(nameList)-1]      // 获取后缀名
	if !utils.Inlist(suffix, WhiteImageList) {
		res.Msg = "非法文件"
		return
	}

	// 判断大小
	// 加float64是因为要进行浮点数除法
	size := float64(file.Size) / float64(1024*1024)
	if size > float64(global.Config.Upload.Size) {
		res.Msg = fmt.Sprintf("图片大小超过设定大小，当前大小为：%.2fMB，设定大小为：%dMB", size, global.Config.Upload.Size)
		return
	}

	// 读取文件内容 hash
	fileObj, err := file.Open()
	if err != nil {
		global.Log.Error(err)
	}
	byteData, err := io.ReadAll(fileObj)
	imageHash := utils.Md5(byteData)
	// 去数据库中查这个图片是否存在
	var bannerModel models.BannerModel
	err = global.DB.Take(&bannerModel, "hash = ?", imageHash).Error
	if err == nil {
		// 找到了
		res.FileName = bannerModel.Path
		res.Msg = "图片已存在"
		return
	}

	fileType := ctype.Local
	res.Msg = "图片上传成功"
	res.IsSuccess = true

	// 是否上传到七牛云
	if global.Config.QiNiu.Enable {
		filePath, err = qiniu.UploadImage(byteData, fileName, global.Config.QiNiu.Prefix)
		if err != nil {
			global.Log.Error(err)
			res.Msg = err.Error()
			return
		}
		res.FileName = filePath
		res.Msg = "上传七牛云成功"
		fileType = ctype.QiNiu
	}

	// 图片入库
	global.DB.Create(&models.BannerModel{
		Path:      filePath,
		Hash:      imageHash,
		Name:      fileName,
		ImageType: fileType,
	})
	return
}

```

调用方：
**只用管与前端的交互，判断处理部分在 service，如上** 
```go
// api/images_api/image_upload.go
package images_api

import (
	"github.com/gin-gonic/gin"
	"gvb_server/global"
	"gvb_server/models/res"
	"gvb_server/service"
	"gvb_server/service/image_ser"
	"os"
)

// 上传图片，返回图片的URL
func (ImagesApi) ImageUploadView(c *gin.Context) {
	// 上传多个图片文件
	form, err := c.MultipartForm()
	if err != nil {
		res.FailWithMessage(err.Error(), c)
		return
	}

	fileList, ok := form.File["images"]
	if !ok {
		res.FailWithMessage("不存在的文件", c)
		return
	}

	// 判断路径是否存在
	// 截断（逐级判断）
	basePath := global.Config.Upload.Path
	// 在Go语言中，os.ReadDir(basePath)函数用于读取指定路径basePath下的所有目录条目，并将它们以文件名排序后返回。
	// 这个函数返回一个FileInfo结构的数组，每个元素代表一个目录条目。
	// 这个函数可以用来列出一个目录下的所有文件和子目录。
	_, err = os.ReadDir(basePath)
	if err != nil {
		// 在Go语言中，os.MkdirAll(basePath, os.ModePerm)函数用于创建指定路径basePath下的所有目录，包括任何必要的父目录。
		// 这个函数返回nil，或者在出错时返回一个错误。
		// 第二个参数os.ModePerm是一个权限位，用于设置所有被该函数创建的目录的读/写/执行权限。
		// 在Unix-like系统中，os.ModePerm等价于07772，意味着用户有权列出、修改和搜索目录中的文件。
		// 如果目录已经存在，os.MkdirAll()函数不会做任何事情，而是返回nil。
		// 递归创建
		err = os.MkdirAll(basePath, os.ModePerm)
		if err != nil {
			global.Log.Error(err)
		}
	}

	// 不存在就创建
	// 图片上传结果数组
	var resList []image_ser.FileUploadResponse

	for _, file := range fileList {
		// 上传文件
		serviceRes := service.ServiceApp.ImageService.ImageUploadService(file)
		// 没有上传成功的话
		if !serviceRes.IsSuccess {
			resList = append(resList, serviceRes)
			continue
		}

		// 上传成功, 不上传到七牛，那么本地还需要保存一下
		if !global.Config.QiNiu.Enable {
			err = c.SaveUploadedFile(file, serviceRes.FileName)
			// 上传失败
			if err != nil {
				global.Log.Error(err)
				serviceRes.Msg = err.Error()
				serviceRes.IsSuccess = false
				resList = append(resList, serviceRes)
				continue
			}
		}

		resList = append(resList, serviceRes)
	}

	res.OKWithData(resList, c)
}

```

## 图片列表

封装一个统一用于列表查询的函数
涵数支持**分页查询**，**模糊匹配**，**预加载**, **where 查询**

### 分页查询

```go
// service/common/service_list.go
package common

import (
	"gorm.io/gorm"
	"gvb_server/global"
	"gvb_server/models"
)

// 分页和搜索
type Option struct {
	models.PageInfo
	Debug bool // 如果为true，就看日志(即.Debug())
}

// 列表页
func ComList[T any](model T, option Option) (list []T, count int64, err error) {
	DB := global.DB
	if option.Debug {
		DB = global.DB.Session(&gorm.Session{Logger: global.MysqlLog})
	}

	// 排序设置
	if option.Sort == "" {
		option.Sort = "created_at desc" // 默认按照时间往前排（从后往前排）  
		//option.Sort = "created_at asc" // 默认按照时间往后排
	}

	// 查找图片数据库中的总数.RowsAffected
	count = DB.Select("id").Find(&list).RowsAffected
	// SELECT `id` FROM `banner_models`比
	// lobal.DB.Debug().Find(&imageList).RowsAffected的
	// SELECT * FROM `banner_models` 好

	offset := (option.Page - 1) * option.Limit // 偏移量，就是当前是从第几页开始的
	if offset < 0 {                            // 即cr.Page为0，那么offset为-1；
		offset = 0
	}
	// 分页查询（常用）
	err = DB.Limit(option.Limit).Offset(offset).Order(option.Sort).Find(&list).Error

	return list, count, err
}
```


如何调用：
```go
// api/images_api/image_list.go
package images_api

import (
	"github.com/gin-gonic/gin"
	"gvb_server/models"
	"gvb_server/models/res"
	"gvb_server/service/common"
)

// ImageListView 图片列表查询页
func (ImagesApi) ImageListView(c *gin.Context) {
	// 分页
	var cr models.PageInfo
	err := c.ShouldBindQuery(&cr)
	if err != nil {
		res.FailWitheCode(res.ArgumentError, c)
		return
	}

	list, count, err := common.ComList(models.BannerModel{}, common.Option{
		PageInfo: cr,
		Debug:    true,
	})

	res.OKWithList(list, count, c)
	return
}
```

#### 分页

sql 实现分页
```go
// 一页展示一条,偏移两条，也就是从第三条开始,获取1条
select * from banner_models limit 1 offset 2;
// golang
// DB.Limit(option.Limit).Offset(offset).Find(&list)
DB.Limit(1).Offset(2).Find(&list)
```

#### 排序

默认排序值设置：
```go
// 排序设置
if option.Sort == "" {
	option.Sort = "created_at desc" // 默认按照时间往前排（从后往前排）  
	//option.Sort = "created_at asc" // 默认按照时间往后排
}
```

数据库部分 (.Order(option.Sort))：
```go
// 分页查询（常用）  
err = DB.Limit(option.Limit).Offset(offset).Order(option.Sort).Find(&list).Error
```

### 删除图片

**对数据库中数据的增，删，改都需要增加事务，即 Hook（钩子）**

对象生命周期：
[Hook | GORM - The fantastic ORM library for Golang, aims to be developer friendly.](https://gorm.io/zh_CN/docs/hooks.html) 

#### 钩子函数 

```go
// models/banner_model.go
package models

import (
	"gorm.io/gorm"
	"gvb_server/global"
	"gvb_server/models/ctype"
	"os"
)

type BannerModel struct {
	MODEL
	Path      string          `json:"path"`                        // 路径图片
	Hash      string          `json:"hash"`                        // 图片的hash值，用于判断重复图片
	Name      string          `gorm:"size:38" json:"name"`         // 图片名称
	ImageType ctype.ImageType `gorm:"default:1" json:"image_type"` // 图片类型，本地还是七牛(默认为1，即本地)
}

// 在同一个事务中更新数据
func (b *BannerModel) BeforeDelete(tx *gorm.DB) (err error) {
	if b.ImageType == ctype.Local {
		// 本地图片，删除，还要删除本地的存储
		err := os.Remove(b.Path)
		if err != nil {
			global.Log.Error(err)
			return err
		}
	}
	return nil
}

```

#### 批量删除

```go
// api/images_api/image_remove.go
package images_api

import (
	"fmt"
	"github.com/gin-gonic/gin"
	"gvb_server/global"
	"gvb_server/models"
	"gvb_server/models/res"
)

// 批量删除图片，根据idList从数据库中进行批量删除
func (ImagesApi) ImageRemoveView(c *gin.Context) {
	var cr models.RemoveRequest // 要删除的图片的id列表
	err := c.ShouldBindJSON(&cr)
	if err != nil {
		res.FailWitheCode(res.ArgumentError, c)
		return
	}

	// 先从数据库中查询对应的信息
	var imageList []models.BannerModel
	count := global.DB.Find(&imageList, cr.IDList).RowsAffected
	if count == 0 {
		res.OKWithMessage("文件不存在", c)
		return
	}

	// 查到啦，就要批量删除喽
	global.DB.Delete(&imageList)
	res.OKWithMessage(fmt.Sprintf("共删除 %d 张图片", count), c)

}

```

### 图片修改

代码：
```go
// api/images_api/image_update.go
package images_api

import (
	"github.com/gin-gonic/gin"
	"gvb_server/global"
	"gvb_server/models"
	"gvb_server/models/res"
)

type ImageUpdateRequest struct {
	ID   uint   `json:"id" binding:"required" msg:"请选择文件id"`
	Name string `json:"name" binding:"required" msg:"请选择文件名称"` // 新名字
}

// ImageUpdateView 修改图片名称(没有修改存储路径上图片的名字，只改数据库中的name)
func (ImagesApi) ImageUpdateView(c *gin.Context) {
	// 转换前端传来的数据
	var cr ImageUpdateRequest
	err := c.ShouldBindJSON(&cr)
	if err != nil {
		// 根据报错信息，确定是因为什么数据没传导致的报错
		res.FailWithError(err, &cr, c)
		return
	}

	// 查找是否存在
	var imageModel models.BannerModel
	err = global.DB.Take(&imageModel, cr.ID).Error
	if err != nil {
		res.FailWithMessage("文件不存在", c)
		return
	}
	// 存在就修改
	err = global.DB.Model(&imageModel).Update("name", cr.Name).Error
	if err != nil {
		res.FailWithMessage(err.Error(), c)
		return
	}

	res.OKWithMessage("图片名称修改成功", c)
	return
}

```

#### 前端数据处理

前端的 json 数据：
```go
type ImageUpdateRequest struct {
	ID   uint   `json:"id" binding:"required" msg:"请选择文件id"`
	Name string `json:"name" binding:"required" msg:"请选择文件名称"` // 新名字
}
```

```go
// utils/valid.go
package utils

import (
	"github.com/go-playground/validator/v10"
	"reflect"
)

// GetValidMsg 返回结构体中的msg参数 ---------------- 参数绑定
func GetValidMsg(err error, obj any) string {
	// 使用的时候，需要传obj的指针
	getObj := reflect.TypeOf(obj)
	// 将err接口断言为具体类型
	if errs, ok := err.(validator.ValidationErrors); ok {
		// 断言成功
		for _, e := range errs {
			// 循环每一个错误信息
			// 根据报错字段名，获取结构体的具体字段
			if f, exits := getObj.Elem().FieldByName(e.Field()); exits {
				msg := f.Tag.Get("msg")
				return msg
			}
		}
	}
	return err.Error()
}

```

使用：

```go
// models/res/response.go
func FailWithError(err error, obj any, c *gin.Context) {
	// 获取报错中的信息
	msg := utils.GetValidMsg(err, obj)
	FailWithMessage(msg, c)
}
```

即，如果前端传的信息不全，那么会有对应提示
![[Pasted image 20231005160417.png]]
**因为这里缺少必须要传的 name 信息**

# 广告管理

## 广告添加

```go
// api/advert_api/advert_create.go
package advert_api

import (
	"github.com/gin-gonic/gin"
	"gvb_server/global"
	"gvb_server/models"
	"gvb_server/models/res"
)

type AdvertRequest struct {
	Title  string `json:"title" binding:"required" msg:"请输入标题"`       // 显示的标题
	Href   string `json:"href" binding:"required,url" msg:"跳转链接非法"`   // 跳转链接
	Images string `json:"images" binding:"required,url" msg:"图片地址非法"` // 图片
	IsShow bool   `json:"is_show" msg:"请选择是否展示"`                      // 是否展示
}

func (AdvertApi) AdvertCreate(c *gin.Context) {
	var cr AdvertRequest
	err := c.ShouldBindJSON(&cr)
	if err != nil {
		res.FailWithError(err, &cr, c)
		return
	}

	// 重复判断
	var advert models.AdvertModel
	err = global.DB.Take(&advert, "title = ?", cr.Title).Error
	if err == nil { // 如果数据库中存在
		res.FailWithMessage("该广告已存在", c)
		return
	}

	err = global.DB.Create(&models.AdvertModel{
		Title:  cr.Title,
		Href:   cr.Href,
		Images: cr.Images,
		IsShow: cr.IsShow,
	}).Error

	if err != nil {
		global.Log.Error(err)
		res.FailWithMessage("添加广告失败", c)
		return
	}

	res.OKWithMessage("添加广告成功", c)
}
```

对于重复值的判断
- 唯一索引，如果某个字段在入库的时候重复，那么 create 就会报错 
- 使用添加的钩子函数，入库之前去查一次
- 简单处理, 添加之前查一次

**上面用的是第三种**

## 广告列表

根据 Referer 判断请求是从哪里发出的
做到差异化数据展示
**判断 Referer 是否包含 admin，如果是（表示后台查询），就全部返回，不是就返回 is_show=true 的 (就只展示需要展示的)**

```go
// api/advert_api/advert_list.go
package advert_api

import (
	"github.com/gin-gonic/gin"
	"gvb_server/models"
	"gvb_server/models/res"
	"gvb_server/service/common"
	"strings"
)

// 分页查询

func (AdvertApi) AdvertListView(c *gin.Context) {
	var cr models.PageInfo
	if err := c.ShouldBindQuery(&cr); err != nil {
		res.FailWitheCode(res.ArgumentError, c)
		return
	}

	referer := c.GetHeader("Referer")
	isShow := true
	if strings.Contains(referer, "admin") {
		// admin来的
		isShow = false
	}
	// 判断Referer是否包含admin，如果是（表示后台查询），就全部返回，不是就返回is_show=true的(就只展示需要展示的)
	list, count, _ := common.ComList(models.AdvertModel{IsShow: isShow}, common.Option{
		PageInfo: cr,
		Debug:    true,
	})

	res.OKWithList(list, count, c)
}

```

## 广告编辑


**用到了结构体转 map 的第三方包 structs，如下** 
```go
// api/advert_api/advert_update.go
package advert_api

import (
	"github.com/fatih/structs"
	"github.com/gin-gonic/gin"
	"gvb_server/global"
	"gvb_server/models"
	"gvb_server/models/res"
)

func (AdvertApi) AdvertUpdateView(c *gin.Context) {
	id := c.Param("id")
	var cr AdvertRequest
	err := c.ShouldBindJSON(&cr)
	if err != nil {
		res.FailWithError(err, &cr, c)
		return
	}

	// 判断广告是否存在
	var advert models.AdvertModel
	err = global.DB.Take(&advert, id).Error
	if err != nil { // 如果数据库中存在
		res.FailWithMessage("广告不存在", c)
		return
	}

	// 结构体转map的第三方包structs
	maps := structs.Map(&cr)
	// Updates 好用嗷嗷嗷嗷嗷嗷嗷嗷嗷嗷
	err = global.DB.Model(&advert).Updates(maps).Error

	// 结构体转map的第三方包

	if err != nil {
		global.Log.Error(err)
		res.FailWithMessage("修改广告失败", c)
		return
	}

	res.OKWithMessage("修改广告成功", c)
}
```

### 结构体转 map 的第三方包

#### 优点

- 主要用于 struct 转 map 
- 还可以判断结构体是否有**空属性**等功能

方法：
[【Golang 接口自动化07】struct转map的三种方式-CSDN博客](https://blog.csdn.net/GDYY3721/article/details/132020762) 

#### 使用第三方库 structs 

使用第三方库 structs 要下载的依赖：
```shell
 go get github.com/fatih/structs
```

测试用法如下：
```go
// testdata/2.structs.go
package main

import (
	"fmt"
	"github.com/fatih/structs"
	"gvb_server/models"
)

type AdvertRequest struct {
	models.MODEL `structs:"-"`
	Title        string `json:"title" binding:"required" msg:"请输入标题" structs:"title"` // 显示的标题
	Href         string `json:"href" binding:"required,url" msg:"跳转链接非法" structs:"-"` // 跳转链接
	Images       string `json:"images" binding:"required,url" msg:"图片地址非法"`           // 图片
	IsShow       bool   `json:"is_show" msg:"请选择是否展示" structs:"is_show"`              // 是否展示
}

func main() {
	u1 := AdvertRequest{
		Title:  "123",
		Href:   "xxx",
		Images: "xxx",
		IsShow: true,
	}
	m3 := structs.Map(&u1)
	fmt.Println(m3)
}
```

运行结果：
![[Pasted image 20231006211525.png]]

较详细用法：
[Golang - Structs 包的使用_golang structs包_叁丶贰壹的博客-CSDN博客](https://blog.csdn.net/yang_kaiyue/article/details/120989276) 

## 广告删除

```go
package advert_api

import (
	"fmt"
	"github.com/gin-gonic/gin"
	"gvb_server/global"
	"gvb_server/models"
	"gvb_server/models/res"
)

// 批量删除广告，根据idList从数据库中进行批量删除
func (AdvertApi) AdvertRemoveView(c *gin.Context) {
	var cr models.RemoveRequest // 要删除的广告的id列表
	err := c.ShouldBindJSON(&cr)
	if err != nil {
		res.FailWitheCode(res.ArgumentError, c)
		return
	}

	// 先从数据库中查询对应的信息
	var advertList []models.AdvertModel
	count := global.DB.Find(&advertList, cr.IDList).RowsAffected
	if count == 0 {
		res.OKWithMessage("广告不存在", c)
		return
	}

	// 查到啦，就要批量删除喽
	global.DB.Delete(&advertList)
	res.OKWithMessage(fmt.Sprintf("共删除 %d 个广告", count), c)

}
```


# swag 集成

## 下载 swag

教程： 
[40.swag文档接入_哔哩哔哩_bilibili](https://www.bilibili.com/video/BV1f24y1G72C/?p=40&spm_id_from=pageDriver&vd_source=5fd2c97b55c9ca08b4bb2e9f07fbea9a) 

### 安装依赖
```shell
// 如果其他项目下载过swag，那么就不需要再运行这个啦
go install github.com/swaggo/swag/cmd/swag@latest


//生成api文档
swag init

浏览器访问
http://ip:host/swagger/index.html
```

然后执行这两个，安装依赖
```shell
go get -u github.com/swaggo/gin-swagger
go get -u github.com/swaggo/files
```

### main. go 部分

代码部分：
**在 main. go 中** 
```go
import (
	_ "docs" // swag init生成后的docs路径
)

// @title API文档
// @version 1.0
// @description API文档
// @host 127.0.0.01:9000  // 写对可以在线测试  
// @BasePath / // 根路径
func main(){}
```

**如下:**  
gvb_server 项目中的使用：
```go
// @title gvb_server API文档  
// @version 1.0  
// @description gvb_server API文档  
// @host 127.0.0.01:8080
// @BasePath / 
```
然后运行：
```shell
//生成api文档
swag init
```

就会自动生成一个 docs 目录，
同时在 main. go 中手动添加 swag init 生成后的 **docs 路径**， 
如：
```go
import (
	_ "docs" // swag init生成后的docs路径
)
```
本项目中为：
![[Pasted image 20231007133734.png]]

### 路由部分

```go
import (
	"github.com/gin-gonic/gin"
	swaggerFiles "github.com/swaggo/files"
	gs "github.com/swaggo/gin-swagger"
)
func Routers( *gin.Engine {
	Router := gin.Default()
	PublicGroup := Router.Group("")
	PublicGroup.GET("/swagger/*any", gs.WrapHandler(swaggerFiles.Handler))
}
```

**如下：**

```go
// 在routers/enter.go 中对应位置加上：
router.GET("/swagger/*any", gs.WrapHandler(swaggerFiles.Handler))

// 然后补齐 gs "github.com/swaggo/gin-swagger" 这句
```

之后**再次 swag init**  即可。

### 浏览器打开

- 上面依次操作之后，重新运行
- 浏览器：127.0.0.1:8080/swagger/index.html 

![[Pasted image 20231007135028.png]]


### 使用

注释：
```go
// @Tags标签
// @summary标题
// @Description 描述，可以有多个
// @Param limit query string false "查询参数,表示单个参数" 
// @Param data body request.Request  true  "表示多个参数,参数是data，body表示json参数" 
// @Router /api/users [post]
// @Produce json
// @success 200 {object} gin.H{"msg":“响应"}
func (a *Api) UsersApi(c *gin.context){}
```

**如下：** 

以**广告部分**为例：
#### 广告管理的增删改查

```go
// AdvertCreateView 添加广告  
// @Tags 广告管理  
// @summary 创建广告  
// @Description 创建广告  
// @Param data body AdvertRequest true "表示多个参数"  
// @Router /api/adverts [post]  
// @Produce json  
// @success 200 {object} res.Response{}  
func (AdvertApi) AdvertCreateView(c *gin.Context) {}

// AdvertRemoveView 批量删除广告，根据idList从数据库中进行批量删除  
// @Tags 广告管理  
// @summary 批量删除广告  
// @Description 批量删除广告  
// @Param data body models.RemoveRequest true "广告的id列表"  
// @Router /api/adverts [delete]  
// @Produce json  
// @success 200 {object} res.Response{data=string}  
func (AdvertApi) AdvertRemoveView(c *gin.Context) {}

// AdvertUpdateView 更新广告  
// @Tags 广告管理  
// @summary 更新广告  
// @Description 更新广告  
// @Param data body AdvertRequest true "广告的一些参数"  
// @Router /api/adverts/:id [put]  
// @Produce json  
// @success 200 {object} res.Response{data=string}  
func (AdvertApi) AdvertUpdateView(c *gin.Context) {}

// AdvertListView 广告列表分页查询  
// @Tags 广告管理  
// @summary 广告列表  
// @Description 广告列表  
// @Param data query models.PageInfo false "查询参数"  
// @Router /api/adverts [get]  
// @Produce json  
// @success 200 {object} res.Response{data=res.ListResponse[models.AdvertModel]}  
func (AdvertApi) AdvertListView(c *gin.Context) {}
```

结果：
![[Pasted image 20231007143811.png]]

# air 项目自动重启

安装下载 air:
```shell
go install github.com/cosmtrek/air@latest
```

之后终端输入
```shell
air
```

**之后任意修改代码，都会自动重新运行**

air **配置文件**：
```shell
air init
```
即，
![[Pasted image 20231007145054.png]]

之后加到. gitignore：

**. gitignore** ： 
- git 为我们提供了一个. gitignore 文件，只要在这个文件中声明哪些文件你不希望添加到 git 中去，这样当你使用 git add .的时候这些文件就会被自动忽略掉。
![[Pasted image 20231007150202.png]]


# 菜单管理

**注意这里有第三张表嗷**

## 添加菜单

```go
// api/menu_api/menu_create.go
package menu_api

import (
	"github.com/gin-gonic/gin"
	"gvb_server/global"
	"gvb_server/models"
	"gvb_server/models/ctype"
	"gvb_server/models/res"
)

type ImageSort struct {
	ImageID uint `json:"image_id"`
	Sort    int  `json:"sort"`
}

type MenuRequest struct {
	Title         string      `json:"title" binding:"required" msg:"请完善菜单名称" structs:"title"`
	Path          string      `json:"path" binding:"required" msg:"请完善菜单路径" structs:"path"`
	Slogan        string      `json:"slogan" structs:"slogan"`
	Abstract      ctype.Array `json:"abstract" structs:"abstract"`
	AbstractTime  int         `json:"abstract_time" structs:""`                                    // 切换时间，单位秒
	BannerTime    int         `json:"banner_time" structs:"abstract_time"`                         // 切换时间，单位秒
	Sort          int         `json:"sort" binding:"required" msg:"请输入菜单序号" structs:"banner_time"` // 菜单序号
	ImageSortList []ImageSort `json:"image_sort_list" structs:"-"`                                 // 具体图片的顺序
}

func (MenuApi) MenuCreateView(c *gin.Context) {
	var cr MenuRequest
	err := c.ShouldBindJSON(&cr)
	if err != nil {
		res.FailWithError(err, &cr, c)
		return
	}

	// 重复值判断
	var menuList []models.MenuModel
	count := global.DB.Find(&menuList, "title = ? or path = ?", cr.Title, cr.Path).RowsAffected
	if count > 0 {
		res.FailWithMessage("菜单已存在", c)
		return
	}

	// MenuModel 入库
	menuModel := models.MenuModel{
		Title:        cr.Title,
		Path:         cr.Path,
		Slogan:       cr.Slogan,
		Abstract:     cr.Abstract,
		AbstractTime: cr.AbstractTime,
		BannerTime:   cr.BannerTime,
		Sort:         cr.Sort,
	}

	err = global.DB.Create(&menuModel).Error

	if err != nil {
		global.Log.Error(err)
		res.FailWithMessage("菜单添加失败", c)
		return
	}

	// 该菜单没有图片
	if len(cr.ImageSortList) == 0 {
		res.OKWithMessage("菜单添加成功", c)
		return
	}

	// 菜单图片表入库
	var menuBannerList []models.MenuBannerModel
	for _, sort := range cr.ImageSortList {
		// 这里也得判断image_id是否真的有这张图片
		menuBannerList = append(menuBannerList, models.MenuBannerModel{
			MenuID:   menuModel.ID,
			BannerID: sort.ImageID,
			Sort:     sort.Sort,
		})
	}

	// 给第三张表入库
	err = global.DB.Create(&menuBannerList).Error
	if err != nil {
		global.Log.Error(err)
		res.FailWithMessage("菜单图片关联失败", c)
		return
	}

	res.OKWithMessage("菜单添加成功", c)
}
```

添加的参数：
```json
{
	"menu_title": "首页",
	"menu_title_en": "index",
	"slogan": "糕小芝",
	"abstract": ["hhhhhhhhhhhh", "gin-vue-blog"],
	"abstract_time": 7,
	"banner_time": 7,
	"sort": 1,
	"image_sort_list": [
	    {"image_id": 13, "sort": 0},
	    {"image_id": 14, "sort": 1}
	]
}
```

其中部分数据不是必须填的，因为页面有很多不同的页面，需求也不一样，所以一些页面的信息不是必写的

## 菜单列表

### 菜单表页面部分
**自定义连接表，如何排序** 

```go
// api/menu_api/menu_list.go
package menu_api

import (
	"github.com/gin-gonic/gin"
	"gvb_server/global"
	"gvb_server/models"
	"gvb_server/models/res"
)

type Banner struct {
	ID   uint   `json:"id"`
	Path string `json:"path"`
}

type MenuResponse struct {
	models.MenuModel
	Banners []Banner `json:"banners"`
}

func (MenuApi) MenuListView(c *gin.Context) {
	// 先查菜单，得到菜单列表，同时查到所有菜单的ID列表
	var menuList []models.MenuModel
	var menuIDList []uint
	// sort desc 从大到小排
	global.DB.Order("sort desc").Find(&menuList).Select("id").Scan(&menuIDList)

	// 查连接表,找到对应menu_id关联的menu_banner
	var menuBanners []models.MenuBannerModel
	// .Preload("BannerModel")联表查询
	global.DB.Preload("BannerModel").Order("sort desc").Find(&menuBanners, "menu_id in ?", menuIDList)

	var menus []MenuResponse
	for _, model := range menuList {
		// model 就是一个菜单, 查找当前菜单对应的图片列表
		var banners []Banner
		for _, banner := range menuBanners {
			if banner.MenuID == model.ID {
				banners = append(banners, Banner{
					ID:   banner.BannerID,
					Path: banner.BannerModel.Path, // 联表查询吼吼吼
				})
			}
		}
		menus = append(menus, MenuResponse{
			MenuModel: model,
			Banners:   banners,
		})
	}

	res.OKWithData(menus, c)
	return
}

```


### 菜单的名称列表

```go
// api/menu_api/menu_name_list.go
package menu_api

import (
	"github.com/gin-gonic/gin"
	"gvb_server/global"
	"gvb_server/models"
	"gvb_server/models/res"
)

type MenuNameResponse struct {
	ID    uint   `json:"id"`
	Title string `json:"title"`
	Path  string `json:"path"`
}

func (MenuApi) MenuNameListView(c *gin.Context) {
	var menuNameList []MenuNameResponse
	global.DB.Model(&models.MenuModel{}).Select("id,title,path").Find(&menuNameList)
	res.OKWithData(menuNameList, c)
}

```

## 菜单更新

第三张表的更新
这里也是**转化成 map 然后更新，像广告编辑一样，用到了 structs 库**

对应菜单创建的 Request 的修改部分
转化成 map 就可以有一些地方不更新, 即实现**选择更新**，配合 **Updates** 使用
[更新 | GORM 中的更新多列部分](https://gorm.io/zh_CN/docs/update.html)   
但是要在 Requet 中标记
**structs: "-"表示不会转化成 map 格式**

```go
// api/menu_api/menu_create.go
type MenuRequest struct {
	Title         string      `json:"title" binding:"required" msg:"请完善菜单名称" structs:"title"`
	Path          string      `json:"path" binding:"required" msg:"请完善菜单路径" structs:"path"`
	Slogan        string      `json:"slogan" structs:"slogan"`
	Abstract      ctype.Array `json:"abstract" structs:"abstract"`
	AbstractTime  int         `json:"abstract_time" structs:""`                                    // 切换时间，单位秒
	BannerTime    int         `json:"banner_time" structs:"abstract_time"`                         // 切换时间，单位秒
	Sort          int         `json:"sort" binding:"required" msg:"请输入菜单序号" structs:"banner_time"` // 菜单序号
	ImageSortList []ImageSort `json:"image_sort_list" structs:"-"`                                 // 具体图片的顺序
}
```

代码：
```go
// api/menu_api/menu_update.go
package menu_api

import (
	"github.com/fatih/structs"
	"github.com/gin-gonic/gin"
	"gvb_server/global"
	"gvb_server/models"
	"gvb_server/models/res"
)

func (MenuApi) MenuUpdateView(c *gin.Context) {
	var cr MenuRequest
	err := c.ShouldBindJSON(&cr)
	if err != nil {
		res.FailWithError(err, &cr, c)
		return
	}
	id := c.Param("id")

	// 先把之前的banner清空
	var menuModel models.MenuModel
	err = global.DB.Take(&menuModel, id).Error
	if err != nil {
		res.FailWithMessage("菜单不存在", c)
		return
	}
	// 联表，把第三张表清空
	global.DB.Model(&menuModel).Association("Banners").Clear()

	// 如果选择了banner,那就添加
	if len(cr.ImageSortList) > 0 {
		// 操作第三张表
		var menuBannerList []models.MenuBannerModel
		for _, imageSort := range cr.ImageSortList {
			menuBannerList = append(menuBannerList, models.MenuBannerModel{
				MenuID:   menuModel.ID,
				BannerID: imageSort.ImageID,
				Sort:     imageSort.Sort,
			})
		}
		err = global.DB.Create(&menuBannerList).Error
		if err != nil {
			res.FailWithMessage("创建菜单图片失败", c)
			return
		}
	}

	// 普通更新，
	// 因为要防范零止问题，所以转化成map来更新数据
	// 结构体转map的第三方包structs
	maps := structs.Map(&cr)
	// Updates 好用嗷嗷嗷嗷嗷嗷嗷嗷嗷嗷
	err = global.DB.Model(&menuModel).Updates(maps).Error

	if err != nil {
		global.Log.Error(err)
		res.FailWithMessage("修改菜单失败", c)
		return
	}

	res.OKWithMessage("修改菜单成功", c)
}
```

### 解决切片 null 值问题

如果有列表查询的接口, 如果列表为空，前端显示的是 null, 如何解决?
**切片是引用数据类型**
```go
// api/menu_api/menu_list.go
//var banners []Banner // 引用数据类型，如果声明了但是没有赋值等价于nil,前端得到的数据会显示null  
// 修改后的代码:  
banners := []Banner{} // 这个写法有波浪线，不太好
// 或者:
var banners = make([]Banner, 0) // 分配一个长度为0的地址
```

## 菜单删除

级联删除

**事务删除**：如果删掉了菜单中的图片，但是菜单删除失败，那么返回给前端就有问题，所以加了事务，就必须都删除才成功，把两个删除事件合并为一个事务

```go
package menu_api

import (
	"fmt"
	"github.com/gin-gonic/gin"
	"gorm.io/gorm"
	"gvb_server/global"
	"gvb_server/models"
	"gvb_server/models/res"
)

func (MenuApi) MenuRemoveView(c *gin.Context) {
	var cr models.RemoveRequest // 要删除的广告的id列表
	err := c.ShouldBindJSON(&cr)
	if err != nil {
		res.FailWitheCode(res.ArgumentError, c)
		return
	}

	// 先从数据库中查询对应的信息
	var menutList []models.MenuModel
	count := global.DB.Find(&menutList, cr.IDList).RowsAffected
	if count == 0 {
		res.OKWithMessage("菜单不存在", c)
		return
	}

	// 查到啦，就要批量删除喽
	// 要先删除关联的第三张表，然后删除菜单表----因为要删除两个东西，所以用事务
	err = global.DB.Transaction(func(tx *gorm.DB) error {
		// 删除关联的第三张表
		err = global.DB.Model(&menutList).Association("Banners").Clear()
		if err != nil {
			global.Log.Error(err)
			return err
		}

		// 删除菜单表
		global.DB.Delete(&menutList)
		if err != nil {
			global.Log.Error(err)
			return err
		}

		// 都成功
		return nil
	})

	if err != nil {
		global.Log.Error(err)
		res.FailWithMessage("删除菜单失败", c)
		return
	}
	res.OKWithMessage(fmt.Sprintf("共删除 %d 个菜单", count), c)

}
```

## 菜单详情

```go
// api/menu_api/menu_detail.go
package menu_api

import (
	"github.com/gin-gonic/gin"
	"gvb_server/global"
	"gvb_server/models"
	"gvb_server/models/res"
)

// MenuDetailView 查单个菜单的细节嗷
func (MenuApi) MenuDetailView(c *gin.Context) {
	// 根据id查菜单
	id := c.Param("id")
	var menuModel models.MenuModel
	err := global.DB.Take(&menuModel, id).Error
	if err != nil {
		res.FailWithMessage("菜单不存在", c)
		return
	}

	// 查连接表
	var menuBannerList []models.MenuBannerModel
	global.DB.Preload("BannerModel").Order("sort desc").Find(&menuBannerList, "menu_id = ?", id)

	// 查对应菜单的图片表
	var bannerList = make([]Banner, 0)
	for _, banner := range menuBannerList {
		bannerList = append(bannerList, Banner{
			ID:   banner.BannerID,
			Path: banner.BannerModel.Path,
		})
	}
	menuResponse := MenuResponse{
		MenuModel: menuModel,
		Banners:   bannerList,
	}
	res.OKWithData(menuResponse, c)
	return
}

```

# 用户管理

## 创建用户 

### 命令行创建用户

设置命令：
```go
// flag/enter.go
package flag

import (
	sys_flag "flag"
)

type Option struct {
	DB   bool
	User string
	// -u admin (创建一个叫admin的管理员)
	// -u user (创建一个叫user的用户)
}

// Parse 解析命令行参数
func Parse() Option {
	// 定义了一个名为 “db” 的命令行标志。这个标志的默认值是 false，并且它的描述是 “初始化数据库”。
	db := sys_flag.Bool("db", false, "初始化数据库")
	user := sys_flag.String("u", "", "创建用户")
	// 解析命令行参数写入注册的flag里
	sys_flag.Parse()
	return Option{
		// DB 字段被设置为 “db” 标志的值（通过 *db 解引用得到）。
		DB:   *db, // false
		User: *user,
	}
	// 如果你在命令行中使用 -db=true 运行你的程序，那么这个 Parse 函数将会返回 { DB: true }。
	// 如果你没有指定 -db 标志，那么它将返回 { DB: false }，因为 false 是 “db” 标志的默认值。
}

// IsWebStop 是否停止web项目
func IsWebStop(option Option) bool {
	if option.DB {
		return true
	}
	return true // 默认返回这个,因为运行完命令之后都是需要停止web项目的，所以这里可以改成true
}

// SwitchOption 根据命令执行不同的函数
func SwitchOption(option Option) {
	if option.DB {
		Makemigrations()
		return
	}

	// 创建用户调用(管理员或普通用户)
	// 因为如果option.User为""，那么就不会走这里，必须是个字符串
	if option.User == "admin" || option.User == "user" {
		CreateUser(option.User)
		return
	}

	// 不符合上面的任意一种情况，那就显示，类似于提示帮助
	sys_flag.Usage()
}
```

使用**命令行输入的方式**创建用户

```go
// flag/create_user.go
package flag

import (
	"fmt"
	"gvb_server/global"
	"gvb_server/models"
	"gvb_server/models/ctype"
	"gvb_server/utils/pwd"
)

// CreateUser 创建用户
// permissins 权限
func CreateUser(permissions string) {
	// 创建用户的逻辑
	// 用户名 昵称 密码 确认密码 邮箱
	var (
		userName   string
		nickName   string
		password   string
		rePassword string
		email      string
	)
	fmt.Printf("请输入用户名")
	fmt.Scan(&userName)
	fmt.Printf("请输入昵称")
	fmt.Scan(&nickName)
	fmt.Printf("请输入密码")
	fmt.Scan(&password)
	fmt.Printf("请再次输入密码")
	fmt.Scan(&rePassword)
	fmt.Printf("请输入邮箱")
	fmt.Scan(&email) // 不用必须输入;因为Scanln问题，所以还是改成必填的Scan

	// 判断用户名是否存在
	var userModel models.UserModel
	err := global.DB.Find(&userModel, "user_name = ?", userName).Error
	if err == nil {
		// 存在
		global.Log.Error("用户名已存在，请重新输入")
		return
	}

	// 校验两次密码
	if password != rePassword {
		global.Log.Error("两次密码不一致，请重新输入")
		return
	}

	// 对密码进行加密（hash）
	hashPwd := pwd.HashPwd(password)
	// 还可以用正则判断密码的强度，还可以用洛必达判断是否属于弱密码

	// 头像问题
	// 1. 默认头像
	// 2. 随机选择头像
	advatar := "/upload/avatars/default.png"

	// 入库
	role := ctype.PermissionUser // 默认是普通用户
	if permissions == "admin" {
		role = ctype.PermissionAdmin
	}

	err = global.DB.Create(&models.UserModel{
		UserName:   userName,
		NickName:   nickName,
		Password:   hashPwd,
		Email:      email,
		Role:       role,
		Avatar:     advatar,
		IP:         "127.0.0.1",
		Addr:       "内网地址",
		SignStatus: ctype.SignEmail,
	}).Error
	if err != nil {
		global.Log.Error(err)
		return
	}
	global.Log.Infof("用户 %s 创建成功", userName)
}

```

使用的命令:
```shell
go run main.go -u user # 创建普通用户
go run main.go -u admin # 创建管理员
```
**这些都是直接创建到数据库里面** 

### 密码 hash

详情：
[go 密码 hash 加密 - 牛奔 - 博客园 (cnblogs.com)](https://www.cnblogs.com/niuben/p/13224221.html) 

### 遗留问题

对于一些不想传的参数，怎么处理
fmt. ScanIn 除了在**第一个**，后面的都会马上赋值

测试代码：
```go
// testdata/3.scan测试.go
package main

import (
	"fmt"
)

func main() {
	var (
		userName string
		email    string
	)
	fmt.Printf("请输入用户名")
	fmt.Scan(&userName)
	fmt.Printf("请输入邮箱")
	fmt.Scanln(&email) // 不用必须输入

	fmt.Println(userName, email)
}
```

## 用户登录

### jwt

- json web token
- 前后端分离项目，**用 jwt 保存登录信息** 
- 它并不是一个具体的技术实现，而更像是─种**标准**。
- JWT 规定了数据传输的结构，一串完整的 JWT 由三段落组成，每个段落用英文句号连接 (.)连接，他们分别是: Header、Payload、Signature (密钥)，所以，常规的 JWT 内容格式是这样的: **AAA. BBB.CCC**
- 并且，这一串内容会 base64 加密; 也就是说 base64 解码就可以看到实际传输的内容。接下来解释一下这些内容都有什么作用。 （即，不要在 jwt 中存敏感信息）

#### 安装
```shell
go get github.com/dgrijalva/jwt-go/v4
```

#### 代码
```go
// utils/jwts/enter.go
package jwts

import (
	"github.com/dgrijalva/jwt-go/v4"
)

type JwtPayLoad struct {
	Username string `json:"username"`  // 用户名
	NickName string `json:"nick_name"` // 昵称
	Role     int    `json:"role"`      // 权限 1.管理员 2.普通用户 3.游客
	UserID   uint   `json:"user_id"`   // 用户id
}

var MySecret []byte

type CustomClaims struct {
	JwtPayLoad
	jwt.StandardClaims
}
```

```go
// utils/jwts/gen_token.go
package jwts

import (
	"github.com/dgrijalva/jwt-go/v4"
	"gvb_server/global"
	"time"
)

// GenToken 创建Token
func GenToken(user JwtPayLoad) (string, error) {
	claim := CustomClaims{
		user,
		jwt.StandardClaims{
			ExpiresAt: jwt.At(time.Now().Add(time.Hour * time.Duration(global.Config.Jwy.Expires))), // 默认两个小时更新
			Issuer:    global.Config.Jwy.Issuer,                                                     // 签发人
		},
	}
	token := jwt.NewWithClaims(jwt.SigningMethodHS256, claim)

	MySecret = []byte(global.Config.Jwy.Secret)
	return token.SignedString(MySecret)
}
```

```go
// utils/jwts/parse_token.go
package jwts

import (
	"errors"
	"fmt"
	"github.com/dgrijalva/jwt-go/v4"
	"gvb_server/global"
)

// ParseToken 解析token
func ParseToken(tokenStr string) (*CustomClaims, error) {
	MySecret = []byte(global.Config.Jwy.Secret)
	token, err := jwt.ParseWithClaims(tokenStr, &CustomClaims{}, func(token *jwt.Token) (interface{}, error) {
		return MySecret, nil
	})
	if err != nil {
		global.Log.Error(fmt.Sprintf("token parse error: %s", err.Error()))
		return nil, err
	}

	if claims, ok := token.Claims.(*CustomClaims); ok && token.Valid {
		return claims, nil
	}
	return nil, errors.New("invalid token") // 错误的token
}
```
#### 测试函数

测试函数中因为要测试的函数，**需要读配置文件**，但是如果没读，就会引起如下报错:
![[Pasted image 20231009105917.png]]

代码：
```go
// testdata/4.jwt测试.go
package main

import (
	"fmt"
	"gvb_server/core"
	"gvb_server/global"
	"gvb_server/utils/jwts"
)

func main() {
	// 读取配置文件
	core.InitConf()
	// 初始化日志
	global.Log = core.InitLogger()

	token, err := jwts.GenToken(jwts.JwtPayLoad{
		Username: "admin",
		NickName: "admin",
		Role:     1,
		UserID:   1,
	})
	fmt.Println(token, err)
	
	// 两个小时过期一次 
	claims, err := jwts.ParseToken("eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VybmFtZSI6ImFkbWluIiwibmlja19uYW1lIjoiYWRtaW4iLCJyb2xlIjoxLCJ1c2VyX2lkIjoxLCJleHAiOjE2OTY4Mzk5NTkuNzA4ODI1fQ.ykRjKIrNOSv8_MN_Uvz25FXIaQ9DR0BNbBKVQGTG_kc")
	fmt.Println(claims, err)
}
```

### 邮箱登录

**前面命令行创建的用户**：
```go
aaa 123
gxz 1234
xz 111
```

- 登录是通过**用户名**登录，不是通过用户昵称，后面修改用户权限的时候可以修改用户昵称，这样不会影响用户登录
登录代码：
```go
// api/user_api/email_login.go
package user_api

import (
	"github.com/gin-gonic/gin"
	"gvb_server/global"
	"gvb_server/models"
	"gvb_server/models/res"
	"gvb_server/utils/jwts"
	"gvb_server/utils/pwd"
)

type EmailLoginRequest struct {
	UserName string `json:"user_name" binding:"required" msg:"请输入用户名"`
	Password string `json:"password" binding:"required" msg:"请输入密码"`
}

func (UserApi) EmailLoginView(c *gin.Context) {
	// 参数绑定
	var cr EmailLoginRequest
	err := c.ShouldBindJSON(&cr)
	if err != nil {
		res.FailWithError(err, &cr, c)
		return
	}

	// 验证用户是否存在
	var userModel models.UserModel
	err = global.DB.Take(&userModel, "user_name = ? or email = ?", cr.UserName, cr.UserName).Error
	if err != nil {
		// 没找到
		global.Log.Warn("用户名不存在")
		res.FailWithMessage("用户名或密码错误", c)
		return
	}

	// 校验密码
	isCheck := pwd.CheckPwd(userModel.Password, cr.Password)
	if !isCheck {
		global.Log.Warn("用户密码错误")
		res.FailWithMessage("用户名或密码错误", c)
		return
	}
	// 登陆成功，生成token
	token, err := jwts.GenToken(jwts.JwtPayLoad{
		NickName: userModel.NickName,
		Role:     int(userModel.Role),
		UserID:   userModel.ID,
	})

	if err != nil {
		global.Log.Error(err)
		res.FailWithMessage("token生成失败", c)
		return
	}

	res.OKWithData(token, c)
}
```

### 对于 web 项目是否停止的优化

优化的是创建用户处命令行参数绑定中出现的问题 --- **flag** 

- 把结构体转成 map --- 因为结构体不能循环，但是 map 可以循环
- 同时方便判断是否有-db 或-u 的参数绑定，如果有那么一定需要停止 web 项目
- 但是如果没有也就是只有 go run main. go 那就正常执行，不用停止 web 项目

修改后的代码：
```go
// flag/enter.go
package flag

import (
	sys_flag "flag"
	"github.com/fatih/structs"
)

type Option struct {
	DB   bool
	User string
	// -u admin (创建一个叫admin的管理员)
	// -u user (创建一个叫user的用户)
}

// Parse 解析命令行参数
func Parse() Option {
	// 定义了一个名为 “db” 的命令行标志。这个标志的默认值是 false，并且它的描述是 “初始化数据库”。
	db := sys_flag.Bool("db", false, "初始化数据库")
	user := sys_flag.String("u", "", "创建用户")
	// 解析命令行参数写入注册的flag里
	sys_flag.Parse()
	return Option{
		// DB 字段被设置为 “db” 标志的值（通过 *db 解引用得到）。
		DB:   *db, // false
		User: *user,
	}
	// 如果你在命令行中使用 -db=true 运行你的程序，那么这个 Parse 函数将会返回 { DB: true }。
	// 如果你没有指定 -db 标志，那么它将返回 { DB: false }，因为 false 是 “db” 标志的默认值。
}

// IsWebStop 是否停止web项目
func IsWebStop(option Option) (f bool) {
	maps := structs.Map(&option) // 结构体转map
	// map可以循环嗷，厉害
	for _, v := range maps {
		switch val := v.(type) { // 因为v是interface，所以不能直接判断，需要转化成实际类型之后分情况判断
		case string:
			if val != "" {
				f = true
			}
		case bool:
			if val == true {
				f = true
			}
		}
	}
	// 上面的switch中只要有一个是true，那就停止web项目
	return f
}

// SwitchOption 根据命令执行不同的函数
func SwitchOption(option Option) {
	if option.DB {
		Makemigrations()
		return
	}

	// 创建用户调用(管理员或普通用户)
	// 因为如果option.User为""，那么就不会走这里，必须是个字符串
	if option.User == "admin" || option.User == "user" {
		CreateUser(option.User)
		return
	}

	// 不符合上面的任意一种情况，那就显示，类似于提示帮助
	//sys_flag.Usage()
}
```

### 用户列表页

- 获取检验 token
- 获取用户列表页分页
- 要区分游客和管理员的界面
- 密码不能展示，加 json: "-"就可以
- 手机号和邮箱需要脱敏
- user_name 要么用星号，要么就不显示，管理员可以看到这个

```go
// api/user_api/user_list.go
package user_api

import (
	"github.com/gin-gonic/gin"
	"gvb_server/models"
	"gvb_server/models/ctype"
	"gvb_server/models/res"
	"gvb_server/service/common"
	"gvb_server/utils/desens"
	"gvb_server/utils/jwts"
)

// UserListView 用户列表页，也要分页
func (UserApi) UserListView(c *gin.Context) {
	// 如何判断是管理员
	token := c.Request.Header.Get("token")
	if token == "" {
		res.FailWithMessage("未携带token", c)
		return
	}
	claims, err := jwts.ParseToken(token)
	if err != nil {
		res.FailWithMessage("token错误", c)
		return
	}

	var page models.PageInfo
	if err := c.ShouldBindQuery(&page); err != nil {
		res.FailWitheCode(res.ArgumentError, c) // 参数错误
		return
	}
	// 分页参数
	// 要区分游客和管理员的界面
	// 密码不能展示，加json:"-"就可以
	// 手机号和邮箱需要脱敏
	// user_name要么用星号，要么就不显示，管理员可以看到这个
	list, count, _ := common.ComList(models.UserModel{}, common.Option{
		PageInfo: page,
		Debug:    true,
	})

	var users []models.UserModel
	for _, user := range list {
		if ctype.Role(claims.Role) != ctype.PermissionAdmin {
			// 非管理员
			user.UserName = ""
		}
		// 脱敏
		user.Tel = desens.DesensitizationTel(user.Tel)
		user.Email = desens.DesensitizationEmail(user.Email)
		users = append(users, user)
	}

	res.OKWithList(users, count, c)
}
```

#### 电话邮箱脱敏

代码：
```go
// utils/desens/enter.go
package desens

import "strings"

// DesensitizationEmail 邮箱脱敏
func DesensitizationEmail(email string) string {
	// 2933756974@qq.com  2****@qq.com
	// yubiniom@yaho.com y****@yaho.com
	eList := strings.Split(email, "@")
	if len(eList) != 2 {
		return ""
	}
	return eList[0][:1] + "****@" + eList[1]
}

// DesensitizationTel 手机号脱敏
func DesensitizationTel(tel string) string {
	// 150 7162 1639
	// 150 **** 1639
	if len(tel) != 11 {
		return ""
	}

	return tel[:3] + "****" + tel[7:] // 每一数的后面切分别在第三个和第七个数字后面切
}
```


## 用户认证中间件

### 登录才行

- 判断是否登录，判断是否是管理员
- 必须登录才能看：用户信息，发布文章等

```go
// middleware/jwt_auth.go
package middleware

import (
	"github.com/gin-gonic/gin"
	"gvb_server/models/res"
	"gvb_server/utils/jwts"
)

// 用户登录的中间件
func JwtAuth() gin.HandlerFunc {
	return func(c *gin.Context) {
		// 如何判断是管理员
		token := c.Request.Header.Get("token")
		if token == "" {
			res.FailWithMessage("未携带token", c)
			c.Abort() // 拦截
			return
		}
		claims, err := jwts.ParseToken(token)
		if err != nil {
			res.FailWithMessage("token错误", c)
			c.Abort() // 拦截
			return
		}
		// 登录的用户
		c.Set("claims", claims) // 给需要使用中间件的函数取claims
	}
}
```

在路由处使用中间件：
```go
// routers/user_router.go
package routers

import (
	"github.com/gin-gonic/gin"
	"gvb_server/api"
	"gvb_server/middleware"
)

func UserRouter(router *gin.RouterGroup) {
	app := api.ApiGroupApp.UserApi
	router.POST("email_login", app.EmailLoginView)
	router.GET("users", middleware.JwtAuth(), app.UserListView)
}
```

同时 app. UserListView 也对应修改
```go
// api/user_api/user_list.go
package user_api

import (
	"github.com/gin-gonic/gin"
	"gvb_server/models"
	"gvb_server/models/ctype"
	"gvb_server/models/res"
	"gvb_server/service/common"
	"gvb_server/utils/desens"
	"gvb_server/utils/jwts"
)

// UserListView 用户列表页，也要分页
func (UserApi) UserListView(c *gin.Context) {
	_claims, _ := c.Get("claims")
	// 断言 (注意_claims的类型)
	claims := _claims.(*jwts.CustomClaims)

	var page models.PageInfo
	if err := c.ShouldBindQuery(&page); err != nil {
		res.FailWitheCode(res.ArgumentError, c) // 参数错误
		return
	}
	// 分页参数
	// 要区分游客和管理员的界面
	// 密码不能展示，加json:"-"就可以
	// 手机号和邮箱需要脱敏
	// user_name要么用星号，要么就不显示，管理员可以看到这个
	list, count, _ := common.ComList(models.UserModel{}, common.Option{
		PageInfo: page,
		Debug:    true,
	})

	var users []models.UserModel
	for _, user := range list {
		if ctype.Role(claims.Role) != ctype.PermissionAdmin {
			// 非管理员
			user.UserName = ""
		}
		// 脱敏
		user.Tel = desens.DesensitizationTel(user.Tel)
		user.Email = desens.DesensitizationEmail(user.Email)
		users = append(users, user)
	}

	res.OKWithList(users, count, c)
}
```

### 管理员才行

```go
// middleware/jwt_auth.go
// JwtAdmin 管理员才有权限的中间件
func JwtAdmin() gin.HandlerFunc {
	return func(c *gin.Context) {
		// 如何判断是管理员
		token := c.Request.Header.Get("token")
		if token == "" {
			res.FailWithMessage("未携带token", c)
			c.Abort() // 拦截
			return
		}
		claims, err := jwts.ParseToken(token)
		if err != nil {
			res.FailWithMessage("token错误", c)
			c.Abort() // 拦截
			return
		}
		// 登录的用户
		if claims.Role != int(ctype.PermissionAdmin) {
			res.FailWithMessage("权限不足", c)
			c.Abort() // 拦截
			return
		}
		c.Set("claims", claims) // 给需要使用中间件的函数取claims
	}
}
```

## 用户修改信息

### 修改权限和昵称

管理员可修改用户的权限和昵称
修改昵称主要是为了防止用户昵称非法，管理员有能力去修改

```go
// api/user_api/user_update_role.go
package user_api

import (
	"github.com/gin-gonic/gin"
	"gvb_server/global"
	"gvb_server/models"
	"gvb_server/models/ctype"
	"gvb_server/models/res"
)

type UserRole struct {
	Role     ctype.Role `json:"role" binding:"required,oneof=1 2 3 4" msg:"权限参数错误"`
	NickName string     `json:"nick_name"` // 防止用户昵称非法,管理员有能力修改
	UserID   uint       `json:"user_id" binding:"required" msg:"用户id错误"`
}

// UserUpdateRoleView 用户权限变更,同时可以修改昵称
func (UserApi) UserUpdateRoleView(c *gin.Context) {
	var cr UserRole
	if err := c.ShouldBindJSON(&cr); err != nil {
		res.FailWithError(err, &cr, c)
		return
	}
	var user models.UserModel
	err := global.DB.Take(&user, cr.UserID).Error
	if err != nil {
		res.FailWithMessage("用户id错误，用户不存在", c)
		return
	}
	err = global.DB.Model(&user).Updates(map[string]any{
		"role":      cr.Role,
		"nick_name": cr.NickName,
	}).Error // 更新权限
	if err != nil {
		global.Log.Error(err)
		res.FailWithMessage("修改权限失败", c)
		return
	}
	res.OKWithMessage("修改权限成功", c)
}

```

### 修改密码

```go
// api/user_api/user_update_password.go
package user_api

import (
	"github.com/gin-gonic/gin"
	"gvb_server/global"
	"gvb_server/models"
	"gvb_server/models/res"
	"gvb_server/utils/jwts"
	"gvb_server/utils/pwd"
)

type UpdatePasswordRequest struct {
	OlePwd string `json:"ole_pwd"` // 旧密码
	Pwd    string `json:"pwd"`     // 新密码
}

// UserUpdatePasswordView 修改登录人id
func (UserApi) UserUpdatePasswordView(c *gin.Context) {
	// 获取token解析后的claims
	_claims, _ := c.Get("claims")
	claims := _claims.(*jwts.CustomClaims)
	// 参数绑定
	var cr UpdatePasswordRequest
	if err := c.ShouldBindJSON(&cr); err != nil {
		res.FailWithError(err, &cr, c)
		return
	}

	// 查找用户是否存在
	var user models.UserModel
	err := global.DB.Take(&user, claims.UserID).Error
	if err != nil {
		res.FailWithMessage("用户不存在", c)
		return
	}

	// 判断密码是否正确
	if !pwd.CheckPwd(user.Password, cr.OlePwd) {
		res.FailWithMessage("密码错误", c)
		return
	}
	// 旧密码正确就修改密码
	hashPwd := pwd.HashPwd(cr.Pwd)
	err = global.DB.Model(&user).Update("password", hashPwd).Error
	if err != nil {
		global.Log.Error(err)
		res.FailWithMessage("密码修改失败", c)
		return
	}
	res.OKWithMessage("密码修改成功", c)
	return

	// 需要注销原本的jwt，这个jwt不是服务端生成的
	// 注销逻辑：
	// 如果用户注销了jwt，那么下次用户再带着这个jwt的时候，就不能让他通过啦
}
```

## 用户退出登录

**用户注销**

- 使用 jwt，它相当于是**客户端生成**（第三方生成），并没有由服务端生成，所以无法手动使 jwt 失效

因为有这种情况：
- 用户 a 原密码 123 登录之后
- 修改密码为 1234
- 中间重新登录，肯定旧密码就是错误的，不能登录
- 但是用户 a 却可以查看用户列表，修改其他的用户权限

注销逻辑：  
- 如果用户注销了 jwt，那么下次用户再带着这个 jwt 的时候，就不能让他通过啦

解决方案：
- 用户注销之后，将用户当前的这个 token 存放在一个地方，并设置一个过期时间，过期时间就是 jwt 的过期时间
- 当用户再携带这个 token 的时候，从那个地方去获取一下，如果存在，就说明这个 token 已经是被注销了的，不能再使用了

**那么这个地方就是 redis，redis 中可以设置过期时间，过期之后就会自动删除**

### redis

#### 工具 phpstudy

安装：
[安装PHPStudy（小皮）V8.1最详细安装教程_小皮phpstudy_wleiloaf的博客-CSDN博客](https://blog.csdn.net/wleiloaf/article/details/121662832) 

使用 redis 教程：
[Phpstudy v8.0使用Redis_phpstudy redis-CSDN博客](https://blog.csdn.net/william_n/article/details/103347949) 

[2021-03-23 - 高性能 Redis 实战_2021 jedis 性能问题_宁小法的博客-CSDN博客](https://blog.csdn.net/william_n/article/details/115113177) 

#### redis 测试代码

下载导包：
```shell
go get github.com/go-redis/redis
```

```go
// testdata/5.redis.go
package main

import (
	"fmt"
	"github.com/go-redis/redis"
	"github.com/sirupsen/logrus"
	"golang.org/x/net/context"
	"time"
)

var rdb *redis.Client

func init() {
	rdb = redis.NewClient(&redis.Options{
		Addr:     "127.0.0.1:6379",
		Password: "123456", // no password set
		DB:       0,        // use default DB
		PoolSize: 100,      // 连接池大小
	})
	// 判断是否超时，是否连接成功
	_, cancel := context.WithTimeout(context.Background(), 500*time.Millisecond)
	defer cancel()
	_, err := rdb.Ping().Result()
	if err != nil {
		logrus.Error(err)
		return
	}
}

func main() {
	// 创建存字符串
	err := rdb.Set("xxx1", "value1", 10*time.Second).Err()
	fmt.Println(err)
	// 查询所有
	cmd := rdb.Keys("*")
	keys, err := cmd.Result()
	fmt.Println(keys, err)

}
```

#### redis 实际使用

配置代码：
```go
package config

import "fmt"

type Redis struct {
	IP       string `json:"ip" yaml:"ip"`
	Port     int    `json:"port" yaml:"port"`
	Password string `json:"password" yaml:"password"`
	PoolSize int    `json:"pool_size" yaml:"pool_size"`
}

func (r Redis) Addr() string {
	return fmt.Sprintf("%s:%d", r.IP, r.Port)
}
```

配置文件 settings. yaml 中对应添加

函数:
```go
// core/redis.go
package core

import (
	"github.com/go-redis/redis"
	"github.com/sirupsen/logrus"
	"golang.org/x/net/context"
	"gvb_server/global"
	"time"
)

func ConnectRedis() *redis.Client {
	return ConnectRedisDB(0)
}

// ConnectRedisDB 根据不同的DB连接不同的redis
func ConnectRedisDB(db int) *redis.Client {
	redisConf := global.Config.Redis
	rdb := redis.NewClient(&redis.Options{
		Addr:     redisConf.Addr(),
		Password: redisConf.Password, // no password set
		DB:       db,                 // use default DB
		PoolSize: redisConf.PoolSize, // 连接池大小
	})

	// 判断是否超时，是否连接成功
	_, cancel := context.WithTimeout(context.Background(), 500*time.Millisecond)
	defer cancel()
	_, err := rdb.Ping().Result()
	if err != nil {
		logrus.Error("redis连接失败 %s", redisConf.Addr())
		return nil
	}
	return rdb
}
```

主函数 main. go 中调用函数：
```go
// 连接redis  
global.Redis = core.ConnectRedis()
// 如果想要设置db，就调用
global.Redis = core.ConnectRedisDB(0) // 0-15
```


### 用户注销

#### 主代码
代码：
```go
// api/user_api/user_logout.go
package user_api

import (
	"github.com/gin-gonic/gin"
	"gvb_server/global"
	"gvb_server/models/res"
	"gvb_server/service"
	"gvb_server/utils/jwts"
)

func (UserApi) LogoutView(c *gin.Context) {
	_claims, _ := c.Get("claims")
	claims := _claims.(*jwts.CustomClaims)

	token := c.Request.Header.Get("token")

	err := service.ServiceApp.UserService.Logout(claims, token)

	if err != nil {
		global.Log.Error(err)
		res.FailWithMessage("注销失败", c)
		return
	}

	res.OKWithMessage("注销成功", c)
}
```

#### 用户注销的逻辑部分

代码：
```go
// service/user_ser/enter.go
package user_ser

import (
	"gvb_server/service/redis_ser"
	"gvb_server/utils/jwts"
	"time"
)

type UserService struct {
}

// Logout 注销操作（逻辑）:计算用户的token的过期时间，并将过期时间放入redis
func (UserService) Logout(claims *jwts.CustomClaims, token string) error {
	// 需要计算距离现在的过期时间(redis需要的过期时间是Duration类型)
	exp := claims.ExpiresAt // 最后的时间
	now := time.Now()       // 现在时间
	// 计算距离现在的过期时间
	diff := exp.Time.Sub(now) // 剩多少秒，单位是Duration

	// 操作redis
	return redis_ser.Logout(token, diff)
}
```
#### redis 操作

- 将注销用户的 token 加入到 redis 中，并设置过期时间，时间一到就会过期
- 同时在中间件代码部分加入判断当前用户的 token 是否在 redis 中，如果在就是已经注销了，那就不能查询或修改登录的或者管理员可以做的事情
- 上述操作写在 service 中

```go
// service/redis_ser/enter.go
package redis_ser

import (
	"gvb_server/global"
	"gvb_server/utils"
	"time"
)

// 注意这里的是普通的函数，没有type RedisService struct{}这个东西
// 这里的函数会被user_ser和jwt_auth调用嗷
// 前缀
const prefix = "logout_"

// Logout 针对注销的操作
func Logout(token string, diff time.Duration) error {
	err := global.Redis.Set(prefix+token, "", diff).Err()
	return err
}

// CheckLogout 检测是否注销就是判断是否在redis中
func CheckLogout(token string) bool {
	keys := global.Redis.Keys(prefix + "*").Val()
	if utils.Inlist(prefix+token, keys) {
		return true // 在里面
	}
	return false // 不在里面
}
```

- **redis_ser 跟其他写在 service 中的函数不同，它只是 service 函数**
- 也就是，它**不像**其他的一样有类似于 type RedisService struct{} 这样的划分类别的东西
- 这是因为，在 **service 中**的 user_ser 中的 enter **需要调用** redis_ser 中的 Logout，如果也像上面那样划分类别，就会包 circle

#### 中间件部分

- 加入了判断当前用户 token 是否在 redis 中

```go
// middleware/jwt_auth.go
package middleware

import (
	"github.com/gin-gonic/gin"
	"gvb_server/models/ctype"
	"gvb_server/models/res"
	"gvb_server/service/redis_ser"
	"gvb_server/utils/jwts"
)

// 用户登录的中间件
func JwtAuth() gin.HandlerFunc {
	return func(c *gin.Context) {
		// 如何判断是管理员
		token := c.Request.Header.Get("token")
		if token == "" {
			res.FailWithMessage("未携带token", c)
			c.Abort() // 拦截
			return
		}
		// 解析token
		claims, err := jwts.ParseToken(token)
		if err != nil {
			res.FailWithMessage("token错误", c)
			c.Abort() // 拦截
			return
		}

		// 判断是否再redis中，如果在redis中，就代表它已经注销过了
		if redis_ser.CheckLogout(token) {
			res.FailWithMessage("token已失效", c)
			c.Abort() // 拦截
			return
		}
		// 登录的用户
		c.Set("claims", claims) // 给需要使用中间件的函数取claims
	}
}

// JwtAdmin 管理员才有权限的中间件
func JwtAdmin() gin.HandlerFunc {
	return func(c *gin.Context) {
		// 如何判断是管理员
		token := c.Request.Header.Get("token")
		if token == "" {
			res.FailWithMessage("未携带token", c)
			c.Abort() // 拦截
			return
		}
		claims, err := jwts.ParseToken(token)
		if err != nil {
			res.FailWithMessage("token错误", c)
			c.Abort() // 拦截
			return
		}

		// 判断是否再redis中，如果在redis中，就代表它已经注销过了
		if redis_ser.CheckLogout(token) {
			res.FailWithMessage("token已失效", c)
			c.Abort() // 拦截
			return
		}

		// 登录的用户
		if claims.Role != int(ctype.PermissionAdmin) {
			res.FailWithMessage("权限不足", c)
			c.Abort() // 拦截
			return
		}

		c.Set("claims", claims) // 给需要使用中间件的函数取claims
	}
}
```


问题：
如果用户注销之后，它的 token 和 token 过期时间放在 redis 中，但是如果过期时间到了，也就会从 redis 中自动删除，那么如果用户注销之后，又等过期时间到了，继续查询或者修改用户信息怎么办？
- redis 中的过期时间是根据用户登录时 token 的过期时间计算的到的
- 如果过期时间到了，那个 token 也失效了，那么还是之前的 token 修改查询也是不行的。
- 如果过期时间没到，用户重新登录了的话，用户的那个 token 就会改变的，之后也跟 redis 中待过期的 token 无关，互不影响

## 用户删除

### 主代码

- 这个接口很少用到，与用户关联的数据很多，后续再完善

```go
// api/user_api/user_remove.go
package user_api  
  
import (  
"fmt"  
"github.com/gin-gonic/gin"  
"gorm.io/gorm"  
"gvb_server/global"  
"gvb_server/models"  
"gvb_server/models/res"  
)  
  
func (UserApi) UserRemoveView(c *gin.Context) {  
var cr models.RemoveRequest // 要删除的用户的id列表  
err := c.ShouldBindJSON(&cr)  
if err != nil {  
res.FailWitheCode(res.ArgumentError, c)  
return  
}  
  
// 先从数据库中查询对应的信息  
var userList []models.UserModel  
count := global.DB.Find(&userList, cr.IDList).RowsAffected  
if count == 0 {  
res.OKWithMessage("用户不存在", c)  
return  
}  
  
// 查到啦，就要批量删除喽  
err = global.DB.Transaction(func(tx *gorm.DB) error {  
// TODO:别除用户，消息表，评论表，用户收藏的文章，用户发布的文章  
// 删除用户  
global.DB.Delete(&userList)  
if err != nil {  
global.Log.Error(err)  
return err  
}  
// 都成功  
return nil  
})  
  
if err != nil {  
global.Log.Error(err)  
res.FailWithMessage("删除用户失败", c)  
return  
}  
res.OKWithMessage(fmt.Sprintf("共删除 %d 个用户", count), c)  
```

## 邮箱绑定

教程：
[66.邮箱操作_哔哩哔哩_bilibili](https://www.bilibili.com/video/BV1f24y1G72C/?p=66&spm_id_from=pageDriver&vd_source=5fd2c97b55c9ca08b4bb2e9f07fbea9a) 
### 测试 demo

下载依赖：
```shell
 go get gopkg.in/gomail.v2
```

QQ 邮箱：
- SMTP 服务器：smtp.qq.com 
- SMTP 端口号：465。必须填这个端口号，否则会报错。
- 身份认证用户名：填完整的邮箱名，如： 2933756974@qq.com  部分。
- 授权码: 发件人的密码, 就是设置-账户-授权码。

配置文件：
```shell
# settings.yaml
email:  
	host: smtp.qq.com  
	port: 465  
	user: 2933756974@qq.com # 发件人的邮箱  
	password: "glmkqxqlarpddefa" # 发件人的密码,就是设置-账户-授权码  
	default_form_email: ""  
	use_ssl: false  
	user_tls: false
```
### session

详情：
[github.com/gin-contrib/sessions教程-CSDN博客](https://blog.csdn.net/qq_16763983/article/details/105049118) 
session 原理：
![[Pasted image 20231012163221.png]]

#### 安装

下载依赖：
```shell
go get github.com/gin-contrib/sessions
```
#### 应用

- 类似于中间件一样的使用
- 这里 session 其实是一个中间件，那么我们同样可以自己写，**自己写**的话就**可以控制 session 的过期时间** 

本次总共有两次应用 :
- 第一次，绑定邮箱的时候，后台发验证码
- 第二次验证验证码
- 后台发验证码，验证码要和本次会话保持一致 --- session 
- 生成4位验证码(公共方法)，将生成的验证码存入 session，要不然用户下一次用的时候，就不知道 code 是发给谁啦

调用代码：

```go
// routers/user_router.go
package routers

import (
	"github.com/gin-contrib/sessions"
	"github.com/gin-contrib/sessions/cookie"
	"github.com/gin-gonic/gin"
	"gvb_server/api"
	"gvb_server/middleware"
)

var store = cookie.NewStore([]byte("secret")) // 这里可以自己在配置文件 settings.yaml 中设置？

func UserRouter(router *gin.RouterGroup) {
	app := api.ApiGroupApp.UserApi
	// 这里session其实是一个中间件，那么我们同样可以自己写，自己写的话就可以控制session的过期时间
	router.Use(sessions.Sessions("sessionid", store))
	router.POST("email_login", app.EmailLoginView)
	router.GET("users", middleware.JwtAuth(), app.UserListView)
	router.PUT("user_role", middleware.JwtAdmin(), app.UserUpdateRoleView)
	router.PUT("user_password", middleware.JwtAuth(), app.UserUpdatePasswordView)
	router.POST("logout", middleware.JwtAuth(), app.LogoutView)
	router.DELETE("users", middleware.JwtAdmin(), app.UserRemoveView)
	router.POST("user_bind_email", middleware.JwtAuth(), app.UserBindEmailView)
}
```


验证码生成代码：
```go
// utils/random/code.go
package random

import (
	"fmt"
	"math/rand"
	"time"
)

var stringCode = ""

func Code(length int) string {
	rand.Seed(time.Now().UnixNano()) // 种子纳秒
	return fmt.Sprintf("%4v", rand.Intn(10000))
}
```

验证码类型：
```go
// plugins/email/send_email.go
package email

import (
	"gopkg.in/gomail.v2"
	"gvb_server/global"
)

type Subject string

const (
	Code  Subject = "平台验证码"
	Note  Subject = "操作通知"
	Alarm Subject = "告警通知"
)

type Api struct {
	Subject Subject
}

func (a Api) Send(name, body string) error {
	return send(name, string(a.Subject), body)
}

func NewCode() Api {
	return Api{
		Subject: Code,
	}
}

func NewNote() Api {
	return Api{
		Subject: Note,
	}
}
func NewAlarm() Api {
	return Api{
		Subject: Alarm,
	}
}

// send 邮件发送， 发给谁，主题，正文
func send(name, subject, body string) error {
	e := global.Config.Email
	return sendMail(
		e.User,
		e.Password,
		e.Host,
		e.Port,
		name,
		e.DefaultFormEmail,
		subject,
		body,
	)
}

// sendMail 发送邮件
func sendMail(userName, authCode, host string, port int, mailTo, sendName string, subject, body string) error {
	m := gomail.NewMessage()
	m.SetHeader("From", m.FormatAddress(userName, sendName)) // 发件人邮箱，默认的发件人名字
	m.SetHeader("To", mailTo)                                // 发送给谁
	m.SetHeader("Subject", subject)                          // 主题
	m.SetBody("text/html", body)                             // 内容
	d := gomail.NewDialer(host, port, userName, authCode)    // authCode 密码
	err := d.DialAndSend(m)
	return err
}
```

使用代码：

- binding 中的参数之间不要有空格
```go
// api/user_api/user_bind_email.go
package user_api

import (
	"fmt"
	"github.com/gin-contrib/sessions"
	"github.com/gin-gonic/gin"
	"gvb_server/global"
	"gvb_server/models"
	"gvb_server/models/res"
	"gvb_server/plugins/email"
	"gvb_server/utils/jwts"
	"gvb_server/utils/pwd"
	"gvb_server/utils/random"
)

type BindEmailRequest struct {
	Email    string  `json:"email" binding:"required,email" msg:"邮箱非法"`
	Code     *string `json:"code"`     // 验证码(方便用户判断用户是否传验证码)
	Password string  `json:"password"` // 邮箱登录的密码是另外设置的
}

// UserBindEmailView 用户绑定邮箱
func (UserApi) UserBindEmailView(c *gin.Context) {
	// 获得解析token得到的东西
	_claims, _ := c.Get("claims")
	// 断言 (注意_claims的类型)
	claims := _claims.(*jwts.CustomClaims)
	// 第一次输入是 邮箱
	// 后台会给这个邮箱发验证码
	var cr BindEmailRequest
	err := c.ShouldBindJSON(&cr)
	if err != nil {
		res.FailWithError(err, &cr, c)
		return
	}
	// 初始化session
	session := sessions.Default(c)
	if cr.Code == nil {
		// 第一次，后台发验证码
		// 验证码要和本次会话保持一致 --- session
		// 生成4位验证码(公共方法)，将生成的验证码存入session，要不然用户下一次用的时候，就不知道code是发给谁啦
		code := random.Code(4)
		// 写入session，可以保持会话
		// 初始化，然后就可以写东西啦
		session.Set("valid_code", code)
		err = session.Save() // !!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!很重要，不能漏掉
		if err != nil {
			global.Log.Error(err)
			res.FailWithMessage("session错误", c)
			return
		}
		err = email.NewCode().Send(cr.Email, "你的验证码是 "+code)
		// 如果这里想处理快一些的话，就加一个go，即
		// go email.NewCode().Send(cr.Email, "你的验证码是 "+code)
		if err != nil {
			global.Log.Error(err)
			fmt.Println(err)
		}
		res.OKWithMessage("验证码已发送，请查收", c)
		return
	}
	// 第二次，用户输入邮箱，验证码，密码
	code := session.Get("valid_code")
	// 校验验证码
	if code != *cr.Code {
		res.FailWithMessage("验证码错误", c)
		return
	}
	// 查找当前用户是否存在
	var user models.UserModel
	err = global.DB.Take(&user, claims.UserID).Error
	if err != nil {
		res.FailWithMessage("用户不存在", c)
		return
	}
	if len(cr.Password) < 4 {
		res.FailWithMessage("密码强度太低", c)
		return
	}
	hashPwd := pwd.HashPwd(cr.Password)
	// 用户绑定邮箱(第一次的邮箱和第二次的邮箱也要做一致性校验)
	// 就是如果收验证码是一个邮箱，绑定验证码又是另一个邮箱，那就很危险，到时候要验证一下，这里后续完善
	err = global.DB.Model(&user).Updates(map[string]any{
		"email":    cr.Email,
		"password": hashPwd,
	}).Error
	if err != nil {
		global.Log.Error(err)
		res.FailWithMessage("绑定邮箱失败", c)
		return
	}
	// 完成绑定
	res.OKWithMessage("邮箱绑定成功", c)
	return
}
```

**遗留问题**：
- 用户绑定邮箱 (第一次的邮箱和第二次的邮箱也要做一致性校验)
- 就是如果收验证码是一个邮箱，绑定验证码又是另一个邮箱，那就很危险，到时候要验证一下，这里后续完善

- `var store = cookie.NewStore ([]byte ("secret"))` // 这里可以自己在配置文件 settings. yaml 中设置？
- 这里 session 其实是一个中间件，那么我们同样可以自己写，自己写的话就可以控制 session 的过期时间

## QQ 登录

### 申请注册

去 QQ 互联申请：
[QQ互联管理中心](https://connect.qq.com/manage.html#/) 

注册一个开发者账号，创建一个网站应用，主要需要三样东西
- APPID
- Secretkey, 
- RedirectUrl：回调地址

```url
// qq登录的跳转地址
https://graph.qq.com/oauth2.0/show?which=Login&display=pc&response_type=code&client_id=101974593&redirect_uri=http://blog.fengfengzhidao.com/login?flag=qd
```

qq 的跳转链接: 
```go
// config/conf_qq.go
package config

import "fmt"

type QQ struct {
	AppID    string `json:"app_id" yaml:"app_id"`
	Key      string `json:"key" yaml:"key"`
	Redirect string `json:"redirect" yaml:"redirect"` // 登录之后的回调地址
}

// GetPath qq的跳转链接
func (q QQ) GetPath() string {
	if q.Key == "" || q.AppID == "" || q.Redirect == "" {
		return ""
	}
	return fmt.Sprintf("https://graph.qq.com/oauth2.0/show?which=Login&display=pc&response_type=code&client_id=%s&redirect_uri=%s", q.AppID, q.Redirect)
}
```

qq 登录回调会有一个 code，我们如何验证？
- `/login? flag=qd` 是回调地址

![[Pasted image 20231012211623.png]]

图解：
![[Pasted image 20231012211855.png]]
- 用 go 做 http 的请求挺麻烦，没有 python 方便
- qq 登录相当于一个插件，所以逻辑代码放在 plugins 文件中


### 基础知识点补充

#### Values
```go
params := url.Values{} // 创建一个新的URL参数值  
// 这些行向URL参数添加了一些必要的字段  
// 如授权类型（grant_type）、客户端ID（client_id）、客户端密钥（client_secret）、授权码（code）和重定向URI（redirect_uri）。  
params.Add("grant_type", "authorization_code")  
params.Add("client_id", q.appID)  
params.Add("client_secret", q.appKey)  
params.Add("code", q.code)  
params.Add("redirect_uri", q.redirect)
```

详情：
[GO Values用法及代码示例 - 纯净天空 (vimsky.com)](https://vimsky.com/examples/usage/golang_net_url_Values.html) 

#### Parse
```go
// 解析URL字符串以创建一个新的URL对象。  
u, err := url.Parse("https://graph.qq.com/oauth2.0/token")  
if err != nil {  
	return err  
}
```

详情：
[GO URL.Parse用法及代码示例 - 纯净天空 (vimsky.com)](https://vimsky.com/examples/usage/golang_net_url_URL_Parse.html) 
内含很多其他用法

### 测试

测试请求接口：
```go
// 测试请求接口  
router.GET("/login", user_api.UserApi{}.QQLoginView)  
// 在浏览器访问：blog.fengfengzhidao.com/login?flag=qq&code=057F987F9A27DFOD6C1E210552667A87，就会被返回code
```

### 代码

QQ 登录逻辑：
```go
// plugins/qq/enter.go
package qq

import (
	"encoding/json"
	"fmt"
	"gvb_server/global"
	"io"
	"io/ioutil"
	"log"
	"net/http"
	"net/url"
	"strings"
)

// 用户信息
type QQInfo struct {
	Nickname string `json:"nickname"`     // 昵称
	Gender   string `json:"gender"`       // 性别
	Avatar   string `json:"figureurl_qq"` // 头像大图
	OpenID   string `json:"open_id"`
}

// QQ登录
type QQLogin struct {
	appID     string
	appKey    string
	redirect  string
	code      string
	accessTok string
	openID    string
}

// NewQQLogin 整个QQ登录的流程逻辑
func NewQQLogin(code string) (qqInfo QQInfo, err error) {
	qqLogin := &QQLogin{
		appID:    global.Config.QQ.AppID,
		appKey:   global.Config.QQ.Key,
		redirect: global.Config.QQ.Redirect,
		code:     code,
	}
	err = qqLogin.GetAccessToken()
	if err != nil {
		return qqInfo, err
	}
	err = qqLogin.GetOpenID()
	if err != nil {
		return qqInfo, err
	}
	qqInfo, err = qqLogin.GetUserInfo()
	if err != nil {
		return qqInfo, err
	}
	qqInfo.OpenID = qqLogin.openID
	return qqInfo, nil
}

// GetAccessToken 获取access_token
// 从QQ OAuth服务获取访问令牌的，这个访问令牌在以后可以用来访问和操作用户的账户数据。
// 并且QQLogin对象将包含新获取的访问令牌。
func (q *QQLogin) GetAccessToken() error {
	// 获取Access_token
	params := url.Values{} // 创建一个新的URL参数值
	// 这些行向URL参数添加了一些必要的字段
	// 如授权类型（grant_type）、客户端ID（client_id）、客户端密钥（client_secret）、授权码（code）和重定向URI（redirect_uri）。
	params.Add("grant_type", "authorization_code")
	params.Add("client_id", q.appID)
	params.Add("client_secret", q.appKey)
	params.Add("code", q.code)
	params.Add("redirect_uri", q.redirect)

	// 解析URL字符串以创建一个新的URL对象。
	u, err := url.Parse("https://graph.qq.com/oauth2.0/token")
	if err != nil {
		return err
	}
	// 将之前创建和填充的参数编码为URL查询字符串，并将其设置为URL对象的原始查询部分。
	u.RawQuery = params.Encode()
	// 执行HTTP GET请求到指定的URL，并获取响应。
	res, err := http.Get(u.String())
	if err != nil {
		return err
	}

	defer res.Body.Close()       // // 延迟关闭响应体，直到包含此行代码的函数返回
	qs, err := parseQS(res.Body) // 解析响应体以获取查询字符串。
	if err != nil {
		return err
	}
	// 从查询字符串中获取访问令牌，并将其设置为QQLogin对象的访问令牌。
	q.accessTok = qs[`access_token`][0]
	return nil
}

// GetOpenID 获取openid
// 从QQ OAuth服务获取开放ID的，这个开放ID在以后可以用来访问和操作用户的账户数据。
// 这个开放ID是用户在QQ OAuth服务中的唯一标识符。
func (q *QQLogin) GetOpenID() error {
	// 获取openid
	// 解析URL字符串以创建一个新的URL对象，其中包含访问令牌（Access Token）。
	u, err := url.Parse(fmt.Sprintf("https://graph.qq.com/oauth2.0/me?access_token=%s", q.accessTok))
	if err != nil {
		return err
	}

	res, err := http.Get(u.String()) // 执行HTTP GET请求到指定的URL，并获取响应。
	if err != nil {
		return err
	}
	defer res.Body.Close() // 延迟关闭响应体，直到包含此行代码的函数返回。

	openID, err := getOpenID(res.Body) // 解析响应体以获取开放ID。
	if err != nil {
		return err
	}
	q.openID = openID // 将获取到的开放ID设置为QQLogin对象的开放ID。
	return nil
}

// GetUserInfo 获取用户信息
func (q *QQLogin) GetUserInfo() (qqInfo QQInfo, err error) {
	params := url.Values{}
	params.Add("access_token", q.accessTok)
	params.Add("oauth_consumer_key", q.appID)
	params.Add("openid", q.openID)
	u, err := url.Parse("https://graph.qq.com/user/get_user_info")
	if err != nil {
		return qqInfo, err
	}
	u.RawQuery = params.Encode()

	res, err := http.Get(u.String())
	data, err := io.ReadAll(res.Body)
	err = json.Unmarshal(data, &qqInfo)
	if err != nil {
		return qqInfo, err
	}
	return qqInfo, nil
}

// parseQS 将HTTP响应的正文解析为键值对的形式
func parseQS(r io.Reader) (val map[string][]string, err error) {
	val, err = url.ParseQuery(readAll(r))
	if err != nil {
		return val, err
	}
	return val, nil
}

// getOpenID 从HTTP响应的正文中解析出openid
func getOpenID(r io.Reader) (string, error) {
	body := readAll(r)
	start := strings.Index(body, `"openid":"`) + len(`"openid":"`)
	if start == -1 {
		return "", fmt.Errorf("openid not found")
	}
	end := strings.Index(body[start:], `"`)
	if end == -1 {
		return "", fmt.Errorf("openid not found")
	}
	return body[start : start+end], nil
}

// readAll 读取所有数据并将其转换为字符串
func readAll(r io.Reader) string {
	b, err := ioutil.ReadAll(r)
	if err != nil {
		log.Fatal(err)
	}
	return string(b)
}
```

api 逻辑：
```go
// api/user_api/qq_login.go
package user_api

import (
	"fmt"
	"github.com/gin-gonic/gin"
	"gvb_server/global"
	"gvb_server/models"
	"gvb_server/models/ctype"
	"gvb_server/models/res"
	"gvb_server/plugins/qq"
	"gvb_server/utils/jwts"
	"gvb_server/utils/pwd"
	"gvb_server/utils/random"
)

// QQLoginView api逻辑
func (UserApi) QQLoginView(c *gin.Context) {
	// 先得到code
	code := c.Query("code")
	if code == "" {
		res.FailWithMessage("没有code", c)
		return
	}
	fmt.Println(code)
	// QQ登录 --- 拿到openID
	qqInfo, err := qq.NewQQLogin(code) // 这里只能调一次，只能请求一次，不然code是失效的也没用
	if err != nil {
		res.FailWithMessage(err.Error(), c)
		return
	}
	openID := qqInfo.OpenID
	// 根据openID判断用户是否存在
	var user models.UserModel
	// token 其他平台的唯一id
	err = global.DB.Take(&user, "token = ?", openID).Error
	if err != nil {
		// 不存在，就注册
		hashPwd := pwd.HashPwd(random.RandString(16))
		user = models.UserModel{
			NickName:   qqInfo.Nickname,
			UserName:   openID,        // qq登录，邮箱+密码，所以不用弄UserName登录，直接openid就行
			Password:   hashPwd,       // 随机生成16位密码
			Avatar:     qqInfo.Avatar, // 头像
			Addr:       "内网",          // 根据ip算地址，目前没有公网ip，所以之后再讲
			Token:      openID,
			IP:         c.ClientIP(),
			Role:       ctype.PermissionUser,
			SignStatus: ctype.SignQQ,
		}
		err = global.DB.Create(&user).Error
		if err != nil {
			global.Log.Error(err)
			res.FailWithMessage("注册失败", c)
			return
		}
		// 注册之后要把token返回给前端，前端才能做登录操作，才能做token的持久化，才能登录
	}

	// 登录操作
	// 登陆成功，生成token
	token, err := jwts.GenToken(jwts.JwtPayLoad{
		NickName: user.NickName,
		Role:     int(user.Role),
		UserID:   user.ID,
	})

	if err != nil {
		global.Log.Error(err)
		res.FailWithMessage("token生成失败", c)
		return
	}

	res.OKWithData(token, c)
}
```

路由：

```go
// routers/user_router.go
package routers

import (
	"github.com/gin-contrib/sessions"
	"github.com/gin-contrib/sessions/cookie"
	"github.com/gin-gonic/gin"
	"gvb_server/api"
	"gvb_server/middleware"
)

var store = cookie.NewStore([]byte("secret")) // // 这里可以自己在配置文件 settings.yaml 中设置?

func UserRouter(router *gin.RouterGroup) {
	app := api.ApiGroupApp.UserApi
	// 这里session其实是一个中间件，那么我们同样可以自己写，自己写的话就可以控制session的过期时间
	router.Use(sessions.Sessions("sessionid", store))
	router.POST("email_login", app.EmailLoginView)
	router.POST("/login", app.QQLoginView) // 这里就是实际用的接口
	router.GET("users", middleware.JwtAuth(), app.UserListView)
	router.PUT("user_role", middleware.JwtAdmin(), app.UserUpdateRoleView)
	router.PUT("user_password", middleware.JwtAuth(), app.UserUpdatePasswordView)
	router.POST("logout", middleware.JwtAuth(), app.LogoutView)
	router.DELETE("users", middleware.JwtAdmin(), app.UserRemoveView)
	router.POST("user_bind_email", middleware.JwtAuth(), app.UserBindEmailView)
}

```

### 补充

#### 随机生成密码 
[Golang生成随机字符串的八种方式与性能测试_Performance_张俭_InfoQ写作社区](https://xie.infoq.cn/article/f274571178f1bbe6ff8d974f3) 

```go
// utils/random/string.go
package random

import "math/rand"

var letters = []rune("abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ0123456789") // 从中选
// RandString 随机生成不同长度的字符串
func RandString(n int) string {
	b := make([]rune, n)
	for i := range b {
		b[i] = letters[rand.Intn(len(letters))]
	}
	return string(b)
}
```

#### switchHosts
- 可以方便的修改 hosts  
- 简单的说，hosts 文件是用于**本地 dns 服务**的，采用 **ip 域名的格式**写在一个**文本文件**当中，Hosts 是一个没有扩展名的系统文件，可以用**记事本**等工具打开
- 其作用就是**将一些常用的网址域名与其对应的 IP 地址建立一个关联“数据库”**，当用户在浏览器中输入一个**需要登录的网址**时，系统会首先**自动从 Hosts 文件**中寻找对应的 IP 地址，一旦找到，系统会立即打开对应网页，如果没有找到，则系统再会**将网址提交 DNS 域名解析服务器**进行 IP 地址的解析。
-  我们在开发**Web项目**过程中，一般会**部署有多套环境**，**网址域名都相同，部署在不同的服务器上**，有开发环境、测试环境、预发布环境、生产环境。经常要**切换Hosts来访问，测试以及验证bug**，如果纯手工修改这会花掉不少时间。

下载教程：
[SwitchHosts下载安装使用_swichhost-CSDN博客](https://blog.csdn.net/libusi001/article/details/108516673) 
使用教程：
[switchHost使用指南_新阿伟先生的博客-CSDN博客](https://blog.csdn.net/weixin_45022563/article/details/123922815) 

### 创建用户

创建用户 api 代码：
```go
// api/user_api/user_create.go
package user_api

import (
	"fmt"
	"github.com/gin-gonic/gin"
	"gvb_server/global"
	"gvb_server/models/ctype"
	"gvb_server/models/res"
	"gvb_server/service/user_ser"
)

type UserCreateRequest struct {
	NickName string     `json:"nick_name" binding:"required" msg:"请输入昵称"`         // 昵称
	UserName string     `json:"user_name" binding:"required" msg:"请输入用户名"`        // 用户名
	Password string     `json:"password" binding:"required" msg:"请输入密码"`          // 密码
	Role     ctype.Role `json:"role" binding:"required,oneof=1 2 3" msg:"权限参数错误"` // 权限 1 管理员 2 普通用户 3 游客
}

func (UserApi) UserCreateView(c *gin.Context) {
	var cr UserCreateRequest
	if err := c.ShouldBindJSON(&cr); err != nil {
		res.FailWithError(err, &cr, c)
		return
	}

	err := user_ser.UserService{}.CreateUser(cr.UserName, cr.NickName, cr.Password, cr.Role, "", c.ClientIP())
	if err != nil {
		global.Log.Error(err)
		res.FailWithMessage(err.Error(), c)
		return
	}
	res.OKWithMessage(fmt.Sprintf("用户 %s 创建成功", cr.UserName), c)
	return
}
```

创建用户逻辑代码：
```go
// service/user_ser/create_user.go
package user_ser

import (
	"errors"
	"gvb_server/global"
	"gvb_server/models"
	"gvb_server/models/ctype"
	"gvb_server/utils/pwd"
)

// 头像问题
// 1. 默认头像
// 2. 随机选择头像
const Avatar = "/upload/avatars/default.png"

func (UserService) CreateUser(userName, nickName, password string, role ctype.Role, email string, ip string) error {
	// 判断用户名是否存在
	var userModel models.UserModel
	err := global.DB.Take(&userModel, "user_name = ?", userName).Error
	if err == nil {
		return errors.New("用户名已存在")
	}

	// 对密码进行加密（hash）
	hashPwd := pwd.HashPwd(password)
	// 还可以用正则判断密码的强度，还可以用洛必达判断是否属于弱密码

	err = global.DB.Create(&models.UserModel{
		UserName:   userName,
		NickName:   nickName,
		Password:   hashPwd,
		Email:      email,
		Role:       role,
		Avatar:     Avatar,
		IP:         ip,
		Addr:       "内网地址", // 地址要根据IP算
		SignStatus: ctype.SignEmail,
	}).Error
	if err != nil {
		return err
	}
	return nil
}
```


# 标签管理

很传统的增删改查

## 增加标签

代码：
```go
// api/tag_api/tag_create.go
package tag_api

import (
	"github.com/gin-gonic/gin"
	"gvb_server/global"
	"gvb_server/models"
	"gvb_server/models/res"
)

type TagRequest struct {
	Title string `json:"title" binding:"required" msg:"请输入标题" structs:"title"` // 显示的标题
}

func (TagApi) TagCreateView(c *gin.Context) {
	var cr TagRequest
	err := c.ShouldBindJSON(&cr)
	if err != nil {
		res.FailWithError(err, &cr, c)
		return
	}

	// 重复判断
	var tag models.TagModel
	err = global.DB.Take(&tag, "title = ?", cr.Title).Error
	if err == nil { // 如果数据库中存在
		res.FailWithMessage("该标签已存在", c)
		return
	}

	err = global.DB.Create(&models.TagModel{
		Title: cr.Title,
	}).Error

	if err != nil {
		global.Log.Error(err)
		res.FailWithMessage("添加标签失败", c)
		return
	}

	res.OKWithMessage("添加标签成功", c)
}
```
## 修改标签

代码：
```go
// api/tag_api/tag_update.go
package tag_api

import (
	"github.com/fatih/structs"
	"github.com/gin-gonic/gin"
	"gvb_server/global"
	"gvb_server/models"
	"gvb_server/models/res"
)

func (TagApi) TagUpdateView(c *gin.Context) {
	id := c.Param("id")
	var cr TagRequest
	err := c.ShouldBindJSON(&cr)
	if err != nil {
		res.FailWithError(err, &cr, c)
		return
	}

	// 判断广告是否存在
	var tag models.TagModel
	err = global.DB.Take(&tag, id).Error
	if err != nil { // 如果数据库中存在
		res.FailWithMessage("标签不存在", c)
		return
	}

	// 结构体转map的第三方包structs
	maps := structs.Map(&cr)
	// Updates 好用嗷嗷嗷嗷嗷嗷嗷嗷嗷嗷
	err = global.DB.Model(&tag).Updates(maps).Error

	if err != nil {
		global.Log.Error(err)
		res.FailWithMessage("修改标签失败", c)
		return
	}

	res.OKWithMessage("修改标签成功", c)
}
```
## 标签列表

代码：
```go
// api/tag_api/tag_list.go
package tag_api

import (
	"github.com/gin-gonic/gin"
	"gvb_server/models"
	"gvb_server/models/res"
	"gvb_server/service/common"
)

func (TagApi) TagListView(c *gin.Context) {
	var cr models.PageInfo
	if err := c.ShouldBindQuery(&cr); err != nil {
		res.FailWitheCode(res.ArgumentError, c)
		return
	}

	list, count, _ := common.ComList(models.TagModel{}, common.Option{
		PageInfo: cr,
		Debug:    true,
	})

	res.OKWithList(list, count, c)
}
```
### 查文章的个数

- 根据当前标签的 id，查找文章库中对应标签 id 的文章的数量，返回给前端

## 标签删除

代码：
```go
// api/tag_api/tag_remove.go
package tag_api

import (
	"fmt"
	"github.com/gin-gonic/gin"
	"gvb_server/global"
	"gvb_server/models"
	"gvb_server/models/res"
)

func (TagApi) TagRemoveView(c *gin.Context) {
	var cr models.RemoveRequest // 要删除的广告的id列表
	err := c.ShouldBindJSON(&cr)
	if err != nil {
		res.FailWitheCode(res.ArgumentError, c)
		return
	}

	// 先从数据库中查询对应的信息
	var tagList []models.TagModel
	count := global.DB.Find(&tagList, cr.IDList).RowsAffected
	if count == 0 {
		res.OKWithMessage("标签不存在", c)
		return
	}

	// 查到啦，就要批量删除喽
	// 如果这个标签下有文章怎么办
	var articleCount int64
	global.DB.Model(&models.ArticleModel{}).Where("tag_id IN ?", cr.IDList).Count(&articleCount)

	if articleCount > 0 {
		res.FailWithMessage("该标签下存在文章，无法删除", c)
		return
	}

	// 标签下没有文章，可以进行批量删除
	global.DB.Delete(&tagList)
	res.OKWithMessage(fmt.Sprintf("共删除 %d 个标签", count), c)

}
```

### 标签有关联文章？

如果这个标签下有关联文章，如何处理？
- 先根据要删除的标签的 id，找文章库，将对应有该标签 id 的文章的标签信息删除，都删除完之后就可以删除当前标签啦
- 或者将这个标签打个标记，然后下次展示文章的时候，如果 hash 当前标签是被删除的，那就不展示，那么 tag_model 就要记录一个 bool 值，没有很大的必要性

# 消息管理

## 消息发送

- 注意传参只用查发件人 id 和收件人 id 和消息内容，其他信息由查表得到

```go
// api/message_api/message_create.go
package message_api

import (
	"github.com/gin-gonic/gin"
	"gvb_server/global"
	"gvb_server/models"
	"gvb_server/models/res"
)

type MessageRequest struct {
	SendUserID uint   `json:"send_user_id" binding:"required"` // 发送人id
	RecvUserID uint   `json:"recv_user_id" binding:"required"` // 接收人id
	Content    string `json:"content" binding:"required"`      // 消息内容
}

// MessageCreateView 当前用户发送消息
func (MessageApi) MessageCreateView(c *gin.Context) {
	var cr MessageRequest
	// SendUserID就是当前登录人的id
	err := c.ShouldBindJSON(&cr)
	if err != nil {
		res.FailWithError(err, &cr, c)
		return
	}
	var sendUser, recvUser models.UserModel
	err = global.DB.Take(&sendUser, cr.SendUserID).Error
	if err != nil {
		res.FailWithMessage("发送人信息不存在", c)
		return
	}
	err = global.DB.Take(&recvUser, cr.RecvUserID).Error
	if err != nil {
		res.FailWithMessage("接收人信息不存在", c)
		return
	}

	err = global.DB.Create(&models.MessageModel{
		SendUserID:       cr.SendUserID,
		SendUserNickName: sendUser.NickName,
		SendUserAvatar:   sendUser.Avatar,
		RevUserID:        cr.RecvUserID,
		RevUserNickName:  recvUser.NickName,
		RevUserAvatar:    recvUser.Avatar,
		IsRead:           false,
		Content:          cr.Content,
	}).Error

	if err != nil {
		res.FailWithMessage("消息发送失败", c)
		return
	}
	res.OKWithMessage("消息发送成功", c)
	return
}
```

## 消息列表

### 作为管理员

显示所有用户的聊天记录

```go
// api/message_api/message_list_all.go
package message_api

import (
	"github.com/gin-gonic/gin"
	"gvb_server/models"
	"gvb_server/models/res"
	"gvb_server/service/common"
)

func (MessageApi) MessageListAllView(c *gin.Context) {
	var cr models.PageInfo
	if err := c.ShouldBindQuery(&cr); err != nil {
		res.FailWitheCode(res.ArgumentError, c)
		return
	}
	list, count, _ := common.ComList(models.MessageModel{}, common.Option{
		PageInfo: cr,
		Debug:    true,
	})
	res.OKWithList(list, count, c)
}

```
### 作为用户

显示对方与我, 我与对方的聊天列表，点击展开就是聊天记录 --- 类似于QQ

判断标准，双方的用户 id 之和，相同就是一组消息

```go
// api/message_api/message_list.go
package message_api

import (
	"fmt"
	"github.com/gin-gonic/gin"
	"gvb_server/global"
	"gvb_server/models"
	"gvb_server/models/res"
	"gvb_server/utils/jwts"
	"time"
)

type Message struct {
	SendUserID       uint      `json:"send_user_id"`       // 发送人id
	SendUserNickName string    `json:"send_user_nickname"` // 发送人昵称
	SendUserAvatar   string    `json:"send_user_avatar"`   // 发送人头像
	RevUserID        uint      `json:"rev_user_id"`        // 接收人id
	RevUserNickName  string    `json:"rev_user_nickname"`  // 接收人昵称
	RevUserAvatar    string    `json:"rev_user_avatar"`    // 接收人头像
	Content          string    `json:"content"`
	CreatedAt        time.Time `json:"created_at"`    // 最新的消息时间
	MessageCount     int       `json:"message_count"` // 消息条数
}

type MessageGroup map[uint]*Message

// MessageListView 类似于QQ聊天页面，展示消息数量和最新的一条消息，没有分页
func (MessageApi) MessageListView(c *gin.Context) {
	_claims, _ := c.Get("claims")
	claims := _claims.(*jwts.CustomClaims)

	var messageGroup = MessageGroup{}
	var messageList []models.MessageModel
	var messages []Message

	fmt.Println(claims.ID)
	// 按升序找到我和其他人的聊天记录，其中我和某个人的聊天记录，对应的id和肯定相同，以此区分我与不同人的聊天
	global.DB.Order("created_at asc").
		Find(&messageList, "send_user_id = ? or rev_user_id = ?", claims.UserID, claims.UserID)

	for _, model := range messageList {
		message := Message{
			SendUserID:       model.SendUserID,
			SendUserNickName: model.SendUserNickName,
			SendUserAvatar:   model.SendUserAvatar,
			RevUserID:        model.RevUserID,
			RevUserNickName:  model.RevUserNickName,
			RevUserAvatar:    model.RevUserAvatar,
			Content:          model.Content,
			CreatedAt:        model.CreatedAt,
			MessageCount:     1,
		}
		idNum := model.SendUserID + model.RevUserID
		val, ok := messageGroup[idNum]
		if !ok {
			// 不存在
			messageGroup[idNum] = &message
			continue
		}
		// 存在
		message.MessageCount = val.MessageCount + 1
		messageGroup[idNum] = &message
	}

	for _, model := range messageGroup {
		messages = append(messages, *model)
	}
	res.OKWithData(messages, c)
	fmt.Println(messages)
	return
}
```

### 消息详情 

- 消息记录

```go
// api/message_api/message_record.go
package message_api

import (
	"github.com/gin-gonic/gin"
	"gvb_server/global"
	"gvb_server/models"
	"gvb_server/models/res"
	"gvb_server/utils/jwts"
)

type MessageRecordRequest struct {
	UserID uint `json:"user_id" binding:"required" msg:"请输入要查询的用户id"`
}

func (MessageApi) MessageRecordView(c *gin.Context) {
	var cr MessageRecordRequest
	err := c.ShouldBindJSON(&cr)
	if err != nil {
		res.FailWithError(err, &cr, c)
		return
	}

	_claims, _ := c.Get("claims")
	claims := _claims.(*jwts.CustomClaims)

	var _messageList []models.MessageModel
	var messageList = make([]models.MessageModel, 0)

	// 按升序找到我和其他人的聊天记录，其中我和某个人的聊天记录，对应的id和肯定相同，以此区分我与不同人的聊天
	global.DB.Order("created_at asc").
		Find(&_messageList, "send_user_id = ? or rev_user_id = ?", claims.UserID, claims.UserID)

	for _, model := range _messageList {
		if model.RevUserID == cr.UserID || model.SendUserID == cr.UserID {
			messageList = append(messageList, model)
		}
	}

	// 点开消息,里面的每―条消息,都从未读变成已读
	for _, model := range messageList {
		model.IsRead = true
		global.DB.Save(&model)
	}
	
	res.OKWithData(messageList, c)
}
```

# Es 操作

## 环境搭建

### windows 版本

教程：
[windows 安装es环境，手把手教学_windows安装es_喜欢猪猪的博客-CSDN博客](https://blog.csdn.net/qq_25580555/article/details/121495924#:~:text=%E7%AC%AC%E4%B8%80%E6%AD%A5%E4%B8%8B%E8%BD%BDes%E7%9A%84,dows%E5%AE%89%E8%A3%85es) 
中的下载链接是没有问题的，直接成功

压缩包解压 -> bin 目录 -> 双击 elasticsearch. bat ->运行如下
![[Pasted image 20231017161946.png]]
-> 即可浏览器访问 localhost: 9200 
![[Pasted image 20231017162032.png]]

如果出现上面的信息，说明已经启动并运行一个 Elasticsearch 节点了

### Grunt 命令行工具

- grunt是一个很方便的构建工具，可以进行打包压缩、测试、执行等等的工作，5.x里之后的**head插件就是通过grunt启动的**。因此需要安装grunt.

- Grunt 依赖 Node.js 所以在安装之前确保你安装了 **Node.js**。然后开始安装 Grunt
- 实际上，安装的并不是 Grunt，而是 **Grunt-cli**，也就是**命令行的 Grunt**，这样你就可以使用 grunt 命令来执行某个项目中的 Gruntfile.js 中定义的 task 。但是要注意，**Grunt-cli 只是一个命令行工具**，用来执行，而不是 Grunt 这个工具本身

安装（管理员身份）：
```shell
C:\windows\system32>npm install -g grunt-cli

added 58 packages in 3s
npm notice
npm notice New major version of npm available! 9.6.5 -> 10.2.0
npm notice Changelog: https://github.com/npm/cli/releases/tag/v10.2.0
npm notice Run npm install -g npm@10.2.0 to update!
npm notice

C:\windows\system32>grunt -version
grunt-cli v1.4.3
```

### head 插件

教程：
[windows环境下elasticsearch安装教程(超详细) - hualess - 博客园 (cnblogs.com)](https://www.cnblogs.com/hualess/p/11540477.html)  

- 网址: [https://github.com/mobz/elasticsearch-head](https://github.com/mobz/elasticsearch-head) 下载安装包
- 解压
- 手动安装 pathomjs（因为我的 node. js 版本不符合，所以手动。。。），详情在下一个板块
- 安装完成之后在 npm run start 运行启动 head 插件
```shell
PS D:\Download2\ESHead\elasticsearch-head-master\elasticsearch-head-master> npm run start

> elasticsearch-head@0.0.0 start
> grunt server

Running "connect:server" (connect) task
Waiting forever...
Started connect web server on http://localhost:9100
```

**至此运行成功运行**
### phantomjs 浏览器

PhantomJS 是一个**无界面的、可脚本编程的 WebKit 浏览器引擎**，其快速，原生支持各种 Web 标准： DOM 处理, CSS 选择器, JSON, Canvas, 和 SVG。

- PhantomJS是一个基于webkit内核、无界面的浏览器，即它就是一个浏览器，只是其内的点击、翻页等人为相关操作需要程序设计实现；
- PhantomJS提供Javascript API接口，可以通过编写JS程序直接与webkit内核交互；
- PhantomJS的应用：无需浏览器的 Web 测试、网页截屏、页面访问自动化、网络监测。

下载教程：
[PhantomJs的用法(一)——下载与安装 - 简书 (jianshu.com)](https://www.jianshu.com/p/80e984ef94d3) 

### 修改 es 使用的参数

```shell
# 增加新的参数，这样head插件可以访问es
http.cors.enabled: true 
http.cors.allow-origin: "*"
@注意，设置参数的时候:后面要有空格！
```

![[Pasted image 20231017203810.png]]
- 修改完配置将es重启,浏览器访问 http://localhost:9100

**到此，Elasticsearch 和 ElasticSearch-head 已经装好了**

### ElasticSearch 安装为 Windows 服务

- elasticsearch 的 bin 目录下有一个 elasticsearch-service.bat
- cmd 进入 bin 目录下执行: elasticsearch-service. bat install
- 查看电脑服务 es 已经存在了 -- **不能和 elasticsearch. bat 同时打开** 
![[Pasted image 20231017205810.png]]

```shell
elasticsearch-service.bat后面还可以执行这些命令
install: 安装Elasticsearch服务
remove: 删除已安装的Elasticsearch服务（如果启动则停止服务）
start: 启动Elasticsearch服务（如果已安装）
stop: 停止服务（如果启动）
manager:启动GUI来管理已安装的服务
```

整体路线教程：
[windows环境下elasticsearch安装教程(超详细) - hualess - 博客园 (cnblogs.com)](https://www.cnblogs.com/hualess/p/11540477.html) 

## es 连接

## 增删改查
