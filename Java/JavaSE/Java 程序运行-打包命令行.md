## 直接编译运行
Java 程序的运行一般分为两个阶段：

> **编译阶段**（将 `.java` 源码文件编译为 `.class` 字节码文件）  
> **运行阶段**（使用 JVM 运行 `.class` 文件）

### 前提准备
配置好 JDK（Java Development Kit）环境
### 具体步骤
假设有如下文件`HelloWorld.java`:
```java
public class HelloWorld {
    public static void main(String[] args) {
        System.out.println("Hello, World!");
    }
}
```
编译java源码：
```sh
javac HelloWorld.java
```
运行：
```sh
java HelloWorld
```
> **注意**：不加 `.class` 后缀，只写类名。

### 有包结构的项目运行
假设 Java 文件结构如下：
```css
src/
 └── com/example/Main.java
```
```java
package com.example;

public class Main {
    public static void main(String[] args) {
        System.out.println("With package!");
    }
}
```
**编译**：
```java
javac -d out src/com/example/Main.java
// -d out：指定编译后的 .class 文件输出到 out/ 目录
```
**运行**：
```java
java -cp out com.example.Main
```
- `-cp` 或 `-classpath`：指定类路径
- `com.example.Main` 是包含完整包名的类名

## 打jar包
在 Java 项目中，“**打包**”通常指的是将编译好的 `.class` 文件（以及其他资源）封装成一个压缩文件（主要是 `.jar` 文件），以便于分发、部署或运行。
### 打包的作用

|作用|说明|
|---|---|
|✅ 部署方便|不需要一堆 `.class` 文件，只需要一个 `.jar` 文件即可部署。|
|✅ 运行方便|可以直接使用 `java -jar xxx.jar` 来运行程序。|
|✅ 管理依赖|可以把第三方库一起打包（可用“可执行 Jar”或“fat jar”）|
|✅ 支持主类入口声明|`jar` 文件可以声明 `Main-Class`，使其可执行。|
|✅ IDE或CI支持|打包是项目构建（build）的核心步骤之一，IDE 和 CI/CD 会用到它。|

### 基础打包命令（使用 `jar` 工具）
前提：你已经使用 `javac` 编译好了 `.class` 文件。

**示例项目结构**：
```
MyApp/
├── HelloWorld.java
├── HelloWorld.class
└── manifest.txt
```
### 打包命令详解

#### **最基本打包命令**（不包含主类信息）

```bash
jar cf HelloWorld.jar HelloWorld.class
```
- `c`：create 创建新的 jar 文件
- `f`：file 输出到文件
- `HelloWorld.jar`：目标 jar 文件名
- `HelloWorld.class`：要打包的类文件

#### **可执行 Jar 包**（带 `main()` 方法）

需要指定 **主类信息**，可以使用 `-m` 指定 manifest 文件，或手动添加。

① 创建 `manifest.txt` 文件，内容如下：
```
Main-Class: HelloWorld
```
> ⚠️ **注意**：`Main-Class:` 后面必须是类的全限定名，且后面必须有换行！

② 打包命令：
```bash
jar cfm HelloWorld.jar manifest.txt HelloWorld.class
```
③ 运行：
```bash
java -jar HelloWorld.jar
```

#### **带包结构的打包**
项目结构：
```
com/example/
└── Main.class
```
打包：
```bash
jar cfe App.jar com.example.Main com/example/Main.class
```
- `e`：entry，指定主类

运行：
```bash
java -jar App.jar
```
### 打包包含依赖（Fat Jar）
单纯用 `jar` 命令不能打包依赖库，如果你有多个 `.jar` 文件要打包一起，建议使用构建工具：
- **Maven**：`maven-shade-plugin`
- **Gradle**：`shadow plugin`

受限需要在pom中配置打包包含依赖的插件，添加如下内容：
```java
			<plugin>
				<groupId>org.apache.maven.plugins</groupId>
				<artifactId>maven-surefire-plugin</artifactId>
				<version>3.2.5</version>
				<configuration>
					<skipTests>true</skipTests> <!--跳过单元测试的插件，因为单元测试在package之前，如果导致失败可以使用这个插件-->
				</configuration>
			</plugin>
			<plugin>
				<groupId>org.apache.maven.plugins</groupId>
				<artifactId>maven-shade-plugin</artifactId>
				<version>3.5.0</version>
				<executions>
					<execution>
						<phase>package</phase>
						<goals>
							<goal>shade</goal>
						</goals>
						<configuration>
							<transformers>
								<transformer
									implementation="org.apache.maven.plugins.shade.resource.ManifestResourceTransformer"> <!--指定主类-->
									<mainClass>com.itranswarp.jerrymouse.Start</mainClass>
								</transformer>
							</transformers>
						</configuration>
					</execution>
				</executions>
			</plugin>
```

```bash
mvn package
```
运行 `mvn package` 命令后，Maven 会对你的 Java 项目进行完整构建，最终在 `target/` 目录下生成打包好的文件，通常是一个或多个 `.jar` 文件。
```bash
your-project/
├── pom.xml
├── src/
│   ├── main/java/...
│   └── test/java/...
├── target/
│   ├── classes/          ← 编译后的主类文件
│   ├── test-classes/     ← 编译后的测试类文件
│   ├── your-app.jar      ← 最终生成的 jar 包（默认名字）
│   └── ...其他文件
```
默认情况下，生成的 `.jar` 文件位于target目录下

> **注意**：如果你的 `pom.xml` 指定了 `Main-Class`，那么可以直接运行这个 jar，否则**运行时需要手动指定主类**，例如`java -cp target/myapp-1.0-SNAPSHOT.jar com.example.Main`

#### 打war包还是jar包
```xml
<groupId>com.itranswarp.sample</groupId>  
<artifactId>hello-webapp</artifactId>  
<version>1.0</version>  
<packaging>war</packaging>
```
通过`<packaging>`标签来决定