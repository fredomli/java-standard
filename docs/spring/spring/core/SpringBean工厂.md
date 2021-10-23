# BeanFactory

本节解释BeanFactory和ApplicationContext容器级别之间的区别以及对引导的影响。

你应该使用ApplicationContext，除非你有很好的理由不这样做，用GenericApplicationContext和它的子类AnnotationConfigApplicationContext作为自定义引导的通用实现。这些是Spring核心容器的主要入口点，用于所有常见目的:加载配置文件、触发类路径扫描、以编程方式注册bean定义和注释类，以及(从5.0开始)注册功能bean定义。

因为ApplicationContext包含了BeanFactory的所有功能，所以除了需要对bean处理进行完全控制的场景外，一般建议使用它而不是普通的BeanFactory。在ApplicationContext(比如GenericApplicationContext实现)的几种检测到bean按照惯例(也就是说,由bean名称或bean类型——特别是,后处理器),而普通DefaultListableBeanFactory不可知论者是关于任何特殊bean。

对于许多扩展容器特性，如注释处理和AOP代理，BeanPostProcessor扩展点是必不可少的。如果只使用普通的DefaultListableBeanFactory，默认情况下不会检测和激活这样的后处理器。这种情况可能会令人困惑，因为您的bean配置实际上没有任何问题。相反，在这种情况下，需要通过额外的设置完全引导容器。

下表列出了BeanFactory和ApplicationContext接口和实现提供的特性。

> * [使用Spring MVC服务Web内容](https://spring.io/guides/gs/serving-web-content/)
> * [Spring安全架构(参考指南)](https://spring.io/guides/topicals/spring-security-architecture/)
> * [Spring Framework官方文档](https://spring.io/projects/spring-framework)
> * [SpringIoc简介](https://github.com/fredomli/java-standard/blob/main/docs/spring/spring/core/SpringIoc容器.md)
> * [Spring容器使用](https://github.com/fredomli/java-standard/blob/main/docs/spring/spring/core/Spring容器使用.md)
> * [SpringIoc简介](https://github.com/fredomli/java-standard/blob/main/docs/spring/spring/core/SpringIoc容器.md)
> * [SpringBean作用域](https://github.com/fredomli/java-standard/blob/main/docs/spring/spring/core/SpringBean作用域.md)
> * [Spring基于注解的容器配置](https://github.com/fredomli/java-standard/blob/main/docs/spring/spring/core/Spring基于注解的容器配置.md)
> * [Spring基于Java的容器配置](https://github.com/fredomli/java-standard/blob/main/docs/spring/spring/core/Spring基于java的容器配置.md)

*(☞ﾟヮﾟ)☞[返回首页README.md address](https://github.com/fredomli/java-standard)*
