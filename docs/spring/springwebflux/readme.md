# Spring WebFlux

## 介绍
> Spring框架中包含的最初web框架Spring web MVC是专门为Servlet API和Servlet容器而构建的。响应式堆栈web框架Spring WebFlux在5.0版本中被添加。它是完全无阻塞的，支持响应流回压，并在Netty、Undertow和Servlet 3.1+容器等服务器上运行。

这两个web框架都反映了它们源模块的名称(Spring -webmvc和Spring -webflux)，并在Spring框架中共存。每个模块都是可选的。应用程序可以使用其中一个或另一个模块，在某些情况下，两者都可以使用——例如，带有响应式WebClient的Spring MVC控制器。

### 概述
为什么要创建Spring WebFlux ?

部分原因是需要一个非阻塞的web堆栈来处理少量线程的并发性，并使用更少的硬件资源进行伸缩。Servlet 3.1确实为非阻塞I/O提供了一个API。然而，使用它会偏离Servlet API的其余部分，其中契约是同步的(Filter, Servlet)或阻塞的(getParameter, getPart)。这是一个新的公共API作为跨任何非阻塞运行时的基础的动机。这一点很重要，因为服务器(如Netty)已经在异步、非阻塞空间中建立了良好的关系。

答案的另一部分是函数式编程。就像在Java 5中添加注释创造了机会(例如带注释的REST控制器或单元测试)，在Java 8中添加lambda表达式也为Java中的函数api创造了机会。这对于非阻塞应用程序和延续风格的api(由CompletableFuture和ReactiveX普及)是一个福音，它们允许异步逻辑的声明式组合。在编程模型级别，Java 8使Spring WebFlux能够提供功能性的web端点和带注释的控制器。



> 参阅
> * [Spring安全架构(参考指南)](https://spring.io/guides/topicals/spring-security-architecture/)
> * [Spring WebFlux官方教程](https://docs.spring.io/spring-framework/docs/current/reference/html/web-reactive.html)

*(☞ﾟヮﾟ)☞[返回首页README.md address](https://github.com/fredomli/java-standard)*
