> 1. [Java IO - 分类(传输，操作) | Java 全栈知识体系](https://www.pdai.tech/md/java/io/java-io-basic-category.html)
> 2. [深入探索 Java IO：基础、实践与最佳实践 - JavaGuidePro](https://javaguidepro.com/java-basic/java-io/)

## 介绍
Java IO（Input/Output）是 Java 编程中用于处理**输入输出操作**的核心模块，主要基于 **流（Stream）** 的概念实现。Java IO 可以分为传统的 **BIO（Blocking IO）** 和新的 **NIO（Non-blocking IO）** 两大体系。

### 字节流和字符流的区别

- 字节流读取单个字节，字符流读取单个字符(一个字符根据编码的不同，对应的字节也不同，如 UTF-8 编码中文汉字是 3 个字节，GBK编码中文汉字是 2 个字节。)
- 字节流用来处理二进制文件(图片、MP3、视频文件)，字符流用来处理文本文件(可以看做是特殊的二进制文件，使用了某种编码，人可以阅读)。

> 简而言之，字节是给计算机看的，字符才是给人看的。

- **字节流**：`InputStream`、`OutputStream`、`FileInputStream`、`FileOutputStream`
- **字符流**：`Reader`、`Writer`、`FileReader`、`FileWriter`