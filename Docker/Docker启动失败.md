## 问题描述
docker启动失败，经过多次重启还是失败
```sh
# sudo systemctl start docker
Job for docker.service failed because the control process exited with error code.
See "systemctl status docker.service" and "journalctl -xe" for details.
```
## 问题排查
```sh
# sudo systemctl status docker.service
docker.service - Docker Application Container Engine
   Loaded: loaded (/usr/lib/systemd/system/docker.service; enabled; vendor preset: enabled)
   Active: failed (Result: exit-code) since Fri 2024-11-15 13:59:19 CST; 35s ago
     Docs: https://docs.docker.com
  Process: 59077 ExecStart=/usr/bin/dockerd -H unix:// --containerd=/run/containerd/containerd.sock (code=exited, status=1/FAILURE)
 Main PID: 59077 (code=exited, status=1/FAILURE)

Nov 15 13:59:19 iZ2vcd6bealyr76cgiidyvZ systemd[1]: docker.service: Main process exited, code=exited, status=1/FAILURE
Nov 15 13:59:19 iZ2vcd6bealyr76cgiidyvZ systemd[1]: docker.service: Failed with result 'exit-code'.
Nov 15 13:59:19 iZ2vcd6bealyr76cgiidyvZ systemd[1]: Failed to start Docker Application Container Engine.
Nov 15 13:59:19 iZ2vcd6bealyr76cgiidyvZ systemd[1]: docker.service: Start request repeated too quickly.
Nov 15 13:59:19 iZ2vcd6bealyr76cgiidyvZ systemd[1]: docker.service: Failed with result 'exit-code'.
Nov 15 13:59:19 iZ2vcd6bealyr76cgiidyvZ systemd[1]: Failed to start Docker Application Container Engine.
Nov 15 13:59:53 iZ2vcd6bealyr76cgiidyvZ systemd[1]: docker.service: Start request repeated too quickly.
Nov 15 13:59:53 iZ2vcd6bealyr76cgiidyvZ systemd[1]: docker.service: Failed with result 'exit-code'.
Nov 15 13:59:53 iZ2vcd6bealyr76cgiidyvZ systemd[1]: Failed to start Docker Application Container Engine
```
第一个命令`systemctl status docker.service`查看不出来有什么问题

```sh
# journalctl -u docker.service
Nov 15 14:25:47 iZ2vcd6bealyr76cgiidyvZ dockerd[59496]: time="2024-11-15T14:25:47.404339042+08:00" level=info msg="Starting up"
Nov 15 14:25:47 iZ2vcd6bealyr76cgiidyvZ dockerd[59496]: time="2024-11-15T14:25:47.427240025+08:00" level=info msg="[graphdriver] trying configured driver: overlay"
Nov 15 14:25:47 iZ2vcd6bealyr76cgiidyvZ dockerd[59496]: failed to start daemon: error initializing graphdriver: [graphdriver] ERROR: the overlay storage-driver has been deprecated and removed; visit https://docs.docker.com/go/storage-driver/ for moredeprecated and removed; visit https://docs.docker.com/go/storage-driver/ for more information: overlay>

```

找到问题所在了

`ERROR: the overlay storage-driver has been deprecated and removed; visit https://docs.docker.com/go/storage-driver/ for moredeprecated and removed; visit https://docs.docker.com/go/storage-driver/ for more information: overlay`

## 原因分析
Docker 会不断进行技术升级和改进，存储驱动方面也不例外。随着时间推移，一些早期的存储驱动（如 `overlay` ）可能因为存在性能瓶颈、兼容性问题或者不符合新的架构设计等原因，被判定为不再适合继续使用，进而被弃用并从 Docker 中移除。官方推荐使用更先进、更稳定的存储驱动来替代它们，以保障 Docker 容器运行的高效性和可靠性。

`overlay2` 是 `overlay` 的升级版，它在功能和性能上有诸多改进。例如，在处理多层容器镜像时效率更高，对文件系统的利用更加合理，能更好地支持大规模容器部署等场景。它已经成为目前比较主流的 Docker 存储驱动选择之一。

## 操作步骤
**编辑配置文件**：使用文本编辑器（如 `vim` 或 `nano` ）打开 Docker 的配置文件 `/etc/docker/daemon.json` ，找到 `"storage-driver": "overlay"` 这一行（如果存在的话），将其修改为 `"storage-driver": "overlay2"` ，示例修改后的配置文件片段如下：

```json
{
    "registry-mirrors": [
       "https://do.nark.eu.org",
        "https://dc.j8.work",
        "https://docker.m.daocloud.io",
        "https://dockerproxy.com",
        "https://docker.mirrors.ustc.edu.cn",
        "https://docker.nju.edu.cn"
    ],
    "insecure-registries": [
        "docker-registry.zjq.com"
    ],
    "log-driver": "json-file",
    "log-opts": {
        "max-size": "10m",
        "max-file": "10"
    },
    "data-root": "/data/docker",
    "storage-driver": "overlay2"
}
```

保存对配置文件的修改后，需要执行以下两个命令来让新的配置应用到 Docker 服务中。首先执行 `systemctl daemon-reload` 命令，这个命令的作用是让 `systemd` 守护进程重新加载配置信息，使其知晓 Docker 服务配置发生了变化；然后再执行 `systemctl restart docker` 命令来重启 Docker 服务，这样 Docker 守护进程就会按照新配置的存储驱动（即 `overlay2` ）来启动运行了。