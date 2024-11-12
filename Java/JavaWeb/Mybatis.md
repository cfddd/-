## 数据库连接池
- 数据库连接池是个容器，负责分配、管理数据库连接(Connection)
- 它允许应用程序重复使用一个现有的数据库连接，而不是再重新建立一个
- 释放空闲时间超过最大空闲时间的连接，来避免因为没有释放连接而引起的数据库连接遗漏
> 优点：**资源重用、提升相应速度、避免链接长久持有而不释放**

```xml
        <!--数据库连接池 druid-->
        <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>druid-spring-boot-starter</artifactId>
            <version>1.2.20</version>
        </dependency>
```
## lombok
当一个实体（entity）类有很多成员的时候，就会显得非常臃肿，set，get方法，重写toString，这些让一个简单的实体非常长，有什么办法简化呢？

Lombok是一个实用的]ava类库，能通过注解的形式自动生成构造器、getter/setter、equals、hashcode、tostring等方法，并可以自动化生成日志变量。

![](http://file.cfd.hhblog.top/myPicture/20240426105121.png)

```java
@Data
public class User {
    private int id;
    private String name;

    private Integer age;

//    public String getName() {
//        return name;
//    }
// ...
}
```