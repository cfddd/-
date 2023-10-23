## 1.新工作台
右上角选择“新建”，然后选择go语言

不要直接赋值demo代码里面的那个，没有mysql和相关的go支持

**因为在1024平台上直接clone代码构建项目不方便，于是选择将所有代码在本地编译好再上传上去**
## 2.go框架需求  
需要gin,gorm和mysql驱动  
```shell  
go get -u github.com/gin-gonic/gin
go get -u github.com/jinzhu/gorm
go get -u github.com/go-sql-driver/mysql
```  
然后再使用`go mod tidy`更新依赖
## 3.mysql配置  

首先在命令行里链接数据库
```shell
mysql -h ${MYSQL_HOST} -P ${MYSQL_PORT} -u ${MYSQL_USER} -p
```
然后创建一个名为douyin的数据库
```shell
create database douyin;
```
在`database/iniMYSQL.go`文件中有如下内容  
```go  
username := "root" //用户名  
password := "123456" //密码  
host := "127.0.0.1" //数据库地址，可以是IP或者域名  
port := 3306 //端口号  
Dbname := "douyin" //数据库名  
timeout := "10s" //超时连接，10秒  
```  
根据自己的mysql配置信息链接  
  
创建一个名为douyin的数据库  
  
然后运行该文件
>`DB.Debug().AutoMigrate(&models.Video{}, &models.Comment{}, models.User{}, &models.Like{}, &models.Post{})`
>这行代码是根据数据模型创建数据库中的表，仅第一次运行时需要使用
>每次关闭1024代码空间都会更换数据库的ip和端口，所以建议使用环境变量的方式给变量写

```go
	username := os.Getenv("MYSQL_USER") //用户名
	password := os.Getenv("MYSQL_PASSWORD")  //密码
	host := os.Getenv("MYSQL_HOST") //数据库地址，可以是IP或者域名
	port, _ := strconv.ParseInt(os.Getenv("MYSQL_PORT"), 10, 32) //端口号
	Dbname := "douyin" //数据库名
	timeout := "10s"     //超时连接，10秒

```
## 4.添加环境变量  
使用到的ffmpeg软件通过添加一个环境变量由程序调用  
  
ffmpeg 软件所在据对位置目录

> **注意**:1024code里面使用的linux环境，ffmpeg.exe无法使用，需要替换掉service/ffmpeg.exe文件为对应系统的版本
> [下载地址](https://johnvansickle.com/ffmpeg/)

D:\goland\simple-demo-main\service\ffmpeg.exe
下载后在1024平台解压
```shell
xz -d ffmpeg-git-amd64-static.tar.xz
tar -xvf ffmpeg-git-amd64-static.tar
#进入解压的文件夹下
./ffmpeg
#如果出现ffmpeg version N-66595-gc2b38619c0-static ...说明成功
```
然后添加环境变量
```shell
vim ~/.bashrc
# 在最后一行加上
export PATH="$PATH:/home/runner/app/simple-demo-main/for1024/ffmpeg-git-20230721-amd64-static"
# 手动更新
source ~/.bashrc
```
## 5.上传在本地编译好的文件
**因为在1024平台上直接clone代码构建项目不方便，于是选择将所有代码在本地编译好再上传上去**

我试了在1024上，遇到了各种问题（操作系统不同，goland本地写的包无法在1024用……）

在goland运行选择编辑运行选项`Run Condifuration`，在环境处加上
```
GOOS=linux;GOARCH=amd64
```
再取消选择`Run after build`

在选择一个输出路径即可

我一共上传了三个文件
- 下载好的ffmpeg压缩包
- 初始化数据库的代码编译的可执行文件
- 项目的可执行文件

> 1024每一次刷新链接，都会更新数据库的ip和端口
> 
> 数据库和表都在，数据也不会没
> 
> 所以我专门写了一个初始化数据库的代码，方便使用

## 6.全部完成，开始运行
在上面的所有内容都检查过后没有问题
就可以开始运行代码了

可执行文件刚被放上去，只是一个普通的文件，没有执行权限，需要用下面的权限让它可以运行
```
chmod +x ./xxx
./xxx
```
运行之后，1024code会给一个链接（感觉直接绑定在了这个8080默认端口上），放在postman里面可以直接访问

## 7.总结
1024code感觉用起来很吃力，很多地方没讲明白

linux基础有一点，但是不多
