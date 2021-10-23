# 基于java的容器配置

本节介绍如何在Java代码中使用注释来配置Spring容器。

## 基本概念:@Bean和@Configuration
Spring新的java配置支持中的中心构件是带有@ configuration注解的类和带有@ bean注解的方法。

@Bean注释用于指示方法实例化、配置和初始化要由Spring IoC容器管理的新对象。对于那些熟悉Spring的<beans/> XML配置的人来说，@Bean注释扮演着与<bean/>元素相同的角色。您可以在任何Spring @Component中使用@ bean注解的方法。但是，它们最常与@Configuration bean一起使用。

用@Configuration注释类表明它的主要用途是作为bean定义的源。此外，@Configuration类允许通过调用同一类中的其他@Bean方法来定义bean间的依赖关系。最简单的@Configuration类如下所示:

```java
@Configuration
public class AppConfig {

    @Bean
    public MyService myService() {
        return new MyServiceImpl();
    }
}
```

面的AppConfig类相当于下面的Spring <beans/> XML:

```xml
<beans>
    <bean id="myService" class="com.acme.services.MyServiceImpl"/>
</beans>
```
> 当@Bean方法在没有使用@Configuration注释的类中声明时，它们被称为以“lite”模式处理。在@Component中甚至在普通的老类中声明的Bean方法被认为是“lite”，包含类的主要目的不同，而@Bean方法是一种额外的用途。例如，服务组件可以通过每个适用组件类上的附加@Bean方法向容器公开管理视图。在这样的场景中，@Bean方法是一种通用工厂方法机制。
> 
> 与完整的@Configuration不同，lite @Bean方法不能声明bean间的依赖关系。相反，它们对包含它们的组件的内部状态进行操作，并且(可选地)对它们可能声明的参数进行操作。因此，这样的@Bean方法不应该调用其他@Bean方法。每个这样的方法实际上只是特定bean引用的工厂方法，没有任何特殊的运行时语义。这里的积极的副作用是不需要在运行时应用CGLIB子类化，因此在类设计方面没有限制(也就是说，包含的类可能是final的等等)
> 
> 在常见的场景中，@Bean方法将在@Configuration类中声明，以确保始终使用“完整”模式，因此跨方法引用被重定向到容器的生命周期管理。这可以防止相同的@Bean方法通过常规Java调用意外被调用，这有助于减少在“lite”模式下操作时难以跟踪的细微错误。

下面几节将深入讨论@Bean和@Configuration注释。但是，首先，我们将介绍使用基于java的配置创建spring容器的各种方法。

## 使用AnnotationConfigApplicationContext实例化Spring容器

下面的章节记录了Spring 3.0中引入的AnnotationConfigApplicationContext。这个通用的ApplicationContext实现不仅能够接受@Configuration类作为输入，还能够接受普通的@Component类和用JSR-330元数据注释的类。

当@Configuration类作为输入提供时，@Configuration类本身被注册为bean定义，并且类中所有声明的@Bean方法也被注册为bean定义。

当提供了@Component和JSR-330类时，它们被注册为bean定义，并且假设像@Autowired或@Inject这样的DI元数据在需要的时候在这些类中使用。


### 简单构建

与实例化ClassPathXmlApplicationContext时使用Spring XML文件作为输入的方式非常相似，您可以使用@Configuration类作为实例化AnnotationConfigApplicationContext时的输入。这允许Spring容器完全不使用xml，如下例所示:

```java
public static void main(String[] args) {
    ApplicationContext ctx = new AnnotationConfigApplicationContext(AppConfig.class);
    MyService myService = ctx.getBean(MyService.class);
    myService.doStuff();
}
```
如前所述，AnnotationConfigApplicationContext并不局限于只使用@Configuration类。任何@Component或JSR-330注释的类都可以作为构造函数的输入，如下例所示:

```java
public static void main(String[] args) {
    ApplicationContext ctx = new AnnotationConfigApplicationContext(MyServiceImpl.class, Dependency1.class, Dependency2.class);
    MyService myService = ctx.getBean(MyService.class);
    myService.doStuff();
}
```
前面的示例假设MyServiceImpl、Dependency1和Dependency2使用@Autowired等Spring依赖注入注释。

### 使用register以编程方式构建容器(Class<?>…)
您可以使用无参数构造函数实例化一个AnnotationConfigApplicationContext，然后使用register()方法配置它。当以编程方式构建AnnotationConfigApplicationContext时，这种方法特别有用。下面的例子展示了如何做到这一点:

```java
public static void main(String[] args) {
    AnnotationConfigApplicationContext ctx = new AnnotationConfigApplicationContext();
    ctx.register(AppConfig.class, OtherConfig.class);
    ctx.register(AdditionalConfig.class);
    ctx.refresh();
    MyService myService = ctx.getBean(MyService.class);
    myService.doStuff();
}
```

### 使用scan(String…)启用组件扫描

要启用组件扫描，你可以像下面这样注释@Configuration类:

```java
@Configuration
@ComponentScan(basePackages = "com.acme") 
public class AppConfig  {
    // ...
}
```

> 有经验的Spring用户可能熟悉来自Spring上下文中的XML声明等价物:名称空间，如下例所示
> ```java
> <beans>
>    <context:component-scan base-package="com.acme"/>
> </beans>
> ```

在上面的例子中，com.acme包被扫描以查找任何带有@ component注解的类，这些类被注册为容器内的Spring bean定义。AnnotationConfigApplicationContext暴露了scan(String…)方法来允许相同的组件扫描功能，如下面的示例所示:

```java
public static void main(String[] args) {
    AnnotationConfigApplicationContext ctx = new AnnotationConfigApplicationContext();
    ctx.scan("com.acme");
    ctx.refresh();
    MyService myService = ctx.getBean(MyService.class);
}
```

> 请记住@Configuration类是用@Component元注释的，所以它们是组件扫描的候选类。在前面的例子中，假设AppConfig是在com. com中声明的。Acme包(或下面的任何包)，它在调用scan()期间被提取。在refresh()时，它的所有@Bean方法都被处理并注册为容器内的bean定义。

### 使用AnnotationConfigWebApplicationContext支持Web应用

AnnotationConfigApplicationContext的WebApplicationContext变体可与AnnotationConfigWebApplicationContext一起使用。您可以在配置Spring ContextLoaderListener servlet侦听器、Spring MVC DispatcherServlet等时使用此实现。下面的web.xml片段配置了一个典型的Spring MVC web应用程序(注意contextClass context-param和init-param的使用):

```xml
<web-app>
    <!-- Configure ContextLoaderListener to use AnnotationConfigWebApplicationContext
        instead of the default XmlWebApplicationContext -->
    <context-param>
        <param-name>contextClass</param-name>
        <param-value>
            org.springframework.web.context.support.AnnotationConfigWebApplicationContext
        </param-value>
    </context-param>

    <!-- Configuration locations must consist of one or more comma- or space-delimited
        fully-qualified @Configuration classes. Fully-qualified packages may also be
        specified for component-scanning -->
    <context-param>
        <param-name>contextConfigLocation</param-name>
        <param-value>com.acme.AppConfig</param-value>
    </context-param>

    <!-- Bootstrap the root application context as usual using ContextLoaderListener -->
    <listener>
        <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
    </listener>

    <!-- Declare a Spring MVC DispatcherServlet as usual -->
    <servlet>
        <servlet-name>dispatcher</servlet-name>
        <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
        <!-- Configure DispatcherServlet to use AnnotationConfigWebApplicationContext
            instead of the default XmlWebApplicationContext -->
        <init-param>
            <param-name>contextClass</param-name>
            <param-value>
                org.springframework.web.context.support.AnnotationConfigWebApplicationContext
            </param-value>
        </init-param>
        <!-- Again, config locations must consist of one or more comma- or space-delimited
            and fully-qualified @Configuration classes -->
        <init-param>
            <param-name>contextConfigLocation</param-name>
            <param-value>com.acme.web.MvcConfig</param-value>
        </init-param>
    </servlet>

    <!-- map all requests for /app/* to the dispatcher servlet -->
    <servlet-mapping>
        <servlet-name>dispatcher</servlet-name>
        <url-pattern>/app/*</url-pattern>
    </servlet-mapping>
</web-app>
```
## 使用@Bean注释

@Bean是一个方法级注释，是对XML `<bean/>`元素的直接模拟。注释支持`<bean/>`提供的一些属性，例如:

* init-method
* destroy-method
* autowiring 
* name

可以在@ configuration注释的类或@ component注释的类中使用@Bean注释。

### 声明一个Bean

要声明bean，可以使用@Bean注释对方法进行注释。使用此方法在指定类型的ApplicationContext中注册bean定义作为方法的返回值。默认情况下，bean名称与方法名称相同。下面的例子展示了一个@Bean方法声明:

```java
@Configuration
public class AppConfig {

    @Bean
    public TransferServiceImpl transferService() {
        return new TransferServiceImpl();
    }
}
```
上述配置与以下Spring XML完全等价:

```xml
<beans>
    <bean id="transferService" class="com.acme.TransferServiceImpl"/>
</beans>
```
这两个声明都使名为transferService的bean在ApplicationContext中可用，绑定到类型为TransferServiceImpl的对象实例，如下图所示:

```xml
transferService -> com.acme.TransferServiceImpl
```
还可以使用接口(或基类)返回类型声明@Bean方法，如下例所示:

```java
@Configuration
public class AppConfig {

    @Bean
    public TransferService transferService() {
        return new TransferServiceImpl();
    }
}
```
但是，这限制了预先类型预测对指定接口类型(TransferService)的可见性。然后，容器只有在受影响的单例bean被实例化后才知道完整类型(TransferServiceImpl)。非惰性单例bean会根据它们的声明顺序进行实例化，因此您可能会看到不同的类型匹配结果，这取决于其他组件何时尝试通过未声明的类型进行匹配(例如@Autowired TransferServiceImpl，它只在transferService bean被实例化之后解析)。

> 如果您始终通过声明的服务接口引用您的类型，则@Bean返回类型可以安全地加入该设计决策。但是，对于实现多个接口的组件，或者对于可能由其实现类型引用的组件，声明最特定的返回类型(至少根据引用bean的注入点的要求指定)是更安全的。

### Bean的依赖关系
带@ bean注释的方法可以有任意数量的参数，用于描述构建该bean所需的依赖项。例如，如果我们的TransferService需要一个AccountRepository，我们可以用一个方法参数来实现这个依赖，如下例所示:

```java
@Configuration
public class AppConfig {

    @Bean
    public TransferService transferService(AccountRepository accountRepository) {
        return new TransferServiceImpl(accountRepository);
    }
}
```
解析机制与基于构造器的依赖项注入非常相似。更多细节请参阅相关部分。

### 接受生命周期回调
使用@Bean注释定义的任何类都支持常规的生命周期回调，并可以使用JSR-250中的@PostConstruct和@PreDestroy注释。有关更多细节，请参阅JSR-250注释。

也完全支持常规的Spring生命周期回调。如果一个bean实现了InitializingBean、DisposableBean或Lifecycle，容器将调用它们各自的方法。

标准的*Aware接口集(如BeanFactoryAware、BeanNameAware、MessageSourceAware、ApplicationContextAware等)也得到了完全支持。

@Bean注释支持指定任意初始化和销毁回调方法，很像Spring XML的bean元素的init-method和destroy-method属性，如下例所示

```java
public class BeanOne {

    public void init() {
        // initialization logic
    }
}

public class BeanTwo {

    public void cleanup() {
        // destruction logic
    }
}

@Configuration
public class AppConfig {

    @Bean(initMethod = "init")
    public BeanOne beanOne() {
        return new BeanOne();
    }

    @Bean(destroyMethod = "cleanup")
    public BeanTwo beanTwo() {
        return new BeanTwo();
    }
}
```

> 默认情况下，使用Java配置定义的具有公共close或shutdown方法的bean将使用销毁回调自动征用。如果您有一个公共的close或shutdown方法，并且您不希望在容器关闭时调用它，那么您可以在bean定义中添加@Bean(destroyMethod="")来禁用默认(推断)模式。
> 
> 对于使用JNDI获得的资源，您可能希望在默认情况下这样做，因为它的生命周期是在应用程序外部管理的。特别是，请确保始终为DataSource执行此操作，因为众所周知，它在Java EE应用服务器上存在问题。
> 
> 下面的示例演示如何防止数据源的自动销毁回调:
> 
> ```java
> @Bean(destroyMethod="")
> public DataSource dataSource() throws NamingException {
>    return (DataSource) jndiTemplate.lookup("MyDS");
> }
> ```
> @ bean方法,您通常使用程序化的JNDI查找,通过使用Spring的JndiTemplate JndiLocatorDelegate助手或直接使用JNDI InitialContext但不是JndiObjectFactoryBean变体(这将迫使你声明返回类型作为FactoryBean类型,而不是实际的目标类型,这使得其他@Bean方法中想要引用这里提供的资源的交叉引用调用更难使用)。

对于上例中的BeanOne，在构造过程中直接调用init()方法同样有效，如下例所示:

```java
@Configuration
public class AppConfig {

    @Bean
    public BeanOne beanOne() {
        BeanOne beanOne = new BeanOne();
        beanOne.init();
        return beanOne;
    }

    // ...
}
```
> 当您直接在Java中工作时，您可以对对象做任何您喜欢的事情，而不总是需要依赖于容器的生命周期。

### 指定Bean范围
Spring包含@Scope注释，以便您可以指定bean的范围。

#### 使用@Scope注释

您可以指定使用@Bean注释定义的bean应该具有特定的作用域。您可以使用Bean作用域部分中指定的任何标准作用域。

默认的作用域是singleton，但是你可以用@Scope注释覆盖它，如下面的例子所示:

```java
@Configuration
public class MyConfiguration {

    @Bean
    @Scope("prototype")
    public Encryptor encryptor() {
        // ...
    }
}
```

#### @Scope和作用域内的代理
Spring提供了一种通过作用域代理处理作用域依赖的方便方法。在使用XML配置时，创建这样一个代理的最简单方法是使用`<aop:scoped-proxy/>`元素。使用@Scope注释在Java中配置bean提供了与proxyMode属性等效的支持。默认是`ScopedProxyMode.DEFAULT`，它通常表示不应该创建范围代理，除非在组件扫描指令级别配置了不同的默认值。您可以指定`ScopedProxyMode.TARGET_CLASS` `ScopedProxyMode.INTERFACES` 或`ScopedProxyMode.NO`。

如果您使用Java将XML参考文档中的作用域代理示例(请参阅作用域代理)移植到我们的@Bean，它类似于以下内容:

```java
// an HTTP Session-scoped bean exposed as a proxy
@Bean
@SessionScope
public UserPreferences userPreferences() {
    return new UserPreferences();
}

@Bean
public Service userService() {
    UserService service = new SimpleUserService();
    // a reference to the proxied userPreferences bean
    service.setUserPreferences(userPreferences());
    return service;
}
```

### 定制Bean命名
默认情况下，配置类使用@Bean方法的名称作为结果bean的名称。但是，可以使用name属性覆盖该功能，如下例所示:

```java
@Configuration
public class AppConfig {

    @Bean("myThing")
    public Thing thing() {
        return new Thing();
    }
}
```

### Bean 别名

正如命名bean中所讨论的，有时需要为单个bean指定多个名称，称为bean别名。@Bean注释的name属性为此目的接受一个String数组。下面的示例演示如何为bean设置多个别名:

```java
@Configuration
public class AppConfig {

    @Bean({"dataSource", "subsystemA-dataSource", "subsystemB-dataSource"})
    public DataSource dataSource() {
        // instantiate, configure and return DataSource bean...
    }
}
```

### Bean描述

有时，为bean提供更详细的文本描述是很有帮助的。当为了监视目的而公开bean(可能是通过JMX)时，这可能特别有用。

要向@Bean添加描述，可以使用@Description注释，如下例所示:

```java
@Configuration
public class AppConfig {

    @Bean
    @Description("Provides a basic example of a bean")
    public Thing thing() {
        return new Thing();
    }
}
```

## 使用@Configuration注释
@Configuration是一个类级注释，表示对象是bean定义的源。@Configuration类通过@ bean注释的方法声明bean。还可以使用@Configuration类上的@Bean方法调用来定义bean间的依赖关系。请参阅基本概念:@Bean和@Configuration获得一般介绍。

### 在Bean之间注入依赖（相互注入）

当bean相互依赖时，表示这种依赖就像让一个bean方法调用另一个bean一样简单，如下面的示例所示:

```java
@Configuration
public class AppConfig {

    @Bean
    public BeanOne beanOne() {
        return new BeanOne(beanTwo());
    }

    @Bean
    public BeanTwo beanTwo() {
        return new BeanTwo();
    }
}
```

在前面的示例中，beanOne通过构造函数注入接收到对beanTwo的引用。

> 只有当@Bean方法在@Configuration类中声明时，这种声明bean间依赖关系的方法才有效。您不能通过使用普通的@Component类来声明bean间的依赖。

### 查找方法注入
如前所述，查找方法注入是一个高级特性，您应该很少使用它。它在单例作用域bean依赖于原型作用域bean的情况下很有用。为这种类型的配置使用Java提供了一种实现该模式的自然方法。下面的例子展示了如何使用查找方法注入:

```java
public abstract class CommandManager {
    public Object process(Object commandState) {
        // grab a new instance of the appropriate Command interface
        Command command = createCommand();
        // set the state on the (hopefully brand new) Command instance
        command.setState(commandState);
        return command.execute();
    }

    // okay... but where is the implementation of this method?
    protected abstract Command createCommand();
}
```
通过使用Java配置，您可以创建CommandManager的子类，其中抽象的createCommand()方法以这样一种方式被覆盖，它查找一个新的(原型)命令对象。下面的例子展示了如何做到这一点:

### 关于基于java的配置如何在内部工作的进一步信息
考虑下面的例子，它显示了一个@Bean注释方法被调用两次:

```java
@Configuration
public class AppConfig {

    @Bean
    public ClientService clientService1() {
        ClientServiceImpl clientService = new ClientServiceImpl();
        clientService.setClientDao(clientDao());
        return clientService;
    }

    @Bean
    public ClientService clientService2() {
        ClientServiceImpl clientService = new ClientServiceImpl();
        clientService.setClientDao(clientDao());
        return clientService;
    }

    @Bean
    public ClientDao clientDao() {
        return new ClientDaoImpl();
    }
}
```
在clientService1()和clientService2()中分别调用了一次clientDao()。由于此方法创建了ClientDaoImpl的一个新实例并返回它，因此您通常会希望有两个实例(每个服务一个实例)。这肯定会有问题:在Spring中，实例化的bean默认具有单例作用域。这就是神奇之处:所有@Configuration类在启动时都被CGLIB子类化了。在子类中，子方法在调用父方法并创建新实例之前，首先检查容器中是否有缓存的(有作用域的)bean。

> 据bean的范围，行为可能会有所不同。我们在这里讨论的是单例。
> 
> 从Spring 3.2开始，不再需要将CGLIB添加到类路径中，因为CGLIB类已经被重新打包到org.springframework.cglib下，并直接包含在Spring核心的JAR中。


> ```
> 由于CGLIB在启动时动态地添加特性，因此存在一些限制。特别是，配置类不能是final类。
> 然而，从4.3开始，配置类允许使用任何构造函数，包括使用@Autowired或单个非默认构造函数声明来实现默认注入。
> 
> 如果您希望避免任何cglib强加的限制，可以考虑在non-@Configuration类上声明@Bean方法
> (例如，在普通的@Component类上声明)。@Bean方法之间的跨方法调用不会被拦截，因此必须在构造函数或方法级别完全依赖依赖项注入。
> ```

## 编写基于java的配置

Spring基于java的配置特性允许编写注释，这可以降低配置的复杂性。

### 使用@Import注释

正如在Spring XML文件中使用<import/>元素来帮助模块化配置一样，@Import注释允许从另一个配置类加载@Bean定义，如下例所示:

```java
@Configuration
public class ConfigA {

    @Bean
    public A a() {
        return new A();
    }
}

@Configuration
@Import(ConfigA.class)
public class ConfigB {

    @Bean
    public B b() {
        return new B();
    }
}
```
现在，在实例化上下文时，不需要同时指定ConfigA.class和ConfigB.class，只需要显式提供ConfigB，如下例所示:

```java
public static void main(String[] args) {
    ApplicationContext ctx = new AnnotationConfigApplicationContext(ConfigB.class);

    // now both beans A and B will be available...
    A a = ctx.getBean(A.class);
    B b = ctx.getBean(B.class);
}
```
这种方法简化了容器实例化，因为只需要处理一个类，而不需要在构造过程中记住大量的@Configuration类。

> 从Spring Framework 4.2开始，@Import也支持对常规组件类的引用，类似于AnnotationConfigApplicationContext。注册方法。如果您想要避免组件扫描，通过使用一些配置类作为入口点来显式定义所有组件，这就特别有用。

#### 向导入的@Bean定义注入依赖项
前面的示例可以工作，但过于简单。在大多数实际场景中，bean跨配置类相互依赖。在使用XML时，这不是问题，因为不涉及编译器，您可以声明ref="someBean"，并信任Spring在容器初始化期间解决它。当使用@Configuration类时，Java编译器会对配置模型施加约束，因为对其他bean的引用必须是有效的Java语法。

幸运的是，解决这个问题很简单。正如我们已经讨论过的，@Bean方法可以有任意数量的参数来描述bean依赖关系。考虑以下更真实的场景，其中有几个@Configuration类，每个类都依赖于其他类中声明的bean:

```java
@Configuration
public class ServiceConfig {

    @Bean
    public TransferService transferService(AccountRepository accountRepository) {
        return new TransferServiceImpl(accountRepository);
    }
}

@Configuration
public class RepositoryConfig {

    @Bean
    public AccountRepository accountRepository(DataSource dataSource) {
        return new JdbcAccountRepository(dataSource);
    }
}

@Configuration
@Import({ServiceConfig.class, RepositoryConfig.class})
public class SystemTestConfig {

    @Bean
    public DataSource dataSource() {
        // return new DataSource
    }
}

public static void main(String[] args) {
    ApplicationContext ctx = new AnnotationConfigApplicationContext(SystemTestConfig.class);
    // everything wires up across configuration classes...
    TransferService transferService = ctx.getBean(TransferService.class);
    transferService.transfer(100.00, "A123", "C456");
}
```
还有另一种方法可以达到同样的效果。请记住，@Configuration类最终只是容器中的另一个bean:这意味着它们可以利用@Autowired和@Value注入以及其他与任何其他bean相同的特性。

> 确保以这种方式注入的依赖项是最简单的类型。@Configuration类在初始化上下文期间很早就被处理了，迫使依赖以这种方式注入可能会导致意外的早期初始化。只要有可能，就诉诸于基于参数的注入，如前面的示例所示。
> 
> 另外，要特别小心通过@Bean定义的BeanPostProcessor和BeanFactoryPostProcessor。这些方法通常应该声明为静态@Bean方法，而不是触发包含它们的配置类的实例化。否则，@Autowired和@Value可能无法在配置类本身上工作，因为可以将它创建为比AutowiredAnnotationBeanPostProcessor更早的bean实例。


下面的示例展示了一个bean如何自动连接到另一个bean:
```java
@Configuration
public class ServiceConfig {

    @Autowired
    private AccountRepository accountRepository;

    @Bean
    public TransferService transferService() {
        return new TransferServiceImpl(accountRepository);
    }
}

@Configuration
public class RepositoryConfig {

    private final DataSource dataSource;

    public RepositoryConfig(DataSource dataSource) {
        this.dataSource = dataSource;
    }

    @Bean
    public AccountRepository accountRepository() {
        return new JdbcAccountRepository(dataSource);
    }
}

@Configuration
@Import({ServiceConfig.class, RepositoryConfig.class})
public class SystemTestConfig {

    @Bean
    public DataSource dataSource() {
        // return new DataSource
    }
}

public static void main(String[] args) {
    ApplicationContext ctx = new AnnotationConfigApplicationContext(SystemTestConfig.class);
    // everything wires up across configuration classes...
    TransferService transferService = ctx.getBean(TransferService.class);
    transferService.transfer(100.00, "A123", "C456");
}
```
@Configuration类中的构造函数注入仅在Spring Framework 4.3中得到支持。还请注意，如果目标bean只定义了一个构造函数，则不需要指定@Autowired。

#### 完全限定的导入bean，便于导航

在前面的场景中，使用@Autowired工作得很好，并提供了所需的模块化，但是确定自动连接bean定义的确切位置仍然有些不明确。例如，作为一个着眼于ServiceConfig的开发人员，您如何确切地知道@Autowired AccountRepository bean是在哪里声明的?它在代码中不是显式的，这可能很好。请记住，用于Eclipse的Spring Tools提供了一些工具，这些工具可以呈现显示所有内容如何连接的图表，这可能就是您所需要的。此外，您的Java IDE可以轻松地找到AccountRepository类型的所有声明和使用，并快速显示返回该类型的@Bean方法的位置。

如果这种歧义是不可接受的，并且您希望在IDE中从一个@Configuration类直接导航到另一个@Configuration类，那么考虑自动装配配置类本身。下面的例子展示了如何做到这一点:

```java
@Configuration
public class ServiceConfig {

    @Autowired
    private RepositoryConfig repositoryConfig;

    @Bean
    public TransferService transferService() {
        // navigate 'through' the config class to the @Bean method!
        return new TransferServiceImpl(repositoryConfig.accountRepository());
    }
}
```
在上述情况下，定义AccountRepository是完全显式的。但是，ServiceConfig现在与RepositoryConfig紧密耦合。这就是权衡。通过使用基于接口或基于抽象类的@Configuration类，可以在一定程度上缓解这种紧密耦合。考虑下面的例子:

```java
@Configuration
public class ServiceConfig {

    @Autowired
    private RepositoryConfig repositoryConfig;

    @Bean
    public TransferService transferService() {
        return new TransferServiceImpl(repositoryConfig.accountRepository());
    }
}

@Configuration
public interface RepositoryConfig {

    @Bean
    AccountRepository accountRepository();
}

@Configuration
public class DefaultRepositoryConfig implements RepositoryConfig {

    @Bean
    public AccountRepository accountRepository() {
        return new JdbcAccountRepository(...);
    }
}

@Configuration
@Import({ServiceConfig.class, DefaultRepositoryConfig.class})  // import the concrete config!
public class SystemTestConfig {

    @Bean
    public DataSource dataSource() {
        // return DataSource
    }

}

public static void main(String[] args) {
    ApplicationContext ctx = new AnnotationConfigApplicationContext(SystemTestConfig.class);
    TransferService transferService = ctx.getBean(TransferService.class);
    transferService.transfer(100.00, "A123", "C456");
}
```
现在ServiceConfig与具体的DefaultRepositoryConfig是松散耦合的，内置的IDE工具仍然有用:您可以很容易地获得RepositoryConfig实现的类型层次结构。通过这种方式，导航@Configuration类及其依赖项与导航基于接口的代码的通常过程没有什么不同。

> 如果你想影响某些bean 类的启动创建订单,考虑将其中一些声明为@Lazy(用于创建在第一次访问,而不是在启动时)或@DependsOn某些其他bean(确保特定的其他bean创建当前bean之前,超出后者的直接依赖关系暗示)。
### 有条件地包含@Configuration类或@Bean方法

根据某些任意的系统状态，有条件地启用或禁用一个完整的@Configuration类，甚至单个的@Bean方法，通常是有用的。一个常见的例子是，只有在Spring环境中启用了特定的概要文件时，才使用@Profile注释激活Bean(有关详细信息，请参阅Bean定义概要文件)。

@Profile注释实际上是通过使用更灵活的@Conditional注释实现的。@Conditional注释指示了特定的org.springframework.context.annotation.Condition实现，在注册@Bean之前应该咨询这些实现。

Condition接口的实现提供了一个返回true或false的matches(…)方法。例如，下面的清单显示了@Profile使用的实际条件实现:

```java
@Override
public boolean matches(ConditionContext context, AnnotatedTypeMetadata metadata) {
    // Read the @Profile annotation attributes
    MultiValueMap<String, Object> attrs = metadata.getAllAnnotationAttributes(Profile.class.getName());
    if (attrs != null) {
        for (Object value : attrs.get("value")) {
            if (context.getEnvironment().acceptsProfiles(((String[]) value))) {
                return true;
            }
        }
        return false;
    }
    return true;
}
```
### 结合Java和XML配置
Spring的@Configuration类支持并不旨在100%完全替代Spring XML。一些工具(如Spring XML名称空间)仍然是配置容器的理想方式。在XML方便或必要的情况下,你有一个选择:要么在容器实例化在一个“以XML为中心”的方式使用,例如,ClassPathXmlApplicationContext或实例化它“以java为中心”的方式通过使用所和@ImportResource注释导入XML。

#### 以xml为中心使用@Configuration类
最好是从XML引导Spring容器，并以一种特别的方式包含@Configuration类。例如，在使用Spring XML的大型现有代码库中，更容易根据需要创建@Configuration类，并从现有XML文件中包含它们。在本节的后面，我们将介绍在这种“以xml为中心”的情况下使用@Configuration类的选项。

#### 将@Configuration类声明为普通Spring `<bean/>`元素
`
请记住，@Configuration类最终是容器中的bean定义。在本系列的示例中，我们创建了一个名为AppConfig的@Configuration类，并将其作为<bean/>定义包含在system-test-config.xml中。因为打开了<context:annotation-config/>，所以容器识别@Configuration注释并正确处理AppConfig中声明的@Bean方法。

下面的例子展示了Java中的一个普通配置类:
```java
@Configuration
public class AppConfig {

    @Autowired
    private DataSource dataSource;

    @Bean
    public AccountRepository accountRepository() {
        return new JdbcAccountRepository(dataSource);
    }

    @Bean
    public TransferService transferService() {
        return new TransferService(accountRepository());
    }
}
```
下面的示例显示了示例system-test-config.xml文件的一部分:

```java
<beans>
    <!-- enable processing of annotations such as @Autowired and @Configuration -->
    <context:annotation-config/>
    <context:property-placeholder location="classpath:/com/acme/jdbc.properties"/>

    <bean class="com.acme.AppConfig"/>

    <bean class="org.springframework.jdbc.datasource.DriverManagerDataSource">
        <property name="url" value="${jdbc.url}"/>
        <property name="username" value="${jdbc.username}"/>
        <property name="password" value="${jdbc.password}"/>
    </bean>
</beans>
```
下面的示例显示了一个可能的jdbc.properties:

```properties
jdbc.url=jdbc:hsqldb:hsql://localhost/xdb
jdbc.username=sa
jdbc.password=123456
```
```java
public static void main(String[] args) {
    ApplicationContext ctx = new ClassPathXmlApplicationContext("classpath:/com/acme/system-test-config.xml");
    TransferService transferService = ctx.getBean(TransferService.class);
    // ...
}
```
> 在system-test-config.xml文件中，AppConfig <bean/>没有声明id元素。虽然这样做是可以接受的，但这是不必要的，因为没有其他bean引用它，而且不太可能按名称从容器中显式地获取它。类似地，DataSource bean只会按类型自动连接，因此不严格要求显式bean id。

### 使用`<context:component-scan/>`来挑选@Configuration类

因为@Configuration是用@Component注释的，所以@Configuration注释的类会自动被组件扫描。使用前面示例中描述的相同场景，我们可以重新定义system-test-config.xml以利用组件扫描。注意，在这种情况下，我们不需要显式声明<context:annotation-config/>，因为<context:component-scan/>启用了相同的功能。

下面的示例显示了修改后的system-test-config.xml文件:

```xml
<beans>
    <!-- picks up and registers AppConfig as a bean definition -->
    <context:component-scan base-package="com.acme"/>
    <context:property-placeholder location="classpath:/com/acme/jdbc.properties"/>

    <bean class="org.springframework.jdbc.datasource.DriverManagerDataSource">
        <property name="url" value="${jdbc.url}"/>
        <property name="username" value="${jdbc.username}"/>
        <property name="password" value="${jdbc.password}"/>
    </bean>
</beans>
```

### @Configuration以类为中心使用@ImportResource的XML

在@Configuration类是配置容器的主要机制的应用程序中，仍然可能需要使用至少一些XML。在这些场景中，您可以使用@ImportResource并根据需要定义尽可能多的XML。这样做实现了一种“以java为中心”的方法来配置容器，并将XML保持在最低限度。下面的示例(包括一个配置类、一个定义bean的XML文件、一个属性文件和主类)展示了如何使用@ImportResource注释来实现“以java为中心”的配置，根据需要使用XML:

```java
@Configuration
@ImportResource("classpath:/com/acme/properties-config.xml")
public class AppConfig {

    @Value("${jdbc.url}")
    private String url;

    @Value("${jdbc.username}")
    private String username;

    @Value("${jdbc.password}")
    private String password;

    @Bean
    public DataSource dataSource() {
        return new DriverManagerDataSource(url, username, password);
    }
}
```

```text
properties-config.xml
<beans>
    <context:property-placeholder location="classpath:/com/acme/jdbc.properties"/>
</beans>
```

```text
jdbc.properties
jdbc.url=jdbc:hsqldb:hsql://localhost/xdb
jdbc.username=sa
jdbc.password=
```
```java
public static void main(String[] args) {
    ApplicationContext ctx = new AnnotationConfigApplicationContext(AppConfig.class);
    TransferService transferService = ctx.getBean(TransferService.class);
    // ...
}
```

> * [使用Spring MVC服务Web内容](https://spring.io/guides/gs/serving-web-content/)
> * [Spring安全架构(参考指南)](https://spring.io/guides/topicals/spring-security-architecture/)
> * [Spring Framework官方文档](https://spring.io/projects/spring-framework)
> * [SpringIoc简介](https://github.com/fredomli/java-standard/blob/main/docs/spring/spring/core/SpringIoc容器.md)
> * [Spring容器使用](https://github.com/fredomli/java-standard/blob/main/docs/spring/spring/core/Spring容器使用.md)
> * [SpringIoc简介](https://github.com/fredomli/java-standard/blob/main/docs/spring/spring/core/SpringIoc容器.md)
> * [SpringBean作用域](https://github.com/fredomli/java-standard/blob/main/docs/spring/spring/core/SpringBean作用域.md)
> * [Spring基于注解的容器配置](https://github.com/fredomli/java-standard/blob/main/docs/spring/spring/core/Spring基于注解的容器配置.md)


*(☞ﾟヮﾟ)☞[返回首页README.md address](https://github.com/fredomli/java-standard)*
