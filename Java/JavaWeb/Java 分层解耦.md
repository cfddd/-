> 感觉实践过一遍就可以，Java和go的项目可能有细微的区别

## 三层架构
- controller：控制层，**接收前端发送的请求，对请求进行处理，并响应数据**
- service：业务逻辑层，**处理具体的业务逻辑**
- dao：数据访问层(Data Access Object)(持久层)，**负责数据访问操作，包括数据的增、删、改、查**

**controller**
```java
@RestController
public class TestEmpController {
    private EmpService empService = new GetData();
    @GetMapping("/test_emp")
    public List<String> TestEmp(){
        return empService.getData();
    }
}
```
**service**
```java
public class GetData implements EmpService {
    private EmpDao empDao = new GenData();

    @Override
    public List<String> getData() {
        return empDao.genData();
    }
}

```
**dao**
```java
public class GenData implements EmpDao {
    @Override
    public List<String> genData() {
        List<String> res = new ArrayList<>();
        for (int i = 0; i < 10; i++) {
            res.add((new Integer(i).toString()));
        }
        return res;
    }
}
```
## 分层解耦
- 内聚:软件中各个功能模块内部的功能联系。
- 耦合:衡量软件中各个层/模块之间的依赖、关联的程度。

> 软件设计原则：**高内聚，低耦合**

简单地说，如果A调用B，就是A和B耦合在了一起。如果我们修改了B，并且A不能再调用B，则需要修改A的代码，对于系统来说是蛮烦的事情。**分层解耦**就是为了解决这个问题提出的。

- **控制反转**
	控制反转: Inversion Of Control，简称IOC。对象的创建控制权由程序自身转移到外部(容器)，这种思想称为控制反转。

- **依赖注入**
	DependencyInjection，简称DI。容器为应用程序提供运行时，所依赖的资源，称之为依赖注入。
- **Bean对象**
	IOC容器中创建、管理的对象，称之为bean。

简单来说就是，**通过外部容器，自动管理/调用类和方法**

## IOC 详解
①.Service层及 Dao层的实现类，交给IOC容器管理。
```java
@Component // 将当前类交给IOC容器管理，成为IOC容器中的Bean，实现控制反转
```
②.为Controller及Service注入运行时，依赖的对象。
```java
@Autowired //运行时IOC容器会提供该类型的bean对象，并赋值给该对象，实现依赖注入
```
③.完整代码

**controller**
```java
@RestController
public class TestEmpController {
    @Autowired //运行时IOC容器会提供该类型的bean对象，并赋值给该对象，实现依赖注入
    private EmpService empService;
    @GetMapping("/test_emp")
    public List<String> TestEmp(){
        return empService.getData();
    }
}
```
**service**
```java
@Component // 将当前类交给IOC容器管理，成为IOC容器中的Bean，实现控制反转
public class GetData implements EmpService {
    @Autowired //运行时IOC容器会提供该类型的bean对象，并赋值给该对象，实现依赖注入
    private EmpDao empDao;

    @Override
    public List<String> getData() {
        return empDao.genData();
    }
}
```
**dao**
```java
@Component // 将当前类交给IOC容器管理，成为IOC容器中的Bean，实现控制反转
public class GetData implements EmpService {
    @Autowired //运行时IOC容器会提供该类型的bean对象，并赋值给该对象，实现依赖注入
    private EmpDao empDao;

    @Override
    public List<String> getData() {
        return empDao.genData();
    }
}

```

> 如果有同样的实现类，会怎么处理

```java
@Component // 将当前类交给IOC容器管理，成为IOC容器中的Bean，实现控制反转
public class GetDataInstead implements EmpService {
    @Autowired //运行时IOC容器会提供该类型的bean对象，并赋值给该对象，实现依赖注入
    private EmpDao empDao;
    @Override
    public List<String> getData() {
        return empDao.genData();
    }
}
/* 报错
Could not autowire. There is more than one bean of 'EmpService' type.
*/
```
 代码和前面的 service里一摸一样，如果有两个相同的实现，spring启动时就会检测出问题报错，阻止启动。

可以选择性的删除，或者**注释掉不想使用的代码的`@Component`注解**，这样我们就实现了代码的解耦和自动装配！
### IOC - Bean的声明

要把某个对象交给IOC容器管理，需要在对应的类上加上如下注解之一!

| 注解        | 说明               | 位置 |
| ----------- | ------------------ | ---- |
| @Component  | 声明bean的基础注解 |    不属于以下三类时，用此注解  |
| @Controller |         @Component的行生注解           |   标注在控制器类上   |
| @Service    |           @Component的衍生注解         |     标注在业务类上 |
| @Repository |         @Component的衍生注解           |     标注在数据访问类上(由于与mybatis整合，用的少) |
每一个bean都有一个**标识名字**，默认是类名，也可以通过Value给其命名（默认开头字母小写）

**service**
```java
@Service//("name") 
public class GetData implements EmpService {
}
```
**dao**
```java
@Repository
public class GetData implements EmpService {
}
```
>**注意：**
>
>声明bean的时候，可以通过value属性指定bean的名字，如果没有指定，默认为类名首字母小写。
>
>使用以上四个注解都可以声明bean，但是在sprinaboot集成web开发中，声明控制器bean只能用@Controller.

### Bean 组件扫描
- 前面声明bean的四大注解，要想生效，还需要被组件扫描注解@ComponentScan扫描。
- @ComponentScan注解虽然没有显式配置，但是实际上已经包含在了启动类声明注解 @SpringBootApplication中，默认扫描的范围是启动类所在包及其子包。
- 可以通过`@ComponentScan({"需要添加扫描的包名……"，"默认扫描包"})`
## DI 依赖注入
在前面提到使用 `@Autowired` 就可以自动注入数据，但是当多个相同的Bean，就会报错的问题，如何解决呢？

### `@Primary`

让当前的类生效（加载`@Service`所在的类之前）
```java
@Primary
@Service
public class GetData implements EmpService {
}
```
### `@Qualifier`：
```java
@RestController
public class TestEmpController {
    @Qualifier("getData")
    @Autowired
    private EmpService empService;
}
```
`@Qualifier`注解中使用value选取使用哪一个Bean

注意开头字母小写（驼峰）
### `@Resource`
```java
@RestController
public class TestEmpController {
    @Resource(name = "getData")
    private EmpService empService;
}
```
同理，使用名字选取

> @Resource与@Autowired区别
> @Autowired 是spring框架提供的注解，而@Resource是IDK提供的注解
> @Autowired 默认是按照类型注入，而@Resource默认是按照名称注入