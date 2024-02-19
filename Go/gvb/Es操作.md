## 环境搭建

拉取镜像

```Python
docker pull elasticsearch:7.12.0
```



创建docker容器挂在的目录：

```Python
mkdir -p D:\goland/gvb_master/gvb_server/elasticsearch/config & mkdir -p D:\goland/gvb_master/gvb_server/elasticsearch/data & mkdir -p D:\goland/gvb_master/gvb_server/elasticsearch/plugins

chmod 777 D:\goland/gvb_master/gvb_server/elasticsearch/data

```

配置文件

```Python
echo "http.host: 0.0.0.0" >> /opt/elasticsearch/config/elasticsearch.yml
```



创建容器

```Python
# linux
docker run --name es -p 9200:9200  -p 9300:9300 -e "discovery.type=single-node" -e ES_JAVA_OPTS="-Xms84m -Xmx512m" -v /opt/elasticsearch/config/elasticsearch.yml:/usr/share/elasticsearch/config/elasticsearch.yml -v /opt/elasticsearch/data:/usr/share/elasticsearch/data -v /opt/elasticsearch/plugins:/usr/share/elasticsearch/plugins -d elasticsearch:7.12.0

# windows
docker run --name es -p 9200:9200  -p 9300:9300 -e "discovery.type=single-node" -e ES_JAVA_OPTS="-Xms84m -Xmx512m" -v G:\\IT\\docker_container\\elasticsearch\\config\\elasticsearch.yml:/usr/share/elasticsearch/config/elasticsearch.yml -v G:\\IT\\docker_container\\elasticsearch\\data:/usr/share/elasticsearch/data -v G:\\IT\docker_container\\elasticsearch\\plugins:/usr/share/elasticsearch/plugins -d elasticsearch:7.12.0

```