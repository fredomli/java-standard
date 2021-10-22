# 类路径扫描和组件管理

本章中的大多数示例使用XML指定在Spring容器中生成每个BeanDefinition的配置元数据。上一节(基于注释的容器配置)演示了如何通过源级注释提供大量配置元数据。然而，即使在这些示例中，“基本”bean定义也是在XML文件中显式定义的，而注释仅驱动依赖项注入。本节描述一个通过扫描类路径隐式检测候选组件的选项。候选组件是符合筛选条件的类，并且具有在容器中注册的相应bean定义。这消除了使用XML执行bean注册的需要。相反，您可以使用注释(例如@Component)、AspectJ类型表达式或您自己的自定义过滤条件来选择哪些类具有已注册到容器中的bean定义。

从Spring 3.0开始，Spring JavaConfig项目提供的许多特性都是核心Spring框架的一部分。这允许您使用Java定义bean，而不是使用传统的XML文件。查看@Configuration、@Bean、@Import和@DependsOn注释，了解如何使用这些新特性。

## @Component和进一步的构造型注解

@Repository注释是任何实现存储库角色或原型(也称为数据访问对象或DAO)的类的标记。该标记的使用包括异常的自动翻译，如异常翻译中所述。

Spring提供了进一步的原型注解:@Component、@Service和@Controller。@Component是任何spring托管组件的通用构造型。@Repository、@Service和@Controller是@Component用于更具体用例的专门化(分别在持久性、服务和表示层中)。因此，你可以用@Component来标注你的组件类，但是，通过用@Repository、@Service或@Controller来标注，你的类更适合通过工具进行处理或与方面相关联。例如，这些构造型注释是切入点的理想目标。@Repository、@Service和@Controller也可以在Spring框架的未来版本中携带额外的语义。因此，如果要在服务层使用@Component还是@Service之间进行选择，@Service显然是更好的选择。类似地，如前所述，@Repository已经被支持作为持久性层中自动异常转换的标记。

## 使用元注释和复合注释

Spring提供的许多注释都可以在您自己的代码中用作元注释。元注释是可以应用于另一个注释的注释。例如，前面提到的@Service注释是用@Component元注释的，如下例所示

```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Component 
public @interface Service {

    // ...
}
```
@Component会让@Service和@Component一样被处理。

您还可以组合元注释来创建“组合注释”。例如，来自Spring MVC的@RestController注释由@Controller和@ResponseBody组成。

此外，组合注释可以选择性地重新声明元注释的属性，以允许自定义。当您只想公开元注释属性的子集时，这可能特别有用。例如，Spring的@SessionScope注释将作用域名称硬编码为会话，但仍然允许定制proxyMode。下面的清单显示了SessionScope注释的定义:

```java
@Target({ElementType.TYPE, ElementType.METHOD})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Scope(WebApplicationContext.SCOPE_SESSION)
public @interface SessionScope {

    /**
     * Alias for {@link Scope#proxyMode}.
     * <p>Defaults to {@link ScopedProxyMode#TARGET_CLASS}.
     */
    @AliasFor(annotation = Scope.class)
    ScopedProxyMode proxyMode() default ScopedProxyMode.TARGET_CLASS;

}
```
然后你可以使用@SessionScope而不需要声明proxyMode，如下所示:

```java
@Service
@SessionScope
public class SessionScopedService {
    // ...
}
```

你也可以覆盖proxyMode的值，如下例所示:

```java
@Service
@SessionScope(proxyMode = ScopedProxyMode.INTERFACES)
public class SessionScopedUserService implements UserService {
    // ...
}
```
要了解更多细节，请参见Spring Annotation Programming Model wiki页面。

## 自动检测类和注册Bean定义
Spring可以自动检测构造型类并向ApplicationContext注册相应的BeanDefinition实例。例如，以下两个类符合这种自动检测条件:

```java
@Service
public class SimpleMovieLister {

    private MovieFinder movieFinder;

    public SimpleMovieLister(MovieFinder movieFinder) {
        this.movieFinder = movieFinder;
    }
}
```

```java
@Repository
public class JpaMovieFinder implements MovieFinder {
    // implementation elided for clarity
}
```
要自动检测这些类并注册相应的bean，需要将@ComponentScan添加到@Configuration类中，其中basePackages属性是这两个类的公共父包。(或者，您可以指定一个逗号或分号或空格分隔的列表，其中包括每个类的父包。)

```java
@Configuration
@ComponentScan(basePackages = "org.example")
public class AppConfig  {
    // ...
}
```
为简洁起见，前面的示例可以使用注释的value属性(即@ComponentScan("org.example"))。

下替代方法使用XML:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:context="http://www.springframework.org/schema/context"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/context
        https://www.springframework.org/schema/context/spring-context.xsd">

    <context:component-scan base-package="org.example"/>

</beans>
```

> 使用`<context:component-scan>`隐式地启用了`<context:annotation-config>`的功能。当使用`<context:component-scan>`时，通常不需要包含`<context:annotation-config>`元素。

> 类路径包的扫描需要类路径中存在相应的目录项。当您使用Ant构建JAR时，请确保您没有激活JAR任务的只文件开关。此外，在某些环境中，基于安全策略，类路径目录可能不会公开——例如，JDK 1.7.0_45或更高版本上的独立应用程序(这需要在您的清单中设置“Trusted-Library”——参见https://stackoverflow.com/questions/19394570/java-jre-7u45-breaks-classloader-getresources)。
> 
> 在JDK 9的模块路径(Jigsaw)上，Spring的类路径扫描通常按照预期工作。但是，请确保在模块信息描述符中导出了组件类。如果希望Spring调用类的非公共成员，请确保它们是“打开的”(也就是说，它们在模块信息描述符中使用一个“打开”声明，而不是一个“导出”声明)。

此外，当您使用组件扫描元素时，AutowiredAnnotationBeanPostProcessor和CommonAnnotationBeanPostProcessor都是隐式包含的。这意味着这两个组件被自动检测并连接在一起—所有这些都不需要在XML中提供任何bean配置元数据。

通过包含值为false的注释配置属性，可以禁用AutowiredAnnotationBeanPostProcessor和CommonAnnotationBeanPostProcessor的注册。

## 使用过滤器自定义扫描

默认情况下，使用@Component、@Repository、@Service、@Controller、@Configuration标注的类，或者使用@Component标注的自定义标注是唯一检测到的候选组件。但是，您可以通过应用自定义过滤器来修改和扩展此行为。将它们添加为@ComponentScan注释的includeFilters或excludeFilters属性(或在XML配置中作为<context:component-scan>元素的<context:include-filter />或<context:exclude-filter />子元素)。每个筛选器元素都需要type和expression属性。过滤选项如下表所示:

|Filter Type|Example Expression|Description|
|---|---|---|
|annotation (default)|org.example.SomeAnnotation|在目标组件的类型级别上出现的注释或元注释。|
|assignable|org.example.SomeClass|目标组件可分配(扩展或实现)的类(或接口)。|
|aspectj|org.example..*Service+|由目标组件匹配的AspectJ类型表达式。|
|regex|org\\.example\\.Default.*|由目标组件的类名匹配的正则表达式。|
|custom|org.example.MyTypeFilter|org.springframework.core.type.TypeFilter接口的自定义实现。|

下面的示例显示了忽略所有@Repository注释并使用“存根”存储库的配置:

```java
@Configuration
@ComponentScan(basePackages = "org.example",
        includeFilters = @Filter(type = FilterType.REGEX, pattern = ".*Stub.*Repository"),
        excludeFilters = @Filter(Repository.class))
public class AppConfig {
    // ...
}
```
下面的清单显示了等价的XML:

```xml
<beans>
    <context:component-scan base-package="org.example">
        <context:include-filter type="regex"
                                expression=".*Stub.*Repository"/>
        <context:exclude-filter type="annotation"
                                expression="org.springframework.stereotype.Repository"/>
    </context:component-scan>
</beans>
```

> 您还可以通过在注释上设置useDefaultFilters=false或通过提供use-default-filters="false"作为<component-scan/>元素的属性来禁用默认过滤器。这有效地禁用了对@Component、@Repository、@Service、@Controller、@RestController或@Configuration注释或元注释的类的自动检测。

## 在组件中定义Bean元数据

Spring组件还可以向容器提供bean定义元数据。您可以使用与在@Configuration注释类中定义bean元数据相同的@Bean注释来实现这一点。下面的例子展示了如何做到这一点:

```java
@Component
public class FactoryMethodComponent {

    @Bean
    @Qualifier("public")
    public TestBean publicInstance() {
        return new TestBean("publicInstance");
    }

    public void doWork() {
        // Component method implementation omitted
    }
}
```

前面的类是一个Spring组件，它的doWork()方法中包含特定于应用程序的代码。但是，它还提供了一个bean定义，该bean定义有一个引用方法publicInstance()的工厂方法。@Bean注释标识工厂方法和其他bean定义属性，例如通过@Qualifier注释标识一个限定符值。其他可以指定的方法级注释有@Scope、@Lazy和自定义限定符注释。

> 除了它在组件初始化中的作用外，您还可以将@Lazy注释放置在用@Autowired或@Inject标记的注入点上。在这个上下文中，它会导致惰性解析代理的注入。

如前所述，支持自动连接字段和方法，还支持@Bean方法的自动连接。下面的例子展示了如何做到这一点:

```java
@Component
public class FactoryMethodComponent {

    private static int i;

    @Bean
    @Qualifier("public")
    public TestBean publicInstance() {
        return new TestBean("publicInstance");
    }

    // use of a custom qualifier and autowiring of method parameters
    @Bean
    protected TestBean protectedInstance(
            @Qualifier("public") TestBean spouse,
            @Value("#{privateInstance.age}") String country) {
        TestBean tb = new TestBean("protectedInstance", 1);
        tb.setSpouse(spouse);
        tb.setCountry(country);
        return tb;
    }

    @Bean
    private TestBean privateInstance() {
        return new TestBean("privateInstance", i++);
    }

    @Bean
    @RequestScope
    public TestBean requestScopedInstance() {
        return new TestBean("requestScopedInstance", 3);
    }
}
```
该示例将String方法参数country自动连接到另一个名为privateInstance的bean上的age属性的值。Spring Expression Language元素通过符号#{< Expression >}定义属性的值。对于@Value注释，表达式解析器被预先配置为在解析表达式文本时查找bean名称。

从Spring Framework 4.3开始，您还可以声明一个类型为InjectionPoint(或其更具体的子类:DependencyDescriptor)的工厂方法参数，以访问触发当前bean创建的请求注入点。注意，这只适用于bean实例的实际创建，而不适用于现有实例的注入。因此，这个特性对于原型范围的bean最有意义。对于其他作用域，工厂方法只看到在给定作用域中触发创建新bean实例的注入点(例如，触发创建惰性单例bean的依赖项)。在这种情况下，您可以使用提供的注入点元数据并注意语义。下面的例子展示了如何使用InjectionPoint:

```java
@Component
public class FactoryMethodComponent {

    @Bean @Scope("prototype")
    public TestBean prototypeInstance(InjectionPoint injectionPoint) {
        return new TestBean("prototypeInstance for " + injectionPoint.getMember());
    }
}
```
常规Spring组件中的@Bean方法的处理方式与Spring @Configuration类中的对应方法不同。不同之处在于@Component类没有使用CGLIB进行增强以拦截方法和字段的调用。CGLIB代理是通过调用@Configuration类中的@Bean方法中的方法或字段创建到协作对象的bean元数据引用的方法。这些方法不是用普通的Java语义调用的，而是通过容器来提供Spring bean的常用生命周期管理和代理，甚至在通过对@Bean方法的编程调用引用其他bean时也是如此。相比之下，在普通的@Component类中调用@Bean方法中的方法或字段具有标准的Java语义，不需要应用特殊的CGLIB处理或其他约束。

> 可以将@Bean方法声明为静态方法，这样就可以在不将包含它们的配置类创建为实例的情况下调用它们。这在定义后处理器bean(例如，类型为BeanFactoryPostProcessor或BeanPostProcessor)时特别有意义，因为这样的bean在容器生命周期的早期被初始化，应该避免在那个时候触发配置的其他部分。
> 
> 对静态@Bean方法的调用从来不会被容器拦截，甚至在@Configuration类中也不会(如本节前面所述)，这是由于技术限制:CGLIB子类化只能覆盖非静态方法。因此，直接调用另一个@Bean方法具有标准的Java语义，导致直接从工厂方法本身返回独立的实例。
> 
> @Bean方法的Java语言可见性对Spring容器中的结果bean定义没有直接影响。您可以自由地在non-@Configuration类中声明您的工厂方法，也可以在任何地方声明静态方法。但是，@Configuration类中的常规@Bean方法需要重写—也就是说，它们不能声明为private或final。
> 
> 在给定组件或配置类的基类上，以及在组件或配置类实现的接口中声明的Java 8默认方法上，也会发现@Bean方法。这允许在组合复杂的配置安排时具有很大的灵活性，甚至可以通过Spring 4.2中的Java 8默认方法实现多重继承。
> 
> 最后，单个类可能包含同一个bean的多个@Bean方法，这是根据运行时可用的依赖关系安排的多个工厂方法。这与在其他配置场景中选择“最贪婪的”构造函数或工厂方法的算法相同:在构造时选择具有最大数量可满足依赖的变量，类似于容器在多个@Autowired构造函数之间的选择。

## 命名个组件

当一个组件作为扫描过程的一部分被自动检测时，它的bean名称将由该扫描程序已知的BeanNameGenerator策略生成。默认情况下，任何包含name值的Spring构造型注释(@Component、@Repository、@Service和@Controller)因此将该名称提供给相应的bean定义。

如果这样的注释不包含任何名称值或任何其他检测到的组件(例如自定义过滤器发现的组件)，默认bean名称生成器将返回未大写的非限定类名。例如，如果检测到以下组件类，则名称为myMovieLister和movieFinderImpl:
```java
@Service("myMovieLister")
public class SimpleMovieLister {
    // ...
}
```
```java
@Repository
public class MovieFinderImpl implements MovieFinder {
    // ...
}
```
如果不想依赖默认的bean命名策略，可以提供自定义的bean命名策略。首先，实现BeanNameGenerator接口，并确保包含一个默认的无参数构造函数。然后，在配置扫描程序时提供完全限定的类名，如下面的注释和bean定义示例所示。

> 如果由于多个自动检测的组件具有相同的非限定类名(即具有相同名称但驻留在不同包中的类)而导致命名冲突，则可能需要配置一个BeanNameGenerator，默认为生成的bean名的完全限定类名。从Spring Framework 5.2.3开始，位于package org.springframework.context.annotation中的fulllyqualifiedannotationbeannamegenerator就可以用于这些目的。


```java
@Configuration
@ComponentScan(basePackages = "org.example", nameGenerator = MyNameGenerator.class)
public class AppConfig {
    // ...
}
```
```xml
<beans>
    <context:component-scan base-package="org.example"
        name-generator="org.example.MyNameGenerator" />
</beans>
```
作为一般规则，考虑在其他组件显式引用它时使用注释指定名称。另一方面，只要容器负责连接，自动生成的名称就足够了。

## 为自动检测组件提供范围

与spring管理的组件一样，自动检测组件的默认和最常见的作用域是单例的。然而，有时您需要一个可以由@Scope注释指定的不同范围。您可以在注释中提供作用域的名称，如下面的示例所示:

```java
@Scope("prototype")
@Repository
public class MovieFinderImpl implements MovieFinder {
    // ...
}
```
> @Scope注释只在具体bean类(对于带注释的组件)或工厂方法(对于@Bean方法)上自省。与XML bean定义相反，没有bean定义继承的概念，类级别的继承层次对于元数据的目的是无关的。

关于特定于web的作用域的详细信息，如Spring上下文中的“请求”或“会话”，请参见请求、会话、应用和WebSocket作用域。与那些作用域的预构建注释一样，您也可以通过使用Spring的元注释方法来编写自己的作用域注释:例如，使用@Scope(“prototype”)注释的自定义注释元，还可能声明自定义作用域代理模式。

> 为了提供范围解析的自定义策略，而不是依赖于基于注释的方法，您可以实现ScopeMetadataResolver接口。确保包含一个默认的无参数构造函数。然后，您可以在配置扫描程序时提供完全限定的类名，如下面的注释和bean定义示例所示:

```java
@Configuration
@ComponentScan(basePackages = "org.example", scopeResolver = MyScopeResolver.class)
public class AppConfig {
    // ...
}
```
```xml
<beans>
    <context:component-scan base-package="org.example" scope-resolver="org.example.MyScopeResolver"/>
</beans>
```
当使用某些非单例作用域时，可能需要为作用域对象生成代理。其原因在作用域bean中描述为依赖关系。为此目的，可以在组件扫描元素上使用作用域代理属性。三个可能的值是:no、interfaces和targetClass。例如，下面的配置将导致标准JDK动态代理:

```java
@Configuration
@ComponentScan(basePackages = "org.example", scopedProxy = ScopedProxyMode.INTERFACES)
public class AppConfig {
    // ...
}
```
```xml
<beans>
    <context:component-scan base-package="org.example" scoped-proxy="interfaces"/>
</beans>
```

## 使用注释提供限定符元数据
@Qualifier注释在使用qualifier的基于微调注释的自动装配中进行了讨论。该节中的示例演示了@Qualifier注释和自定义限定符注释的使用，以便在解析自动装配候选项时提供细粒度控制。因为这些示例是基于XML bean定义的，所以限定符元数据是通过使用XML中bean元素的限定符或元子元素在候选bean定义上提供的。当依赖类路径扫描来自动检测组件时，您可以在候选类上提供带有类型级别注释的限定符元数据。下面三个例子演示了这种技术:

```java
@Component
@Qualifier("Action")
public class ActionMovieCatalog implements MovieCatalog {
    // ...
}
```
```java
@Component
@Genre("Action")
public class ActionMovieCatalog implements MovieCatalog {
    // ...
}
```
```java
@Component
@Offline
public class CachingMovieCatalog implements MovieCatalog {
    // ...
}
```

与大多数基于注释的替代方法一样，请记住注释元数据是绑定到类定义本身的，而XML的使用允许同一类型的多个bean在其限定符元数据中提供变化，因为元数据是按实例而不是按类提供的。

## 生成候选组件索引

虽然类路径扫描非常快，但是可以通过在编译时创建一个静态候选列表来提高大型应用程序的启动性能。在此模式下，所有作为组件扫描目标的模块都必须使用此机制。

> 您现有的@ComponentScan或`<context:component-scan/>`指令必须保持不变，以请求上下文扫描某些包中的候选包。当ApplicationContext检测到这样的索引时，它会自动使用它，而不是扫描类路径。

要生成索引，请向每个包含组件扫描指令目标组件的模块添加额外的依赖项。下面的例子展示了如何使用Maven:

```xml
<dependencies>
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-context-indexer</artifactId>
        <version>5.3.12</version>
        <optional>true</optional>
    </dependency>
</dependencies>
```
spring-context-indexer工件生成jar文件中包含的META-INF/spring.components文件。

> >在IDE中使用此模式时，必须将spring上下文索引器注册为注释处理器，以确保在更新候选组件时索引是最新的。

> 当在类路径中找到META-INF/spring.components文件时，该索引将自动启用。如果索引是部分可用于一些图书馆(或者用例),但不能为整个应用程序,你可以回到常规类路径安排(好像没有索引在场)通过设置spring.index.ignore为true,要么作为一个JVM系统属性或通过SpringProperties机制。

> * [使用Spring MVC服务Web内容](https://spring.io/guides/gs/serving-web-content/)
> * [Spring安全架构(参考指南)](https://spring.io/guides/topicals/spring-security-architecture/)
> * [Spring Framework官方文档](https://spring.io/projects/spring-framework)
> * [SpringIoc简介](https://github.com/fredomli/java-standard/blob/main/docs/spring/spring/core/SpringIoc容器.md)
> * [Spring容器使用](https://github.com/fredomli/java-standard/blob/main/docs/spring/spring/core/Spring容器使用.md)
> * [SpringIoc简介](https://github.com/fredomli/java-standard/blob/main/docs/spring/spring/core/SpringIoc容器.md)
> * [SpringBean作用域](https://github.com/fredomli/java-standard/blob/main/docs/spring/spring/core/SpringBean作用域.md)
> * [Spring基于注解的容器配置](https://github.com/fredomli/java-standard/blob/main/docs/spring/spring/core/Spring基于注解的容器配置.md)


*(☞ﾟヮﾟ)☞[返回首页README.md address](https://github.com/fredomli/java-standard)*
