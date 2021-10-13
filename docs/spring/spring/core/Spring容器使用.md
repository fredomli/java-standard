# 使用Spring容器

## 介绍
`ApplicationContext`是高级工厂的接口，该工厂能够维护不同bean及其依赖项的注册表。通过使用方法T getBean(String name, Class<T> requiredType)，您可以检索bean的实例。

`ApplicationContext`允许您读取和访问bean定义，如下面的示例所示:

```java
// create and configure beans
ApplicationContext context = new ClassPathXmlApplicationContext("services.xml", "daos.xml");

// retrieve configured instance
PetStoreService service = context.getBean("petStore", PetStoreService.class);

// use configured instance
List<String> userList = service.getUsernameList();
```

在Groovy配置中，引导看起来非常相似。它有一个不同的上下文实现类，它支持groovy(但也理解XML bean定义)。下面的示例展示了Groovy的配置:

```java
ApplicationContext context = new GenericGroovyApplicationContext("services.groovy", "daos.groovy");
```

最灵活的变体是将GenericApplicationContext与reader委托结合使用——例如，将XmlBeanDefinitionReader用于XML文件，如下面的示例所示:

```java
GenericApplicationContext context = new GenericApplicationContext();
new XmlBeanDefinitionReader(context).loadBeanDefinitions("services.xml", "daos.xml");
context.refresh();
```
您还可以对Groovy文件使用GroovyBeanDefinitionReader，如下例所示:

```java
GenericApplicationContext context = new GenericApplicationContext();
new GroovyBeanDefinitionReader(context).loadBeanDefinitions("services.groovy", "daos.groovy");
context.refresh();
```

您可以在同一个ApplicationContext上混合和匹配这样的读取器委托，从不同的配置源读取bean定义。

然后可以使用getBean检索bean的实例。ApplicationContext接口有一些用于检索bean的其他方法，但是，理想情况下，应用程序代码不应该使用它们。实际上，应用程序代码根本不应该调用getBean()方法，因此完全不依赖于Spring api。例如，Spring与web框架的集成为各种web框架组件(如控制器和jsf管理的bean)提供了依赖注入，允许您通过元数据(如自动装配注释)声明对特定bean的依赖关系。



> 参阅
> * [使用Spring MVC服务Web内容](https://spring.io/guides/gs/serving-web-content/)
> * [Spring安全架构(参考指南)](https://spring.io/guides/topicals/spring-security-architecture/)
> * [Spring Framework官方文档](https://spring.io/projects/spring-framework)
> * [SpringIoc简介](https://github.com/fredomli/java-standard/blob/main/docs/spring/spring/core/SpringIoc容器.md)



*(☞ﾟヮﾟ)☞[返回首页README.md address](https://github.com/fredomli/java-standard)*
