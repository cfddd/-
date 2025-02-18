![](http://file.cfd.hhblog.top//myPicture/20250101224228.png)
> 其中实线代表继承，虚线代表实线

- `IUserService`接口 继承 `IService`接口  
- `ServiceImpl` 有默认实现` IService`接口  
- `UserServiceImpl`类 实现` IUserService`接口，同时继承 `ServiceImpl<BaseMapper<User>,User>`

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

## 批量新增
开启rewriteBatchedStatements=true参数，可以自动凭借批处理中的sql语句，减少网络IO次数