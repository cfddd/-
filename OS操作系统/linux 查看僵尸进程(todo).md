**事情的起因：**
```bash
[root@iZ2vcd6bealyr76cgiidyvZ ~]# netstat -tunpl |grep 2049
tcp        0      0 0.0.0.0:2049            0.0.0.0:*               LISTEN      -                   
tcp6       0      0 :::2049                 :::*                    LISTEN      -                   
[root@iZ2vcd6bealyr76cgiidyvZ ~]# kill -
-bash: kill: : invalid signal specification
[root@iZ2vcd6bealyr76cgiidyvZ ~]# service net restart
Redirecting to /bin/systemctl restart net.service
Failed to restart net.service: Unit net.service not found.

```
上面的命令是查看 2049 端口被那个端口占用了



```
PS D:\GitCargo\frp\frps\amd64> python D:\GitCargo\frp\hitme.py
成功连接到目标主机！
连接已关闭
PS D:\GitCargo\frp\frps\amd64>
```