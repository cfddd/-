配置数据库信息
就是把数据库文件信息以sql文件导出，然后进入mysql容器里面再导入

数据卷怎么上传？不会(看了官网的文档，使用ORS上传到dockerHub上，然后再拉下来，然后替换就可以了，感觉也没有很方便……)！

至于为什么不写进启动的命令里，是因为每次启动都会调用这些命令行，所以只在第一次部署的时候注入

> 导出的命令是`mysqldump -u root -p -P 13306 gva > D:\goland\gin-vue-admin\dump.sql `
>
> 该命令是用于从一个 SQL 文件中恢复一个数据库的。但是，如果您的 SQL 文件是用 Windows PowerShell 和 mysqldump 命令创建的，可能会出现编码问题。因为 PowerShell 的默认编码是 UTF-16，而 MySQL 不支持这种编码1。这可能导致您的 SQL 文件中出现一些不可识别的字符，从而引发错误。
>
> 解决这个问题的方法之一是，使用 --result-file 选项来生成 ASCII 格式的输出文件1。例如：
>
> `mysqldump -u root -p -P 13306 gva --result-file=D:\goland\gin-vue-admin\dump.sql`
> 
>-P是宿主机链接数据库的端口
>
> 然后，您可以用这个文件来恢复数据库：

下面是把sql文件导入数据库的命令

```
docker stop gva-server 
# 先关闭server容器（反正在数据库迁移后需要重启server容器）

docker cp dump.sql gva-mysql:/
# 复制文件dump.sql到gva-mysql容器里面

docker exec -it gva-mysql /bin/bash
# 进入gva-mysql容器

mysql -u root -p --binary-mode --force gva < ./dump.sql
# 导入sql文件
```
## 在docker-compose中设置mysql案例
```
  mysql:
    image: mysql:8.0.21       # 如果您是 arm64 架构：如 MacOS 的 M1，请修改镜像为 image: mysql/mysql-server:8.0.21
    container_name: gva-mysql
    command: mysqld --character-set-server=utf8mb4 --collation-server=utf8mb4_unicode_ci #设置utf8字符集
    restart: always
    ports:
      - "13306:3306"  # host物理直接映射端口为13306
    environment:
      MYSQL_ROOT_PASSWORD: '123456' # root管理员用户密码
      MYSQL_DATABASE: 'gva' # 初始化启动时要创建的数据库的名称
      MYSQL_USER: 'root'
      MYSQL_PASSWORD: '123456'
    volumes: 
      - ./initSQL:/docker-entrypoint-initdb.d/   # 把这个目录挂载到mysql中/docker-entrypoint-initdb.d/，该路径下容器启动时自动执行sql文件，用于初始化数据库。
      - mysql:/var/lib/mysql

    networks:
      network:
        ipv4_address: 177.7.0.13

```
如上图中的volumes