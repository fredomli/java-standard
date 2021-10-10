# Spring Security

## 1. 介绍

> Spring Security is a powerful and highly customizable authentication and access-control framework. It is the de-facto standard for securing Spring-based applications.
>
> Spring Security is a framework that focuses on providing both authentication and authorization to Java applications. Like all Spring projects, the real power of Spring Security is found in how easily it can be extended to meet custom requirements

Spring Security是一个强大的、高度可定制的身份验证和访问控制框架。它是保护基于spring的应用程序的事实上的标准。
Spring Security是一个专注于为Java应用程序提供身份验证和授权的框架。与所有Spring项目一样，Spring Security的真正强大之处在于它可以很容易地扩展以满足定制需求

### 特点：  
* 对身份验证和授权的全面和可扩展支持
* 针对会话固定、点击劫持、跨站点请求伪造等攻击的保护
* Servlet API的集成
* 与Spring Web MVC的可选集成
* Much more…

注意：更多请参考 [官方文档](https://spring.io/projects/spring-security#overview)

## 2. 入门指南
> 该指南旨在在15-30分钟内,为Spring开发任务构建入门应用程序提供快速、实用的指导。 

具体参考官方文档 [保护Web应用程序](https://spring.io/guides/gs/securing-web/)  

### 2.1 获得一个Web应用程序
> 本指南将引导您使用Spring Security保护的资源创建一个简单的web应用程序。  

### 2.2 你将构建怎样的应用
> 您将构建一个Spring MVC应用程序，该应用程序使用一个由固定用户列表支持的登录表单来保护页面。  

*准备：*
* 15分钟时间
* 最喜欢文本编辑器或IDE
* JDK1.8或更高版本
* Gradle4+ 或 Maven3.2+
* 也可以直接将代码导入IDEA中:
  * [Spring Tool Suite (STS)](Spring Tool Suite (STS))
  * [IntelliJ IDEA](https://spring.io/guides/gs/intellij-idea/)
    

### 2.3 初始化一个Spring项目
> 像大多数Spring Getting Started指南一样，您可以从头开始并完成每个步骤，或者您可以绕过您已经熟悉的基本设置步骤。无论哪种方式，最终都会得到可工作的代码。  

要从头开始，请继续阅读[开始一个Spring初始化项目](https://github.com/fredomli/java-standard/blob/main/docs/spring/security/readme.md:53) 。

需要跳过初始化项目的步骤可以执行下列操作：
* 下载并解压缩本指南的源代码库，或者使用Git克隆它:Git clone https://github.com/spring-guides/gs-securing-web.git
* cd到gs-securing-web/initial
* 跳转到后面步骤`创建一个不安全的应用程序。`


### 2.4 开始一个Spring初始化项目

## 3. 主题
> 可在一个小时或更少的时间内阅读和理解，提供比入门指南更广泛或更主观的内容。  

具体参考官方文档 [Spring Security 构建风格](https://spring.io/guides/topicals/spring-security-architecture)
## 4. 文档
每一个Spring项目都有对应的文档，它非常详细地解释了如何使用项目特性以及使用它们可以实现什么。  

![spring-security-docs](https://gitee.com/fredomli/fredomli-picture/raw/picgo/static/images/wordpress/spring-security-document.png)
### 快速开始一个项目
> Quickstart Your Project (开始你的项目，使用 Spring Initializr)  
> Bootstrap your application with [Spring Initializr](https://start.spring.io/) .

*(☞ﾟヮﾟ)☞[返回首页README.md address](https://github.com/fredomli/java-standard)*
