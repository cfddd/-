克隆了main分支到linux虚拟机中，记录部署出现的问题

项目缺少更具体的安装文档，有的地方不好理解
## 1.直接运行`install/amd64/fobrain/install.sh`无法完成安装
有如下报错，缺少
1. docker 的文件
2. 镜像
```log
XXX
2
系统配置初始化... 
XXX
XXX
5
安装Docker环境... 
XXX
/bin/cp: cannot stat 'docker/bin/*': No such file or directory
/bin/cp: cannot stat 'docker/docker.service': No such file or directory
{
    "ipv6": true,
    "fixed-cidr-v6": "2001:db8:1::/64",
    "data-root": "/data/docker/data-root"
}
Synchronizing state of docker.service with SysV service script with /lib/systemd/systemd-sysv-install.
Executing: /lib/systemd/systemd-sysv-install enable docker
XXX
6
安装系统依赖环境... 
XXX
/bin/cp: cannot stat 'fobrain/*': No such file or directory
XXX
6.3
安装系统依赖环境... 
XXX
sed: can't read /data/fobrain/docker-compose.yaml: No such file or directory
XXX
6.6
安装系统依赖环境... 
XXX
XXX
6.8
安装系统依赖环境... 
XXX
XXX
15
加载Es镜像文件... 
XXX
open images/es: no such file or directory
XXX
20
加载Redis镜像文件... 
XXX
open images/redis: no such file or directory
XXX
25
加载MySql镜像文件... 
XXX
open images/mysql: no such file or directory
XXX
35
加载FOBrain镜像文件... 
XXX
open images/fobrain: no such file or directory
XXX
70
配置系统防火墙... 
XXX
Failed to enable unit: Unit file firewalld.service does not exist.
Failed to start firewalld.service: Unit firewalld.service not found.
vm.overcommit_memory = 1
net.core.somaxconn = 1024
XXX
75
开启系统访问端口... 
XXX
./install.sh: line 186: firewall-cmd: command not found
./install.sh: line 187: firewall-cmd: command not found
./install.sh: line 189: firewall-cmd: command not found
./install.sh: line 191: firewall-cmd: command not found
sudo: firewall-cmd: command not found
XXX
90
安装系统系统项... 
XXX
/bin/cp: cannot stat '/home/cfd/GoCargo/fobrain/install/amd64/fobrain/fobrain/fobrain.service': No such file or directory
chmod: cannot access '/usr/lib/systemd/system/fobrain.service': No such file or directory
sed: can't read /usr/lib/systemd/system/fobrain.service: No such file or directory
unlink: cannot unlink '/data/fobrain/fobrain.service': No such file or directory
XXX
95
启动系统... 
XXX
Failed to enable unit: Unit file fobrain.service does not exist.
Failed to start fobrain.service: Unit fobrain.service not found.
等待系统启动完成....
等待系统启动完成....
等待系统启动完成....
等待系统启动完成....
等待系统启动完成....
等待系统启动完成....
等待系统启动完成....
等待系统启动完成....
等待系统启动完成....
等待系统启动完成....
等待系统启动完成....
等待系统启动完成....
等待系统启动完成....
等待系统启动完成....
等待系统启动完成....
```
## 2. 通过dockerfile构建fobrain镜像
dockerfile 的所在路径没有 ./fobrain/fobrain，无法复制

注释掉后可以顺利构建，不知道有没有影响？

这是是否是需要把项目编译成二进制再复制到容器里面运行？

但是dockerfile `RUN` 后面的命令也没有使用fobrain这个二进制文件

```sh
$ sudo docker build -t fobrain .
Dockerfile:14
--------------------
  12 |     COPY ./update/dockerUpdate /usr/sbin/dockerUpdate
  13 |     # copy fobrain
  14 | >>> COPY ./fobrain/fobrain /usr/sbin/fobrain
  15 |     COPY ./fobrain/fobrain.conf /etc/fobrain/fobrain.conf
  16 |     # copy nginx
--------------------
ERROR: failed to solve: failed to compute cache key: failed to calculate checksum of ref 5af5e7b2-de47-468f-b675-744c26b82bec::xtr2n6t6kqs8vbi869u4xn00u: "/fobrain/fobrain": not found
```
## 3.不用docker，直接运行
main.go输出了一个 fobrain，就结束了

用了README.md里面的命令也是一样无法运行
## 4.总结
可不可以把安装文档在写详细一点啊
我通过阅读docker-compose才发现还要RabbitMQ，一开始连需要哪些都不知道