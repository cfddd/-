```java
public interface MyInterface {
    void abstractMethod();
    default void defaultMethod() {
        System.out.println("This is a default method");
    }
}
```
## 为什么接口需要默认实现

**向接口添加方法是诸多问题的罪恶之源**。

一旦接口发生变化，实现这些接口的类往往也需要更新，提供新添方法的实现才能适配接口的变化。一旦发布，所有实现类都必须实现，代价极高。

> 当然，还有其他方式能够实现对API的改进，但是都不是明智的选择。
> 
> 比如，你可以为你的API创建不同的发布版本，同时维护老版本和新版本，但这是非常费时费力的，原因如下。其一，这增加了你作为类库的设计者**维护类库的复杂度**。其次，类库的用户不得不同时使用一套代码的两个版本，而这会增大内存的消耗，**延长程序的载入时间**，因为这种方式下项目使用的类文件数量更多了。

这就是引入默认方法的目的：它让类可以自动地继承接口的一个默认实现。

它让类库的设计者放心地改进应用程序接口，无需担忧对遗留代码的影响，这是因为实现更新接口的类现在会自动继承一个默认的方法实现。
## 默认方法的使用
默认方法在Java 8的API中已经大量地使用了，例如Collection接口的stream方法、List接口的sort方法。Predicate、Function以及Comparator也引入了新的默认方法，比如Predicate.and或者Function.andThen（记住，函数式接口只包含一个抽象方法，默认方法是种非抽象方法）

接下来介绍使用默认方法的两种用例：**可选方法**和**行为的多继承**

### 可选方法
在Java 8中，Iterator接口就为remove方法提供了一个默认实现，如下所示：
```
interface Iterator<T> {
	boolean hasNext();
	T next();
	default void remove() {
		throw new UnsupportedOperationException();
	}
}
```

通过这种方式，你可以减少无效的模板代码。实现Iterator接口的每一个类都不需要再声明一个空的remove方法了，因为它现在已经有一个默认的实现。

### 行为的多继承
Java的类只能继承单一的类，但是一个类可以实现多接口。下面是Java API中对ArrayList类的定义：
```java
public class ArrayList<E> extends AbstractList<E> implements List<E>, RandomAccess, Cloneable,Serializable, Iterable<E>, Collection<E> {
}
```

> **注意**：继承不应该成为你一谈到代码复用就试图倚靠的万精油。比如，从一个拥有100个方法及字段的类进行继承就不是个好主意，因为这其实会引入不必要的复杂性。你完全可以使用代理有效地规避这种窘境。
> 
> 这就是为什么有的时候我们发现有些类被刻意地声明为final类型：声明为final的类不能被其他的类继承，避免发生这样的反模式，防止核心代码的功能被污染。


## 编译器多继承的规则
1. 类中的方法优先级最高。类或父类中声明的方法的优先级高于任何声明为默认方法的优先级。
2. 如果无法依据第一条进行判断，那么子接口的优先级更高：函数签名相同时，优先选择拥有最具体实现的默认方法的接口，即如果B继承了A，那么B就比A更加具体。
3. 最后，如果还是无法判断，继承了多个接口的类必须通过显式覆盖和调用期望的方法，显式地选择使用哪一个默认方法的实现。

```java
interface A {
    public void hello() { System.out.println("Hello from A"); }
}
interface B extends A {
    default void hello() { System.out.println("Hello from B"); }
}
class C implements A,B {} // 调用 hello() → A 的方法
```
按照规则(2)，应该选择的是提供了最具体实现的默认方法的接口。由于B比A更具体，所以应该选择B的hello方法。

```java
public class D implements A{ }
public class C extends D implements B, A {
	public static void main(String... args) {
		new C().hello();
	}
}
```
依据规则(1)，类中声明的方法具有更高的优先级。D并未覆盖hello方法，可是它实现了接口A。所以它就拥有了接口A的默认方法。规则(2)说如果类或者父类没有对应的方法，那么就应该选择提供了最具体实现的接口中的方法。

由于B更加具体，所以程序会再次打印输出“Hello from B”。
### 冲突的发生
```java
public interface A {  
    void hello() {  
        System.out.println("Hello from A");  
    }  
}  
public interface B {  
    void hello() {  
        System.out.println("Hello from B");  
    }  
}  
public class C implements B, A { }
```
这时规则(2)就无法进行判断了，因为从编译器的角度看没有哪一个接口的实现更加具体，两个都差不多。A接口和B接口的hello方法都是有效的选项。

Java编译器这时就会抛出一个编译错误，因为它无法判断哪一个方法更合适：“Error: class C inherits unrelated defaults for hello()from types B and A.”
### 冲突的解决
能显式地决定你希望在C中使用哪一个方法
```java
interface A {
    default void hello() { System.out.println("Hello from A"); }
}
interface B {
    default void hello() { System.out.println("Hello from B"); }
}
class C implements A, B {
    @Override
    public void hello() {
        A.super.hello(); // 或 B.super.hello();
    }
}
```
必须**显式重写**，并指定调用哪个接口的默认方法。

### 菱形继承问题
```java
public interface A{  
    default void hello(){  
        System.out.println("Hello from A");  
    }  
}  
public interface B extends A { }  
public interface C extends A { }  
public class D implements B, C {  
    public static void main(String... args) {  
        new D().hello();  
    }  
}
```
际上只有一个方法声明可以选择。只有A声明了一个默认方法。由于这个接口是D的父接口，代码会打印输出“Hello from A”。

> 如果B中也提供了一个默认的hello方法，并且函数签名跟A中的方法也完全一致，这时会发生什么情况呢？
> 
> 根据规则(2)，编译器会选择提供了更具体实现的接口中的方法。由于B比A更加具体，所以编译器会选择B中声明的默认方法。

java中菱形继承没有c++中复杂，编译器做了很多的工作