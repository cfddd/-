## java反射能干什么
通过反射可以获取任意一个类的所有属性和方法，还可以调用这些方法和属性
### 主要场景
- 像 Spring/Spring Boot、MyBatis 等等框架，**使用了动态代理，而动态代理的实现依赖反射**。
- Java 中的一大利器 **注解** 的实现也用到了反射。
## java为什么能支持反射
Java 支持反射主要是基于以下几个方面的设计和实现：

### 字节码的可获取性
- 在 Java 中，类的字节码信息是可以被获取和分析的。当 Java 程序被编译后，会生成对应的字节码文件（.class 文件），这些字节码文件包含了类的完整结构信息，如类的成员变量、方法、构造函数等的定义和签名。Java 虚拟机（JVM）在运行时可以通过类加载器加载这些字节码文件，并将其解析为内部的类对象表示。这种字节码的可获取性为反射提供了基础，使得程序能够在运行时动态地获取类的信息，进而实现对类的各种操作。
### 类加载机制
- Java 的类加载机制是支持反射的重要环节。类加载器负责将类的字节码加载到 JVM 中，并生成对应的类对象。在类加载过程中，JVM 会对字节码进行验证、准备等操作，同时将类的相关信息存储在类对象中，以便后续的使用。通过类加载器，Java 能够在运行时动态地加载和链接类，这使得反射可以在程序运行期间根据需要加载不同的类，并获取其信息和调用其方法，实现了对类的动态操作能力。
### 反射相关的 API
- Java 提供了一套丰富的反射 API，这些 API 位于`java.lang.reflect`包中。通过这些 API，开发人员可以在运行时获取类的各种信息，如通过`Class`类的静态方法`forName()`可以根据类的全限定名获取对应的类对象；通过`Class`类的`getFields()`、`getMethods()`、`getConstructors()`等方法可以获取类的成员变量、方法、构造函数等信息；通过`Method`类的`invoke()`方法可以动态地调用对象的方法等。这些反射 API 为开发人员提供了一种强大的工具，使得他们能够在运行时以编程的方式操作类和对象，实现各种灵活的功能。
### 面向对象的设计原则
- Java 作为一种面向对象的编程语言，其设计原则也为反射的实现提供了支持。面向对象的编程强调将数据和操作数据的方法封装在类中，通过继承和多态性实现代码的复用和扩展。反射机制与面向对象的设计原则相契合，它允许在运行时动态地获取和操作类的信息，进一步增强了 Java 面向对象编程的灵活性和扩展性。通过反射，开发人员可以在不修改原有类的基础上，根据不同的需求动态地创建对象、调用方法、访问成员变量等，实现了对面向对象编程的更高层次的控制和扩展。
### 动态性需求
- 在实际的软件开发中，常常会遇到一些需要在运行时动态地获取和操作类的情况，如框架的开发、插件化架构的实现等。反射机制为满足这些动态性需求提供了一种有效的解决方案。例如，在一个插件化的系统中，主程序可以通过反射动态地加载和调用不同的插件类，而无需在编译时就确定所有的类和方法调用。这种动态性使得 Java 应用程序能够更加灵活地适应不同的业务需求和变化，提高了软件的可扩展性和可维护性。
## java反射的优缺点
**优点**：可以让咱们的代码更加灵活、为各种框架提供开箱即用的功能提供了便利

**缺点**：让我们在运行时有了分析操作类的能力，这同样也增加了安全问题。比如可以无视泛型参数的安全检查（泛型参数的安全检查发生在编译时）。另外，反射的性能也要稍差点，不过，对于框架来说实际是影响不大的。
## 反射实战
> [Java 反射（Reflection） | 菜鸟教程](https://www.runoob.com/java/java-reflection.html)
### 1. 获取 Class 对象
每个类在 JVM 中都有一个与之相关的 Class 对象。可以通过以下方式获取 Class 对象：

**通过类字面量**
```java
Class<?> clazz = String.class;
```
**通过对象实例：**
```java
String str = "Hello";
Class<?> clazz = str.getClass();
```
**通过 Class.forName() 方法：**
```java
Class<?> clazz = Class.forName("java.lang.String");
```

### 2. 创建对象

可以使用反射动态创建对象：
```java
Class<?> clazz = Class.forName("java.lang.String");
Object obj = clazz.getDeclaredConstructor().newInstance();
```

### 3. 访问字段

可以通过反射访问和修改类的字段：
```java
Class<?> clazz = Person.class;
Field field = clazz.getDeclaredField("name");
field.setAccessible(true); // 如果字段是私有的，需要设置为可访问
Object value = field.get(personInstance); // 获取字段值
field.set(personInstance, "New Name"); // 设置字段值
```

### 4. 调用方法

可以通过反射调用类的方法：
```java
Class<?> clazz = Person.class;
Method method = clazz.getMethod("sayHello");
method.invoke(personInstance);

Method methodWithArgs = clazz.getMethod("greet", String.class);
methodWithArgs.invoke(personInstance, "World");
```
### 5. 获取构造函数

可以使用反射获取和调用构造函数：
```java
Class<?> clazz = Person.class;
Constructor<?> constructor = clazz.getConstructor(String.class, int.class);
Object obj = constructor.newInstance("John", 30);
```
### 6. 获取接口和父类

可以使用反射获取类实现的接口和父类：
```java
Class<?> clazz = Person.class;

// 获取所有接口
Class<?>[] interfaces = clazz.getInterfaces();
for (Class<?> i : interfaces) {
    System.out.println("Interface: " + i.getName());
}

// 获取父类
Class<?> superClass = clazz.getSuperclass();
System.out.println("Superclass: " + superClass.getName());
```

## 小结
Java 反射（Reflection）是一个强大的特性，它允许程序在运行时查询、访问和修改类、接口、字段和方法的信息。反射提供了一种动态地操作类的能力，这在很多框架和库中被广泛使用，例如Spring框架的依赖注入。

使用反射创建对象的步骤是：
1. 创建类对象（`类.class`，`Class.forName("全类名")`，`对象.getClass()`）
2. 使用类对象的构造器生成对象