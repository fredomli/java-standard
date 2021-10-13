#Spring IOC 容器

参考文档的这一部分涵盖了Spring框架绝对不可或缺的所有技术。
其中最重要的是Spring框架的控制反转(IoC)容器。在对Spring框架的IoC容器进行全面讨论之后，还对Spring的面向方面编程(AOP)技术进行了全面的介绍。Spring框架有它自己的AOP框架，它在概念上容易理解，并且成功地解决了Java企业编程中80%的AOP需求的最佳点。

## Spring IoC容器和bean简介
本章介绍了Spring框架实现的反转控制(IoC)原理。IoC也称为依赖注入(DI)。在此过程中，对象仅通过构造函数参数、工厂方法参数或在对象实例被构造或从工厂方法返回后设置的属性来定义它们的依赖项(即它们使用的其他对象)。然后容器在创建bean时注入这些依赖项。

从根本上说，这个过程是bean本身通过直接构造类或Service Locator模式等机制来控制依赖项的实例化或位置的相反过程(因此得名“控制倒置”)。

org.springframework.beans和org.springframework.context包是Spring框架的IoC容器的基础。BeanFactory接口提供了一种高级的配置机制，能够管理任何类型的对象。ApplicationContext是BeanFactory的子接口。它补充道:

* 更容易与Spring的AOP特性集成
* 消息资源处理(用于国际化)
* 事件发布
* 应用程序层特定的上下文，如WebApplicationContext用于web应用程序。

简而言之，BeanFactory提供了配置框架和基本功能，而ApplicationContext添加了更多特定于企业的功能。ApplicationContext是BeanFactory的一个完整超集，仅在本章描述Spring的IoC容器时使用。有关使用BeanFactory而不是ApplicationContext的更多信息，请参阅[BeanFactory](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-beanfactory) 。 

在Spring中，由Spring IoC容器管理的构成应用程序主干的对象称为bean。bean是由Spring IoC容器实例化、组装和管理的对象。否则，bean只是应用程序中的众多对象之一。bean及其之间的依赖关系反映在容器使用的配置元数据中。

## 容器概述
`org.springframework.context.ApplicationContext`接口表示Spring IoC容器，并负责实例化、配置和组装bean。容器通过读取配置元数据获得关于要实例化、配置和组装哪些对象的指令。配置元数据以XML、Java注释或Java代码表示。它允许您表达组成应用程序的对象以及这些对象之间丰富的相互依赖关系。

Spring提供了ApplicationContext接口的几个实现。在独立应用程序中，创建ClassPathXmlApplicationContext或FileSystemXmlApplicationContext的实例是很常见的。虽然XML一直是定义配置元数据的传统格式，但您可以通过提供少量的XML配置以声明方式支持这些额外的元数据格式，指示容器使用Java注释或代码作为元数据格式。

在大多数应用程序场景中，不需要显式用户代码来实例化Spring IoC容器的一个或多个实例。例如，在一个web应用程序场景中，应用程序的web. XML文件中简单的8行(大约)web描述符XML样板文件就足够了(参见web应用程序的方便应用程序上下文实例化)。如果您使用Eclipse的Spring Tools(一个基于Eclipse的开发环境)，那么只需点击几下鼠标或敲击几下键盘，就可以轻松地创建这个样板配置。

一个简单的web.xml文件：  
```xml
<context-param>
    <param-name>contextConfigLocation</param-name>
    <param-value>/WEB-INF/daoContext.xml /WEB-INF/applicationContext.xml</param-value>
</context-param>

<listener>
    <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
</listener>
```
下面的图表显示了Spring如何工作的高级视图。您的应用程序类与配置元数据相结合，以便在创建和初始化ApplicationContext之后，您就拥有了一个完全配置的可执行系统或应用程序。

![container-magic.png](https://gitee.com/fredomli/fredomli-picture/raw/picgo/static/images/wordpress/container-magic.png)


## 配置元数据
如上面的图表所示，Spring IoC容器使用一种配置元数据形式。这个配置元数据表示作为应用程序开发人员，您如何告诉Spring容器实例化、配置和组装应用程序中的对象。

配置元数据传统上以简单而直观的XML格式提供，本章的大部分内容都使用这种格式来传达Spring IoC容器的关键概念和特性。

> 基于xml的元数据并不是唯一允许的配置元数据形式。Spring IoC容器本身与实际编写配置元数据的格式完全解耦。现在，许多开发人员为他们的Spring应用程序选择基于java的配置。

示例：
```java
@Configuration
public class AppConfig {

    @Bean
    public MyService myService() {
        return new MyServiceImpl();
    }
}
```

前面的AppConfig类相当于下面的Spring <beans/> XML:

```xml
<beans>
    <bean id="myService" class="com.acme.services.MyServiceImpl"/>
</beans>
```
* 基于注释的配置:Spring 2.5引入了对基于注释的配置元数据的支持。
* 基于java的配置:从Spring 3.0开始，Spring JavaConfig项目提供的许多特性都成为了核心Spring框架的一部分。因此，您可以通过使用Java而不是XML文件来定义应用程序类外部的bean。要使用这些新特性，请参阅@Configuration、@Bean、@Import和@DependsOn注释。

Spring配置由容器必须管理的至少一个(通常是多个)bean定义组成。基于xml的配置元数据将这些bean配置为顶层<beans/>元素中的<bean/>元素。Java配置通常在@Configuration类中使用@ bean注释的方法。

这些bean定义对应于组成应用程序的实际对象。通常，您需要定义服务层对象、数据访问对象(dao)、表示对象(如Struts Action实例)、基础设施对象(如Hibernate SessionFactories)、JMS队列，等等。通常，我们不会在容器中配置细粒度的域对象，因为创建和加载域对象通常是dao和业务逻辑的职责。但是，您可以使用Spring与AspectJ的集成来配置在IoC容器控制之外创建的对象。

```xml  
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="..." class="...">  
        <!-- collaborators and configuration for this bean go here -->
    </bean>

    <bean id="..." class="...">
        <!-- collaborators and configuration for this bean go here -->
    </bean>

    <!-- more bean definitions go here -->

</beans>
```
* id属性是一个字符串，用于标识各个bean定义。
* class属性定义bean的类型，并使用完全限定的类名。

id属性的值指的是协作对象。本例中没有显示用于引用协作对象的XML。有关更多信息，请参阅依赖性。

## 实例化一个容器
提供给ApplicationContext构造函数的位置路径或路径是资源字符串，允许容器从各种外部资源(如本地文件系统、Java CLASSPATH等)加载配置元数据。

```java
ApplicationContext context = new ClassPathXmlApplicationContext("services.xml", "daos.xml");
```
> 在了解了Spring的IoC容器之后，您可能想了解更多关于Spring的Resource抽象(如参考资料中所述)的信息，它提供了一种方便的机制，可以从URI语法中定义的位置读取InputStream。特别是，资源路径用于构造应用程序上下文，如应用程序上下文和资源路径中所述。

以下示例显示了服务层对象(services.xml)配置文件:  

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd">

    <!-- services -->

    <bean id="petStore" class="org.springframework.samples.jpetstore.services.PetStoreServiceImpl">
        <property name="accountDao" ref="accountDao"/>
        <property name="itemDao" ref="itemDao"/>
        <!-- additional collaborators and configuration for this bean go here -->
    </bean>

    <!-- more bean definitions for services go here -->

</beans>
```

下面的示例展示了数据访问对象dao .xml文件:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="accountDao"
        class="org.springframework.samples.jpetstore.dao.jpa.JpaAccountDao">
        <!-- additional collaborators and configuration for this bean go here -->
    </bean>

    <bean id="itemDao" class="org.springframework.samples.jpetstore.dao.jpa.JpaItemDao">
        <!-- additional collaborators and configuration for this bean go here -->
    </bean>

    <!-- more bean definitions for data access objects go here -->

</beans>
```
在前面的示例中，服务层由PetStoreServiceImpl类和两个类型为JpaAccountDao和JpaItemDao的数据访问对象组成(基于JPA对象-关系映射标准)。property name元素引用JavaBean属性的名称，ref元素引用另一个bean定义的名称。id和ref元素之间的这种链接表示协作对象之间的依赖关系。有关配置对象依赖项的详细信息，请参见依赖项。

### 组合基于xml的配置元数据
让bean定义跨越多个XML文件可能很有用。通常，每个XML配置文件代表体系结构中的逻辑层或模块。

您可以使用应用程序上下文构造函数从所有这些XML片段加载bean定义。这个构造函数接受多个Resource位置，如上一节所示。或者，使用<import/>元素的一次或多次出现来从另一个或多个文件加载bean定义。下面的例子展示了如何做到这一点:

```xml
<beans>
    <import resource="services.xml"/>
    <import resource="resources/messageSource.xml"/>
    <import resource="/resources/themeSource.xml"/>

    <bean id="bean1" class="..."/>
    <bean id="bean2" class="..."/>
</beans>
```

在前面的示例中，外部bean定义从三个文件加载:services.xml、messageSource.xml和themeSource.xml。所有位置路径都是相对于执行导入的定义文件而言的，因此services.xml必须位于执行导入的文件相同的目录或类路径位置，而messageSource.xml和themeSource.xml必须位于导入文件下方的资源位置。

如您所见，前导斜杠被忽略。但是，考虑到这些路径是相对的，最好完全不使用斜杠。根据Spring Schema，要导入的文件的内容(包括顶级的<beans/>元素)必须是有效的XML bean定义。

> 使用相对的"../”路径。s引用父目录中的文件是可能的，但不推荐使用。这样做会在当前应用程序之外的文件上创建一个依赖项。特别地，这个引用不推荐用于类路径:url(例如，classpath:../services.xml)，运行时解析过程选择“最近的”类路径根，然后查看它的父目录。类路径配置更改可能导致选择不同的、不正确的目录。
>
> 您总是可以使用完全限定的资源位置而不是相对路径:例如，file:C:/config/services.xml或classpath:/config/services.xml。但是，请注意，您将应用程序的配置耦合到特定的绝对位置。对于这样的绝对位置，通常最好保持一个间接的位置——例如，通过“${…}”占位符，在运行时根据JVM系统属性解析。

名称空间本身提供了导入指令特性。除了普通bean定义之外，Spring还提供了一些XML名称空间选项，例如上下文和util名称空间。



> 参阅
> * [使用Spring MVC服务Web内容](https://spring.io/guides/gs/serving-web-content/)
> * [Spring安全架构(参考指南)](https://spring.io/guides/topicals/spring-security-architecture/)
> * [Spring Framework官方文档](https://spring.io/projects/spring-framework)

*(☞ﾟヮﾟ)☞[返回首页README.md address](https://github.com/fredomli/java-standard)*
