# Spring Bean 作用域

当您创建bean定义时，您将创建一个配方，用于创建由该bean定义定义的类的实际实例。bean定义是一个菜谱的想法很重要，因为这意味着，与类一样，您可以从一个菜谱创建许多对象实例。

您不仅可以控制要插入到从特定bean定义创建的对象中的各种依赖项和配置值，还可以控制从特定bean定义创建的对象的范围。这种方法功能强大且灵活，因为您可以通过配置选择创建的对象的范围，而不必在Java类级别上烘烤对象的范围。可以将bean定义为在多个作用域之一中部署。Spring框架支持六个作用域，其中四个只有在你使用web感知的ApplicationContext时才可用。您还可以创建自定义作用域。

下表描述了支持的范围:

|范围|描述|
|---|---|
|singleton|(默认)作用域为每个Spring IoC容器的单个对象实例定义单个bean。|
|prototype|将单个bean定义定义为任意数量的对象实例。|
|request|将单个bean定义限定为单个HTTP请求的生命周期。也就是说，每个HTTP请求都有自己的bean实例，它是在单个bean定义的后面创建的。仅在web感知的Spring ApplicationContext上下文中有效。|
|session|将单个bean定义限定为HTTP会话的生命周期。仅在web感知的Spring ApplicationContext上下文中有效。|
|application|将单个bean定义作用于ServletContext的生命周期。仅在web感知的Spring ApplicationContext上下文中有效。|
|websocket|将单个bean定义作用于WebSocket的生命周期。仅在web感知的Spring ApplicationContext上下文中有效。|

> 从Spring 3.0开始，线程作用域是可用的，但默认情况下没有注册。有关更多信息，请参阅SimpleThreadScope的文档。有关如何注册此或任何其他自定义作用域的说明，请参见使用自定义作用域。

## 单例的范围
只管理一个单例bean的一个共享实例，所有对具有一个或多个ID的bean的请求都将导致Spring容器返回一个特定的bean实例。

换句话说，当您定义一个bean定义并且它的作用域为单例时，Spring IoC容器会创建由该bean定义定义的对象的一个实例。这个单实例存储在这样的单例bean的缓存中，对该命名bean的所有后续请求和引用都会返回缓存的对象。下面的图片展示了单例作用域是如何工作的:

![singleton](https://gitee.com/fredomli/fredomli-picture/raw/picgo/static/images/wordpress/singleton.png)

Spring的单例bean概念不同于四人组(Gang of Four, GoF)模式书中定义的单例模式。GoF单例对对象的作用域进行硬编码，使得每个ClassLoader只创建一个特定类的实例。Spring单例的作用域最好描述为每个容器和每个bean。这意味着，如果您在单个Spring容器中为特定类定义一个bean，那么Spring容器将创建且仅创建由该bean定义的类的一个实例。单例作用域是Spring中的默认作用域。要在XML中将bean定义为单例，可以定义如下示例所示的bean:

```xml
<bean id="accountService" class="com.something.DefaultAccountService"/>

<!-- the following is equivalent, though redundant (singleton scope is the default) -->
<bean id="accountService" class="com.something.DefaultAccountService" scope="singleton"/>
```

## 原型范围
bean部署的非单例原型作用域导致每次对特定bean发出请求时都要创建一个新的bean实例。也就是说，该bean被注入到另一个bean中，或者您通过容器上的getBean()方法调用请求它。作为一条规则，您应该对所有有状态bean使用原型作用域，对无状态bean使用单例作用域。

下图说明了Spring原型的作用域:

![prototype](https://gitee.com/fredomli/fredomli-picture/raw/picgo/static/images/wordpress/prototype.png) 

数据访问对象(DAO)通常不配置为原型，因为典型的DAO不持有任何会话状态。对我们来说，重用单例图的核心更容易。)

下面的示例用XML将bean定义为原型:

```xml
<bean id="accountService" class="com.something.DefaultAccountService" scope="prototype"/>
```
与其他作用域相比，Spring并不管理原型bean的完整生命周期。容器实例化、配置和组装一个原型对象，并将其传递给客户端，而不需要进一步记录该原型实例。因此，尽管初始化生命周期回调方法在所有对象上都被调用，而不管作用域如何，但在原型的情况下，配置的销毁生命周期回调不会被调用。客户端代码必须清理原型作用域对象，并释放原型bean所持有的昂贵资源。与其他作用域相比，Spring并不管理原型bean的完整生命周期。容器实例化、配置和组装一个原型对象，并将其传递给客户端，而不需要进一步记录该原型实例。因此，尽管初始化生命周期回调方法在所有对象上都被调用，而不管作用域如何，但在原型的情况下，配置的销毁生命周期回调不会被调用。客户端代码必须清理原型作用域对象，并释放原型bean所持有的昂贵资源。要让Spring容器释放由原型作用域bean持有的资源，请尝试使用自定义bean后处理器，它保存了对需要清理的bean的引用。

在某些方面，Spring容器在原型作用域bean方面的角色可以替代Java new操作符。超过这一点的所有生命周期管理都必须由客户端处理。(有关Spring容器中bean的生命周期的详细信息，请参阅生命周期回调。)

## 带有原型bean依赖关系的单例bean

当您使用依赖于原型bean的单例作用域bean时，请注意在实例化时解析依赖项。因此，如果您将一个原型作用域的bean依赖注入到一个单例作用域的bean中，那么将实例化一个新的原型bean，然后将依赖注入到单例bean中。原型实例是曾经提供给单例作用域bean的唯一实例。

但是，假设您希望单例作用域bean在运行时重复获得原型作用域bean的新实例。不能将原型作用域的bean依赖注入到单例bean中，因为这种注入只发生一次，即当Spring容器实例化单例bean并解析和注入其依赖项时。如果您不止一次地在运行时需要一个原型bean的新实例，请参阅方法注入。

## 请求、会话、应用和WebSocket作用域

只有当您使用web感知的Spring ApplicationContext实现(如XmlWebApplicationContext)时，request、session、application和websocket作用域才可用。如果将这些作用域与常规Spring IoC容器(如ClassPathXmlApplicationContext)一起使用，则会抛出一个举报未知bean作用域的IllegalStateException。

### 初始化Web配置

为了在请求、会话、应用程序和websocket级别(web作用域bean)支持bean的作用域，在定义bean之前需要进行一些小的初始配置。(这种初始设置对于标准作用域是不需要的:单例和原型。)

如何完成这个初始设置取决于特定的Servlet环境。
如果您访问Spring Web MVC中的作用域bean，实际上是在由Spring DispatcherServlet处理的请求中，不需要进行特殊设置。DispatcherServlet已经公开了所有相关的状态。

如果你使用Servlet 2.5 web容器，在Spring的DispatcherServlet之外处理请求(例如，当使用JSF或Struts时)，你需要注册org.springframework.web.context.request.RequestContextListener ServletRequestListener。对于Servlet 3.0+，这可以通过使用WebApplicationInitializer接口以编程方式完成。或者，对于旧的容器，在你的web应用程序的web.xml文件中添加以下声明:

```xml
<web-app>
    ...
    <listener>
        <listener-class>
            org.springframework.web.context.request.RequestContextListener
        </listener-class>
    </listener>
    ...
</web-app>
```
或者，如果监听器设置有问题，可以考虑使用Spring的RequestContextFilter。过滤器映射依赖于周围的web应用程序配置，所以您必须适当地更改它。下面的列表显示了web应用程序的过滤部分:

```xml
<web-app>
    ...
    <filter>
        <filter-name>requestContextFilter</filter-name>
        <filter-class>org.springframework.web.filter.RequestContextFilter</filter-class>
    </filter>
    <filter-mapping>
        <filter-name>requestContextFilter</filter-name>
        <url-pattern>/*</url-pattern>
    </filter-mapping>
    ...
</web-app>
```
DispatcherServlet、RequestContextListener和RequestContextFilter都做完全相同的事情，即绑定HTTP请求对象到服务该请求的线程。这使得请求和会话作用域的bean可以在调用链的更深处使用。

### Request 作用域

考虑以下bean定义的XML配置:

```xml
<bean id="loginAction" class="com.something.LoginAction" scope="request"/>
```

Spring容器通过为每个HTTP请求使用LoginAction bean定义来创建LoginAction bean的新实例。也就是说，loginAction bean的作用域是在HTTP请求级别。您可以随心所欲地更改已创建实例的内部状态，因为从相同的loginAction bean定义创建的其他实例看不到这些状态更改。它们是针对个别要求的。当请求完成处理时，将丢弃作用域为该请求的bean。

当使用注释驱动的组件或Java配置时，可以使用@RequestScope注释将组件分配到请求范围。下面的例子展示了如何做到这一点:

```java
@RequestScope
@Component
public class LoginAction {
    // ...
}
```

### Session 作用域

考虑以下bean定义的XML配置:
```xml
<bean id="userPreferences" class="com.something.UserPreferences" scope="session"/>
```

Spring容器通过在单个HTTP会话的生命周期中使用UserPreferences bean定义来创建UserPreferences bean的新实例。换句话说，userPreferences bean有效地限定在HTTP会话级别。与请求范围内bean一样,你可以改变内部状态的实例创建尽可能多的你想要的,知道其他HTTP会话实例也使用相同的实例创建userPreferences bean定义看不到这些变化状态,因为他们是特定于一个单独的HTTP会话。

当使用注释驱动的组件或Java配置时，您可以使用@SessionScope注释将组件分配到会话作用域。

```java
@SessionScope
@Component
public class UserPreferences {
    // ...
}
```

### Application 作用域

考虑以下bean定义的XML配置:
```xml
<bean id="appPreferences" class="com.something.AppPreferences" scope="application"/>
```

Spring容器通过对整个web应用使用一次AppPreferences bean定义来创建一个AppPreferences bean的新实例。也就是说，appPreferences bean的作用域在ServletContext级别，并存储为常规的ServletContext属性。这有点类似于弹簧单例bean,但在两个重要方面不同:它是一个单例每ServletContext不是每Spring ApplicationContext(可能有几个在任何给定的web应用程序),它实际上是暴露,因此可见ServletContext属性。

当使用注释驱动的组件或Java配置时，您可以使用@ApplicationScope注释将组件分配给应用程序范围。下面的例子展示了如何做到这一点:

```java
@ApplicationScope
@Component
public class AppPreferences {
    // ...
}
```

### 作用域bean作为依赖项
Spring IoC容器不仅管理对象(bean)的实例化，还管理协作者(或依赖项)的连接。如果您想将(例如)一个HTTP请求作用域的bean注入到另一个寿命较长的作用域bean中，您可以选择注入一个AOP代理来代替该作用域bean。也就是说，您需要注入一个代理对象，该对象公开与作用域对象相同的公共接口，但也可以从相关作用域(例如HTTP请求)检索实际目标对象，并将方法调用委托给实际对象。

> 您还可以在作用域为单例的bean之间使用`<aop:scoped-proxy/>`，然后引用通过一个可序列化的中间代理，从而能够在反序列化时重新获得目标单例bean。
>
> 当对作用域原型bean声明`<aop:scoped-proxy/>`时，共享代理上的每个方法调用都会导致创建一个新的目标实例，然后将调用转发到该目标实例。
> 
> 而且，限定范围的代理并不是以生命周期安全的方式从较短范围访问bean的唯一方法。你也可以声明你的注射点(也就是构造函数或setter参数或autowired的字段)作为ObjectFactory < MyTargetBean >,允许getObject()调用来检索当前实例对需求每次需要——没有分别持有实例或存储它。
> 
> 作为扩展变量，您可以声明ObjectProvider<MyTargetBean>，它提供了几个额外的访问变量，包括getIfAvailable和getIfUnique。
> 
> 它的JSR-330变体称为Provider，与Provider<MyTargetBean>声明一起使用，并在每次检索尝试时调用相应的get()。在这里了解更多关于JSR-330的详细信息。


下面示例中的配置只有一行，但是理解“为什么”和“如何”是很重要的:  

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:aop="http://www.springframework.org/schema/aop"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/aop
        https://www.springframework.org/schema/aop/spring-aop.xsd">

    <!-- an HTTP Session-scoped bean exposed as a proxy -->
    <bean id="userPreferences" class="com.something.UserPreferences" scope="session">
        <!-- instructs the container to proxy the surrounding bean -->
        <aop:scoped-proxy/> 
    </bean>

    <!-- a singleton-scoped bean injected with a proxy to the above bean -->
    <bean id="userService" class="com.something.SimpleUserService">
        <!-- a reference to the proxied userPreferences bean -->
        <property name="userPreferences" ref="userPreferences"/>
    </bean>
</beans>
```
要创建这样的代理，需要将子元素<aop:作用域代理/>插入到作用域bean定义中(请参阅选择要创建的代理类型和基于XML schema的配置)。为什么作用域在请求、会话和自定义作用域级别的bean定义需要<aop:作用域代理/>元素?考虑下面的单例bean定义，并将其与您需要为上述作用域定义的内容进行对比(注意下面的userPreferences bean定义是不完整的):

```xml
<bean id="userPreferences" class="com.something.UserPreferences" scope="session"/>

<bean id="userManager" class="com.something.UserManager">
    <property name="userPreferences" ref="userPreferences"/>
</bean>
```

在前面的示例中，使用对HTTP会话作用域bean (userPreferences)的引用注入了单例bean (userManager)。这里的要点是userManager bean是一个单例:它在每个容器中只实例化一次，而且它的依赖项(在本例中只有一个userPreferences bean)也只注入一次。这意味着userManager bean只操作完全相同的userPreferences对象(即最初注入它的对象)。

当将寿命较短的作用域bean注入寿命较长的作用域bean时(例如，将HTTP session作用域协作bean作为依赖项注入到单例bean中)，这不是您想要的行为。相反，您需要一个userManager对象，并且，对于HTTP会话的生命周期，您需要一个特定于HTTP会话的userPreferences对象。因此，容器创建一个对象，该对象公开与UserPreferences类完全相同的公共接口(理想情况下是一个UserPreferences实例对象)，该对象可以从范围机制(HTTP请求、会话等)获取真正的UserPreferences对象。容器将此代理对象注入userManager bean，该bean不知道此UserPreferences引用是一个代理。在本例中，当UserManager实例调用依赖项注入的UserPreferences对象上的方法时，它实际上是在调用代理上的方法。然后，代理从(在本例中)HTTP Session获取真实的UserPreferences对象，并将方法调用委托给检索到的真实UserPreferences对象。

因此，在将请求范围和会话范围的bean注入协作对象时，需要进行以下(正确且完整的)配置，如下面的示例所示:

```xml
<bean id="userPreferences" class="com.something.UserPreferences" scope="session">
    <aop:scoped-proxy/>
</bean>

<bean id="userManager" class="com.something.UserManager">
    <property name="userPreferences" ref="userPreferences"/>
</bean>
```

### 选择要创建的代理类型
默认情况下，当Spring容器为用<aop:scoped-proxy/>元素标记的bean创建代理时，将创建一个基于cglib的类代理。

> CGLIB代理只拦截公共方法调用!不要在这样的代理上调用非公共方法。它们不会委托给实际的作用域目标对象。

或者，您可以将Spring容器配置为为这些作用域bean创建标准的基于JDK接口的代理，方法是为`<aop:scoped-proxy/>`元素的`proxy-target-class`属性的值指定`false`。使用JDK基于接口的代理意味着您不需要在应用程序类路径中添加额外的库来影响此类代理。然而，这也意味着限定范围bean的类必须实现至少一个接口，并且所有被注入限定范围bean的协作者必须通过其中一个接口引用该bean。

基于接口的代理示例如下:  

```xml
<!-- DefaultUserPreferences implements the UserPreferences interface -->
<bean id="userPreferences" class="com.stuff.DefaultUserPreferences" scope="session">
    <aop:scoped-proxy proxy-target-class="false"/>
</bean>

<bean id="userManager" class="com.stuff.UserManager">
    <property name="userPreferences" ref="userPreferences"/>
</bean>
```
有关选择基于类或基于接口的代理的更多详细信息，请参阅代理机制。

## 自定义作用域
bean作用域机制是可扩展的。您可以定义自己的作用域，甚至可以重新定义现有的作用域，尽管后者被认为是不好的做法，而且您不能覆盖内置的单例和原型作用域。

### 创建自定义作用域
要将您的自定义作用域集成到Spring容器中，您需要实现org.springframework.beans.factory.config.Scope接口，该接口将在本节中描述。要了解如何实现自己的作用域，请参阅Spring框架本身提供的作用域实现和Scope javadoc，后者更详细地解释了需要实现的方法。

Scope接口有四个方法来从作用域获取对象、从作用域删除对象和销毁对象。

例如，会话范围实现返回会话范围的bean(如果它不存在，则方法将其绑定到会话以供将来引用后返回该bean的新实例)。下面的方法从底层作用域返回对象:

```java
Object get(String name, ObjectFactory<?> objectFactory)
```

例如，会话范围实现从底层会话中删除会话范围的bean。应该返回对象，但是如果没有找到指定名称的对象，可以返回null。下面的方法将对象从底层作用域中移除:

```java
Object remove(String name)
```

下面的方法注册了一个回调函数，当范围被销毁或范围内指定的对象被销毁时，该回调函数应该被调用:

```java
void registerDestructionCallback(String name, Runnable destructionCallback)
```

有关销毁回调的更多信息，请参阅javadoc或Spring作用域实现。  
下面的方法获取底层作用域的对话标识符:

```java
String getConversationId()
```

这个标识符对于每个作用域都是不同的。对于会话范围的实现，此标识符可以是会话标识符

### 使用自定义作用域
在编写和测试一个或多个自定义作用域实现之后，需要让Spring容器知道您的新作用域。下面的方法是向Spring容器注册一个新的Scope的中心方法:

```java
void registerScope(String scopeName, Scope scope);
```
此方法在ConfigurableBeanFactory接口上声明，该接口可通过Spring附带的大多数具体ApplicationContext实现上的BeanFactory属性获得。

registerScope(..)方法的第一个参数是与作用域关联的唯一名称。Spring容器中此类名称的例子有singleton和prototype。registerScope(..)方法的第二个参数是您希望注册和使用的自定义Scope实现的实际实例。

假设您编写了自定义Scope实现，然后按下一个示例所示注册它。

> 下一个示例使用SimpleThreadScope，它包含在Spring中，但默认情况下没有注册。对于您自己的自定义Scope实现，说明是相同的。

Java
```java
Scope threadScope = new SimpleThreadScope();
beanFactory.registerScope("thread", threadScope);
```
然后，您可以创建遵循自定义Scope范围规则的bean定义，如下所示:
```xml
<bean id="..." class="..." scope="thread">
```
使用自定义范围实现，您将不再局限于范围的程序化注册。您还可以通过使用CustomScopeConfigurer类声明地进行Scope注册，如下例所示:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:aop="http://www.springframework.org/schema/aop"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/aop
        https://www.springframework.org/schema/aop/spring-aop.xsd">

    <bean class="org.springframework.beans.factory.config.CustomScopeConfigurer">
        <property name="scopes">
            <map>
                <entry key="thread">
                    <bean class="org.springframework.context.support.SimpleThreadScope"/>
                </entry>
            </map>
        </property>
    </bean>

    <bean id="thing2" class="x.y.Thing2" scope="thread">
        <property name="name" value="Rick"/>
        <aop:scoped-proxy/>
    </bean>

    <bean id="thing1" class="x.y.Thing1">
        <property name="thing2" ref="thing2"/>
    </bean>

</beans>
```

> 当您在FactoryBean实现的<bean>声明中放置<aop:scoped-proxy/>时，有作用域的是工厂bean本身，而不是getObject()返回的对象。

> * [使用Spring MVC服务Web内容](https://spring.io/guides/gs/serving-web-content/)
> * [Spring安全架构(参考指南)](https://spring.io/guides/topicals/spring-security-architecture/)
> * [Spring Framework官方文档](https://spring.io/projects/spring-framework)
> * [SpringIoc简介](https://github.com/fredomli/java-standard/blob/main/docs/spring/spring/core/SpringIoc容器.md)
> * [Spring容器使用](https://github.com/fredomli/java-standard/blob/main/docs/spring/spring/core/Spring容器使用.md)
> * [SpringIoc简介](https://github.com/fredomli/java-standard/blob/main/docs/spring/spring/core/SpringIoc容器.md)


*(☞ﾟヮﾟ)☞[返回首页README.md address](https://github.com/fredomli/java-standard)*

