# Spring 环境抽象（Environment Abstraction）

Environment接口是集成在容器中的一个抽象概念，它建模应用程序环境的两个关键方面:profiles 和properties。

概要文件是一组已命名的逻辑bean定义，只有在给定的概要文件处于活动状态时才向容器注册。bean可以分配给一个配置文件，无论该配置文件是用XML定义的还是用注释定义的。与概要文件相关的Environment对象的作用是确定哪些概要文件(如果有的话)当前是活动的，以及哪些概要文件(如果有的话)在缺省情况下应该是活动的。

属性在几乎所有的应用程序中都扮演着重要的角色，并且可能起源于各种来源:属性文件、JVM系统属性、系统环境变量、JNDI、servlet上下文参数、特别的Properties对象、Map对象，等等。与属性相关的Environment对象的作用是为用户提供一个方便的服务接口，用于配置属性源并从中解析属性。

## Bean Definition Profiles

Bean定义概要文件在核心容器中提供了一种机制，允许在不同的环境中注册不同的Bean。“环境”这个词对不同的用户有不同的含义，这个特性可以帮助很多用例，包括:
* 在开发中使用内存中的数据源，而在QA或生产中从JNDI（Java Naming and Directory Interface,Java命名和目录接口）中查找相同的数据源。
* 仅在将应用程序部署到性能环境时注册监视基础设施。
* 注册客户A和客户B部署的定制bean实现。

考虑需要数据源的实际应用程序中的第一个用例。在测试环境中，配置可能类似如下:

```java
@Bean
public DataSource dataSource() {
    return new EmbeddedDatabaseBuilder()
        .setType(EmbeddedDatabaseType.HSQL)
        .addScript("my-schema.sql")
        .addScript("my-test-data.sql")
        .build();
}
```
现在考虑如何将该应用程序部署到QA或生产环境中，假设应用程序的数据源已注册到生产应用程序服务器的JNDI目录中。我们的dataSource bean现在看起来如下所示:

```java
@Bean(destroyMethod="")
public DataSource dataSource() throws Exception {
    Context ctx = new InitialContext();
    return (DataSource) ctx.lookup("java:comp/env/jdbc/datasource");
}
```
问题是如何根据当前环境在使用这两种变体之间进行切换。随着时间的推移，Spring用户设计了许多方法来完成这一任务，通常依赖于系统环境变量和XML <import/>语句的组合，这些语句包含${占位符}标记，根据环境变量的值解析到正确的配置文件路径。Bean定义概要文件是一个核心容器特性，它为这个问题提供了解决方案。

如果我们将前面特定于环境的bean定义示例中所示的用例一般化，我们最终会需要在某些上下文中注册某些bean定义，而不是在其他上下文中注册。您可以说您想在情况a中注册特定的bean定义概要文件，在情况b中注册不同的概要文件。我们首先更新配置以反映这一需求。

### Using @Profile
当一个或多个指定的概要文件处于活动状态时，@Profile注释允许您指示组件有资格注册。使用前面的例子，我们可以重写dataSource配置如下:

```java
@Configuration
@Profile("development")
public class StandaloneDataConfig {

    @Bean
    public DataSource dataSource() {
        return new EmbeddedDatabaseBuilder()
            .setType(EmbeddedDatabaseType.HSQL)
            .addScript("classpath:com/bank/config/sql/schema.sql")
            .addScript("classpath:com/bank/config/sql/test-data.sql")
            .build();
    }
}
```
```java
@Configuration
@Profile("production")
public class JndiDataConfig {

    @Bean(destroyMethod="")
    public DataSource dataSource() throws Exception {
        Context ctx = new InitialContext();
        return (DataSource) ctx.lookup("java:comp/env/jdbc/datasource");
    }
}
```
> @ bean方法,如前所述,您通常选择使用程序化的JNDI查找,通过使用Spring的JndiTemplate / JndiLocatorDelegate助手或直接使用JNDI InitialContext JndiObjectFactoryBean变体,但不是早些时候显示这将迫使你声明返回类型作为FactoryBean类型。


配置文件字符串可以包含简单的配置文件名称(例如，生产)或配置文件表达式。概要表达式允许表达更复杂的概要逻辑(例如，production & us-east)。profile表达式中支持以下操作符:
* !: A logical “not” of the profile
* &: A logical “and” of the profiles
* |: A logical “or” of the profiles

> 如果不使用括号，就不能混合使用&和|操作符。例如，production & us-east | eu-central不是一个有效的表达。它必须表达为生产和(美国-东部|欧盟-中部)。

```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Profile("production")
public @interface Production {
}
```

> 如果@Configuration类被标记为@Profile，那么所有与该类关联的@Bean方法和@Import注释都会被绕过，除非一个或多个指定的概要文件处于活动状态。如果@Component或@Configuration类被标记为@Profile({"p1"， "p2"})，则该类不会被注册或处理，除非配置文件'p1'或'p2'已经被激活。
> 
> 如果给定的概要文件以NOT操作符(!)为前缀，则只有在概要文件不活动时才会注册带注释的元素。例如，给定@Profile({"p1"， "!p2"})，如果配置文件'p1'是活动的，或者配置文件'p2'不是活动的，则会发生注册。

还可以在方法级声明@Profile，以只包含配置类的一个特定bean(例如，用于特定bean的可选变体)，如下面的示例所示:

```java
@Configuration
public class AppConfig {

    @Bean("dataSource")
    @Profile("development") 
    public DataSource standaloneDataSource() {
        return new EmbeddedDatabaseBuilder()
            .setType(EmbeddedDatabaseType.HSQL)
            .addScript("classpath:com/bank/config/sql/schema.sql")
            .addScript("classpath:com/bank/config/sql/test-data.sql")
            .build();
    }

    @Bean("dataSource")
    @Profile("production") 
    public DataSource jndiDataSource() throws Exception {
        Context ctx = new InitialContext();
        return (DataSource) ctx.lookup("java:comp/env/jdbc/datasource");
    }
}
```
* standaloneDataSource方法仅在开发概要文件中可用。
* jndiDataSource方法仅在生产概要文件中可用。

> 在@Bean方法上使用@Profile时，可能会出现一个特殊的场景:在重载具有相同Java方法名的@Bean方法的情况下(类似于构造函数重载)，需要在所有重载方法上一致声明@Profile条件。如果条件不一致，重载方法中只有第一个声明的条件才重要。因此，@Profile不能用于选择具有特定参数签名的重载方法。同一个bean的所有工厂方法之间的解析遵循Spring在创建时的构造函数解析算法。
> 
> 如果您想定义具有不同概要文件条件的可选bean，可以使用@Bean name属性，使用指向相同bean名称的不同Java方法名，如前面的示例所示。如果参数签名都是相同的(例如,所有的变量都不带参数工厂方法),这是唯一的方式来表示这种安排在第一时间有效的Java类(因为只能有一个方法的名称和参数签名)。

### XML Bean定义配置文件

对应的XML是`<beans>`元素的profile属性。我们前面的示例配置可以用两个XML文件重写，如下所示:
```xml
<beans profile="development"
    xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:jdbc="http://www.springframework.org/schema/jdbc"
    xsi:schemaLocation="...">

    <jdbc:embedded-database id="dataSource">
        <jdbc:script location="classpath:com/bank/config/sql/schema.sql"/>
        <jdbc:script location="classpath:com/bank/config/sql/test-data.sql"/>
    </jdbc:embedded-database>
</beans>
```

```xml
<beans profile="production"
    xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:jee="http://www.springframework.org/schema/jee"
    xsi:schemaLocation="...">

    <jee:jndi-lookup id="dataSource" jndi-name="java:comp/env/jdbc/datasource"/>
</beans>
```
也可以避免在同一个文件中分割和嵌套`<beans/>`元素，如下例所示:

```xml
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:jdbc="http://www.springframework.org/schema/jdbc"
    xmlns:jee="http://www.springframework.org/schema/jee"
    xsi:schemaLocation="...">

    <!-- other bean definitions -->

    <beans profile="development">
        <jdbc:embedded-database id="dataSource">
            <jdbc:script location="classpath:com/bank/config/sql/schema.sql"/>
            <jdbc:script location="classpath:com/bank/config/sql/test-data.sql"/>
        </jdbc:embedded-database>
    </beans>

    <beans profile="production">
        <jee:jndi-lookup id="dataSource" jndi-name="java:comp/env/jdbc/datasource"/>
    </beans>
</beans>
```

spring bean.XSD受到限制，只允许这些元素作为文件中的最后一个元素。这应该有助于提供灵活性，而不会在XML文件中引起混乱。

> XML对等体不支持前面描述的概要文件表达式。但是，可以通过使用!操作符。也可以通过嵌套概要文件应用逻辑“和”，如下面的示例所示:
> 
> ```xml
>  <beans xmlns="http://www.springframework.org/schema/beans"
>    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
>    xmlns:jdbc="http://www.springframework.org/schema/jdbc"
>    xmlns:jee="http://www.springframework.org/schema/jee"
>    xsi:schemaLocation="...">
>
>    <!-- other bean definitions -->
>
>    <beans profile="production">
>        <beans profile="us-east">
>            <jee:jndi-lookup id="dataSource" jndi-name="java:comp/env/jdbc/datasource"/>
>        </beans>
>    </beans>
></beans> 
> ```
> 在前面的示例中，如果生产配置文件和us-east配置文件都是活动的，则会公开dataSource bean。

### 激活一个配置文件
现在我们已经更新了配置，我们仍然需要指示Spring哪个配置文件是活动的。如果我们现在启动示例应用程序，我们将看到抛出NoSuchBeanDefinitionException，因为容器无法找到名为dataSource的Spring bean。

激活配置文件有几种方法，但最直接的方法是通过ApplicationContext可用的Environment API以编程方式执行。下面的例子展示了如何做到这一点:

```java
AnnotationConfigApplicationContext ctx = new AnnotationConfigApplicationContext();
ctx.getEnvironment().setActiveProfiles("development");
ctx.register(SomeConfig.class, StandaloneDataConfig.class, JndiDataConfig.class);
ctx.refresh();
```
此外，您还可以通过spring.profiles.active属性声明地激活概要文件，该属性可以通过web.xml中的系统环境变量、JVM系统属性、servlet上下文参数指定，甚至可以作为JNDI中的一个条目(请参阅PropertySource抽象)。在集成测试中，可以使用spring-test模块中的@ActiveProfiles注释来声明活动概要文件(请参阅带有环境概要文件的上下文配置)。

注意，配置文件不是一个“非此即彼”的命题。您可以一次激活多个配置文件。通过编程，您可以为setActiveProfiles()方法提供多个配置文件名称，该方法接受String…下面的示例激活多个配置文件:

```java
ctx.getEnvironment().setActiveProfiles("profile1", "profile2");
```
spring.profiles.active声明性地接受逗号分隔的配置文件名列表，如下例所示:

```properties
-Dspring.profiles.active="profile1,profile2"
```

### 默认配置文件

默认配置文件表示默认启用的配置文件。考虑下面的例子:

```java
@Configuration
@Profile("default")
public class DefaultDataConfig {

    @Bean
    public DataSource dataSource() {
        return new EmbeddedDatabaseBuilder()
            .setType(EmbeddedDatabaseType.HSQL)
            .addScript("classpath:com/bank/config/sql/schema.sql")
            .build();
    }
}
```

如果没有激活配置文件，则创建数据源。您可以将此视为为一个或多个bean提供默认定义的一种方法。如果启用了任何配置文件，则不应用默认配置文件。

您可以通过在Environment上使用setDefaultProfiles()或声明性地使用spring.profiles.default属性来更改默认概要文件的名称。

## PropertySource抽象
Spring的Environment抽象在属性源的可配置层次结构上提供搜索操作。考虑以下清单:

```java
ApplicationContext ctx = new GenericApplicationContext();
Environment env = ctx.getEnvironment();
boolean containsMyProperty = env.containsProperty("my-property");
System.out.println("Does my environment contain the 'my-property' property? " + containsMyProperty);
```
在前面的代码片段中，我们看到了询问Spring是否为当前环境定义了my-property属性的高级方法。为了回答这个问题，Environment对象对一组PropertySource对象执行搜索。PropertySource是对任何键值对源的简单抽象，Spring的StandardEnvironment配置了两个PropertySource对象—一个表示JVM系统属性集(system . getproperties())，另一个表示系统环境变量集(system .getenv())。

> 这些默认属性源用于StandardEnvironment，以便在独立应用程序中使用。StandardServletEnvironment使用额外的默认属性源进行填充，包括servlet配置和servlet上下文参数。它可以选择启用JndiPropertySource。详细信息请参见javadoc。

具体来说，当您使用StandardEnvironment时，如果在运行时出现了my-property系统属性或my-property环境变量，则调用environment . containsproperty ("my-property")将返回true。

> 执行的搜索是分层的。默认情况下，系统属性优先于环境变量。因此，如果在调用env.getProperty(“my-property”)期间恰好在两个地方设置了my-property属性，则系统属性值“获胜”并返回。注意，属性值并没有被合并，而是被前面的条目完全覆盖了。
> 
> 对于一个通用的standardservletenenvironment，完整的层次结构如下所示，最高优先级的条目位于顶部:
> 
> * ServletConfig参数(如果适用-例如，DispatcherServlet上下文)
> * ServletContext参数(web.xml上下文参数条目)
> * JNDI环境变量(java:comp/env/ entries)
> * JVM系统属性(-D命令行参数)
> * JVM系统环境(操作系统环境变量)

最重要的是，整个机制是可配置的。也许您有一个想要集成到此搜索中的自定义属性源。为此，实现并实例化您自己的PropertySource，并将其添加到当前环境的PropertySource集合中。下面的例子展示了如何做到这一点:

```java
ConfigurableApplicationContext ctx = new GenericApplicationContext();
MutablePropertySources sources = ctx.getEnvironment().getPropertySources();
sources.addFirst(new MyPropertySource());
```
在前面的代码中，MyPropertySource在搜索中具有最高的优先级。如果它包含my-property属性，则检测并返回该属性，这有利于任何其他PropertySource中的任何my-property属性。MutablePropertySources API公开了许多方法，允许对属性源集进行精确操作。

## Using @PropertySource
@PropertySource注释为向Spring的环境中添加PropertySource提供了一种方便的声明性机制。

给定一个名为app.properties的文件，其中包含键值对testbean.name=myTestBean，下面的@Configuration类使用@PropertySource，调用testBean.getName()返回myTestBean:

```java
@Configuration
@PropertySource("classpath:/com/myco/app.properties")
public class AppConfig {

    @Autowired
    Environment env;

    @Bean
    public TestBean testBean() {
        TestBean testBean = new TestBean();
        testBean.setName(env.getProperty("testbean.name"));
        return testBean;
    }
}
```
@PropertySource资源位置中的任何${…}占位符都会根据已经在环境中注册的属性源集进行解析，如下面的示例所示:

```java
@Configuration
@PropertySource("classpath:/com/${my.placeholder:default/path}/app.properties")
public class AppConfig {

    @Autowired
    Environment env;

    @Bean
    public TestBean testBean() {
        TestBean testBean = new TestBean();
        testBean.setName(env.getProperty("testbean.name"));
        return testBean;
    }
}
```
假设我的。占位符出现在一个已经注册的属性源中(例如，系统属性或环境变量)，占位符解析为相应的值。如果没有，则默认使用default/path。如果没有指定默认值，且无法解析属性，则抛出IllegalArgumentException。

> 根据Java 8的约定，@PropertySource注释是可重复的。但是，所有这样的@PropertySource注释需要在相同的级别声明，要么直接在配置类上声明，要么作为相同自定义注释中的元注释声明。不推荐混合直接注释和元注释，因为直接注释有效地覆盖了元注释。


## 语句中的占位符解析

过去，元素中的占位符的值只能根据JVM系统属性或环境变量解析。现在已经不是这样了。因为Environment抽象集成在整个容器中，所以很容易通过它路由占位符的解析。这意味着您可以以您喜欢的任何方式配置解析过程。您可以更改搜索系统属性和环境变量的优先级，或者完全删除它们。您还可以根据需要将自己的属性源添加到该组合中。

具体地说，无论客户财产在哪里定义，只要它在环境中可用，以下声明都有效:


```xml
<beans>
    <import resource="com/bank/service/${customer}-config.xml"/>
</beans>
```

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
