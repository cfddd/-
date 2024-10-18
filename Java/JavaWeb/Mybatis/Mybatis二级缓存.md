> 1. [五分钟，带你彻底掌握MyBatis的缓存工作原理-腾讯云开发者社区-腾讯云 (tencent.com)](https://cloud.tencent.com/developer/article/1766286)
> 2. [100道Java面试题之：说一说Mybatis里面的缓存机制_哔哩哔哩_bilibili](https://www.bilibili.com/video/BV1YR4y127Yt/?spm_id_from=333.337.search-card.all.click&vd_source=2d885cb62bb9393fa8a5379c72eabd82)
> 3. [深入浅出 MyBatis 的一级、二级缓存机制 - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/388720463)

## Mybatis缓存机制
为了提高查询效率，Mybatis带有缓存机制

在MyBatis中存在两种缓存，即**一级缓存**和**二级缓存**。

## 一级缓存

**作用范围**：一级缓存是在会话(SqlSession)层面实现的，只能在同一个SqlSession中，跨SqlSession是无效的。

MyBatis中一级缓存是默认开启的，不需要任何配置。

> 一级缓存什么时候会被清除？

一级缓存的清除主要有以下两个地方：

- 获取缓存之前会先进行判断用户是否配置了flushCache=true属性，如果配置了则会清除一级缓存。
- MyBatis全局配置属性localCacheScope配置为Statement时，那么完成一次查询就会清除缓存。
- 在执行commit，rollback，update方法时会清空一级缓存。
## 二级缓存 

**作用范围**：跨会话共享的，**在MyBatis中二级缓存的作用域是namespace，也就是作用范围是同一个命名空间**。

在MyBatis中为了实现二级缓存，专门用了一个装饰器来维护：CachingExecutor。

### 二级缓存的创建和使用
1. 创建一级缓存的CacheKey
2. 获取二级缓存
3. 如果没有获取到二级缓存则执行被包装的Executor对象中的query方法，此时会走一级缓存中的流程。
4. 查询到结果之后将结果进行缓存（放入二级和对应一级缓存）。

需要注意的是**在事务提交之前，并不会真正存储到二级缓存，而是先存储到一个临时属性，等事务提交之后才会真正存储到二级缓存**。

## 小结
一级缓存是SqlSession层面的，不同的缓存之间是隔离的

二级缓存是命名空间层面的，可以共享