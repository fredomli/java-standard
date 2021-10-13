# Spring Container Bean 概述

Spring IoC容器管理一个或多个bean。这些bean是使用您提供给容器的配置元数据创建的(例如，以XML <bean/>定义的形式)。

在容器本身中，这些bean定义被表示为BeanDefinition对象，其中包含(以及其他信息)以下元数据:

* 包限定的类名:通常是被定义的bean的实际实现类。
* Bean行为配置元素，它声明Bean在容器中应该如何行为(范围、生命周期回调等等)。
* 对bean执行其工作所需的其他bean的引用。这些引用也称为协作者或依赖项。
* 要在新创建的对象中设置的其他配置设置—例如，池的大小限制或在管理连接池的bean中使用的连接数。

此元数据转换为组成每个bean定义的一组属性。下表描述了这些属性:

|Property|Explained|
|-------|-------|
|Class|Instantiating Beans|
|Name|Naming Beans|
|Scope|Bean Scopes|
|Constructor arguments|Dependency Injection|
|Properties|Dependency Injection|
|Autowiring mode|Autowiring Collaborators|
|Lazy initialization mode|Lazy-initialized Beans|
|Initialization method|Initialization Callbacks|
|Destruction method|Destruction Callbacks|

除了包含关于如何创建特定bean的信息的bean定义之外，ApplicationContext实现还允许在容器外部(由用户)创建的现有对象的注册。这是通过getBeanFactory()方法访问ApplicationContext的BeanFactory来实现的，该方法返回BeanFactory DefaultListableBeanFactory实现。DefaultListableBeanFactory通过registerSingleton(..)和registerBeanDefinition(..)方法支持此注册。然而，典型的应用程序只能使用通过常规bean定义元数据定义的bean。

> Bean元数据和手动提供的单例实例需要尽早注册，以便容器在自动装配和其他自省步骤中对它们进行正确的推理。虽然在一定程度上支持覆盖现有元数据和现有单例实例，但官方并不支持在运行时注册新bean(同时对工厂进行实时访问)，这可能会导致并发访问异常、bean容器中的不一致状态，或者两者都有。

## Naming Beans
每个bean都有一个或多个标识符。这些标识符在承载bean的容器中必须是唯一的。bean通常只有一个标识符。但是，如果需要多个，则可以考虑使用别名。

在基于xml的配置元数据中，可以使用id属性、name属性或两者都指定bean标识符。id属性允许您指定一个id。通常，这些名称是字母数字('myBean'， 'someService'等)，但它们也可以包含特殊字符。如果希望为bean引入其他别名，还可以在name属性中指定它们，由逗号(，)、分号(;)或空格分隔。作为一个历史说明，在Spring 3.1之前的版本中，id属性被定义为xsd: id类型，这限制了可能的字符xsd:string

您不需要为bean提供名称或id。如果不显式提供名称或id，容器将为该bean生成唯一名称。但是，如果您想通过名称引用该bean，则必须通过使用ref元素或Service Locator样式查找来提供名称。不提供名称的动机与使用内部bean和自动装配协作者有关。

### Bean命名约定

> 在命名bean时，约定使用标准Java约定作为实例字段名称。也就是说，bean名称以小写字母开头，并从那里开始采用驼峰式大小写。这些名称的示例包括accountManager、accountService、userDao、loginController等等。
> 一致地命名bean可以使您的配置更容易阅读和理解。此外，如果您使用Spring AOP，那么在将通知应用到一组按名称相关的bean时，它会有很大的帮助。

```text
通过在类路径中扫描组件，Spring为未命名组件生成bean名，遵循前面描述的规则:本质上，采用简单的类名并将其初始字符转换为小写。
但是，在(不寻常的)特殊情况下，当有多个字符并且第一个和第二个字符都是大写时，原始的大小写将被保留。
这些规则与java.beans. introspector . decitalize (Spring在这里使用的)定义的规则相同。
```

### 给Bean另取别名
在bean定义本身中，您可以为bean提供多个名称，方法是使用id属性指定的最多一个名称和name属性中任意数量的其他名称的组合。这些名称可以是相同bean的等效别名，在某些情况下很有用，例如允许应用程序中的每个组件使用特定于该组件本身的bean名称来引用公共依赖项。  

然而，在实际定义bean的地方指定所有别名并不总是足够的。有时需要为别处定义的bean引入别名。这种情况在大型系统中很常见，其中配置在每个子系统之间被分割，每个子系统都有自己的一组对象定义。在基于xml的配置元数据中，可以使用<alias/>元素来完成此任务。下面的例子展示了如何做到这一点:

```xml
<alias name="fromName" alias="toName"/>s
```

在这种情况下，一个名为fromName的bean(在同一个容器中)也可以在使用了这个别名定义之后被称为toName。







> 参阅
> * [使用Spring MVC服务Web内容](https://spring.io/guides/gs/serving-web-content/)
> * [Spring安全架构(参考指南)](https://spring.io/guides/topicals/spring-security-architecture/)
> * [Spring Framework官方文档](https://spring.io/projects/spring-framework)
> * [SpringIoc简介](https://github.com/fredomli/java-standard/blob/main/docs/spring/spring/core/SpringIoc容器.md)
> * [Spring容器使用](https://github.com/fredomli/java-standard/blob/main/docs/spring/spring/core/Spring容器使用.md)



*(☞ﾟヮﾟ)☞[返回首页README.md address](https://github.com/fredomli/java-standard)*












