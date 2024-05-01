## 配置优先级
application.properties > application.yml > application.yaml

springboot 还支持**Java系统属性**和**命令行参数**的方式进行配置，**命令行参数**>**Java系统属性**
- **Java系统属性**：VMOption
- **命令行参数**：program argument

这两个属性均大于上面三个文件的配置信息

一般在将springboot打包成为jar包后，使用`java -c xxx.jar`后面再跟上**Java系统属性**和**命令行参数**即可启动

## Bean 管理
### 获取Bean对象
![](http://douyin.cfddfc.online/myPicture/20240501214828.png)

```java
@SpringBootTest
public class Get {
    @Autowired
    private ApplicationContext applicationContext;  // IOC 容器对象
    private Object testBeanController;

    @Test
    public void testGetBean(){
        testBeanController = applicationContext.getBean("testBeanController");
        System.out.println(testBeanController);
		// com.example.springbootappdemo.demos.Contorller.TestBeanController@4a9b3956
        testBeanController = applicationContext.getBean(TestBeanController.class);
        System.out.println(testBeanController);
        // com.example.springbootappdemo.demos.Contorller.TestBeanController@4a9b3956
        testBeanController = applicationContext.getBean("testBeanController",TestBeanController.class);
        System.out.println(testBeanController);
        // com.example.springbootappdemo.demos.Contorller.TestBeanController@4a9b3956
    }
}
```

可以看到这里的Bean对象从IOC容器中拿到的地址都是相同的。即，**默认是单例模式的**

### Bean 的作用域
![](http://douyin.cfddfc.online/myPicture/20240501220303.png)

可以通过`@scope`来配置作用域
```java
@Scope("prototype")
// @Lazy       // 直到第一次运行时才会初始化，然后放到IOC容器当中
@RestController
public class TestBeanController {
    TestBeanController(){
        System.out.println("Constructor run ...");
    }
    @GetMapping("/test_bean")
    public List<String> TestBean(){
        return new ArrayList<>(10);
    }
}

@SpringBootTest
public class TestBean {
    @Autowired
    private ApplicationContext applicationContext;  // IOC 容器对象
    private Object testBeanController;

    @Test
    public void testGetBean(){
        testBeanController = applicationContext.getBean("testBeanController");
        System.out.println(testBeanController);
        testBeanController = applicationContext.getBean(TestBeanController.class);
        System.out.println(testBeanController);
        testBeanController = applicationContext.getBean("testBeanController",TestBeanController.class);
        System.out.println(testBeanController);
        // com.example.springbootappdemo.demos.Contorller.TestBeanController@4a9b3956
        // com.example.springbootappdemo.demos.Contorller.TestBeanController@4a9b3956
        // com.example.springbootappdemo.demos.Contorller.TestBeanController@4a9b3956
    }

    @Test
    public void testScope(){
        for (int i = 0; i < 10; i++) {
            testBeanController = applicationContext.getBean("testBeanController");
            System.out.println(testBeanController);
        }
        // 默认自动初始化Bean
        /*
        Constructor run ...
        间隔一段时间，运行单侧内容后
        com.example.springbootappdemo.demos.Contorller.TestBeanController@4a9b3956 * 10 ...
         */
        // 使用@lazy 使用时才会初始化Bean容器
        /*
        Constructor run ...
        com.example.springbootappdemo.demos.Contorller.TestBeanController@2e45a357
        */
        // 使用@Scope("prototype")，每次获取Bean都重新初始化
        /*
        Constructor run ...
        com.example.springbootappdemo.demos.Contorller.TestBeanController@203b953c
        Constructor run ...
        com.example.springbootappdemo.demos.Contorller.TestBeanController@730bea0
        ...
         */

    }
}
```


默认singleton的bean，在容器启动时被创建，可以使用@Lazy注解来延迟初始化(延迟到第一次使用时)

prototype的bean，每一次使用该bean的时候都会创建一个新的实例。

实际开发当中，绝大部分的Bean是单例的，也就是说绝大部分Bean不需要配置scope属性。

### 第三方 Bean
如果要管理的bean对象来自于第三方(不是自定义的)，是无法用 @Component及衍生注解声明bean的，就需要用到 @Bean注解

若要管理的第三方bean对象，建议对这些bean进行集中分类配置，可以通过 @Configuration 注解声明一个配置类。

```java
@Configuration
public class ThirdBeanConfig {
    // 声明第三方Bean
    @Bean   // 将当前方法的返回值对象交给IOC容器管理，成为IOC容器bean
            // 通过@Bean注解的name/value属性指定bean名称，如果未指定，默认是方法名
    public SAXReader saxReader(GetData getData){// 自动根据参数类型，完成第三方Bean所需的依赖注入
        System.out.println(getData);    // com.example.springbootappdemo.service.impl.GetData@7698b7a4
        return new SAXReader();
    }
}

```

```java
    @Autowired
    private SAXReader saxReader;
    @Test
    public void testThirdBean() throws DocumentException {
        Document document = saxReader.read(this.getClass().getClassLoader().getResource("data/cfd.xml"));
        Element rootElement = document.getRootElement();
        String name = rootElement.element("name").getText();
        String age = rootElement.element("age").getText();

        System.out.println("name : " + name);
        System.out.println("age : " + age);
    }
    @Test
    public void getThirdBean(){
        Object saxReader = applicationContext.getBean("saxReader");
        System.out.println(saxReader);  // org.dom4j.io.SAXReader@52f9e8bb
    }
```

> **注意**
> 
> 通过@Bean注解的name或value属性可以声明bean的名称，如果不指定，默认bean的名称就是方法名。
> 
> 如果第三方bean需要依赖其它bean对象，直接在bean定义方法中设置形参即可，容器会根据类型自动装配。