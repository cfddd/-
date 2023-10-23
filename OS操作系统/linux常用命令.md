[40个最常用的Linux命令行大全 - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/420247468)
[常用的Linux命令（面试/工作必备） - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/458483707)
**一、关机重启**

```text
shutdown -h now        立刻关机
shutdown -h 5          5分钟后关机
poweroff               立刻关机
shutdown -r now        立刻重启
shutdown -r 5          5分钟后重启
reboot                 立刻重启
```

**二、目录操作**

```text
pwd		        查看当前工作目录
clear 			清除屏幕
cd ~			当前用户目录
cd /			根目录
cd -			上一次访问的目录
cd ..			上一级目录
```

**查看目录内信息**

```text
ll		        查看当前目录下内容
ls                      查看当前目录的相信内容内容
```

创建目录

```text
mkdir    aaa            在当前目录下创建一个名为aaa的目录
mkdir    /usr/aaa       在指定目录下创建一个名为aaa的目录
```

**搜索命令**

```text
find / -name 'b'        查询根目录下（包括子目录），名以b的目录和文件； 
find / -name 'b*'	查询根目录下（包括子目录），名以b开头的目录和文件； 
```

**重命名**

```text
mv 原先目录 文件的名称   mv test001 test 
```

**剪切命令**

```text
mv	/aaa /bb			将根目录下的aaa目录，移动到bbb目录下
mv	bbbb usr/bbb			将当前目录下的bbbb目录，移动到usr目录下，并且修改名称为bbb；
mv	bbb usr/aaa			将当前目录下的bbbb目录，移动到usr目录下，并且修改名称为aaa；
```

**复制目录**

```text
cp /usr/tmp/aaa  /usr                   将/usr/tmp目录下的aaa目录复制到 /usr目录下面
```

**删除目录**

```text
rm -r /bbb			普通删除，询问你是否删除每一个文件
rm -rf /bbb			强制删除/目录下的bbb目录。如果bbb目录中还有子目录，也会被强制删除，不会提示；
```

**三、文件操作**

**删除文件**

```text
rm -r a.java		       删除当前目录下的a.java文件
rm -rf a.java		       强制删除当前目录下的a.java文件
rm -rf ./a*		       强制删除当前目录下以a开头的所有文件；
rm -rf ./*		       强制删除当前目录下所有文件（慎用）；
```

**创建文件**

```text
touch test
```

**修改文件**

```text
vi/vim 文件名
点击键盘i进入编辑模式
ESC 退出编辑模式到命令行模式
退出编辑：    :q
强制退出：    :q!
保存并退出：  :wq
```

**文件的查看**

```text
cat：看最后一屏
more：百分比显示
less：翻页查看
tail：指定行数或者动态查看
```

**四、创建与删除软连接**

**创建软连接**

```text
ln -s /usr/local/app /data
注意：创建软连接时，data目录后不加 / (加上后是查找其下一级目录)
```

**删除软连接**

```text
ln -s /usr/local/app /data
```

**五、压缩和解压缩**

**压缩**

```text
tar -cvf start.tar a.java b.java		//将当前目录下a.java、b.java打包
tar -cvf start.tar ./*			        //将当前目录下的所欲文件打包压缩成haha.tar文件
tar -zcvf start.tar.gz a.java b.java	        //将当前目录下a.java、b.java打包
tar -zcvf start.tar.gz ./*		        //将当前目录下的所欲文件打包压缩成start.tar.gz文件
```

解压缩

```text
tar -xvf start.tar				//解压start.tar压缩包，到当前文件夹下；
tar -xvf start.tar -C usr/local 		//解压start.tar压缩包，到/usr/local目录下；

tar -zxvf start.tar.gz			         //解压start.tar.gz压缩包，到当前文件夹下；
tar -zxvf start.tar.gz -C usr/loca            	//解压start.tar.gz压缩包，到/usr/local目录下；
```

压缩(zip)

```text
zip lib.zip tomcat.jar					//将单个文件压缩(lib.zip)
zip -r lib.zip lib/					//将目录进行压缩(lib.zip)
zip -r lib.zip tomcat-embed.jar xml-aps.jar		//将多个文件压缩为zip文件(lib.zip)	
```

解压缩(unzip)

```text
unzip file1.zip  					//解压一个zip格式压缩包
unzip -d /usr/app/com.lydms.test.zip			//将`test.zip`包，解压到指定目录下`/usr/app/`
```

**六、查找命令**

**grep**

```text
ps -ef | grep sshd                  查找指定ssh服务进程 
ps -ef | grep sshd | grep -v grep   查找指定服务进程，排除gerp身 
ps -ef | grep sshd -c               查找指定进程个数 
```

**find**

```text
find . -name "*.log" -ls           在当前目录查找以.log结尾的文件，并显示详细信息。 
find /root/ -perm 600              查找/root/目录下权限为600的文件 
find . -type f -name "*.log"       查找当目录，以.log结尾的普通文件 
find . -type d | sort              查找当前所有目录并排序 
find . -size +100M                 查找当前目录大于100M的文件
```

**七、yum常用命令**

```text
yum install iptables-services	   下载并安装iptables
yum list			   列出当前系统中安装的所有包
yum search package_name		   在rpm仓库中搜寻软件包
yum update package_name.rpm	   更新当前系统中所有安装的rpm包
yum update package_name		   更新一个rpm包
yum remove package_name	           删除一个rpm包
yum clean all			   删除所有缓存的包和头文件
```

**八、系统服务**

```text
service iptables status           查看iptables服务的状态
service iptables start            开启iptables服务
service iptables stop             停止iptables服务
service iptables restart          重启iptables服务 
chkconfig iptables off            关闭iptables服务的开机自启动
chkconfig iptables on             开启iptables服务的开机自启动
```

**九、用户管理**

```text
su - 用户名                         切换用户，并且切换目录
exit                               退出当前登录账户
注意：su 不接用户名，可以切换到 root ，但是不推荐使用，因为不安全
```

**which**

```text
/etc/passwd 是用于保存用户信息的文件
/usr/bin/passwd 是用于修改用户密码的程序
which 命令可以查看执行命令所在位置，例如：
which ls
# 输出
# /bin/ls
which useradd
# 输出
# /usr/sbin/useradd
```

**十、其他命令**

**查看当前目录：pwd**

```text
命令：pwd     查看当前目录路径
```

**查看进程：ps -ef**

```text
命令：ps -ef    查看所有正在运行的进程
```

**结束进程：kill**

```text
命令：kill pid 或者 kill -9 pid(强制杀死进程)           pid:进程号
```

**网络通信命令：**

```text
ifconfig：查看网卡信息
命令：ifconfig 或 ifconfig | more
```

**ping：查看与某台机器的连接情况**

```text
命令：ping ip
```

**netstat -an：查看当前系统端口**

```text
命令：netstat -an
```

**搜索指定端口**

```text
命令：netstat -an | grep 8080
```

**配置网络**

```text
命令：setup
```

**重启网络**

```text
命令：service network restart
```

**关闭防火墙**

```text
命令：chkconfig iptables off
     iptables -L;
     iptables -F;
     service iptables stop
```

**修改文件权限**

```text
命令：chmod 777
```

**清屏**

```text
命令：ctrl + l
```