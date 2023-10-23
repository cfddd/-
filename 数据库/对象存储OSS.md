## 对象存储OSS（Object Storage Service）
**特点**
- **容量无限大**：可以到 EB 级，多少数据都能存的下
- **持久可靠**：11个 9 甚至以上的可靠性，数据丢失的概率比中五百万的概率还要低 2-3 个量级
- **低成本**：1 部高清电影存 1 年，差不多也就几块钱人民币
- **使用方便**：支持 REST 接口，主要操作为 PUT/GET/DELETE等，使用非常简单。

**应用场景：**
日志、文本、音频、视频、图片、安装包、备份、前端js文件等

## 对象存储使用
对象（Object）是OSS存储数据的基本单元，也被称为OSS的文件。

和传统的文件系统不同，Object没有文件目录层级结构的关系。

1. 创建Bucket
2. 开发业务逻辑
3. 测试
### restful接口
对象存储对外提供的一般都是Restful风格的接口。

和gin类似
![[Pasted image 20230818215848.png]]
### MultiUpload接口
解决大文件的问题
![[Pasted image 20230818220002.png]]
### Listprefix接口

查看bucket里面的对象
![[Pasted image 20230818220023.png]]

### 关于运行代码

刚再阿里云上申请完，但是控制台里显示"数据还在准备中"

官网里面的文档非常详细了，不需要做任何补充

### bucket创建就可以使用
[OSS管理控制台 (aliyun.com)](https://oss.console.aliyun.com/bucket/oss-cn-beijing/cfddfc/overview)
在这里可以看到**endpoint**
当然是选择外网访问的第一个啦
![[Pasted image 20230819202350.png]]
在头像触摸可以出来accessKey管理，新建一个就可以使用。

分别是acessKey和acessSecretKey，在之后登录会用到

还有STS访客临时登录等等，文档里面很详细

---
### 在goland里面用
发现完全没必要自己写了啊，找到了阿里云的 [OSS SDK for Go](https://github.com/aliyun/aliyun-oss-go-sdk)

- 执行命令`go get github.com/aliyun/aliyun-oss-go-sdk/oss`获取远程代码包。
- 在您的代码中使用`import "github.com/aliyun/aliyun-oss-go-sdk/oss"`引入OSS Go SDK的包。

这样就应该可以在大项目里面用了

---
[科普贴：什么是对象存储 - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/426079235)
[OSS阿里云免费试用 - 阿里云 (aliyun.com)](https://free.aliyun.com/?crowd=personal&utm_content=m_1000371090)
[什么是对象存储OSS_对象存储 OSS-阿里云帮助中心 (aliyun.com)](https://help.aliyun.com/zh/oss/product-overview/what-is-oss)
[网站前端部署最佳实践-对象存储加CDN实现低成本支撑大规模用户_哔哩哔哩_bilibili](https://www.bilibili.com/video/BV1Jg4y1F7kb/?spm_id_from=333.337.search-card.all.click&vd_source=2d885cb62bb9393fa8a5379c72eabd82)

```go
package main

import (
"fmt"
"github.com/aliyun/aliyun-oss-go-sdk/oss"
)

func main() {
	client, err := oss.New("Endpoint", "AccessKeyId", "AccessKeySecret")
	if err != nil {
		// HandleError(err)  
	}

	lsRes, err := client.ListBuckets()
	if err != nil {
		// HandleError(err)  
	}

	for _, bucket := range lsRes.Buckets {
		fmt.Println("Buckets:", bucket.Name)
	}

	bucket, err := client.Bucket("my-bucket")
	if err != nil {
		// HandleError(err)  
	}

	lsRest, err := bucket.ListObjects()
	if err != nil {
		// HandleError(err)  
	}

	for _, object := range lsRest.Objects {
		fmt.Println("Objects:", object.Key)
	}
}
```