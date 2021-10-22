# Spring 自定义 Bean 属性

Spring框架提供了许多接口，您可以使用这些接口自定义bean的性质。本节将它们归类如下:

* 实现生命周期回调
* `ApplicationContextAware` 和 `BeanNameAware`
* 其他 Aware 到接口

## 生命周期回调
要与容器对bean生命周期的管理交互，可以实现Spring InitializingBean和DisposableBean接口。容器对前者调用afterPropertiesSet()，对后者调用destroy()，以让bean在初始化和销毁bean时执行某些操作。

> JSR-250 @PostConstruct和@PreDestroy注释通常被认为是在现代Spring应用程序中接收生命周期回调的最佳实践。使用这些注释意味着您的bean没有耦合到spring特定的接口。具体请参见使用@PostConstruct和@PreDestroy。
> 
> 如果不想使用JSR-250注释，但仍然希望消除耦合，请考虑init-method和destroy-method bean定义元数据。

在内部，Spring框架使用BeanPostProcessor实现来处理它能找到的任何回调接口，并调用适当的方法。如果您需要自定义特性或Spring默认不提供的其他生命周期行为，您可以自己实现BeanPostProcessor。有关更多信息，请参见容器扩展点。

除了初始化和销毁回调，spring管理的对象还可以实现Lifecycle接口，以便这些对象可以参与启动和关闭过程，这是由容器自己的生命周期驱动的。

*本节描述生命周期回调接口。*

### 初始化回调
`org.springframework.beans.factory.initializingbean`接口允许容器在bean上设置了所有必要的属性后，bean执行初始化工作。InitializingBean接口指定了一个方法:

```java
void afterPropertiesSet() throws Exception;
```
我们建议您不要使用InitializingBean接口，因为它不必要地将代码与Spring耦合在一起。或者，我们建议使用@PostConstruct注释或指定POJO初始化方法。在基于xml的配置元数据的情况下，可以使用init-method属性指定具有无效无参数签名的方法的名称。在Java配置中，可以使用@Bean的initMethod属性。请参见接收生命周期回调。考虑下面的例子:

```xml
<bean id="exampleInitBean" class="examples.ExampleBean" init-method="init"/>
```

```java
public class ExampleBean {

    public void init() {
        // do some initialization work
    }
}
```

前面的示例几乎与下面的示例(包含两个清单)具有完全相同的效果:

```xml
<bean id="exampleInitBean" class="examples.AnotherExampleBean"/>
```

```java
public class AnotherExampleBean implements InitializingBean {

    @Override
    public void afterPropertiesSet() {
        // do some initialization work
    }
}
```

然而，前面两个示例中的第一个并没有将代码与Spring耦合起来。

### 销毁回调
实现disposable bean接口可以让bean在容器被销毁时获得回调。DisposableBean接口指定了一个方法:

```java
void destroy() throws Exception;
```

我们建议您不要使用DisposableBean回调接口，因为它不必要地将代码与Spring耦合在一起。另外，我们建议使用@PreDestroy注释或指定bean定义支持的泛型方法。对于基于xml的配置元数据，您可以在<bean/>上使用destroy-method属性。在Java配置中，可以使用@Bean的destroyMethod属性。请参见接收生命周期回调。考虑以下定义:

```xml
<bean id="exampleInitBean" class="examples.ExampleBean" destroy-method="cleanup"/>
```

```java
public class ExampleBean {

    public void cleanup() {
        // do some destruction work (like releasing pooled connections)
    }
}
```
前面的定义与下面的定义几乎完全相同:

```xml
<bean id="exampleInitBean" class="examples.AnotherExampleBean"/>
```

```java
public class AnotherExampleBean implements DisposableBean {

    @Override
    public void destroy() {
        // do some destruction work (like releasing pooled connections)
    }
}
```
您可以为<bean>元素的destroy-method属性指定一个特殊的(推断的)值，该值指示Spring自动检测特定bean类上的公共关闭或关闭方法。(因此，任何实现java.lang.AutoCloseable或java.io.Closeable的类都可以匹配。)您还可以在<beans>元素的Default - Destroy -method属性上设置这个特殊的(推断的)值，以便将此行为应用到整个bean集(请参阅默认初始化和Destroy方法)。注意，这是Java配置的默认行为。

### 默认初始化和销毁方法
当编写不使用特定于spring的InitializingBean和DisposableBean回调接口的初始化和销毁方法回调时，通常编写具有init()、initialize()、dispose()等名称的方法。理想情况下，这种生命周期回调方法的名称在项目中是标准化的，以便所有开发人员使用相同的方法名称，并确保一致性。

您可以将Spring容器配置为“查找”每个bean上的命名初始化并销毁回调方法名称。这意味着，作为应用程序开发人员，您可以编写应用程序类并使用名为init()的初始化回调，而不必为每个bean定义配置init-method="init"属性。Spring IoC容器在创建bean时调用该方法(并且符合前面描述的标准生命周期回调契约)。此特性还强制初始化和销毁方法回调的一致命名约定。

假设您的初始化回调方法名为init()，而您的destroy回调方法名为destroy()。你的类就像下面这个例子中的类:

```java
public class DefaultBlogService implements BlogService {

    private BlogDao blogDao;

    public void setBlogDao(BlogDao blogDao) {
        this.blogDao = blogDao;
    }

    // this is (unsurprisingly) the initialization callback method
    public void init() {
        if (this.blogDao == null) {
            throw new IllegalStateException("The [blogDao] property must be set.");
        }
    }
}
```
然后，您可以在bean中使用该类，类似如下:

```xml
<beans default-init-method="init">

    <bean id="blogService" class="com.something.DefaultBlogService">
        <property name="blogDao" ref="blogDao" />
    </bean>

</beans>
```
在顶级`<`beans/>`元素属性上出现default-init-method属性，会导致Spring IoC容器将bean类上名为init的方法识别为初始化方法回调。在创建和组装bean时，如果bean类有这样的方法，就会在适当的时候调用它。

通过在顶级`<beans/>`元素上使用default-destroy-method属性，可以类似地(在XML中)配置destroy方法回调。

如果现有的bean类已经具有按约定命名的回调方法，那么您可以通过使用`<bean/>`本身的init-method和destroy-method属性指定(在XML中)方法名称来覆盖默认值。

Spring容器保证在提供了所有依赖项的bean之后立即调用已配置的初始化回调。因此，对原始bean引用调用初始化回调，这意味着AOP拦截器等还没有应用到bean。首先完全创建目标bean，然后应用带有拦截器链的AOP代理(例如)。如果目标bean和代理是分别定义的，那么您的代码甚至可以绕过代理与原始目标bean交互。因此，将拦截器应用于init方法是不一致的，因为这样做会将目标bean的生命周期与它的代理或拦截器耦合在一起，并且在代码直接与原始目标bean交互时留下奇怪的语义。


### 组合生命周期机制
从Spring 2.5开始，你有三个选项来控制bean的生命周期行为:
* InitializingBean和DisposableBean回调接口
* 自定义init()和destroy()方法
* @PostConstruct和@PreDestroy注释。您可以组合这些机制来控制给定的bean。

> 如果为bean配置了多个生命周期机制，并且每种机制都配置了不同的方法名，那么每个配置的方法都将按照下面列出的顺序运行。但是，如果为多个生命周期机制配置了相同的方法名(例如，初始化方法的init())，则该方法只运行一次，如上一节所述。

#### 为同一个bean配置的多个生命周期机制(具有不同的初始化方法)如下所示:
1. 用@PostConstruct注释的方法
2. afterPropertiesSet()由InitializingBean回调接口定义
3. 自定义配置的init()方法

#### Destroy方法的调用顺序相同:
1. 使用@PreDestroy注释的方法
2. 由DisposableBean回调接口定义的destroy()
3. 自定义配置的destroy()方法

### 启动和关闭回调

Lifecycle接口为任何具有自己生命周期需求(比如启动和停止某个后台进程)的对象定义了基本方法:

```java
public interface Lifecycle {

    void start();

    void stop();

    boolean isRunning();
}
```
任何spring管理的对象都可以实现Lifecycle接口。然后，当ApplicationContext本身接收到启动和停止信号(例如，对于运行时的停止/重启场景)时，它将这些调用级联到该上下文中定义的所有生命周期实现。它通过委托给LifecycleProcessor来实现，如下所示:

```java
public interface LifecycleProcessor extends Lifecycle {

    void onRefresh();

    void onClose();
}
```
注意，LifecycleProcessor本身就是Lifecycle接口的扩展。它还添加了另外两个方法，用于对正在刷新和关闭的上下文作出反应。

> 注意，常规的org.springframework.context.Lifecycle接口是一个显式的启动和停止通知的简单契约，并不意味着在上下文刷新时自动启动。对于特定bean的自动启动(包括启动阶段)的细粒度控制，可以考虑实现org.springframework.context.SmartLifecycle。
> 
> 另外，请注意，停止通知并不保证在毁灭之前到来。在常规关闭时，在传播常规销毁回调之前，所有生命周期bean都会首先收到停止通知。但是，在上下文生命周期内的热刷新或在停止刷新尝试时，只调用销毁方法。

启动和关闭调用的顺序可能很重要。如果任意两个对象之间存在“依赖”关系，依赖方在其依赖项之后开始，在其依赖项之前停止。然而，有时，直接的依赖关系是未知的。您可能只知道某种类型的对象应该先于另一种类型的对象启动。在这些情况下，SmartLifecycle接口定义了另一个选项，即在其超接口上定义的getPhase()方法。下面的清单显示了phase界面的定义:

```java
public interface Phased {

    int getPhase();
}
```
SmartLifecycle接口的定义如下所示:
```java
public interface SmartLifecycle extends Lifecycle, Phased {

    boolean isAutoStartup();

    void stop(Runnable callback);
}
```

### 在非web应用程序中优雅地关闭Spring IoC容器
本节仅适用于非web应用程序。Spring基于web的ApplicationContext实现已经有了适当的代码，可以在相关web应用程序关闭时优雅地关闭Spring IoC容器。`

如果在非web应用程序环境中使用Spring的IoC容器(例如，在富客户端桌面环境中)，请向JVM注册一个关闭钩子。这样做可以确保良好地关闭并调用单例bean上的相关destroy方法，以便释放所有资源。您仍然必须正确地配置和实现这些销毁回调。

要注册一个shutdown钩子，调用在ConfigurableApplicationContext接口上声明的registerShutdownHook()方法，如下例所示:

```java
import org.springframework.context.ConfigurableApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;

public final class Boot {

    public static void main(final String[] args) throws Exception {
        ConfigurableApplicationContext ctx = new ClassPathXmlApplicationContext("beans.xml");

        // add a shutdown hook for the above context...
        ctx.registerShutdownHook();

        // app runs here...

        // main method exits, hook is called prior to the app shutting down...
    }
}
```
____
## ApplicationContextAware and BeanNameAware

## Other Aware Interfaces


> * [使用Spring MVC服务Web内容](https://spring.io/guides/gs/serving-web-content/)
> * [Spring安全架构(参考指南)](https://spring.io/guides/topicals/spring-security-architecture/)
> * [Spring Framework官方文档](https://spring.io/projects/spring-framework)
> * [SpringIoc简介](https://github.com/fredomli/java-standard/blob/main/docs/spring/spring/core/SpringIoc容器.md)
> * [Spring容器使用](https://github.com/fredomli/java-standard/blob/main/docs/spring/spring/core/Spring容器使用.md)
> * [SpringIoc简介](https://github.com/fredomli/java-standard/blob/main/docs/spring/spring/core/SpringIoc容器.md)
> * [SpringBean作用域](https://github.com/fredomli/java-standard/blob/main/docs/spring/spring/core/SpringBean作用域.md)


*(☞ﾟヮﾟ)☞[返回首页README.md address](https://github.com/fredomli/java-standard)*

