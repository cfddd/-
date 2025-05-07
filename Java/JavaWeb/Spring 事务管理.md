## 概念
一组操作的集合，要么一起成功，要么一起失败

## @Transactional
- 位置:业务(service)层的方法上、类上、接口上
- 作用:将当前方法交给spring进行事务管理
	- 方法执行前，开启事务;
	- 成功执行完毕，提交事务;
	- 出现异常，回滚事务

> **注意：** 默认情况下，只有出现 RuntimeException 才回滚异常。rollbackFor属性用于控制出现何种异常类型，回滚事务。
> 
> 使用@Transactional(rollbackFor =Exception.class)可以让所有异常类型都回滚 

## 传播行为
事务传播行为:指的就是当一个事务方法被另一个事务方法调用时，这个事务方法应该如何进行事务控制。

举个例子，TransactionA开启后，调用了TransactionB方法，那么入股如果TransactionB回滚了，TransactionA该做什么呢？

![](http://file.cfd.hhblog.top/myPicture/20240426154642.png)

```java
@Transactional
public void A(){
	try{
		int i = 1 / 0 //报错，必然会滚
	}
	finally{
		userservice.B(); // 仍然会运行
	}
}

@Transactional(proganation = Proganation.REQUIRED)
public void B(){

}
```
上面的B方法会加入到A的事务中，在默认值的情况下，因为是同一个事务，所以**B也会回滚**

更换为`Proganation.REQUIRES_NEW` 创建一个新的事务即可

## 内部方法调用-事务失效
> https://blog.csdn.net/u012102536/article/details/121995272

```java
public void autoUpdate() {  
        // 直接内部调用，会导致事务失效！
        executeUpdate(idList);  
}  

// 事务失效
@Transactional(rollbackFor = Throwable.class)
public void executeUpdate(List<Integer> idList){}
```
当 `autoUpdate` 方法调用类内 `executeUpdate` 方法时，会发现事务失效，因为`@Transactional`实际上是使用了Spring AOP，而AOP是通过代理Bean对象才实现的，而类内方法的调用会导致Bean方法看不到这个雷内方法，导致事务失效！
### 解决方案
方案一（使用代理调用）：
```java
// 在类中添加代理对象注入
@Autowired
private Service service; // 当前类的自身引用

public void autoUpdate() {
    // ...其他代码
    // 直接调用改为通过代理对象调用
    service.executeUpdate(paperList);
    // ...其他代码
}
```
方案二（方法移至独立类）：
```java
// 新建事务服务类
@Component
public class TransactionalService {
    @Transactional(rollbackFor = Throwable.class)
    public void executeUpdate(List<Integer> idList) {
        // 原executeUpdatePapersStatus方法代码
    }
}

// 原代码修改：
@Autowired
private TransactionalService transactionalService;

public void autoUpdate() {
    // ...其他代码
    transactionalService.executeUpdate(idList);
    // ...其他代码
}
```
方案三（使用手动触发事务）：
```java
@Autowired
private PlatformTransactionManager transactionManager;

public void autoUpdatePapersStatusOfDimission() {
    // ...其他代码
    new TransactionTemplate(transactionManager).execute(status -> {
        executeUpdate(idList);
        return null;
    });
}
```