## 封装
- **定义**：
	- 封装是指将对象的状态信息（属性）和行为（方法）隐藏在对象内部，只通过定义好的接口（公共方法）来访问和操作这些信息。简单来说，就是把对象的具体实现细节包裹起来，不让外部直接访问，就像把东西放在一个盒子里，只通过特定的开口来拿取。
- **示例**：
	- 以一个简单的银行账户类为例。账户有余额（balance）这个属性，还有存款（deposit）和取款（withdraw）这两个行为。

```java
class BankAccount {
    private double balance;

    public void deposit(double amount) {
        if (amount > 0) {
            balance += amount;
        }
    }

    public void withdraw(double amount) {
        if (amount > 0 && balance >= amount) {
            balance -= amount;
        }
    }

    public double getBalance() {
        return balance;
    }
}
```
- 在这个例子中，`balance`属性被声明为`private`，这意味着它不能被外部类直接访问。外部只能通过`deposit`、`withdraw`和`getBalance`这些公共方法来间接操作余额。这样就实现了对`balance`属性的封装，保证了余额数据的安全性和一致性，避免了外部随意修改余额可能导致的问题。
- **优点**：
    - **信息隐藏**：通过封装，可以隐藏对象内部的复杂实现细节。例如，在银行账户类中，`deposit`和`withdraw`方法内部可能涉及到更复杂的业务逻辑，如记录交易日志、更新账户状态等，但这些细节对外部使用者是隐藏的，使用者只需要关心如何调用这些方法来完成存款和取款操作。
    - **数据安全性**：封装可以防止外部代码对对象的属性进行不合理的修改。比如，在银行账户类中，通过封装`balance`属性，避免了外部直接将余额修改为负数等不合理的值，保证了数据的合法性和安全性。
## **继承**
- **定义**：
	- 继承是一种创建新类（子类或派生类）的方式，新类可以继承现有类（父类或基类）的属性和方法，并且可以在此基础上添加新的属性和方法或者重写父类的方法。它体现了类与类之间的 “is - a” 关系，即子类是父类的一种特殊情况。
- **示例**：
	- 假设我们有一个动物（Animal）类作为父类，它有属性如名字（name）和行为如发出声音（makeSound）。

```java
class Animal {
    private String name;

    public Animal(String name) {
        this.name = name;
    }

    public void makeSound() {
        System.out.println("Animal makes a sound.");
    }
}
```
- 然后我们创建一个狗（Dog）类作为子类，它继承自动物类。狗类除了继承动物类的属性和方法外，还可以有自己特有的行为，如摇尾巴（wagTail）。
```java
class Dog extends Animal {
    public Dog(String name) {
        super(name);
    }

    @Override
    public void makeSound() {
        System.out.println("Dog barks.");
    }

    public void wagTail() {
        System.out.println("Dog wags its tail.");
    }
}
```
- 在这个例子中，`Dog`类通过`extends`关键字继承了`Animal`类。它继承了`Animal`类的`name`属性和`makeSound`方法，并且重写了`makeSound`方法来体现狗的叫声特点。同时，`Dog`类还添加了自己特有的`wagTail`方法。
- **优点**：
    - **代码复用**：通过继承，子类可以直接使用父类的属性和方法，避免了重复编写相同的代码。例如，在多个不同的动物子类（如猫、狗、牛等）中，不需要重新定义动物的基本属性（如名字）和一些通用的行为（如吃东西），可以直接从动物类继承这些内容。
    - **层次结构清晰**：继承可以建立起类的层次结构，使得代码的组织结构更加清晰。在面向对象的设计中，这种层次结构可以很好地体现现实世界中的对象关系，方便对复杂系统进行建模和管理。
## **多态**
- **定义**：
	- 多态是指同一个行为（方法）在不同的对象上会产生不同的效果。它主要通过方法重写（Override）和方法重载（Overload）来实现，另外还有接口实现的多态。在运行时，系统会根据对象的实际类型来决定调用哪个具体的方法。
- **示例（方法重写）**：
	- 继续以上面的动物类和狗类为例。我们可以创建一个动物数组，里面存放不同类型的动物对象。
```java
class Main {
    public static void main(String[] args) {
        Animal[] animals = new Animal[2];
        animals[0] = new Dog("Buddy");
        animals[1] = new Animal("Generic Animal");
        for (Animal animal : animals) {
            animal.makeSound();
        }
    }
}
```
- 在这个例子中，`animals`数组中存放了一个`Dog`对象和一个`Animal`对象。当遍历数组并调用`makeSound`方法时，对于`Dog`对象，会调用`Dog`类中重写后的`makeSound`方法（输出 “Dog barks.”），而对于`Animal`对象，会调用`Animal`类中的`makeSound`方法（输出 “Animal makes a sound.”）。这就是多态的体现，同样是`makeSound`方法，在不同的对象上有不同的行为。
- **示例（方法重载）**：
    - 假设有一个计算器（Calculator）类，它有一个加法（add）方法。

```java
class Calculator {
    public int add(int num1, int num2) {
        return num1 + num2;
    }

    public double add(double num1, double num2) {
        return num1 + num2;
    }
}
```

- 这个类中有两个`add`方法，它们的参数类型不同，这就是方法重载。当调用`add`方法时，编译器会根据传入的参数类型来决定调用哪个具体的`add`方法。如果传入两个整数，就调用第一个`add`方法；如果传入两个双精度浮点数，就调用第二个`add`方法。
- **优点**：
    - **提高代码的可扩展性和可维护性**：多态使得代码更加灵活，可以方便地添加新的子类而不需要修改调用代码。例如，在动物的例子中，如果要添加一个新的动物类别（如猫），只需要创建一个新的猫类并继承动物类，重写`makeSound`方法来体现猫的叫声特点，而`main`程序中的循环代码不需要修改，就可以正确地调用猫类的`makeSound`方法。
    - **增强代码的通用性**：通过多态，可以编写更加通用的代码。比如在处理集合中的对象时，不需要知道集合中对象的具体类型，只需要知道它们都继承自某个父类或者实现了某个接口，就可以统一调用它们的方法，让代码更加简洁和高效。