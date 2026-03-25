> made by :_CHATGPT_

## 常见命令

```sh
mvn test -Dtest=XxxTest -o # 跳过 依赖下载（mvn 默认会跑 dependency:resolve），但是编译步骤会保留。
mvn -o test              # 离线模式，不下载依赖

mvn test -DskipTests     # 只编译不跑单测

mvn verify -DskipTests   # 验证构建流程，但不跑单测
```

## 🔧 基础构建

- **`mvn clean`**

清理 `target/` 编译输出目录。

- **`mvn compile`**

编译主代码。

- **`mvn test`**

编译并执行测试代码。

- **`mvn package`**

打包项目（生成 jar/war）。

- **`mvn install`**

打包并安装到本地仓库，给本地其他项目用。

- **`mvn deploy`**

发布到远程仓库。

  

---

  

## 📦 依赖管理

- **`mvn dependency:tree`**

打印依赖树，排查冲突神器。

- **`mvn dependency:list`**

列出所有依赖。

- **`mvn dependency:sources`**

下载依赖的源码。

- **`mvn dependency:resolve`**

强制解析并下载依赖。

- **`mvn dependency:analyze`**

分析项目依赖，看看哪些依赖没用到 / 哪些用到但没声明。

  

---

  

## 🧪 测试相关

- **`mvn test -Dtest=TestClass`**

只跑指定测试类。

- **`mvn test -Dtest=TestClass#methodName`**

只跑指定测试方法。

- **`mvn -DskipTests package`**

打包时跳过测试（编译测试类，但不运行）。

- **`mvn -Dmaven.test.skip=true package`**

完全跳过测试（不编译，不运行）。

  

---

  

## 🚀 运行项目

- **`mvn exec:java -Dexec.mainClass="com.example.Main"`**

直接运行 main 方法（需要配置 `exec-maven-plugin`）。

- **`mvn spring-boot:run`**

运行 Spring Boot 项目。

  

---

  

## ⚡ 性能优化

- **`mvn clean install -T 4`**

多线程构建（这里是 4 线程）。

- **`mvn -o package`**

离线模式，不访问远程仓库。

  

---

  

## 🧹 其他实用

- **`mvn help:effective-pom`**

打印合并后的完整 POM（父子 + 依赖管理都展开）。

- **`mvn versions:display-dependency-updates`**

显示依赖可升级版本（需要 `versions-maven-plugin`）。

- **`mvn help:describe -Dcmd=compile`**

查看命令的详细说明。