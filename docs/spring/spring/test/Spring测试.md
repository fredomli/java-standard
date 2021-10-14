# Spring 测试

## 概述
本章涵盖了Spring对集成测试和单元测试的最佳实践的支持。Spring团队提倡测试驱动开发(TDD)。春天的团队发现,正确使用控制反转(IoC)的确是简化单元测试和集成测试(setter方法和适当的构造函数类的存在使他们更容易连接在一起测试,而无需建立服务定位器注册中心和类似的结构)。

测试是企业软件开发中不可分割的一部分。本章重点讨论IoC原则对单元测试的附加价值，以及Spring框架支持集成测试的好处。(对企业中测试的全面处理超出了本

## 单元测试
与传统Java EE开发相比，依赖项注入应该使您的代码对容器的依赖程度降低。组成应用程序的pojo在JUnit或TestNG测试中应该是可测试的，通过使用new操作符实例化对象，而不需要Spring或任何其他容器。可以使用模拟对象(结合其他有价值的测试技术)单独测试代码。如果您遵循Spring的架构建议，那么代码库的干净分层和组件化将有助于更容易地进行单元测试。例如，您可以通过存根或模拟DAO或存储库接口来测试服务层对象，而无需在运行单元测试时访问持久数据。


真正的单元测试通常运行得非常快，因为不需要设置运行时基础设施。强调将真正的单元测试作为开发方法的一部分可以提高您的生产力。你可能不需要测试章节的这一节来帮助你为基于iocb的应用程序编写有效的单元测试。然而，对于某些单元测试场景，Spring框架提供了模拟对象和测试支持类，这将在本章中描述。


## 模拟对象
Spring包括许多专门用于mocking的包:

* Environment
* JNDI
* Servlet API
* Spring Web Reactive

### Environment（环境）
`org.springframework.mock.env`包包含了`Environment`和`PropertySource`抽象的模拟实现(请参阅Bean定义概要文件和PropertySource抽象)。MockEnvironment和MockPropertySource对于为依赖于特定环境属性的代码开发容器外测试很有用。

### JNDI
`org.springframework.mock.jndi`包包含JNDI SPI的部分实现，您可以使用它为测试套件或独立应用程序建立一个简单的JNDI环境。例如，如果JDBC DataSource实例在测试代码中与在Java EE容器中绑定到相同的JNDI名称，那么您可以在测试场景中重用应用程序代码和配置，而无需进行修改。

> 注意：
> 从Spring Framework 5.2开始，org.springframework.mock.jndi包中的模拟JNDI支持正式被弃用，取而代之的是来自第三方(如Simple-JNDI)的完整解决方案。

### Servlet API

`org.springframework.mock.web`包包含一组完整的Servlet API模拟对象，这些对象对于测试web上下文、控制器和过滤器非常有用。这些模拟对象是针对Spring的Web MVC框架使用的，通常比使用动态模拟对象(如EasyMock)或替代Servlet API模拟对象(如MockObjects)更方便。

从Spring Framework 5.0开始，org.springframework.mock.web中的模拟对象都是基于Servlet 4.0 API的。

Spring MVC Test框架构建在模拟Servlet API对象之上，为Spring MVC提供集成测试框架。看到MockMvc。

### Spring Web Reactive
`org.springframework.mock.http.server.reactive`包包含用于WebFlux应用程序的ServerHttpRequest和ServerHttpResponse的模拟实现。`org.springframework.mock.web.server`包包含一个模拟ServerWebExchange，它依赖于那些模拟请求和响应对象。

MockServerHttpRequest和MockServerHttpResponse都扩展了作为服务器特定实现的相同抽象基类，并与它们共享行为。例如，模拟请求一旦创建就不可变，但是您可以使用ServerHttpRequest中的mutate()方法来创建修改后的实例。

为了使模拟响应正确地实现写契约并返回写完成句柄(即Mono<Void>)，它在默认情况下使用带有cache().then()的Flux，这将缓冲数据并使其可用于测试中的断言。应用程序可以设置自定义的写函数(例如，测试无限流)。

WebTestClient建立在模拟请求和响应的基础上，以支持在没有HTTP服务器的情况下测试WebFlux应用程序。客户端还可以用于运行服务器的端到端测试。


## 单元测试支持类

Spring包括许多可以帮助进行单元测试的类。它们可分为两类:

* General Testing Utilities（一般的测试工具）
* Spring MVC Testing Utilities（Spring MVC 测试工具）

### 一般的测试工具
`org.springframework.test.util`包包含几个用于单元测试和集成测试的通用实用程序。

ReflectionTestUtils是一组基于反射的实用程序方法。在测试以下用例的应用程序代码时，您需要更改常量的值、设置非公共字段、调用非公共setter方法或调用非公共配置或生命周期回调方法，您可以在这些测试场景中使用这些方法:

* 允许私有或受保护字段访问的ORM框架(如JPA和Hibernate)，而不是对域实体中的属性使用公共setter方法。
* Spring对注解(如@Autowired、@Inject和@Resource)的支持，这些注解为私有或受保护的字段、setter方法和配置方法提供依赖注入。
* 在生命周期回调方法中使用@PostConstruct和@PreDestroy等注释。

AopTestUtils是一组与aop相关的实用程序方法。您可以使用这些方法获取对隐藏在一个或多个Spring代理后面的底层目标对象的引用。例如，如果您使用EasyMock或Mockito等库将bean配置为动态模拟，并且模拟被包装在Spring代理中，那么您可能需要直接访问底层模拟，以在其上配置期望并执行验证。关于Spring的核心AOP实用程序，请参阅AopUtils和AopProxyUtils。

### Spring MVC测试工具

`org.springframework.test.web`包包含ModelAndViewAssert，您可以将其与JUnit、TestNG或任何其他用于处理Spring MVC ModelAndView对象的单元测试的测试框架组合使用。

> 单元测试Spring MVC控制器:
> 要将Spring MVC控制器类作为pojo进行单元测试，可以使用ModelAndViewAssert和MockHttpServletRequest、MockHttpSession等来自Spring的Servlet API模拟。为了将Spring MVC和REST Controller类与Spring MVC的WebApplicationContext配置进行彻底的集成测试，请使用Spring MVC测试框架。

## 集成测试

本节(本章剩下的大部分内容)涵盖了Spring应用程序的集成测试。它包括以下主题:

* Overview
* Goals of Integration Testing
* JDBC Testing Support
* Annotations
* Spring TestContext Framework
* MockMvc

> * [使用Spring MVC服务Web内容](https://spring.io/guides/gs/serving-web-content/)
> * [Spring安全架构(参考指南)](https://spring.io/guides/topicals/spring-security-architecture/)
> * [Spring Framework官方文档](https://spring.io/projects/spring-framework)
> * [SpringIoc简介](https://github.com/fredomli/java-standard/blob/main/docs/spring/spring/core/SpringIoc容器.md)
> * [Spring容器使用](https://github.com/fredomli/java-standard/blob/main/docs/spring/spring/core/Spring容器使用.md)
> * [SpringIoc简介](https://github.com/fredomli/java-standard/blob/main/docs/spring/spring/core/SpringIoc容器.md)


*(☞ﾟヮﾟ)☞[返回首页README.md address](https://github.com/fredomli/java-standard)*
