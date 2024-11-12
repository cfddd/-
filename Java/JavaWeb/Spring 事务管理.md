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