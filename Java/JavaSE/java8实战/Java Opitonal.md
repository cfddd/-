`Optional` 是对 null 的“替代，不是替身”，本质是引导你**主动处理无值的场景**，并不能完全消除 null 的存在，但能显式表达、减少错误。

Optional具有如下特点：
- **返回值为 Optional，参数不用 Optional**
    - 公共方法返回 Optional，清晰表达“可能没有值”
    - 方法参数不使用 Optional，直接传 null 或不传
- **使用 orElseGet 替代 orElse**（避免不必要的实例化）
- **链式 map / flatMap 替代嵌套 if 判空**
- **避免 get() 方法使用**（相当于提前埋下 NPE）

## 使用场景实例

### 1. 避免多层 null 判断

传统写法：

```java
String name = user != null ? 
              user.getAddress() != null ?
              user.getAddress().getCity() : null : null;
```

Optional 写法：

```java
Optional.ofNullable(user)
        .map(User::getAddress)
        .map(Address::getCity)
        .orElse("defaultCity");
```

### 2. 替代返回值为 null 的方法

不推荐：

```java
public User findUser(String id) {
    return userMap.get(id); // 可能为 null
}
```

推荐：

```java
public Optional<User> findUser(String id) {
    return Optional.ofNullable(userMap.get(id));
}
```

## 常见反模式（Anti-Patterns）

|反模式|问题描述|替代方案|
|---|---|---|
|1. `Optional` 作为方法参数|与设计理念相违，调用者仍要构造 Optional|使用 null 或重载方法|
|2. 使用 `optional.get()`|违背其设计目的，会抛异常|使用 `orElse` / `orElseThrow`|
|3. `Optional` 字段作为类成员|会导致序列化问题和语义不清|使用普通字段配合注释或文档说明|
|4. `Optional.of(null)`|会抛出 NPE|使用 `Optional.ofNullable()`|
|5. 乱用 `orElse()` 传递开销大的默认值|即使不为空也会创建默认值|使用 `orElseGet()` 延迟执行|
