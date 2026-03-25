[Java泛型 | GeekIBLi](https://geekibli.github.io/wiki/Java%E6%B3%9B%E5%9E%8B/)
**上界通配符**
```java
public static void processPages(List<? extends PageDTO> pages) {
    // 只能接受PageDTO及其子类的List
}
```
**下界通配符**
```java
public static void addPages(List<? super SpecificPageDTO> pages) {
    // 只能接受SpecificPageDTO及其父类的List
}
```
**不可以同时使用上下界**
```java
// ❌ 这种语法是错误的，Java不支持
public static <T extends PageDTO & super PageDTO> void addPages(List<T> pages) {
    // 编译错误：Syntax error on token "super"
}
```