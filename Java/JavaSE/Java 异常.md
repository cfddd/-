## Exception 和 Error 有什么区别

Error类和Exception类都是继承Throwable类

Exception 是指程序在正常运行中，可以预料的意外情况，可能并且应该被捕获，进行处理

Error 是指在正常情况下，不大可能出现的情况，通常会导致程序不可恢复，所以捕获就没有意义（例如OOM，已经无法继续执行了）

## Exception 受检异常和非受检异常
受检异常（Checked Exception）
- 编译器会检查此类异常，必须在代码中显式地捕获或声明抛出，编译时检查。
- 使用 try-catch 块捕获，或者在方法签名中使用 throws 关键字声明。
- 例如 IOException, SQLException

```java
public void readFile() throws IOException {
    FileReader file = new FileReader("somefile.txt");
    BufferedReader fileInput = new BufferedReader(file);
    fileInput.read();
}
```

非受检异常（Unchecked Exception）
- 编译器不会检查此类异常，程序员可以选择捕获或不捕获，运行时检查。如果一直未捕获非受检异常，会沿着调用栈想上抛出，直到被捕获或者导致程序终止
- 可以使用 try-catch 块捕获，但不是强制性的。
- 例如：NullPointerException, ArrayIndexOutOfBoundsException

```java
public void divide(int a, int b) {
    if (b == 0) {
        throw new ArithmeticException("Division by zero");
    }
    int result = a / b;
}
```
## 捕获 Exception 的基本原则
1. 尽量不要捕获类似 Exception、Throwable 这样的通用异常，而是捕获特定异常
2. 不要生吞异常，打印一个e.printStackTrace()就结束，无法定位问题
3. 早抛，晚捕