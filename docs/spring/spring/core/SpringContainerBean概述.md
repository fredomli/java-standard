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
<alias name="fromName" alias="toName"/>
```

在这种情况下，一个名为fromName的bean(在同一个容器中)也可以在使用了这个别名定义之后被称为toName。

例如，子系统A的配置元数据可以通过subsystemA-dataSource的名称引用数据源。子系统B的配置元数据可以通过subsystemB-dataSource的名称引用数据源。当组成使用这两个子系统的主应用程序时，主应用程序通过myApp-dataSource的名称引用数据源。要使这三个名称都指向同一个对象，您可以向配置元数据添加以下别名定义:

```xml
<alias name="myApp-dataSource" alias="subsystemA-dataSource"/>
<alias name="myApp-dataSource" alias="subsystemB-dataSource"/>
```
现在，每个组件和主应用程序都可以通过唯一的名称来引用dataSource，并且保证不会与任何其他定义(有效地创建了一个名称空间)发生冲突，但它们引用的是相同的bean。

> 注意：  
> 如果使用javaconconfiguration，则可以使用@Bean注释提供别名。有关详细信息，请参阅使用@Bean注释。

## 实例化bean
bean定义本质上是创建一个或多个对象的方法。当被请求时，容器会查看指定bean的配方，并使用由该bean定义封装的配置元数据来创建(或获取)实际对象。

如果使用基于xml的配置元数据，则指定要在<bean/>元素的class属性中实例化的对象的类型(或类)。这个类属性(在内部是BeanDefinition实例上的class属性)通常是强制的。(有关异常，请参见使用实例工厂方法实例化和Bean定义继承。)你可以通过以下两种方式使用Class属性:

* 通常，在容器本身通过反射式调用其构造函数直接创建bean的情况下，指定要构造的bean类，这有点类似于使用new操作符的Java代码。
* 要指定包含用于创建对象的静态工厂方法的实际类，在不太常见的情况下，容器调用类上的静态工厂方法来创建bean。调用静态工厂方法返回的对象类型可以是相同的类，也可以是完全不同的类。

> 意：嵌套的类名  
> 如果想为嵌套类配置bean定义，可以使用嵌套类的二进制名称或源名称。
> 例如，如果你在com中有一个叫做SomeThing的类。example包，这个SomeThing类有一个静态嵌套类OtherThing，它们可以用美元符号($)或点(.)分隔。因此，bean定义中的class属性的值应该是`com.example.SomeThing$OtherThing`或`com.example.SomeThing.OtherThing`的东西。

### 使用构造函数实例化

当您通过构造函数方法创建bean时，所有普通类都可以被Spring使用并与Spring兼容。也就是说，正在开发的类不需要实现任何特定的接口，也不需要以特定的方式编码。只需指定bean类就足够了。但是，根据您对特定bean使用的IoC类型，您可能需要一个默认(空)构造函数。

Spring IoC容器实际上可以管理您希望它管理的任何类。它并不局限于管理真正的javabean。大多数Spring用户更喜欢实际的javabean，只有默认(无参数)构造函数和适当的setter和getter，它们是按照容器中的属性建模的。您还可以在容器中有更多奇异的非bean样式的类。例如，如果您需要使用一个完全不符合JavaBean规范的遗留连接池，Spring也可以管理它。

使用基于xml的配置元数据，您可以如下指定您的bean类:  
```xml
<bean id="exampleBean" class="examples.ExampleBean"/>

<bean name="anotherExample" class="examples.ExampleBeanTwo"/>
```
有关向构造函数提供参数(如果需要)以及在构造对象之后设置对象实例属性的机制的详细信息，请参见依赖注入项(Injecting Dependencies.)。

### 使用静态工厂方法实例化

在定义使用静态工厂方法创建的bean时，使用class属性指定包含静态工厂方法的类，并使用名为factory-method的属性指定工厂方法本身的名称。您应该能够调用这个方法(稍后将使用可选参数进行描述)并返回一个活动对象，该对象随后将被视为通过构造函数创建的。这种bean定义的一个用途是在遗留代码中调用静态工厂。

```xml
<bean id="clientService"
    class="examples.ClientService"
    factory-method="createInstance"/>
```
下面的示例显示了一个可以使用前面的bean定义的类:

```java
public class ClientService {
    private static ClientService clientService = new ClientService();
    private ClientService() {}

    public static ClientService createInstance() {
        return clientService;
    }
}
```

有关向工厂方法提供(可选)参数以及在从工厂返回对象后设置对象实例属性的机制的详细信息，请参阅详细的依赖项和配置。

### 使用实例工厂方法进行实例化

与通过静态工厂方法进行实例化类似，使用实例工厂方法进行实例化将从容器调用现有bean的非静态方法来创建新bean。要使用这种机制，请保留class属性为空，并在factory-bean属性中指定当前容器(或父容器或祖先容器)中bean的名称，该容器包含要调用的实例方法来创建对象。使用factory-method属性设置工厂方法本身的名称。下面的示例演示如何配置这样的bean:

```xml
<!-- the factory bean, which contains a method called createInstance() -->
<bean id="serviceLocator" class="examples.DefaultServiceLocator">
    <!-- inject any dependencies required by this locator bean -->
</bean>

<!-- the bean to be created via the factory bean -->
<bean id="clientService"
    factory-bean="serviceLocator"
    factory-method="createClientServiceInstance"/>
```
下面的例子显示了相应的类:

```java
public class DefaultServiceLocator {

    private static ClientService clientService = new ClientServiceImpl();

    public ClientService createClientServiceInstance() {
        return clientService;
    }
}
```
个工厂类也可以包含多个工厂方法，如下例所示:

```java
<bean id="serviceLocator" class="examples.DefaultServiceLocator">
    <!-- inject any dependencies required by this locator bean -->
</bean>

<bean id="clientService"
    factory-bean="serviceLocator"
    factory-method="createClientServiceInstance"/>

<bean id="accountService"
    factory-bean="serviceLocator"
    factory-method="createAccountServiceInstance"/>
```
下面的例子显示了相应的类:

```java
public class DefaultServiceLocator {

    private static ClientService clientService = new ClientServiceImpl();

    private static AccountService accountService = new AccountServiceImpl();

    public ClientService createClientServiceInstance() {
        return clientService;
    }

    public AccountService createAccountServiceInstance() {
        return accountService;
    }
}
```
这种方法表明，工厂bean本身可以通过依赖项注入(DI)进行管理和配置。请参阅依赖项和配置的详细信息。

在Spring文档中，“factory bean”是指在Spring容器中配置并通过实例或静态工厂方法创建对象的bean。相比之下，FactoryBean(注意大小写)指的是特定于spring的FactoryBean实现类。进入翻译页面

### 确定Bean的运行时类型

确定特定bean的运行时类型并非易事。bean元数据定义中的指定类只是一个初始类引用，可能与声明的工厂方法组合在一起，或者是一个FactoryBean类，该类可能导致bean的不同运行时类型，或者，对于实例级工厂方法(通过指定的工厂bean名称解析)，根本没有设置。此外，AOP代理可以用基于接口的代理包装一个bean实例，该代理有限地公开目标bean的实际类型(仅公开其实现的接口)。

了解特定bean的实际运行时类型的推荐方法是使用BeanFactory。getType调用指定的bean名。这将考虑上述所有情况，并返回BeanFactory的对象类型。getBean调用将返回相同的bean名称。

> 参阅
> * [使用Spring MVC服务Web内容](https://spring.io/guides/gs/serving-web-content/)
> * [Spring安全架构(参考指南)](https://spring.io/guides/topicals/spring-security-architecture/)
> * [Spring Framework官方文档](https://spring.io/projects/spring-framework)
> * [SpringIoc简介](https://github.com/fredomli/java-standard/blob/main/docs/spring/spring/core/SpringIoc容器.md)
> * [Spring容器使用](https://github.com/fredomli/java-standard/blob/main/docs/spring/spring/core/Spring容器使用.md)
> * [Spring依赖注入](https://github.com/fredomli/java-standard/blob/main/docs/spring/spring/core/Spring依赖注入概述.md)



*(☞ﾟヮﾟ)☞[返回首页README.md address](https://github.com/fredomli/java-standard)*












