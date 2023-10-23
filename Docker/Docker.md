[什么是Docker？看这一篇干货文章就够了！ - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/187505981)
[只要一小时，零基础入门Docker - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/23599229)
[启动 · Docker -- 从入门到实践 (docker-practice.github.io)](https://docker-practice.github.io/zh-cn/container/run.html)
[docker快速入门：20分钟学会用 docker部署服务 (zhihu.com)](https://www.zhihu.com/tardis/zm/art/385852508?source_id=1005)
## 用Docker部署自己的项目

1. **安装 Docker Desktop：** 首先确保您已经在 Windows 上安装了 Docker Desktop。您可以从 Docker 官方网站下载并安装 Docker Desktop。

2. **启动 Docker Desktop：** 打开 Docker Desktop 应用程序，确保 Docker 已成功启动。

3. **拉取 MySQL 镜像：** 打开命令行终端（如 PowerShell 或 CMD），运行以下命令来拉取 MySQL 镜像：

   ```sh
   docker pull mysql
   ```

4. **创建 MySQL 容器：** 运行以下命令来创建 MySQL 容器，并设置数据库密码。这里使用了环境变量来配置 MySQL 容器：

   ```sh
   docker run -d --name my-mysql -e MYSQL_ROOT_PASSWORD=my-secret-pw -p 3306:3306 mysql
   ```

   这会创建一个名为 `my-mysql` 的 MySQL 容器，并将容器内部的 3306 端口映射到主机的 3306 端口。
	>可能出现的问题，端口被占用（数据库mysql默认3306）
	>`docker: Error response from daemon: Ports are not available: exposing port TCP 0.0.0.0:3306 -> 0.0.0.0:0: listen tcp 0.0.0.0:3306: bind: Only one usage of each socket address (protocol/network address/port) is normally permitted.`
	>解决办法：改端口或者杀进程
1. **连接到 MySQL 容器：** 使用 MySQL 客户端连接到 MySQL 容器。您可以使用一些 MySQL 客户端工具，如 MySQL Workbench、Navicat 或命令行工具。以下是如何使用命令行连接到 MySQL 容器的示例：

   ```sh
   mysql -h 127.0.0.1 -P 3306 -u root -p
   ```

   输入之前设置的密码 `my-secret-pw`，即可连接到 MySQL 容器。

6. **设置环境变量：** 如果您的应用程序需要使用环境变量，可以在 Docker 启动容器时通过 `-e` 参数设置环境变量。例如，您可以在运行容器时设置数据库连接信息：

   ```sh
   docker run -d --name my-app -e DB_HOST=my-mysql -e DB_PORT=3306 -e DB_USER=root -e DB_PASSWORD=my-secret-pw my-app-image
   ```

   其中 `my-app-image` 是您的应用程序镜像。

请注意，以上步骤仅提供了一个基本的示例来演示如何在 Docker 中运行 MySQL 和设置环境变量。在实际项目中，您可能需要更多的配置和安全性考虑。另外，如果您需要持久化数据，可以考虑使用 Docker 卷来保存数据库数据。

最后，您可以根据您的项目需要，将应用程序代码放入 Docker 容器中，并在 Dockerfile 中设置相关的环境变量和依赖项。