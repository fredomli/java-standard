# 容器扩展点

通常，应用程序开发人员不需要子类化ApplicationContext实现类。相反，可以通过插入特殊集成接口的实现来扩展Spring IoC容器。下面几节将描述这些集成接口。

## 使用BeanPostProcessor自定义bean
BeanPostProcessor接口定义了回调方法，您可以实现这些回调方法来提供您自己的(或覆盖容器的默认)实例化逻辑、依赖项解析逻辑等等。如果希望在Spring容器完成bean的实例化、配置和初始化之后实现一些自定义逻辑，可以插入一个或多个自定义BeanPostProcessor实现。

您可以配置多个BeanPostProcessor实例，并且可以通过设置order属性来控制这些BeanPostProcessor实例运行的顺序。只有当BeanPostProcessor实现了Ordered接口时，才能设置此属性。如果您编写自己的BeanPostProcessor，也应该考虑实现Ordered接口。有关详细信息，请参阅BeanPostProcessor和Ordered接口的javadoc。请参见关于BeanPostProcessor实例的编程注册的说明。











> * [使用Spring MVC服务Web内容](https://spring.io/guides/gs/serving-web-content/)
> * [Spring安全架构(参考指南)](https://spring.io/guides/topicals/spring-security-architecture/)
> * [Spring Framework官方文档](https://spring.io/projects/spring-framework)
> * [SpringIoc简介](https://github.com/fredomli/java-standard/blob/main/docs/spring/spring/core/SpringIoc容器.md)
> * [Spring容器使用](https://github.com/fredomli/java-standard/blob/main/docs/spring/spring/core/Spring容器使用.md)
> * [SpringIoc简介](https://github.com/fredomli/java-standard/blob/main/docs/spring/spring/core/SpringIoc容器.md)
> * [SpringBean作用域](https://github.com/fredomli/java-standard/blob/main/docs/spring/spring/core/SpringBean作用域.md)


*(☞ﾟヮﾟ)☞[返回首页README.md address](https://github.com/fredomli/java-standard)*

