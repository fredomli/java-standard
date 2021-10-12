# Spring Framework

## 简介
> Spring使创建Java企业应用程序变得很容易。它提供了在企业环境中使用Java语言所需的一切，支持Groovy和Kotlin作为JVM上的替代语言，并根据应用程序的需要灵活地创建多种体系结构。从Spring Framework 5.1开始，Spring需要JDK 8+ (Java SE 8+)并提供对JDK 11 LTS的开箱即用支持。Java SE 8 update 60建议作为Java 8的最小补丁版本，但一般建议使用最近的补丁版本。


Spring支持广泛的应用程序场景。在大型企业中，应用程序通常存在很长时间，并且必须在JDK和应用程序服务器上运行，这些服务器的升级周期超出了开发人员的控制。其他的可能作为一个嵌入服务器的单个jar运行，可能在云环境中。还有一些可能是不需要服务器的独立应用程序(如批处理或集成工作负载)。

Spring是开源的。它有一个大型且活跃的社区，提供基于各种真实世界用例的持续反馈。这帮助Spring在很长一段时间内成功地发展。

### 什么是我们应该知道的
“Spring”这个词在不同的语境中有不同的含义。它可以用来引用Spring Framework项目本身，它是一切开始的地方。随着时间的推移，其他Spring项目已经构建在Spring框架之上。通常，当人们说“Spring”时，他们指的是整个项目家族。本参考文档关注的是基础:Spring框架本身。

Spring框架分为模块。应用程序可以选择它们需要的模块。核心是核心容器的模块，包括配置模型和依赖注入机制。除此之外，Spring框架还为不同的应用程序架构提供了基础支持，包括消息传递、事务性数据和持久性以及web。它还包括基于servlet的Spring MVC web框架，同时还包括Spring WebFlux响应式web框架。

关于模块的注意事项:Spring的框架jar允许部署到JDK 9的模块路径(“Jigsaw”)。为了在支持jigsaw的应用程序中使用，Spring Framework 5 jar附带了“Automatic-Module-Name”清单条目，它定义了稳定的语言级模块名称(“Spring . jar”)。核心”、“春天。Context "等)独立于jar工件名称(jar遵循相同的命名模式，使用"-"而不是"."，例如。“spring核心”和“spring上下文”)。当然，在JDK 8和9+上，Spring框架jar在类路径上一直工作得很好。

## Spring和Spring框架的历史

* Servlet API (JSR 340)
* WebSocket API (JSR 356)
* Concurrency Utilities (JSR 236)
* JSON Binding API (JSR 367)
* Bean Validation (JSR 303)
* JPA (JSR 338)
* JMS (JSR 914)
* 以及用于事务协调的JTA/JCA设置(如果必要的话)。

Spring框架还支持依赖注入(JSR 330)和通用注解(JSR 250)规范，应用程序开发人员可以选择使用这些规范，而不是Spring框架提供的特定于Spring的机制。

Spring Framework 5.0,Spring需要Java EE 7水平(例如Servlet 3.1 +, JPA 2.1 +)最小,同时提供开箱即用的集成与更新的API在Java EE 8级(例如Servlet 4.0, JSON绑定API)在运行时当遇到。这使得Spring与Tomcat 8和9、WebSphere 9和JBoss EAP 7完全兼容。

随着时间的推移，Java EE在应用程序开发中的角色已经发生了变化。在Java EE和Spring的早期，创建应用程序是为了部署到应用服务器上。今天，在Spring Boot的帮助下，应用程序以devops和云友好的方式创建，并嵌入Servlet容器，无需更改。从Spring Framework 5开始，WebFlux应用程序甚至不直接使用Servlet API，可以在非Servlet容器的服务器(如Netty)上运行。

春天继续创新和进化。除了Spring框架，还有其他项目，比如Spring Boot、Spring Security、Spring Data、Spring Cloud、Spring Batch等等。重要的是要记住，每个项目都有自己的源代码存储库、问题跟踪器和发布节奏。


## 设计理念

当你学习一个框架时，不仅要知道它做什么，还要知道它遵循什么原则。以下是Spring框架的指导原则:
* 在每个层次上提供选择。Spring允许您尽可能推迟设计决策。例如，您可以通过配置切换持久性提供程序，而无需更改代码。对于许多其他基础设施以及与第三方api的集成来说也是如此。
* 容纳不同的观点。Spring拥抱灵活性，对于应该如何做事情不固执己见。它以不同的视角支持广泛的应用需求。
* 保持强大的向后兼容性。Spring的发展经过了精心的管理，使得版本之间几乎没有什么破坏性的变化。Spring支持精心选择的JDK版本和第三方库，以方便依赖Spring的应用程序和库的维护。
* 关注API设计。Spring团队花了大量的时间和精力来制作直观的、跨多个版本支持多年的api。
* 为代码质量设定高标准。Spring框架非常强调有意义的、最新的和准确的javadoc。它是极少数能够声明干净的代码结构，并且包之间没有循环依赖关系的项目之一。

## 快速开始

如果您刚刚开始使用Spring，您可能希望通过创建一个基于[Spring boot](https://spring.io/projects/spring-boot) 的应用程序来开始使用Spring框架。Spring Boot提供了一种快速(而且有主见)的方法来创建基于Spring的生产就绪应用程序。它基于Spring框架，更倾向于约定而不是配置，并且旨在让您尽可能快地启动和运行。

您可以使用[start.spring.io](https://start.spring.io/) 来生成一个基本项目，或者遵循“入门”指南中的一项，例如入门构建[RESTful Web](https://spring.io/guides/gs/rest-service/) 服务。除了更容易理解之外，这些指南都是以任务为中心的，而且大多数都是基于Spring Boot的。它们还涵盖了Spring组合中的其他项目，您在解决特定问题时可能需要考虑这些项目。

> 参阅
> * [使用Spring MVC服务Web内容](https://spring.io/guides/gs/serving-web-content/)
> * [Spring安全架构(参考指南)](https://spring.io/guides/topicals/spring-security-architecture/)
> * [Spring Framework官方文档](https://spring.io/projects/spring-framework)

*(☞ﾟヮﾟ)☞[返回首页README.md address](https://github.com/fredomli/java-standard)*
