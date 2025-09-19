## 原理
![](http://file.cfd.hhblog.top//myPicture/20250101224228.png)
> 其中实线代表继承，虚线代表实线

- `IUserService`接口 继承 `IService`接口  
- `ServiceImpl` 有默认实现` IService`接口  
- `UserServiceImpl`类 实现` IUserService`接口，同时继承 `ServiceImpl<BaseMapper<User>,User>`
## 示例
**UserServiceImpl.java**

```java
@Service
public class UserServiceImpl extends ServiceImpl<BaseMapper<User>,User> implements IUserService {
}

```

**IUserService.java**

```java
public interface IUserService extends IService<User> {  
}
```

**User.java**

```java
@TableName("user")
@Data
@AllArgsConstructor
@NoArgsConstructor
public class User {
    private Long id;
    private Long role_id;
    private String name;

    private Integer age;

    private String account;
    private String password;
    /** 用户头像 */
    private String avatar;
}
```

**测试IService功能的使用**

```java
@SpringBootTest
public class test {

    @Autowired
    IUserService iUserService;

    @Test
    public void testQuery() {
        List<User> list = iUserService.listByIds(Arrays.asList(1L,2L,3L));
    }
}
```

> 使用ctrl + shift + t 可以快捷创建单元测试

## 为什么要使用？
web项目中按职责划分三层，分别是controller、service、dao，使用mybatis-plus之后可以省略很多重复代码的编写，甚至直接用dao就可以满足业务上的需求，但是我们还是要维持项目的三层结构，以保持项目的可拓展型。所以可以用`public class UserServiceImpl extends ServiceImpl<BaseMapper<User>,User> implements IUserService`这种形式来维持。
### 如何在此基础上添加新的方法
点击`ServiceImpl`方法：
```java
public class ServiceImpl<M extends BaseMapper<T>, T> extends CrudRepository<M, T> implements IService<T> {  
  
}
```
可以发现他继承了`CrudRepository`
```java
public abstract class CrudRepository<M extends BaseMapper<T>, T> extends AbstractRepository<M, T> {  
  
    @Autowired  
    protected M baseMapper;
    // ...
    }
```
可以自动实现注入类型为M的Mapper对象，protect类型让我们可以直接访问在同一个包中的这个对象

```java
@Service  
public class UserServiceImpl extends ServiceImpl<BaseMapper<User>,User> implements IUserService {  
    public void QuickUpdate(User user) {
       baseMapper.updateValueByKey(user.getId(), user.getUserName());  
    }  
}
```
## 批量新增
开启rewriteBatchedStatements=true参数，可以自动凭借批处理中的sql语句，减少网络IO次数