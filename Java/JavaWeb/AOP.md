## AOP 是什么？

AOP:Aspect Oriented Programming (面向切面编程、面向方面编程)，其实就是面向特定方法编程。
### 静态代理和动态代理
> 参考自[深入理解Java动态代理 - 一起学Java的文章 - 知乎](https://zhuanlan.zhihu.com/p/347141071)

**静态代理代码**

[静态代理](https://github.com/cfddd/SpringBootAppDemo/blob/master/src/test/java/com/example/springbootappdemo/staticProxyModels/StaticProxyModelTests.java)


**动态代理参考代码**

[动态代理](https://github.com/cfddd/SpringBootAppDemo/blob/master/src/test/java/com/example/springbootappdemo/dynamicProxyModels/DynamicProxyModelTests.java)

## Spring AOP 快速入门
**添加依赖**
```xml
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-aop</artifactId>
        </dependency>
```
**service调用耗时**
```java
@Slf4j
@Component
@Aspect
public class TimeAspect {
    // 4.@Around决定那些方法使用下面的代理
    @Around("execution(* com.example.springbootappdemo.service.*.*(..))")   // 切入点表达式
    public Object recordTime(ProceedingJoinPoint joinPoint) throws Throwable {
        // 1.记录开始时间
        long begin = System.currentTimeMillis();

        // 2.调用原始方法运行
        Object result = joinPoint.proceed();

        // 3.计算结束时间，计算耗时操作
        long end = System.currentTimeMillis();
        log.info(joinPoint.getSignature() + " used " + (end - begin) + " ms");

        return result;
    }
}
```

简单地说，这里的`joinPoint`就是即将被运行的代理对象。在运行前被 `recordTime` 方法代理，直到运行`proceed`方法就会运行原本的方法。

> AOP的使用场景主要是**记录操作日志、权限控制、事务管理**等

## 详细介绍
| 名称            | 介绍                                            |
| ------------- | --------------------------------------------- |
| **JoinPoint** | **连接点**，可以被AOP控制的方法（方法的执行信息）                  |
| **Around**    | **通知**，指那些重复的逻辑，共性方法（封装为一个方法）                 |
| **PointCut**  | **切入点**，匹配**连接点**并运行，**通知**仅会在**切入点方法执行时**被使用 |
| **Aspect**    | **切面**，描述**通知和切入点**的对应关系                      |
| **Target**    | **目标对象**，**通知**所应用到的对象                        |
### 通知
1. `@Around`:环绕通知，此注解标注的通知方法在目标方法**前、后**都被执行
2. `@Before`:前置通知，此注解标注的通知方法在目标方法**前**被执行
3. `@After` :后置通知，此注解标注的通知方法在目标方法**后**被执行，无论是否有异常都会执行
4. `@AfterReturning `:返回后通知，此注解标注的通知方法在目标方法后被执行，有异常不会执行
5. `@AfterThrowing `:异常后通知，此注解标注的通知方法**发生异常后执行**

```java
@Slf4j
@Component
@Aspect
public class TimeAspect {
    // 4.@Around决定那些方法使用下面的代理
    @Around("execution(* com.example.springbootappdemo.service.*.*(..))")   // 切入点表达式
    public Object recordTime(ProceedingJoinPoint joinPoint) throws Throwable {
        // 1.记录开始时间
        long begin = System.currentTimeMillis();

        // 2.调用原始方法运行
        Object result = joinPoint.proceed();

        // 3.计算结束时间，计算耗时操作
        long end = System.currentTimeMillis();
        log.info(joinPoint.getSignature() + " used " + (end - begin) + " ms");

        return result;
    }
    @Before("execution(* com.example.springbootappdemo.service.*.*(..))")
    public void Before(){
        log.info("before ...");
    }
    @After("execution(* com.example.springbootappdemo.service.*.*(..))")
    public void After(){
        log.info("After ...");
    }
    // 正常运行返回后才会运行（抛出异常不会运行）
    @AfterReturning("execution(* com.example.springbootappdemo.service.*.*(..))")
    public void AfterReturning(){
        log.info("AfterReturning ...");
    }
    @AfterThrowing("execution(* com.example.springbootappdemo.service.*.*(..))")
    public void AfterThrowing(){
        log.info("exception aroused");
    }
}
```

**切入点表达式抽取**
```java
@Slf4j
@Component
@Aspect
public class TimeAspect {
    @Pointcut("execution(* com.example.springbootappdemo.service.*.*(..))")
    private void pc(){}
    @Before("pc()")
    public void Before(){
        log.info("before ...");
    }
    @After("pc()")
    public void After(){
        log.info("After ...");
    }
}
```
引入其他包里面的切入点表达式也是可以的（需要是public方法）
### 通知顺序
当一个方法有多个切面，执行的顺序是什么样的？

答：按照**类名字符串**的排序运行的，先进先出
- **before正序运行**
- **after逆序运行**


那如何自定义排序规则？

答：用 `@Order(num)`加在切面类上
- **before数字小的先运行**
- **after数字大的先运行**
```java
@Slf4j
@Component
@Aspect
@Order(1)
public class TimeAspect {
	//...
}

```
### 切入点表达式
#### execution
1. `execution (...)`: 根据方法的签名来匹配
2. `@annotation(...)`: 根据注解匹配

execution主要根据方法的返回值、包名、类名、方法名、方法参数等信息来匹配，语法为:
```java
execution(访问修饰符?返回值︰包名.类名.?方法名(方法参数) throws异常?)
```

`?`代表可以省略，比如异常、访问修饰符（public、private）
```java
@Slf4j
@Component
@Aspect
public class CutPoint {
    @Pointcut("execution(public String com.example.springbootappdemo.demos.Contorller.FileUploadController." +
            "upload(java.lang.String," +
            "org.springframework.web.multipart," +
            "javax.servlet.http)" +
            "throws java.io.IOException)")
    public void pc(){}
}
```

`*`：单个独立的任意符号

`..`：多个连续的任意符号，可以通配任意级的包，任意数量的参数

可以使用 `||` 来连接多个切入点表达式（还有`&&、!`）

> **最佳实践**
> 
> **所有业务方法名在命名时尽量规范**，方便切入点表达式快速匹配。如:查询类方法都是 find 开头，更新类方法都是 update开头。
> 
> 描述切入点方法通常**基于接口描述**，而不是直接描述实现类，**增强拓展性**。
> 
> 在满足业务需要的前提下，**尽量缩小切入点的匹配范围**。如:包名匹配尽量不使用…，使用*匹配单个包。

#### `@annotation`
标识有特定注解的的方法

根据指定的注解，匹配切入点表达式
```java
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface MyLog {
}
```

```java
    // 使用了@MyLog注解的方法会自动执行下面的函数
    @Pointcut("@annotation(com.example.springbootappdemo.aop.MyLog)")
    public void pd(){}

    @Before("pd()")
    public void Before(){
        log.info("before ...");
    }
    @After("pd()")
    public void After(){
        log.info("After ...");
    }
```
### 连接点
在Spring中用JoinPoint抽象了连接点，用它可以获得方法执行时的相关信息，如目标类名、方法名、方法参数等。

对于@Around通知，获取连接点信息只能使用ProceedingJoinPoint

对于其他四种通知，获取连接点信息只能使用JoinPoint ，它是 ProceedingJoinPoint 的父类型

```java
@Slf4j
@Component
@Aspect
public class JoinPoints {
    @Pointcut("execution(* com.example.springbootappdemo.service.*.*(..))")
    private void pc(){}
    @Around("pc()")   // 切入点表达式
    public Object recordTime(ProceedingJoinPoint joinPoint) throws Throwable {


        String className = joinPoint.getTarget().getClass().getName();
        log.info("className " + className);

        String methodName = joinPoint.getSignature().getName();
        log.info("methodName " + methodName);

        Object[] args = joinPoint.getArgs();
        log.info("形参 " + Arrays.toString(args));

        Object result = joinPoint.proceed();
        log.info("returning" + result.toString());

        return result;
    }
    @Before("pc()")
    public void Before(JoinPoint joinPoint)
    {
        log.info("before ...");
    }
}

```
## MyLog
```java
// 自定义Log操作日支
@Retention(RetentionPolicy.RUNTIME) // 运行时
@Target(ElementType.METHOD)         // 方法
public @interface MyLog {
}



@Slf4j
@Component
@Aspect // 使用时加上@MyLog就可以
public class MyLog {
    @Autowired
    private LogMapper logMapper;
    @Around("@annotation(com.example.springbootappdemo.anno.MyLog)") // MyLog注解所在位置
    public Object record(ProceedingJoinPoint proceedingJoinPoint) throws Throwable {

//        String jwt = request.getHeader("token");
//        Claims claims = JwtUtils.parseJWT(jwt);
//        Integer operatorId = (Integer) claims.get("id");
        Integer operatorId = (Integer) 1;
        LocalDateTime localDateTime = LocalDateTime.now();
        Long begin = System.currentTimeMillis();
        String className = proceedingJoinPoint.getTarget().getClass().getName();
        String methodName = proceedingJoinPoint.getSignature().getName();
        String args = Arrays.toString(proceedingJoinPoint.getArgs());

        Object result = proceedingJoinPoint.proceed();
        Long cost = System.currentTimeMillis() - begin;
        String returnValue = JSONObject.toJSONString(result);

        Log OPlog = new Log(
                null,
                operatorId,
                localDateTime,
                className,
                methodName,
                args,
                returnValue,
                cost);
        logMapper.create(OPlog);
        log.info("操作 ： " + OPlog);
        return result;
    }
}
```