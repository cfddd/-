>[文档概览 | frp (gofrp.org)](https://gofrp.org/zh-cn/docs/overview/)
>[fatedier/frp: A fast reverse proxy to help you expose a local server behind a NAT or firewall to the internet. (github.com)](https://github.com/fatedier/frp)
>[snowdreamtech/frpc - Docker Image](https://hub.docker.com/r/snowdreamtech/frpc)

frps.ini

```
[common]
bind_addr = 0.0.0.0
bind_port = 9999
dashboard_port = 9998
vhost_http_port = 10080
vhost_https_port = 10443
token = abcd12345
dashboard_user = admin11
dashboard_pwd = admin11

[cfd]
type = tcp
local_ip = 172.99.0.1
local_port = 2048
remote_port = 9997
```

docker pull ryaning/frps
docker run --restart=always --network host -d -v ~/frp/frps.ini:/etc/frp/frps.ini --name frps ryaning/frps

frpc.ini

```

[common]
server_addr = 47.109.59.237
server_port = 9999
token = abcd12345

[cfd]
type = tcp
local_ip = 172.99.0.1
local_port = 2048
remote_port = 9997
```

docker pull ryaning/frpc
docker run --restart=always --network host -d -v ~/frpc.ini:/etc/frp/frpc.ini --name frpc ryaning/frpc