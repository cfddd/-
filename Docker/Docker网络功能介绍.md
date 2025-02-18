[‍​⁠​⁠‬﻿​⁠⁠‍‍​​​⁠﻿​​​​​​‌‌‍‍﻿​⁠‍​​‍​‬​‍​​‌​﻿⁠​​​​⁠‍day02-Docker - 飞书云文档](https://b11et3un53m.feishu.cn/wiki/MWQIw4Zvhil0I5ktPHwcoqZdnec)
## 普通容器网络
首先，我们查看下MySQL容器的详细信息，重点关注其中的网络IP地址：

```Bash
# 1.用基本命令，寻找Networks.bridge.IPAddress属性
docker inspect mysql
# 也可以使用format过滤结果
docker inspect --format='{{range .NetworkSettings.Networks}}{{println .IPAddress}}{{end}}' mysql
# 得到IP地址如下：
172.17.0.2

# 2.然后通过命令进入dd容器
docker exec -it dd bash

# 3.在容器内，通过ping命令测试网络
ping 172.17.0.2
# 结果
PING 172.17.0.2 (172.17.0.2) 56(84) bytes of data.
64 bytes from 172.17.0.2: icmp_seq=1 ttl=64 time=0.053 ms
64 bytes from 172.17.0.2: icmp_seq=2 ttl=64 time=0.059 ms
64 bytes from 172.17.0.2: icmp_seq=3 ttl=64 time=0.058 ms
```

发现可以互联，没有问题。


但是，容器的网络IP其实是一个虚拟的IP，其值并不固定与某一个容器绑定，如果我们在开发时写死某个IP，而在部署时很可能MySQL容器的IP会发生变化，连接会失败。

所以，我们必须借助于docker的网络功能来解决这个问题，官方文档：

[https://docs.docker.com/engine/reference/commandline/network/](https://docs.docker.com/engine/reference/commandline/network/)

常见命令有：

|**命令**|**说明**|**文档地址**|
|---|---|---|
|docker network create|创建一个网络|[docker network create](https://docs.docker.com/engine/reference/commandline/network_create/)|
|docker network ls|查看所有网络|[docs.docker.com](https://docs.docker.com/engine/reference/commandline/network_ls/)|
|docker network rm|删除指定网络|[docs.docker.com](https://docs.docker.com/engine/reference/commandline/network_rm/)|
|docker network prune|清除未使用的网络|[docs.docker.com](https://docs.docker.com/engine/reference/commandline/network_prune/)|
|docker network connect|使指定容器连接加入某网络|[docs.docker.com](https://docs.docker.com/engine/reference/commandline/network_connect/)|
|docker network disconnect|使指定容器连接离开某网络|[docker network disconnect](https://docs.docker.com/engine/reference/commandline/network_disconnect/)|
|docker network inspect|查看网络详细信息|[docker network inspect](https://docs.docker.com/engine/reference/commandline/network_inspect/)|
## 自定义网络

```Bash
# 1.首先通过命令创建一个网络
docker network create hmall

# 2.然后查看网络
docker network ls
# 结果：
NETWORK ID     NAME      DRIVER    SCOPE
639bc44d0a87   bridge    bridge    local
403f16ec62a2   hmall     bridge    local
0dc0f72a0fbb   host      host      local
cd8d3e8df47b   none      null      local
# 其中，除了hmall以外，其它都是默认的网络

# 3.让dd和mysql都加入该网络，注意，在加入网络时可以通过--alias给容器起别名
# 这样该网络内的其它容器可以用别名互相访问！
# 3.1.mysql容器，指定别名为db，另外每一个容器都有一个别名是容器名
docker network connect hmall mysql --alias db
# 3.2.db容器，也就是我们的java项目
docker network connect hmall dd

# 4.进入dd容器，尝试利用别名访问db
# 4.1.进入容器
docker exec -it dd bash
# 4.2.用db别名访问
ping db
# 结果
PING db (172.18.0.2) 56(84) bytes of data.
64 bytes from mysql.hmall (172.18.0.2): icmp_seq=1 ttl=64 time=0.070 ms
64 bytes from mysql.hmall (172.18.0.2): icmp_seq=2 ttl=64 time=0.056 ms
# 4.3.用容器名访问
ping mysql
# 结果：
PING mysql (172.18.0.2) 56(84) bytes of data.
64 bytes from mysql.hmall (172.18.0.2): icmp_seq=1 ttl=64 time=0.044 ms
64 bytes from mysql.hmall (172.18.0.2): icmp_seq=2 ttl=64 time=0.054 ms
```
OK，现在无需记住IP地址也可以实现容器互联了。

## 总结

- 在自定义网络中，可以给容器起多个别名，默认的别名是容器名本身
    
- 在同一个自定义网络中的容器，可以通过别名互相访问