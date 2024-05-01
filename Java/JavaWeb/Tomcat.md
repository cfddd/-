## Web服务器Tomcat

- **概念:** Tomcat是Apache 软件基金会一个核心项目，是一个开源免费的轻量级Web服务器，支持Servlet/SP少量]avaEE规范。
- **JavaEE:** Java Enterprise Edition，Java企业版。
- **Tomcat 也被称为 Web容器、Servlet容器。** Servlet程序需要依赖于 Tomcat才能运行
- 官网:https://tomcat.apache.org/
> JavaSE：标准版
> 
> JavaEE：企业版，指ava企业级开发的技术规范总和。包含13项技术规范: JDBC、INDI、EJB.RMI、JSP、Servlet、XML、JMS、Java IDL、JTS、JTA、JavaMail、JAF
> 
> JavaME：小型版-嵌入式应用

1. Web服务器
对HTTP协议操作进行封装，简化web程序开发。部署web项目，对外提供网上信息浏览服务
2. Tomcat
一个轻量级的web服务器，支持servlet、jsp等少量javaEE规范。也被称为web容器、servlet容器

## 基本使用
把项目放在tomcat/webapps下即可自动运行
### SpringBoot项目打包
基于Springboot开发的web应用程序，内置了tomcat服务器，当启
动类运行时，会自动启动内嵌的tomcat服务器。