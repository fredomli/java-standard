# 基于注解的容器配置

> 在配置Spring时，注释比XML更好吗?  
> 引入基于注释的配置提出了这样一个问题:这种方法是否比XML“更好”。简短的回答是“视情况而定”。长期的答案是，每种方法都有其优点和缺点，通常，由开发人员决定哪种策略更适合他们。由于注释的定义方式，注释在其声明中提供了大量上下文，从而导致配置更简短、更简洁。  
> 然而，XML擅长连接组件，而无需修改它们的源代码或重新编译它们。一些开发人员更喜欢接近源代码进行连接，而另一些开发人员则认为带注释的类不再是pojo，而且配置变得分散且更难控制。  
> 
> 无论哪种选择，Spring都可以容纳这两种风格，甚至将它们混合在一起。值得指出的是，通过它的JavaConfig选项，Spring允许以一种非侵入性的方式使用注释，而不涉及目标组件的源代码，而且，就工具而言，所有配置样式都由用于Eclipse的Spring Tools支持。


基于注释的配置提供了XML设置的另一种选择，它依赖于连接组件的字节码元数据，而不是尖括号声明。开发人员不使用XML来描述bean连接，而是通过在相关类、方法或字段声明上使用注释将配置转移到组件类本身。正如在示例:AutowiredAnnotationBeanPostProcessor中所提到的，将BeanPostProcessor与注释结合使用是扩展Spring IoC容器的常用方法。例如，Spring 2.0引入了使用@Required注释强制执行必需属性的可能性。Spring 2.5使遵循同样的通用方法来驱动Spring的依赖注入成为可能。本质上，@Autowired注释提供了与Autowiring collaborator中描述的相同的功能，但具有更细粒度的控制和更广泛的适用性。Spring 2.5还增加了对JSR-250注释的支持，比如@PostConstruct和@PreDestroy。Spring 3.0增加了对javax中包含的JSR-330 (Java依赖注入)注释的支持。例如@Inject和@Named。有关这些注释的详细信息可以在相关部分找到。

> 注释注入在XML注入之前执行。因此，XML配置覆盖了通过两种方法连接的属性的注释。

和往常一样，您可以将post-processor注册为单个bean定义，但也可以通过在基于xml的Spring配置中包含以下标记来隐式注册它们(注意包含了上下文名称空间):

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:context="http://www.springframework.org/schema/context"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/context
        https://www.springframework.org/schema/context/spring-context.xsd">

    <context:annotation-config/>

</beans>
```
`<context:annotation-config/>`元素隐式注册以下后处理器:

* ConfigurationClassPostProcessor
* AutowiredAnnotationBeanPostProcessor
* CommonAnnotationBeanPostProcessor
* PersistenceAnnotationBeanPostProcessor
* EventListenerMethodProcessor

`<context:annotation-config/>`只在定义它的应用程序上下文中查找bean上的注释。这意味着，如果您将`<context:annotation-config/>`放在DispatcherServlet的WebApplicationContext中，它只检查您的控制器中的@Autowired bean，而不是您的服务。更多信息请参见DispatcherServlet。

## @Required
@Required注释应用于bean属性设置方法，如下例所示:

```java
public class SimpleMovieLister {

    private MovieFinder movieFinder;

    @Required
    public void setMovieFinder(MovieFinder movieFinder) {
        this.movieFinder = movieFinder;
    }

    // ...
}
```

此注释指出，必须在配置时通过bean定义中的显式属性值或通过自动装配来填充受影响的bean属性。如果未填充受影响的bean属性，容器将抛出异常。这允许紧急和显式的失败，避免以后出现NullPointerException或类似的实例。我们仍然建议将断言放入bean类本身(例如，放入init方法)。这样做将强制执行那些必需的引用和值，即使在容器外部使用该类时也是如此。

必须将RequiredAnnotationBeanPostProcessor注册为一个bean，以支持@Required注释。

> @ required注释和RequiredAnnotationBeanPostProcessor正式弃用Spring框架5.1,使用构造函数注入所需设置的(或一个自定义实现InitializingBean.afterPropertiesSet()或一个自定义@PostConstruct method)和bean属性setter方法)。

## 使用 @Autowired
> 在本节包含的示例中，JSR 330的@Inject注释可以用来代替Spring的@Autowired注释。点击这里了解更多细节。

你可以将@Autowired注释应用到构造函数中，如下例所示:
```java
public class MovieRecommender {

    private final CustomerPreferenceDao customerPreferenceDao;

    @Autowired
    public MovieRecommender(CustomerPreferenceDao customerPreferenceDao) {
        this.customerPreferenceDao = customerPreferenceDao;
    }

    // ...
}
```

从Spring Framework 4.3开始，如果目标bean只定义一个构造函数开始，就不再需要在这样的构造函数上使用@Autowired注释。但是，如果有几个构造函数可用，并且没有主/默认构造函数，那么至少有一个构造函数必须使用@Autowired来注释，以便指示容器使用哪个构造函数。有关细节，请参阅构造函数解析的讨论。

你也可以将@Autowired注解应用到传统的setter方法上，如下面的例子所示:

```java
public class SimpleMovieLister {

    private MovieFinder movieFinder;

    @Autowired
    public void setMovieFinder(MovieFinder movieFinder) {
        this.movieFinder = movieFinder;
    }

    // ...
}
```

您还可以将注释应用于具有任意名称和多个参数的方法，如下面的示例所示:

```java
public class MovieRecommender {

    private MovieCatalog movieCatalog;

    private CustomerPreferenceDao customerPreferenceDao;

    @Autowired
    public void prepare(MovieCatalog movieCatalog,
            CustomerPreferenceDao customerPreferenceDao) {
        this.movieCatalog = movieCatalog;
        this.customerPreferenceDao = customerPreferenceDao;
    }

    // ...
}
```
你也可以将@Autowired应用到字段中，甚至将其与构造函数混合使用，如下面的示例所示:

```java
public class MovieRecommender {

    private final CustomerPreferenceDao customerPreferenceDao;

    @Autowired
    private MovieCatalog movieCatalog;

    @Autowired
    public MovieRecommender(CustomerPreferenceDao customerPreferenceDao) {
        this.customerPreferenceDao = customerPreferenceDao;
    }

    // ...
}
```
> 确保您的目标组件(例如MovieCatalog或CustomerPreferenceDao)始终是由您用于@ autowiredannotated注入点的类型声明的。否则，注入可能会在运行时由于“没有找到类型匹配”错误而失败。  
> 
> 对于通过类路径扫描找到的xml定义的bean或组件类，容器通常事先知道具体的类型。然而，对于@Bean工厂方法，您需要确保声明的返回类型具有足够的表达性。对于实现多个接口的组件，或者对于可能由其实现类型引用的组件，考虑在工厂方法上声明最特定的返回类型(至少按照引用bean的注入点所要求的特定类型)。

您还可以通过向期望该类型数组的字段或方法添加@Autowired注释，指示Spring从ApplicationContext提供特定类型的所有bean，如下例所示:

```java
public class MovieRecommender {

    @Autowired
    private MovieCatalog[] movieCatalogs;

    // ...
}
```

这同样适用于类型化集合，如下例所示:

```java
public class MovieRecommender {

    private Set<MovieCatalog> movieCatalogs;

    @Autowired
    public void setMovieCatalogs(Set<MovieCatalog> movieCatalogs) {
        this.movieCatalogs = movieCatalogs;
    }

    // ...
}
```
> 注意：
> 如果希望数组或列表中的项按特定顺序排序，您的目标bean可以实现org.springframework.core.Ordered接口，或者使用@Order或标准@Priority注释。否则，它们的顺序遵循容器中相应目标bean定义的注册顺序。
> 
> 您可以在目标类级别和@Bean方法上声明@Order注释，可能是针对单个bean定义(在多个定义使用同一bean类的情况下)。@Order值可能会影响注入点的优先级，但请注意，它们不会影响单例启动顺序，单例启动顺序是由依赖关系和@DependsOn声明决定的正交问题。
> 
> 注意，标准javax.annotation.Priority注释在@Bean级别不可用，因为它不能在方法上声明。它的语义可以通过在每个类型的单个bean上结合@Order值和@Primary来建模。

即使是类型化的Map实例也可以自动连接，只要预期的键类型是String。映射值包含所有期望类型的bean，键包含相应的bean名称，如下例所示:

```java
public class MovieRecommender {

    private Map<String, MovieCatalog> movieCatalogs;

    @Autowired
    public void setMovieCatalogs(Map<String, MovieCatalog> movieCatalogs) {
        this.movieCatalogs = movieCatalogs;
    }

    // ...
}
```
默认情况下，当给定注入点没有匹配的候选bean可用时，自动装配将失败。对于声明的数组、集合或映射，至少需要一个匹配的元素。

默认的行为是将注释的方法和字段视为必需的依赖项。你可以像下面的例子中演示的那样改变这个行为，通过将一个非必需的注入点标记为非必需的，从而使框架跳过一个不可满足的注入点(例如，通过将@Autowired中的required属性设置为false):

```java
public class SimpleMovieLister {

    private MovieFinder movieFinder;

    @Autowired(required = false)
    public void setMovieFinder(MovieFinder movieFinder) {
        this.movieFinder = movieFinder;
    }

    // ...
}
```
如果一个非必需的方法的依赖项(或者它的一个依赖项，在有多个参数的情况下)不可用，那么它根本不会被调用。在这种情况下，根本不会填充非必需字段，只保留其默认值。

注入构造函数和工厂方法参数是一种特殊情况，因为@Autowired中的required属性有一些不同的含义，因为Spring的构造函数解析算法可能会处理多个构造函数。默认情况下，构造函数和工厂方法参数实际上是必需的，但在单构造函数场景中有一些特殊规则，比如如果没有匹配的bean可用，多元素注入点(数组、集合、映射)解析为空实例。这允许使用一种通用的实现模式，在这种模式下，所有依赖项都可以在一个唯一的多参数构造函数中声明——例如，声明为一个没有@Autowired注释的公共构造函数。

> 任何给定bean类只有一个构造函数可以声明@Autowired，并将所需属性设置为true，这表明作为Spring bean使用时要自动连接的构造函数。因此，如果required属性保留其默认值true，则只有一个构造函数可以使用@Autowired进行注释。如果多个构造函数声明了注释，那么它们都必须声明required=false，以便被认为是自动装配的候选对象(类似于XML中的autotowire =constructor)。
> 
> 将选择具有最多依赖项的构造函数，这些依赖项可以通过在Spring容器中匹配bean来满足。如果所有候选函数都不满意，那么将使用主/默认构造函数(如果存在)。类似地，如果一个类声明了多个构造函数，但是没有一个构造函数被@Autowired注释，那么将使用主/默认构造函数(如果存在)。如果一个类只声明了一个构造函数，那么即使没有注释，它也总是会被使用。注意，带注释的构造函数不一定是公共的。
> 
> 建议使用@Autowired的required属性，而不是setter方法上已弃用的@Required注释。将required属性设置为false表示该属性不是自动连接所需的，如果无法自动连接，则忽略该属性。另一方面，@Required更强，因为它强制使用容器支持的任何方法设置属性，如果没有定义值，则会引发相应的异常。

或者，您可以通过Java 8的Java .util来表示特定依赖项的非必需性质。可选，示例如下:

```java
public class SimpleMovieLister {

    @Autowired
    public void setMovieFinder(Optional<MovieFinder> movieFinder) {
        ...
    }
}
```

从Spring Framework 5.0开始，你也可以使用@Nullable注释(任何包中的任何类型的注释——例如JSR-305中的javax.annotation.Nullable)，或者仅仅利用Kotlin内置的空安全支持:

```java
public class SimpleMovieLister {

    @Autowired
    public void setMovieFinder(@Nullable MovieFinder movieFinder) {
        ...
    }
}
```

您还可以为那些众所周知的可解析依赖项使用@Autowired: BeanFactory、ApplicationContext、Environment、ResourceLoader、ApplicationEventPublisher和MessageSource。这些接口及其扩展的接口(如ConfigurableApplicationContext或ResourcePatternResolver)会自动解析，不需要特殊设置。下面的例子自动连接一个ApplicationContext对象:

```java
public class MovieRecommender {

    @Autowired
    private ApplicationContext context;

    public MovieRecommender() {
    }

    // ...
}
```

> @Autowired、@Inject、@Value和@Resource注解是由Spring BeanPostProcessor实现处理的。这意味着您不能在自己的BeanPostProcessor或BeanFactoryPostProcessor类型(如果有的话)中应用这些注释。这些类型必须通过使用XML或Spring @Bean方法显式地“连接”起来。

## 使用@Primary微调基于注释的自动装配

由于按类型自动装配可能会导致多个候选人，因此通常有必要对选择过程有更多的控制。实现这一点的一种方法是使用Spring的@Primary注释。@Primary表示当多个bean是自动连接到单值依赖项的候选bean时，应该优先考虑某个特定bean。如果在候选bean中恰好存在一个主bean，那么它将成为自动连接的值。

考虑下面的配置，它将firstMovieCatalog定义为主MovieCatalog:

```java
@Configuration
public class MovieConfiguration {

    @Bean
    @Primary
    public MovieCatalog firstMovieCatalog() { ... }

    @Bean
    public MovieCatalog secondMovieCatalog() { ... }

    // ...
}
```

通过前面的配置，下面的MovieRecommender将与firstMovieCatalog自动连接:

```java
public class MovieRecommender {

    @Autowired
    private MovieCatalog movieCatalog;

    // ...
}
```
相应的bean定义如下:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:context="http://www.springframework.org/schema/context"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/context
        https://www.springframework.org/schema/context/spring-context.xsd">

    <context:annotation-config/>

    <bean class="example.SimpleMovieCatalog" primary="true">
        <!-- inject any dependencies required by this bean -->
    </bean>

    <bean class="example.SimpleMovieCatalog">
        <!-- inject any dependencies required by this bean -->
    </bean>

    <bean id="movieRecommender" class="example.MovieRecommender"/>

</beans>
```

## 微调带有限定符的基于注释的自动装配
@Primary是在可以确定一个主要候选对象时使用多个实例的类型自动装配的有效方法。当您需要更多地控制选择过程时，可以使用Spring的@Qualifier注释。您可以将限定符值与特定的参数关联起来，缩小类型匹配的范围，以便为每个参数选择特定的bean。在最简单的情况下，这可以是一个简单的描述性值，如下面的示例所示:

```java
public class MovieRecommender {

    @Autowired
    @Qualifier("main")
    private MovieCatalog movieCatalog;

    // ...
}
```
你也可以在单独的构造函数参数或方法参数上指定@Qualifier注释，如下例所示:

```java
public class MovieRecommender {

    private MovieCatalog movieCatalog;

    private CustomerPreferenceDao customerPreferenceDao;

    @Autowired
    public void prepare(@Qualifier("main") MovieCatalog movieCatalog,
            CustomerPreferenceDao customerPreferenceDao) {
        this.movieCatalog = movieCatalog;
        this.customerPreferenceDao = customerPreferenceDao;
    }

    // ...
}
```
下面的示例显示了相应的bean定义。

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:context="http://www.springframework.org/schema/context"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/context
        https://www.springframework.org/schema/context/spring-context.xsd">

    <context:annotation-config/>

    <bean class="example.SimpleMovieCatalog">
        <qualifier value="main"/> 

        <!-- inject any dependencies required by this bean -->
    </bean>

    <bean class="example.SimpleMovieCatalog">
        <qualifier value="action"/> 

        <!-- inject any dependencies required by this bean -->
    </bean>

    <bean id="movieRecommender" class="example.MovieRecommender"/>

</beans>
```
对于回滚匹配，bean名称被认为是默认限定符值。因此，可以使用id为main而不是嵌套限定符元素来定义bean，从而得到相同的匹配结果。然而，尽管您可以使用此约定按名称引用特定的bean， @Autowired从根本上讲是关于使用可选语义限定符的类型驱动注入。这意味着限定符值(即使使用bean名称回退)在类型匹配集中始终具有收缩语义。它们在语义上并不表示对唯一bean id的引用。好的限定符值是main、EMEA或persistent，它们表示独立于bean id的特定组件的特征，在匿名bean定义(如前面示例中的bean)的情况下，bean id可能是自动生成的。

限定符也应用于类型化集合，如前所述—例如，设置<MovieCatalog>。在这种情况下，根据声明的限定符，所有匹配的bean都作为集合注入。这意味着限定符不必是唯一的。相反，它们构成了过滤标准。例如，您可以定义多个具有相同限定符值“action”的MovieCatalog bean，所有这些bean都被注入到一个用@Qualifier(“action”)注释的Set<MovieCatalog>中。

> 让限定符值在类型匹配候选对象中针对目标bean名进行选择，不需要在注入点使用@Qualifier注释。如果没有其他解析指示符(例如限定符或主标记)，对于非惟一依赖关系情况，Spring将注入点名称(即字段名称或参数名称)与目标bean名称匹配，并选择同名的候选对象(如果有的话)。

也就是说，如果您打算通过名称表示注释驱动的注入，那么不要主要使用@Autowired，即使它能够在类型匹配的候选对象中通过bean名称进行选择。相反，使用JSR-250 @Resource注释，它在语义上定义为通过其惟一名称标识特定的目标组件，声明的类型与匹配过程无关。@Autowired具有相当不同的语义:在按类型选择候选bean之后，指定的String限定符值只在这些类型选择的候选bean中考虑(例如，将帐户限定符与标记有相同限定符标签的bean相匹配)。

对于本身定义为集合、Map或数组类型的bean， @Resource是一个很好的解决方案，它通过唯一名称引用特定的集合或数组bean。也就是说，从4.3开始，只要元素类型信息保存在@Bean返回类型签名或集合继承层次结构中，就可以通过Spring的@Autowired类型匹配算法匹配集合、Map和数组类型。在这种情况下，可以使用限定符值在相同类型的集合中进行选择，如上一段所述。

从4.3开始，@Autowired还将自身引用视为注入(即对当前被注入的bean的引用)。请注意，自注入是一种后备方法。对其他组件的常规依赖总是具有优先级。从这个意义上说，自我引用并不参与常规的候选人选择，因此它从来都不是主要的。相反，它们的优先级总是最低。在实践中，您应该仅将自身引用作为最后的手段(例如，通过bean的事务代理调用同一实例上的其他方法)。在这种情况下，请考虑将受影响的方法分解到单独的委托bean中。或者，您可以使用@Resource，它可以通过当前bean的唯一名称获取回当前bean的代理。

> 尝试将来自@Bean方法的结果注入同一个配置类实际上也是一个自引用场景。要么在方法签名中惰性地解析此类引用(与配置类中的自动连接字段相反)，要么将受影响的@Bean方法声明为静态方法，将它们与包含的配置类实例及其生命周期解耦。否则，只在回退阶段考虑此类bean，而选择其他配置类上的匹配bean作为主要候选(如果可用)。

@Autowired应用于字段、构造函数和多参数方法，允许在参数级别通过限定符注释缩小范围。相比之下，@Resource只支持具有单个参数的字段和bean属性设置方法。因此，如果注入目标是构造函数或多参数方法，则应该坚持使用限定符。

您可以创建自己的自定义限定符注释。为此，定义一个注释并在定义中提供@Qualifier注释，如下面的示例所示:

```java
@Target({ElementType.FIELD, ElementType.PARAMETER})
@Retention(RetentionPolicy.RUNTIME)
@Qualifier
public @interface Genre {

    String value();
}
```

```java
public class MovieRecommender {

    @Autowired
    @Genre("Action")
    private MovieCatalog actionCatalog;

    private MovieCatalog comedyCatalog;

    @Autowired
    public void setComedyCatalog(@Genre("Comedy") MovieCatalog comedyCatalog) {
        this.comedyCatalog = comedyCatalog;
    }

    // ...
}
```

接下来，您可以为候选bean定义提供信息。您可以添加<qualifier/>标签作为<bean/>标签的子元素，然后指定类型和值来匹配您的自定义限定符注释。类型根据注释的完全限定类名进行匹配。另一种方便的方法是，如果不存在名称冲突的风险，您可以使用短类名。下面的例子演示了这两种方法:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:context="http://www.springframework.org/schema/context"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/context
        https://www.springframework.org/schema/context/spring-context.xsd">

    <context:annotation-config/>

    <bean class="example.SimpleMovieCatalog">
        <qualifier type="Genre" value="Action"/>
        <!-- inject any dependencies required by this bean -->
    </bean>

    <bean class="example.SimpleMovieCatalog">
        <qualifier type="example.Genre" value="Comedy"/>
        <!-- inject any dependencies required by this bean -->
    </bean>

    <bean id="movieRecommender" class="example.MovieRecommender"/>

</beans>
```
在Classpath Scanning和Managed Components中，您可以看到一种基于注释的替代方法，以替代在XML中提供限定符元数据。具体地说，请参见使用注释提供限定符元数据。

在某些情况下，使用没有值的注释可能就足够了。当注释用于更通用的目的，并且可以跨几种不同类型的依赖项应用时，这一点非常有用。例如，您可以提供一个脱机目录，在没有Internet连接时可以搜索它。首先，定义简单的注释，如下例所示:

```java
@Target({ElementType.FIELD, ElementType.PARAMETER})
@Retention(RetentionPolicy.RUNTIME)
@Qualifier
public @interface Offline {

}
```
然后将注释添加到要自动连接的字段或属性，如下面的示例所示:

```java
public class MovieRecommender {

    @Autowired
    @Offline 
    private MovieCatalog offlineCatalog;

    // ...
}
```
现在bean定义只需要一个限定符类型，如下例所示:

```xml
<bean class="example.SimpleMovieCatalog">
    <qualifier type="Offline"/> 
    <!-- inject any dependencies required by this bean -->
</bean>
```

您还可以定义自定义限定符注释，除了接受简单的value属性外，还接受命名属性。如果在要自动连接的字段或参数上指定了多个属性值，那么bean定义必须匹配所有这样的属性值，将其视为自动连接候选值。例如，考虑以下注释定义:

```java
@Target({ElementType.FIELD, ElementType.PARAMETER})
@Retention(RetentionPolicy.RUNTIME)
@Qualifier
public @interface MovieQualifier {

    String genre();

    Format format();
}
```

在这种情况下，Format是一个enum，定义如下:

```java
public enum Format {
    VHS, DVD, BLURAY
}
```
要自动连接的字段使用自定义限定符进行注释，并包含两个属性的值:genre和format，如下面的示例所示:

```java
public class MovieRecommender {

    @Autowired
    @MovieQualifier(format=Format.VHS, genre="Action")
    private MovieCatalog actionVhsCatalog;

    @Autowired
    @MovieQualifier(format=Format.VHS, genre="Comedy")
    private MovieCatalog comedyVhsCatalog;

    @Autowired
    @MovieQualifier(format=Format.DVD, genre="Action")
    private MovieCatalog actionDvdCatalog;

    @Autowired
    @MovieQualifier(format=Format.BLURAY, genre="Comedy")
    private MovieCatalog comedyBluRayCatalog;

    // ...
}
```

最后，bean定义应该包含匹配的限定符值。这个示例还演示了可以使用bean元属性来代替`<qualifier/>`元素。如果可用，`<qualifier/>`元素及其属性优先，但如果没有这样的限定符，自动装配机制将退回到`<meta/>`标签中提供的值，如下面的最后两个bean定义所示:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:context="http://www.springframework.org/schema/context"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/context
        https://www.springframework.org/schema/context/spring-context.xsd">

    <context:annotation-config/>

    <bean class="example.SimpleMovieCatalog">
        <qualifier type="MovieQualifier">
            <attribute key="format" value="VHS"/>
            <attribute key="genre" value="Action"/>
        </qualifier>
        <!-- inject any dependencies required by this bean -->
    </bean>

    <bean class="example.SimpleMovieCatalog">
        <qualifier type="MovieQualifier">
            <attribute key="format" value="VHS"/>
            <attribute key="genre" value="Comedy"/>
        </qualifier>
        <!-- inject any dependencies required by this bean -->
    </bean>

    <bean class="example.SimpleMovieCatalog">
        <meta key="format" value="DVD"/>
        <meta key="genre" value="Action"/>
        <!-- inject any dependencies required by this bean -->
    </bean>

    <bean class="example.SimpleMovieCatalog">
        <meta key="format" value="BLURAY"/>
        <meta key="genre" value="Comedy"/>
        <!-- inject any dependencies required by this bean -->
    </bean>

</beans>
```

## 使用泛型作为自动装配限定符

除了@Qualifier注释外，还可以使用Java泛型类型作为限定的隐式形式。例如，假设您有以下配置:

```java
@Configuration
public class MyConfiguration {

    @Bean
    public StringStore stringStore() {
        return new StringStore();
    }

    @Bean
    public IntegerStore integerStore() {
        return new IntegerStore();
    }
}
```
假设前面的bean实现了一个泛型接口(即Store`<String>`和Store`<Integer>`)，你可以@autotowire Store接口，并将泛型用作限定符，如下例所示:

```java
@Autowired
private Store<String> s1; // <String> qualifier, injects the stringStore bean

@Autowired
private Store<Integer> s2; // <Integer> qualifier, injects the integerStore bean
```

泛型限定符在自动装配列表、Map实例和数组时也适用。下面的例子自动连接了一个通用的List:

```java
// Inject all Store beans as long as they have an <Integer> generic
// Store<String> beans will not appear in this list
@Autowired
private List<Store<Integer>> s;
```
## 使用CustomAutowireConfigurer

CustomAutowireConfigurer是一个BeanFactoryPostProcessor，它允许您注册自己的自定义限定符注释类型，即使它们没有使用Spring的@Qualifier注释。下面的示例演示如何使用CustomAutowireConfigurer:

```xml
<bean id="customAutowireConfigurer"
        class="org.springframework.beans.factory.annotation.CustomAutowireConfigurer">
    <property name="customQualifierTypes">
        <set>
            <value>example.CustomQualifier</value>
        </set>
    </property>
</bean>
```
AutowireCandidateResolver通过以下方法确定自动装配候选对象:  
* 每个bean定义的自动连接候选值
* `<beans/>`元素上可用的任何默认自动连线候选模式
* @Qualifier注释和在CustomAutowireConfigurer注册的任何自定义注释的存在

当多个bean限定为自动装配候选对象时，“主”的确定如下:如果候选对象中恰好有一个bean定义的主属性设置为true，则选择该bean。

## 使用@Resource注入

Spring还通过在字段或bean属性设置器方法上使用JSR-250 @Resource注释(javax.annotation.Resource)来支持注入。这是Java EE中的一种常见模式:例如，在jsf管理的bean和JAX-WS端点中。Spring也支持Spring管理对象的这种模式。

@Resource接受一个name属性。默认情况下，Spring将该值解释为要注入的bean名。换句话说，它遵循by-name语义，如下例所示:

```java
public class SimpleMovieLister {

    private MovieFinder movieFinder;

    @Resource(name="myMovieFinder") 
    public void setMovieFinder(MovieFinder movieFinder) {
        this.movieFinder = movieFinder;
    }
}
```

如果没有显式指定名称，则默认名称派生自字段名或setter方法。对于字段，它接受字段名。对于setter方法，它接受bean属性名。下面的例子将把名为movieFinder的bean注入到它的setter方法中:

```java
public class SimpleMovieLister {

    private MovieFinder movieFinder;

    @Resource
    public void setMovieFinder(MovieFinder movieFinder) {
        this.movieFinder = movieFinder;
    }
}
```

> 与注释一起提供的名称由CommonAnnotationBeanPostProcessor感知的ApplicationContext解析为bean名称。如果显式配置Spring的SimpleJndiBeanFactory，则可以通过JNDI解析这些名称。但是，我们建议您依赖默认行为，并使用Spring的JNDI查找功能来保持间接级别。

在没有明确指定名称的@Resource使用情况下，与@Autowired类似，@Resource会找到一个主类型匹配，而不是特定的命名bean，并解析众所周知的可解析依赖项:BeanFactory、ApplicationContext、ResourceLoader、ApplicationEventPublisher和MessageSource接口。

因此，在下面的示例中，customerPreferenceDao字段首先查找名为“customerPreferenceDao”的bean，然后返回到与customerPreferenceDao类型匹配的主类型:

```java
public class MovieRecommender {

    @Resource
    private CustomerPreferenceDao customerPreferenceDao;

    @Resource
    private ApplicationContext context; 

    public MovieRecommender() {
    }

    // ...
}
```

## 使用 @Value
@Value通常用于注入外部化属性:

```java
@Component
public class MovieRecommender {

    private final String catalog;

    public MovieRecommender(@Value("${catalog.name}") String catalog) {
        this.catalog = catalog;
    }
}
```
配置如下:

```java
@Configuration
@PropertySource("classpath:application.properties")
public class AppConfig { }
```

以下 application.properties 文件:
```yaml
catalog.name=MovieCatalog
```

在这种情况下，catalog参数和字段将等于MovieCatalog值。

Spring提供了一个默认的、宽松的嵌入式值解析器。它将尝试解析属性值，如果不能解析，属性名(例如${catalog.name})将作为值注入。如果你想严格控制不存在的值，你应该声明一个PropertySourcesPlaceholderConfigurer bean，如下面的例子所示:

```java
@Configuration
public class AppConfig {

    @Bean
    public static PropertySourcesPlaceholderConfigurer propertyPlaceholderConfigurer() {
        return new PropertySourcesPlaceholderConfigurer();
    }
}
```

> 当使用JavaConfig配置PropertySourcesPlaceholderConfigurer时，@Bean方法必须是静态的。

如果无法解析任何${}占位符，使用上述配置可确保Spring初始化失败。也可以使用setPlaceholderPrefix、setPlaceholderSuffix或setvalueseseparator等方法来定制占位符。

Spring Boot默认配置一个PropertySourcesPlaceholderConfigurer bean，它将从application.properties和application.yml文件。

Spring提供的内置转换器支持允许自动处理简单的类型转换(例如到Integer或int)。多个逗号分隔的值可以自动转换为字符串数组而不需要额外的努力。
可以提供如下默认值:

```java
@Component
public class MovieRecommender {

    private final String catalog;

    public MovieRecommender(@Value("${catalog.name:defaultCatalog}") String catalog) {
        this.catalog = catalog;
    }
}
```
Spring BeanPostProcessor在幕后使用ConversionService来处理将@Value中的String值转换为目标类型的过程。如果你想为自己的自定义类型提供转换支持，你可以提供自己的ConversionService bean实例，如下例所示:

```java
@Configuration
public class AppConfig {

    @Bean
    public ConversionService conversionService() {
        DefaultFormattingConversionService conversionService = new DefaultFormattingConversionService();
        conversionService.addConverter(new MyCustomConverter());
        return conversionService;
    }
}
```
当@Value包含一个SpEL表达式时，该值将在运行时动态计算，如下例所示:

```java
@Component
public class MovieRecommender {

    private final String catalog;

    public MovieRecommender(@Value("#{systemProperties['user.catalog'] + 'Catalog' }") String catalog) {
        this.catalog = catalog;
    }
}
```
SpEL还支持使用更复杂的数据结构:

```java
@Component
public class MovieRecommender {

    private final Map<String, Integer> countOfMoviesPerCatalog;

    public MovieRecommender(
            @Value("#{{'Thriller': 100, 'Comedy': 300}}") Map<String, Integer> countOfMoviesPerCatalog) {
        this.countOfMoviesPerCatalog = countOfMoviesPerCatalog;
    }
}
```

## 使用@PostConstruct和@PreDestroy
CommonAnnotationBeanPostProcessor不仅可以识别@Resource注释，还可以识别JSR-250生命周期注释:javax.annotation.PostConstruct和javax.annotation.PreDestroy。在Spring 2.5中引入的对这些注释的支持提供了一种替代初始化回调和销毁回调中描述的生命周期回调机制的方法。如果CommonAnnotationBeanPostProcessor是在Spring ApplicationContext中注册的，那么携带这些注释之一的方法将在生命周期中的同一点被调用，与相应的Spring生命周期接口方法或显式声明的回调方法相同。在以下示例中，缓存在初始化时被预填充，在销毁时被清除:

```java
public class CachingMovieLister {

    @PostConstruct
    public void populateMovieCache() {
        // populates the movie cache upon initialization...
    }

    @PreDestroy
    public void clearMovieCache() {
        // clears the movie cache upon destruction...
    }
}
```

有关组合各种生命周期机制的效果的详细信息，请参见组合生命周期机制。

> 与@Resource一样，@PostConstruct和@PreDestroy注释类型也是从JDK 6到8的标准Java库的一部分。然而，整个javax。注释包在JDK 9中与核心Java模块分离，并最终在JDK 11中被删除。如果需要，javax。注释api构件现在需要通过Maven Central获得，只需像其他库一样添加到应用程序的类路径中。


> * [使用Spring MVC服务Web内容](https://spring.io/guides/gs/serving-web-content/)
> * [Spring安全架构(参考指南)](https://spring.io/guides/topicals/spring-security-architecture/)
> * [Spring Framework官方文档](https://spring.io/projects/spring-framework)
> * [SpringIoc简介](https://github.com/fredomli/java-standard/blob/main/docs/spring/spring/core/SpringIoc容器.md)
> * [Spring容器使用](https://github.com/fredomli/java-standard/blob/main/docs/spring/spring/core/Spring容器使用.md)
> * [SpringIoc简介](https://github.com/fredomli/java-standard/blob/main/docs/spring/spring/core/SpringIoc容器.md)
> * [SpringBean作用域](https://github.com/fredomli/java-standard/blob/main/docs/spring/spring/core/SpringBean作用域.md)


*(☞ﾟヮﾟ)☞[返回首页README.md address](https://github.com/fredomli/java-standard)*
