## 普通的在goland中设置编译为linux可执行程序

1、 去掉 Run after build 

2、在Environment 上写入参数： GOARCH=amd64;GOOS=linux

## 重定向
goland中的编译设置界面如下
![](addition/Pasted%20image%2020231004145246.png)
我通过上面的方法编译出的linux可执行程序，在命令行里面重定向不了，大概是输入为空或者没有配置的原因吧
