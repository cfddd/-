
### **🏷️ 标题**

> 目标：用来沉淀**一个点**，可分享、可复用、可快速回忆。

> 建议用 Markdown 写，每篇控制在 1～2 页内。

> 简短 + 能让人一眼明白主题。

> 例如：LocalDateTime 精度问题与替代方案、分布式锁的可靠解锁策略

---

### **🔍 背景 / 问题**

  

> 说明你为啥研究这个点，它出现在哪种场景下。

  

示例：

  

> 在导出任务中使用 LocalDateTime 比较时，发现时间偶尔出现反常排序，怀疑与纳秒精度有关。

---

### **⚙️ 现象 / 案例复现**

  

> 用一小段代码 / 日志 / 截图展示问题。

```
LocalDateTime t1 = LocalDateTime.now();
Thread.sleep(1);
LocalDateTime t2 = LocalDateTime.now();
System.out.println(t2.isAfter(t1)); // 有时返回 false
```

---

### **🧩 原理分析**

  

> 讲清「为什么」会出现这个问题。

> 可以画图、列出源码片段、或者逻辑流程。

  

示例：

  

> LocalDateTime 不包含时区信息，底层取 System.currentTimeMillis() 转换成纳秒，但存在舍入误差，

> 导致微秒级比较时精度不足。

---

### **💡 解决方案**

  

> 重点放清晰、可靠、可落地的做法。

|**方案**|**优点**|**缺点**|**适用场景**|
|---|---|---|---|
|使用 Instant.now()|精度高、可比较|与业务时区脱离|时间戳比较|
|使用 System.currentTimeMillis()|简单|毫秒级|性能统计类|

---

### **🔧 实践建议**

  

> 讲点有经验感的总结，比如：

  

- 在业务中统一时间类型，不混用 LocalDateTime 与 Instant。
    
- 日志时间建议使用 UTC + ISO 格式。
    

---

### **📚 参考资料**

  

> 列出你查的源码 / 文档 / 文章。

  

- [Java Time API 官方文档](https://docs.oracle.com/javase/8/docs/api/java/time/package-summary.html)
    
- StackOverflow: _Why LocalDateTime precision varies across JVMs_
    

---

### **✏️ 总结一句话**

  

> 类似“记忆钉”，方便以后检索。

> 🧷 _LocalDateTime 不适合做高精度时间比较，用 Instant 才稳。_

