> [40个最常用的Linux命令行大全 - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/420247468)
> [常用的Linux命令（面试/工作必备） - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/458483707)
>[Linux 关机命令（超详细） - 犬小哈教程 (quanxiaoha.com)](https://www.quanxiaoha.com/linux-command/linux-shutdown.html)

## 常用命令
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

**解压缩**

```text
tar -xvf start.tar				//解压start.tar压缩包，到当前文件夹下；
tar -xvf start.tar -C usr/local 		//解压start.tar压缩包，到/usr/local目录下；

tar -zxvf start.tar.gz			         //解压start.tar.gz压缩包，到当前文件夹下；
tar -zxvf start.tar.gz -C usr/loca            	//解压start.tar.gz压缩包，到/usr/local目录下；
```

**压缩(zip)**

```text
zip lib.zip tomcat.jar					//将单个文件压缩(lib.zip)
zip -r lib.zip lib/					//将目录进行压缩(lib.zip)
zip -r lib.zip tomcat-embed.jar xml-aps.jar		//将多个文件压缩为zip文件(lib.zip)	
```

**解压缩(unzip)**

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
- -t (tcp) 仅显示tcp相关选项
- -u (udp)仅显示udp相关选项
- -n 拒绝显示别名，能显示数字的全部转化为数字
- -l 仅列出在Listen(监听)的服务状态
- -p 显示建立相关链接的程序名
**搜索指定端口**

```text
命令：netstat -an | grep 8080
```
**查看占用端口进程的PID**

```bash
netstat -tunlp|grep {port}
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
> [Linux文件权限详解_linux 文件权限-CSDN博客](https://blog.csdn.net/lv8549510/article/details/85406215)


```text
命令：chmod 777
```

**清屏**

```text
命令：ctrl + l
```

## 查看磁盘、内存使用情况
### df 显示磁盘分区上可以使用的磁盘空间
显示指定磁盘文件的可用空间。如果没有文件名被指定，则所有当前被挂载的文件系统的可用空间将被显示。默认情况下，磁盘空间将以 1KB为单位进行显示，除非环境变量 POSIXLY_CORRECT 被指定，那样将以512字节为单位进行显示。

### 使用方式

**df [选项] [文件]**
命令参数
```

-a 列出所有的文件系统，包括系统特有的 /proc 等文件系统
-k 以 KBytes 的容量显示各文件系统。命令 df -k 同命令 df
-m 以 MBytes 的容量显示各文件系统
-h 以人们较易阅读的 GBytes、MBytes、KBytes 等格式自行显示
-H 等于“-h”，但是计算式，1K=1000，而不是1K=1024
-i 不用磁盘容量，而以 inode 的数量来显示
-l 只显示本地文件系统。命令 df -l 同命令 df
--no-sync 忽略 sync 命令
-P 输出格式为POSIX
--sync 在取得磁盘信息前，先执行sync命令
-T  连同该磁盘分区的文件系统名称（例如 xfs）也列出
--block-size=<区块大小> 指定区块大小
-t <文件系统类型> 只显示选定文件系统的磁盘信息
-x <文件系统类型> 不显示选定文件系统的磁盘信息
--help 显示帮助信息
--version 显示版本信息
```
### 使用实例
实例1：显示磁盘使用情况

```
[root@server1 ~]# df
Filesystem            1K-blocks    Used Available Use% Mounted on
/dev/mapper/rhel-root  17811456 1196004  16615452   7% /
devtmpfs                 497216       0    497216   0% /dev
tmpfs                    508188       0    508188   0% /dev/shm
tmpfs                    508188    6736    501452   2% /run
tmpfs                    508188       0    508188   0% /sys/fs/cgroup
/dev/vda1               1038336  141508    896828  14% /boot
tmpfs                    101640       0    101640   0% /run/user/0
```

1、Filesystem：代表文件系统对应的设备文件的路径名（一般是硬盘上的分区）；

2、1K-blocks：说明下面的数字单位是  1KB，可利用 -h 或 -m 来改变容量；

3、Used：使用掉的磁盘空间；

4、Available：也就是剩下的磁盘空间大小；用户也许会感到奇怪的是，第3，4列块数之和不等于第2列中的块数。这是因为缺省的每个分区都留了少量空间供系统管理员使用。即使遇到普通用户空间已满的情况，管理员仍能登录和留有解决问题所需的工作空间清单中；

5、Use%：就是磁盘的使用率，如果使用率高达 90%  以上，最好注意一下，免得容量不足造成系统问题，例如最容易占满的  /var/spool/mail  这个保存邮件的目录；

6、Mounted  on：就是磁盘的挂载目录（挂载点）。

### 实例2：以inode模式来显示磁盘使用情况

```
[root@server1 ~]# df -i
Filesystem             Inodes IUsed   IFree IUse% Mounted on
/dev/mapper/rhel-root 8910848 35051 8875797    1% /
devtmpfs               124304   374  123930    1% /dev
tmpfs                  127047     1  127046    1% /dev/shm
tmpfs                  127047   410  126637    1% /run
tmpfs                  127047    16  127031    1% /sys/fs/cgroup
/dev/vda1              524288   328  523960    1% /boot
tmpfs                  127047     1  127046    1% /run/user/0
```
### 实例3：列出文件系统的类型

```
[root@server1 ~]# df -T
Filesystem            Type     1K-blocks    Used Available Use% Mounted on
/dev/mapper/rhel-root xfs       17811456 1196004  16615452   7% /
devtmpfs              devtmpfs    497216       0    497216   0% /dev
tmpfs                 tmpfs       508188       0    508188   0% /dev/shm
tmpfs                 tmpfs       508188    6740    501448   2% /run
tmpfs                 tmpfs       508188       0    508188   0% /sys/fs/cgroup
/dev/vda1             xfs        1038336  141508    896828  14% /boot
tmpfs                 tmpfs       101640       0    101640   0% /run/user/0
```
### 实例4：显示目前磁盘空间和使用情况 （最常用）

```
[root@server1 ~]# df -h
Filesystem             Size  Used Avail Use% Mounted on
/dev/mapper/rhel-root   17G  1.2G   16G   7% /
devtmpfs               486M     0  486M   0% /dev
tmpfs                  497M     0  497M   0% /dev/shm
tmpfs                  497M  6.6M  490M   2% /run
tmpfs                  497M     0  497M   0% /sys/fs/cgroup
/dev/vda1             1014M  139M  876M  14% /boot
tmpfs                  100M     0  100M   0% /run/user/0
 
 
[root@server1 ~]# df -H
Filesystem             Size  Used Avail Use% Mounted on
/dev/mapper/rhel-root   19G  1.3G   18G   7% /
devtmpfs               510M     0  510M   0% /dev
tmpfs                  521M     0  521M   0% /dev/shm
tmpfs                  521M  7.0M  514M   2% /run
tmpfs                  521M     0  521M   0% /sys/fs/cgroup
/dev/vda1              1.1G  145M  919M  14% /boot
tmpfs                  105M     0  105M   0% /run/user/0
 
 
[root@server1 ~]# df -lh
Filesystem             Size  Used Avail Use% Mounted on
/dev/mapper/rhel-root   17G  1.2G   16G   7% /
devtmpfs               486M     0  486M   0% /dev
tmpfs                  497M     0  497M   0% /dev/shm
tmpfs                  497M  6.6M  490M   2% /run
tmpfs                  497M     0  497M   0% /sys/fs/cgroup
/dev/vda1             1014M  139M  876M  14% /boot
tmpfs                  100M     0  100M   0% /run/user/0
 
 
[root@server1 ~]# df -k
Filesystem            1K-blocks    Used Available Use% Mounted on
/dev/mapper/rhel-root  17811456 1196004  16615452   7% /
devtmpfs                 497216       0    497216   0% /dev
tmpfs                    508188       0    508188   0% /dev/shm
tmpfs                    508188    6740    501448   2% /run
tmpfs                    508188       0    508188   0% /sys/fs/cgroup
/dev/vda1               1038336  141508    896828  14% /boot
tmpfs                    101640       0    101640   0% /run/user/0

```
说明：

-h更具目前磁盘空间和使用情况 以更易读的方式显示

-H根上面的-h参数相同，不过在根式化的时候，采用1000而不是1024进行容量转换

-k以单位显示磁盘的使用情况

-l显示本地的分区的磁盘空间使用率，如果服务器nfs了远程服务器的磁盘，那么在df上加上-l后系统显示的是过滤nsf驱动器后的结果

-i显示inode的使用情况。linux采用了类似指针的方式管理磁盘空间影射。这也是一个比较关键应用

## du 显示每个文件和目录的磁盘使用空间
显示每个文件和目录的磁盘使用空间

### 使用方式

**df** 
命令参数
```
-a或-all  列出所you的文件与目录容量，因为默认仅统计目录下面的文件量  
-b或-bytes  显示目录或文件大小时，以byte为单位。   
-c或--total  除了显示个别目录或文件的大小外，同时也显示所有目录或文件的总和。
-k或--kilobytes  以KB(1024bytes)为单位输出。
-m或--megabytes  以MB为单位输出。   
-s或--summarize  仅显示总量，只列出最后加总的值，而不列出每个个别的目录占用容量。
-S或--separate-dirs   不包括子目录下的总计，与 -s 有点差别
-h或--human-readable  以K，M，G为单位，提高信息的可读性。
-x或--one-file-xystem  以一开始处理时的文件系统为准，若遇上其它不同的文件系统目录则略过。
-L<符号链接>或--dereference<符号链接> 显示选项中所指定符号链接的源文件大小。   
-X<文件>或--exclude-from=<文件>  在<文件>指定目录或文件。   
--exclude=<目录或文件>         略过指定的目录或文件。    
-D或--dereference-args   显示指定符号链接的源文件大小。   
-H或--si  与-h参数相同，但是K，M，G是以1000为换算单位。   
-l或--count-links   重复计算硬件链接的文件。   
```
### 使用实例

**实例1：显示目录或者文件所占空间**
```
[root@server1 ~]# du
48	./nginx-1.14.2/auto/cc
...
2468	./nginx-1.15.8/objs
9464	./nginx-1.15.8
20980	.
 
```
说明：
直接输入 du 没有加任何选项时，则 du 会分析【目前所在目录】的文件与目录所占用的磁盘空间。
但是，实际显示时，仅显示目录容量（不含文件），因此（.）目录有很多文件没有列出来。
所以全部的目录相加不会等于（.）的容量，此外，输出的数据为 1K 大小的容量单位

**实例2：显示指定文件所占空间**
```
[root@server1 ~]# du date.txt 
4	date.txt
```
**实例3：查看指定目录的所占空间**
```
[root@server1 ~]# du nginx-1.14.2
48	nginx-1.14.2/auto/cc
...
9348	nginx-1.14.2
```
**实例4：显示多个文件所占空间**
```
[root@server1 ~]# du nginx-1.14.2.tar.gz nginx-1.15.8.tar.gz 
992	nginx-1.14.2.tar.gz
1004	nginx-1.15.8.tar.gz
```
**实例5：方便阅读的格式显示（常用）**
```
[root@server1 ~]# du -h nginx-1.14.2
48K	nginx-1.14.2/auto/cc
...
9.2M	nginx-1.14.2
```
**实例6：文件和目录都显示**

```
[root@server1 ~]# du -ah nginx-1.14.2
4.0K	nginx-1.14.2/auto/cc/acc
...
8.0K	nginx-1.14.2/src/stream/ngx_stream_upstream_round_robin.h
12K	nginx-1.14.2/src/stream/ngx_stream_upstream_zone_module.c
32K	nginx-1.14.2/src/stream/ngx_stream_variables.c
4.0K	nginx-1.14.2/src/stream/ngx_stream_variables.h
8.0K	nginx-1.14.2/src/stream/ngx_stream_write_filter_module.c
...
9.2M	nginx-1.14.2
```
**实例7：按照空间大小排序**

```
[root@server1 ~]# du |sort -nr|more 
20980	.
9464	./nginx-1.15.8
940	./nginx-1.14.2/objs/src/http
888	./nginx-1.15.8/src/core
388	./nginx-1.15.8/src/http/v2
140	./nginx-1.15.8/contrib/vim
136	./.vim
40	./nginx-1.15.8/conf
...
0	./nginx-1.14.2/objs/src/http/modules/perl
```
**实例8：输出当前目录下各个子目录所使用的空间（常用）**

```
[root@server1 ~]# du -h --max-depth=1
9.2M	./nginx-1.14.2
136K	./.vim
9.3M	./nginx-1.15.8
21M	.
```
不带--max-depth参数，那么将循环列出文件夹下所有文件和文件夹占用的空间，带此参数，则是指定深入目录的层数。

## Linux du命令和df命令区别
1、du ：是通过搜索文件来计算每个文件的大小然后累加，du能看到的文件只是一些当前存在的，没有被删除的。他计算的大小就是当前他认为存在的所有文件大小的累加和。
2、 df： 通过文件系统来快速获取空间大小的信息，当我们删除一个文件的时候，这个文件不是马上就在文件系统当中消失了，而是暂时消失了，当所有程序都不用时，才会根据OS的规则释放掉已经删除的文件，df记录的是通过文件系统获取到的文件的大小，他比du强的地方就是能够看到已经删除的文件，而且计算大小的时候，把这一部分的空间也加上了，更精确了。
当文件系统也确定删除了该文件后，这时候du与df就一致了。

3、free 显示内存使用情况
free指令会显示内存的使用情况，包括实体内存，虚拟的交换文件内存，共享内存区段，以及系统核心使用的缓冲区等。
使用方式

**free** 
命令参数
```
-b 　以Byte为单位显示内存使用情况。
-k 　以KB为单位显示内存使用情况。
-m 　以MB为单位显示内存使用情况。
-h 　以合适的单位显示内存使用情况，最大为三位数，自动计算对应的单位值。单位有：
	B = bytes
	K = kilos
	M = megas
	G = gigas
	T = teras
	
-o 　不显示缓冲区调节列。
-s<间隔秒数> 　持续观察内存使用状况。
-t 　显示内存总和列。
-V 　显示版本信息。
```
**实例1：显示内存使用信息**
```
[root@server1 ~]# free
total used free shared buffers cached
Mem: 254772 184568 70204 0 5692 89892
-/+ buffers/cache: 88984 165788
Swap: 524280 65116 459164

Mem行（单位均为M）：
    total   系统总的可用物理内存大小
    
    used    已被使用的物理内存大小
    
    free    还有多少物理内存可用
    
    shared  被共享使用的物理内存大小
    
    buff/cache  被 buffer 和 cache 使用的物理内存大小
    
    available   还可以被 应用程序 使用的物理内存大小
    
   (-/+ buffers/cache)行：
		（-buffers/cache）: 真正使用的内存数，指的是第一部分的 used - buffers - cached
		（+buffers/cache）: 可用的内存数，指的是第一部分的 free + buffers + cached
```
Swap行指交换分区		

**实例2:以总和的形式查询内存的使用信息**

```
[root@server1 ~]# free -t
total used free shared buffers cached
Mem: 254772 184868 69904 0 5936 89908
-/+ buffers/cache: 89024 165748
Swap: 524280 65116 459164
Total: 779052 249984 529068
```

**实例3:周期性的查询内存使用信息**
```
[root@server1 ~]# free -s 10 //每10s 执行一次命令
total used free shared buffers cached
Mem: 254772 187628 67144 0 6140 89964
-/+ buffers/cache: 91524 163248
Swap: 524280 65116 459164

total used free shared buffers cached
Mem: 254772 187748 67024 0 6164 89940
-/+ buffers/cache: 91644 163128
Swap: 524280 65116 459164
```
## 使用top命令监控系统进程
top：“实时查看” ，按q退出 (实时动态显示)

```
   -a　　# 将进程按照使用内存排序

　　-b　　# 批处理的模式显示进程信息，输出结果可以传递给其他程序或写入到文件中，配合-n使用，一直打到-n设置的阈值

　　-c　　# 显示进程的整个命令路径，而不是只显示命令名称

　　-d　　# 指定每两次屏幕信息刷新之间的时间间隔

　　-H　　# 指定这个可以显示每个线程的情况，否则就是进程的总的状态

　　-i　　# 不显示闲置或者僵死的进程状态　　

　　-n　　# top输出信息更新的次数，完成后将推出top命令

　　-p　　# 显示指定的进程信息
```
键入 top,显示如下信息
```
top - 14:27:26 up 4:22, 1 user, load average: 0.08, 0.03, 0.05
Tasks:  96 total,   2 running,  94 sleeping,   0 stopped,   0 zombie
%Cpu(s):  0.0 us,  0.3 sy,  0.0 ni, 99.7 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st
KiB Mem :  1865284 total,  1460360 free,    96264 used,   308660 buff/cache
KiB Swap:  1048572 total,  1048572 free,        0 used.  1595216 avail Mem 

   PID USER      PR  NI    VIRT    RES    SHR S %CPU %MEM     TIME+ COMMAND            
  2029 root      20   0       0      0      0 R  0.3  0.0   0:00.14 kworker/0:1 
```
第一行：任务队列信息，同uptime命令的执行结果
　　　

　　14:27:26　　　　# 当前系统时间
　　up  4:26　　　　# 系统已经运行了4个半小时
　　1 user　　　　　# 当前有1个用户登录系统
　　load average: 0.08, 0.03, 0.05　　　　# 1分钟，5分钟，15分钟的平均负载情况

第二行：Tasks为任务（进程）。上面的信息显示为
　　共有96个进程，处于运行状态的有2个，94个在休眠，stoped状态0个，zombie状态有0个

第三行：CPU状态信息
　　us　　# 用户空间占用CPU的百分比
　　sy　　# 内核空间占用cpu的百分比
　　ni　　# 改变优先级的进程占用CPU的百分比
　　id　　# 空闲CPU百分比
　　wa　　# I/O等待只用CPU的百分比
　　hi　　# 硬中断占用CPU的百分比
　　si　　# 软中断
　　st　　# 虚拟机占用CPU的百分比

第四行：内存状态

　　total　　# 物理内存总量
　　used　　　# 使用中的内存总量
　　free　　　# 空闲内存总量
　　buffers　　# 缓冲的内存量

第五行：swap交换分区信息
　　total　　# 交换分区总量
　　used　　　# 使用的交换区总量
　　free　　　　# 空闲交换区总量
　　cached　　# 缓存的内存量

第六行：空行

第七行：给出的各进程（任务）的状态监控
　　PID　　# 进程iD
　　USER　　# 进程所有者
　　PR　　　　# 进程优先级
　　NI　　　　# nice值，负值表示高优先级，正值表示低优先级
　　VIRT　　　　# 进程使用的虚拟内存总量，单位为KB
　　RES　　　　# 进程使用的，未被换出的物理内存大小，单位KB
　　SHR　　　　# 共享内存大小，单位为kb
　　S　　　　　　# 进程状态，D=不可中断的睡眠状态，R=运行，S=睡眠，T=跟踪/停止，Z=僵尸进程
　　%CPU　　　　# 上次更新到现在的CPU时间占用百分比
　　%MEM　　　　# 进程使用的物理内存百分比
　　TIME+　　　　# 进程使用的物理内存百分比
　　COMMAND　　# 进程名称

## 挂起进程
[linux后台运行、挂起、恢复进程相关命令_linux 在另一个session回复被挂起的程序-CSDN博客](https://blog.csdn.net/koberonaldo24/article/details/103136125)
```
nohup <command> &
```
## 查询域名对应的ip
查看域名对应的ip，无论在windows还是在linux下都是用 `nslookup`
```shell
PS C:\Users\DELL> nslookup baidu.com
服务器:  cache.ahwhtel.net.cn
Address:  202.102.213.68

非权威应答:
名称:    baidu.com
Addresses:  110.242.68.66
          39.156.66.10
```
## 用户权限
> [Ubuntu/Linux用户管理与权限管理（超详细解析）_ubuntu nologin-CSDN博客](https://blog.csdn.net/yl19870518/article/details/100776136)
> [Ubuntu 20.04操作基础（8：用户权限相关命令） - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/560862357)