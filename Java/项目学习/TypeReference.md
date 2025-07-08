保留泛型类型信息，以便在反序列化 JSON 字符串时能够正确地转换为复杂的泛型对象。

## 解决的问题
Java 的泛型在编译后会被擦除，例如：

```java
// 错误示例：无法正确反序列化泛型类型 
String json = "[\"apple\", \"banana\"]"; 
List<String> list = objectMapper.readValue(json, List.class); // 丢失泛型信息，得到List<Object>
```

## TypeReference 的作用
`TypeReference` 通过**匿名内部类**的方式捕获泛型信息，从而在运行时保留类型细节。

```java
import com.fasterxml.jackson.core.type.TypeReference;
import com.fasterxml.jackson.databind.ObjectMapper;

// JSON字符串
String json = "{\"name\":\"Alice\",\"age\":30}";

// 使用TypeReference指定具体类型
Map<String, Integer> map = ObjectMapper.readValue(
    json, 
    new TypeReference<Map<String, Integer>>() {} // 匿名内部类捕获泛型信息
);

// 正确获取类型参数
Integer age = map.get("age"); // 类型安全
```