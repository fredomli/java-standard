# Spring 依赖

> 典型的企业应用程序不是由单个对象(用Spring的话说就是bean)组成的。即使是最简单的应用程序，也有几个对象协同工作，以呈现最终用户所看到的一致应用程序。下一节将解释如何从定义许多独立的bean定义过渡到一个完全实现的应用程序，在该应用程序中，对象相互协作以实现目标。

## 依赖注入

依赖项注入(DI)是一个过程，在此过程中，对象仅通过构造函数参数、工厂方法参数或对象实例被构造或从工厂方法返回后设置的属性来定义它们的依赖项(即它们工作的其他对象)。然后容器在创建bean时注入这些依赖项。从根本上说，这个过程与bean本身相反(因此得名“控制倒置”)，它通过使用类的直接构造或Service Locator模式来控制依赖项的实例化或位置。

使用依赖注入原则的代码更清晰，并且在向对象提供依赖时解耦更有效。对象不查找它的依赖项，也不知道依赖项的位置或类。因此，您的类变得更容易测试，特别是当依赖关系是在接口或抽象基类上时，这允许在单元测试中使用存根或模拟实现。

依赖注入有两种主要的变体:基于构造函数的依赖注入和基于setter的依赖注入。

### 基于构造器的依赖注入
基于构造器的依赖注入是通过容器调用带有许多参数的构造器来实现的，每个参数表示一个依赖项。调用带有特定参数的静态工厂方法来构造bean几乎是等效的，本讨论以类似的方式对待构造函数和静态工厂方法的参数。下面的示例显示了一个只能通过构造函数注入依赖项的类:

```java
public class SimpleMovieLister {

    // the SimpleMovieLister has a dependency on a MovieFinder
    private final MovieFinder movieFinder;

    // a constructor so that the Spring container can inject a MovieFinder
    public SimpleMovieLister(MovieFinder movieFinder) {
        this.movieFinder = movieFinder;
    }

    // business logic that actually uses the injected MovieFinder is omitted...
}
```
注意，这个类没有什么特别之处。它是一个POJO，不依赖于容器特定的接口、基类或注释。

### 构造函数参数解析

```java
package x.y;

public class ThingOne {

    public ThingOne(ThingTwo thingTwo, ThingThree thingThree) {
        // ...
    }
}
```
假设ThingTwo和ThingThree类没有继承关系，就不存在潜在的歧义。因此，下面的配置工作正常，您不需要在<constructor-arg/>元素中显式指定构造函数参数索引或类型

```xml
<beans>
    <bean id="beanOne" class="x.y.ThingOne">
        <constructor-arg ref="beanTwo"/>
        <constructor-arg ref="beanThree"/>
    </bean>

    <bean id="beanTwo" class="x.y.ThingTwo"/>

    <bean id="beanThree" class="x.y.ThingThree"/>
</beans>
```
当引用另一个bean时，类型是已知的，可以进行匹配(如上例所示)。当使用简单类型时，例如<value>true</value>， Spring无法确定值的类型，因此在没有帮助的情况下无法按类型匹配。考虑以下类:


```java
package examples;

public class ExampleBean {

    // Number of years to calculate the Ultimate Answer
    private final int years;

    // The Answer to Life, the Universe, and Everything
    private final String ultimateAnswer;

    public ExampleBean(int years, String ultimateAnswer) {
        this.years = years;
        this.ultimateAnswer = ultimateAnswer;
    }
}
```
#### 构造函数参数类型匹配
在前面的场景中，如果通过使用type属性显式指定构造函数参数的类型，容器可以使用与简单类型匹配的类型，如下例所示:

```xml
<bean id="exampleBean" class="examples.ExampleBean">
    <constructor-arg type="int" value="7500000"/>
    <constructor-arg type="java.lang.String" value="42"/>
</bean>
```

#### 构造函数参数指标
可以使用index属性显式指定构造函数参数的索引，如下例所示:

```xml
<bean id="exampleBean" class="examples.ExampleBean">
    <constructor-arg index="0" value="7500000"/>
    <constructor-arg index="1" value="42"/>
</bean>
```
除了解决多个简单值的歧义之外，指定索引还解决构造函数具有相同类型的两个参数的歧义。

注意: 索引是基于0的,即从0开始。

#### 构造函数参数的名字
还可以使用构造函数参数名来消除值的歧义，如下面的示例所示:

```xml
<bean id="exampleBean" class="examples.ExampleBean">
    <constructor-arg name="years" value="7500000"/>
    <constructor-arg name="ultimateAnswer" value="42"/>
</bean>
```
请记住，要使此工作开箱即用，您的代码必须在启用调试标志的情况下进行编译，以便Spring可以从构造函数中查找参数名称。如果您不能或不想使用调试标志来编译代码，您可以使用JDK注释@ConstructorProperties来显式地命名构造函数参数。然后，样例类必须如下所示:

```java
package examples;

public class ExampleBean {

    // Fields omitted

    @ConstructorProperties({"years", "ultimateAnswer"})
    public ExampleBean(int years, String ultimateAnswer) {
        this.years = years;
        this.ultimateAnswer = ultimateAnswer;
    }
}
```

### 基于Setter依赖注入
基于setter的DI是由容器在调用无参数构造函数或无参数静态工厂方法实例化bean后调用bean上的setter方法来实现的。

下面的示例显示了一个只能通过使用纯setter注入进行依赖注入的类。这个类是传统的Java。它是一个POJO，不依赖于容器特定的接口、基类或注释。

```java
public class SimpleMovieLister {

    // the SimpleMovieLister has a dependency on the MovieFinder
    private MovieFinder movieFinder;

    // a setter method so that the Spring container can inject a MovieFinder
    public void setMovieFinder(MovieFinder movieFinder) {
        this.movieFinder = movieFinder;
    }

    // business logic that actually uses the injected MovieFinder is omitted...
}
```
ApplicationContext支持其管理的bean的基于构造函数和基于setter的依赖注入。在已经通过构造函数方法注入了一些依赖项之后，它还支持基于setter的DI。

您可以以BeanDefinition的形式配置依赖项，并将其与PropertyEditor实例结合使用，以将属性从一种格式转换为另一种格式。然而，大多数Spring用户并不直接使用这些类(即编程方式)，而是使用XML bean定义、带注释的组件(即使用@Component、@Controller等注释的类)或基于java的@Configuration类中的@Bean方法。然后这些源在内部转换为BeanDefinition的实例，并用于加载整个Spring IoC容器实例。

### 基于构造函数还是基于setter的依赖注入?

由于您可以混合使用基于构造函数和基于setter的DI，对于强制性依赖项使用构造函数，对于可选依赖项使用setter方法或配置方法是一个很好的经验法则。注意，在setter方法上使用@Required注释可以使属性成为必需依赖项;但是，最好使用带参数编程验证的构造函数注入。

Spring团队通常提倡构造函数注入，因为它允许您将应用程序组件实现为不可变对象，并确保所需的依赖项不为空。而且，构造函数注入的组件总是以完全初始化的状态返回给客户机(调用)代码。顺便提一下，大量的构造函数参数是一种糟糕的代码味道，这意味着类可能有太多的责任，应该重构以更好地处理适当的关注点分离。

Setter注入主要应该只用于可选依赖项，这些依赖项可以在类中分配合理的默认值。否则，必须在代码使用依赖项的任何地方执行非空检查。setter注入的一个好处是，setter方法使该类的对象能够在稍后进行重新配置或重新注入。因此，通过JMX mbean进行管理是setter注入的一个引人注目的用例。

参阅使用对特定类最有意义的依赖注入风格。有时，在处理您没有源代码的第三方类时，您可以自行选择。例如，如果第三方类不公开任何setter方法，那么构造函数注入可能是DI的唯一可用形式。

### 依赖解析过程

容器按照如下方式执行bean依赖关系解析:

* ApplicationContext是用描述所有bean的配置元数据创建和初始化的。配置元数据可以由XML、Java代码或注释指定。
* 对于每个bean，其依赖关系都以属性、构造函数参数或静态工厂方法参数(如果使用该方法而不是普通构造函数)的形式表示。这些依赖项是在实际创建bean时提供给bean的。
* 每个属性或构造函数参数都是要设置的值的实际定义，或对容器中另一个bean的引用。
* 每个作为值的属性或构造函数参数将从其指定的格式转换为该属性或构造函数参数的实际类型。默认情况下，Spring可以将字符串格式提供的值转换为所有内置类型，比如int、long、string、boolean等等。

Spring容器在创建容器时验证每个bean的配置。但是，在实际创建bean之前，bean属性本身不会设置。在创建容器时创建单例作用域的bean并将其设置为预实例化(默认值)。作用域在Bean作用域中定义。否则，只在请求时创建bean。创建bean可能会导致创建一个bean图，因为bean的依赖项及其依赖项的依赖项(等等)都是创建和分配的。请注意，这些依赖项之间的解析不匹配可能会在稍后出现——也就是说，在第一次创建受影响的bean时出现。


#### 循环依赖

* 如果主要使用构造函数注入，可能会创建不可解析的循环依赖场景。
* 例如:类A通过构造函数注入需要类B的实例，而类B通过构造函数注入需要类A的实例。如果将类A和类B的bean配置为相互注入，Spring IoC容器会在运行时检测到这个循环引用，并抛出一个BeanCurrentlyInCreationException。
* 一种可能的解决方案是编辑一些类的源代码，由setter而不是构造函数来配置。或者，避免构造函数注入，只使用setter注入。换句话说，尽管不推荐这样做，但您可以使用setter注入配置循环依赖项。
* 与典型情况(没有循环依赖关系)不同，bean a和bean B之间的循环依赖关系迫使其中一个bean在完全初始化自己之前被注入到另一个bean中(这是一个经典的先有鸡还是先有蛋的场景)。

一般来说，您可以相信Spring会做正确的事情。它在容器加载时检测配置问题，比如对不存在的bean和循环依赖项的引用。Spring尽可能晚地在实际创建bean时设置属性和解析依赖项。这意味着，如果在创建对象或其中一个依赖项时出现问题，则正确加载的Spring容器稍后可以在请求对象时生成异常——例如，bean会因为缺少或无效的属性而抛出异常。这可能会延迟一些配置问题的可见性，这就是为什么ApplicationContext实现默认预实例化单例bean。在实际需要这些bean之前，您需要花费一些时间和内存来创建它们，在创建ApplicationContext时(而不是稍后)就会发现配置问题。您仍然可以覆盖这个默认行为，以使单例bean延迟初始化，而不是主动预实例化。

如果不存在循环依赖项，那么当一个或多个协作bean被注入到依赖bean中时，每个协作bean在被注入到依赖bean之前都要完成配置。这意味着，如果bean A依赖于bean B, Spring IoC容器在调用bean A上的setter方法之前完全配置bean B。换句话说，bean被实例化(如果它不是一个预实例化的单例)，它的依赖被设置，和相关的生命周期方法(如配置的init方法或InitializingBean回调方法)被调用。


### 依赖注入的例子
下面的示例为基于设置器的DI使用基于xml的配置元数据。Spring XML配置文件的一小部分指定了如下bean定义:

```xml
<bean id="exampleBean" class="examples.ExampleBean">
    <!-- setter injection using the nested ref element -->
    <property name="beanOne">
        <ref bean="anotherExampleBean"/>
    </property>

    <!-- setter injection using the neater ref attribute -->
    <property name="beanTwo" ref="yetAnotherBean"/>
    <property name="integerProperty" value="1"/>
</bean>

<bean id="anotherExampleBean" class="examples.AnotherBean"/>
<bean id="yetAnotherBean" class="examples.YetAnotherBean"/>
```
下面的示例显示了相应的ExampleBean类:

```java
public class ExampleBean {

    private AnotherBean beanOne;

    private YetAnotherBean beanTwo;

    private int i;

    public void setBeanOne(AnotherBean beanOne) {
        this.beanOne = beanOne;
    }

    public void setBeanTwo(YetAnotherBean beanTwo) {
        this.beanTwo = beanTwo;
    }

    public void setIntegerProperty(int i) {
        this.i = i;
    }
}
```

在前面的示例中，声明了setter以匹配XML文件中指定的属性。下面的例子使用了基于构造函数的DI:

```xml
<bean id="exampleBean" class="examples.ExampleBean">
    <!-- constructor injection using the nested ref element -->
    <constructor-arg>
        <ref bean="anotherExampleBean"/>
    </constructor-arg>

    <!-- constructor injection using the neater ref attribute -->
    <constructor-arg ref="yetAnotherBean"/>

    <constructor-arg type="int" value="1"/>
</bean>

<bean id="anotherExampleBean" class="examples.AnotherBean"/>
<bean id="yetAnotherBean" class="examples.YetAnotherBean"/>
```
下面的示例显示了相应的ExampleBean类:

```java
public class ExampleBean {

    private AnotherBean beanOne;

    private YetAnotherBean beanTwo;

    private int i;

    public ExampleBean(
        AnotherBean anotherBean, YetAnotherBean yetAnotherBean, int i) {
        this.beanOne = anotherBean;
        this.beanTwo = yetAnotherBean;
        this.i = i;
    }
}
```
bean定义中指定的构造函数参数用作ExampleBean构造函数的参数。  

现在考虑这个例子的一个变体，在这里，Spring不是使用构造函数，而是被告知调用一个静态工厂方法来返回对象的一个实例:

```xml
<bean id="exampleBean" class="examples.ExampleBean" factory-method="createInstance">
    <constructor-arg ref="anotherExampleBean"/>
    <constructor-arg ref="yetAnotherBean"/>
    <constructor-arg value="1"/>
</bean>

<bean id="anotherExampleBean" class="examples.AnotherBean"/>
<bean id="yetAnotherBean" class="examples.YetAnotherBean"/>
```

下面的示例显示了相应的ExampleBean类:

```java
public class ExampleBean {

    // a private constructor
    private ExampleBean(...) {
        ...
    }

    // a static factory method; the arguments to this method can be
    // considered the dependencies of the bean that is returned,
    // regardless of how those arguments are actually used.
    public static ExampleBean createInstance (
        AnotherBean anotherBean, YetAnotherBean yetAnotherBean, int i) {

        ExampleBean eb = new ExampleBean (...);
        // some other operations...
        return eb;
    }
}
```

静态工厂方法的参数是由<constructor-arg/>元素提供的，就像实际使用了构造函数一样。工厂方法返回的类的类型不必与包含静态工厂方法的类的类型相同(尽管在本例中是相同的)。实例(非静态)工厂方法可以以本质上相同的方式使用(除了使用factory-bean属性而不是class属性)，因此我们在这里不讨论这些细节。

## 依赖关系和配置细节

> 如前一节所述，您可以将bean属性和构造函数参数定义为对其他托管bean(合作者)的引用，或内联定义的值。Spring基于xml的配置元数据为此目的在其<property/>和<constructor-arg/>元素中支持子元素类型。

### 直接写入值(原语、字符串等)

<property/>元素的value属性将属性或构造函数参数指定为人类可读的字符串表示形式。Spring的转换服务用于将这些值从String转换为属性或参数的实际类型。下面的示例显示了正在设置的各种值:

```xml
<bean id="myDataSource" class="org.apache.commons.dbcp.BasicDataSource" destroy-method="close">
    <!-- results in a setDriverClassName(String) call -->
    <property name="driverClassName" value="com.mysql.jdbc.Driver"/>
    <property name="url" value="jdbc:mysql://localhost:3306/mydb"/>
    <property name="username" value="root"/>
    <property name="password" value="misterkaoli"/>
</bean>
```
下面的示例使用[p命名空间](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-p-namespace) 进行更简洁的XML配置:

```xml
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:p="http://www.springframework.org/schema/p"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
    https://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="myDataSource" class="org.apache.commons.dbcp.BasicDataSource"
        destroy-method="close"
        p:driverClassName="com.mysql.jdbc.Driver"
        p:url="jdbc:mysql://localhost:3306/mydb"
        p:username="root"
        p:password="misterkaoli"/>

</beans>
```
前面的XML更简洁。然而，打字错误是在运行时而不是设计时发现的，除非您使用的IDE(如IntelliJ IDEA或Spring Tools for Eclipse)在创建bean定义时支持自动属性完成。强烈推荐这种IDE援助。

您还可以配置java.util.Properties实例，如下所示:

```xml
<bean id="mappings"
    class="org.springframework.context.support.PropertySourcesPlaceholderConfigurer">

    <!-- typed as a java.util.Properties -->
    <property name="properties">
        <value>
            jdbc.driver.className=com.mysql.jdbc.Driver
            jdbc.url=jdbc:mysql://localhost:3306/mydb
        </value>
    </property>
</bean>
```

Spring容器通过使用JavaBeans PropertyEditor机制将<value/>元素中的文本转换为java.util.Properties实例。这是一个很好的快捷方式，也是Spring团队喜欢使用嵌套的<value/>元素而不是value属性样式的少数地方之一。

### idref 元素

idref元素只是将容器中另一个bean的id(一个字符串值-不是引用)传递给<constructor-arg/>或<property/>元素的一种防错误方法。下面的例子展示了如何使用它:

```xml
<bean id="theTargetBean" class="..."/>

<bean id="theClientBean" class="...">
    <property name="targetName">
        <idref bean="theTargetBean"/>
    </property>
</bean>
```

前面的bean定义代码段(在运行时)与下面的代码段完全相同:

```xml
<bean id="theTargetBean" class="..." />

<bean id="client" class="...">
    <property name="targetName" value="theTargetBean"/>
</bean>
```

第一种形式比第二种形式更可取，因为使用idref标记可以让容器在部署时验证所引用的已命名bean是否实际存在。在第二个变体中，没有对传递给客户端bean的targetName属性的值执行验证。只有在实际实例化客户机bean时才会发现输入错误(很可能导致致命的结果)。如果客户端bean是一个原型bean，则可能只有在容器部署很久之后才会发现这种输入错误和由此产生的异常。

> 注意：
> 4.0 bean XSD不再支持idref元素的本地属性，因为它不再提供常规bean引用的值。当升级到4.0模式时，将现有的idref本地引用更改为idref bean。
> 

### 对其他bean的引用(合作者)

ref元素是`<constructor-arg/>`或 `<property/>` 定义元素中的最后一个元素。在这里，您将bean的指定属性的值设置为容器管理的另一个bean(合作者)的引用。被引用的bean是要设置其属性的bean的依赖项，并且在设置属性之前根据需要初始化它。(如果合作者是一个单例bean，它可能已经被容器初始化了。)所有引用最终都是对另一个对象的引用。范围和验证取决于您是否通过bean或父属性指定其他对象的ID或名称。

通过<ref/>标记的bean属性指定目标bean是最通用的形式，它允许创建对同一容器或父容器中的任何bean的引用，而不管它是否在同一XML文件中。bean属性的值可以与目标bean的id属性相同，也可以与目标bean的name属性中的一个值相同。下面的例子展示了如何使用ref元素:

```xml
<ref bean="someBean"/>
```

通过父属性指定目标bean将创建对当前容器的父容器中的bean的引用。父属性的值可以与目标bean的id属性相同，也可以与目标bean的name属性中的值相同。目标bean必须位于当前bean的父容器中。

当您有容器的层次结构，并且希望使用与父bean同名的代理将现有bean包装在父容器中时，您应该主要使用此bean引用变体。下面的两个清单展示了如何使用父属性:

```xml
<!-- in the parent context -->
<bean id="accountService" class="com.something.SimpleAccountService">
    <!-- insert dependencies as required here -->
</bean>
```

```xml
<!-- in the child (descendant) context -->
<bean id="accountService" <!-- bean name is the same as the parent bean -->
    class="org.springframework.aop.framework.ProxyFactoryBean">
    <property name="target">
        <ref parent="accountService"/> <!-- notice how we refer to the parent bean -->
    </property>
    <!-- insert other configuration and dependencies as required here -->
</bean>
```

> 注意：
> 4.0 bean XSD不再支持ref元素的本地属性，因为它不再提供常规bean引用的值。当升级到4.0模式时，将现有的`ref local`引用更改为`ref bean`。

### 内联Bean

`<property/>`或`<constructor-arg/>`元素中的`<bean/>`元素定义了一个内部`bean`，如下面的示例所示:

```xml
<bean id="outer" class="...">
    <!-- instead of using a reference to a target bean, simply define the target bean inline -->
    <property name="target">
        <bean class="com.example.Person"> <!-- this is the inner bean -->
            <property name="name" value="Fiona Apple"/>
            <property name="age" value="25"/>
        </bean>
    </property>
</bean>
```

内部bean定义不需要定义ID或名称。如果指定了，容器不使用这样的值作为标识符。容器在创建时也会忽略scope标志，因为内部bean总是匿名的，并且总是与外部bean一起创建的。不可能独立地访问内部bean，也不可能将它们注入到外围bean之外的协作bean中。

作为一种极端情况，可以从自定义作用域接收回调——例如，对于包含在单例bean中的请求作用域内部bean。内部bean实例的创建与它的包含bean绑定，但是销毁回调让它参与请求作用域的生命周期。这不是一个常见的场景。内部bean通常只是共享其包含bean的作用域。


### 集合(Collections)

`<list/>`、`<set/>`、`<map/>`和`<props/>`元素分别设置Java Collection类型list、set、map和properties的属性和参数。下面的例子展示了如何使用它们:

```xml
<bean id="moreComplexObject" class="example.ComplexObject">
    <!-- results in a setAdminEmails(java.util.Properties) call -->
    <property name="adminEmails">
        <props>
            <prop key="administrator">administrator@example.org</prop>
            <prop key="support">support@example.org</prop>
            <prop key="development">development@example.org</prop>
        </props>
    </property>
    <!-- results in a setSomeList(java.util.List) call -->
    <property name="someList">
        <list>
            <value>a list element followed by a reference</value>
            <ref bean="myDataSource" />
        </list>
    </property>
    <!-- results in a setSomeMap(java.util.Map) call -->
    <property name="someMap">
        <map>
            <entry key="an entry" value="just some string"/>
            <entry key="a ref" value-ref="myDataSource"/>
        </map>
    </property>
    <!-- results in a setSomeSet(java.util.Set) call -->
    <property name="someSet">
        <set>
            <value>just some string</value>
            <ref bean="myDataSource" />
        </set>
    </property>
</bean>
```
映射键或值或集合值的值也可以是以下元素中的任何一个:

```xml
bean | ref | idref | list | set | map | props | value | null
```

### 集合合并

> Spring容器也支持合并集合。应用程序开发人员可以定义一个父元素`<list/>`， `<map/>`， `<set/>`或`<props/>`，并让子元素`<list/>`， `<map/>`， `<set/>`或`<props/>`继承和覆盖父元素集合中的值。也就是说，子集合的值是父集合和子集合的元素合并的结果，子集合的元素覆盖父集合中指定的值。

关于合并的这一节讨论了父-子bean机制。不熟悉父bean和子bean定义的读者可能希望在继续之前阅读相关章节。
下面的例子演示了集合合并:

```xml
<beans>
    <bean id="parent" abstract="true" class="example.ComplexObject">
        <property name="adminEmails">
            <props>
                <prop key="administrator">administrator@example.com</prop>
                <prop key="support">support@example.com</prop>
            </props>
        </property>
    </bean>
    <bean id="child" parent="parent">
        <property name="adminEmails">
            <!-- the merge is specified on the child collection definition -->
            <props merge="true">
                <prop key="sales">sales@example.com</prop>
                <prop key="support">support@example.co.uk</prop>
            </props>
        </property>
    </bean>
<beans>
```

意，在子bean定义的adminEmails属性的<props/>元素上使用了merge=true属性。当容器解析并实例化子bean时，结果实例有一个adminEmails Properties集合，其中包含将子bean的adminEmails集合与父bean的adminEmails集合合并的结果。结果如下所示:

```text
administrator=administrator@example.com
sales=sales@example.com
support=support@example.co.uk
```
子Properties集合的值集从父集合<props/>继承所有属性元素，子集合的支持值覆盖父集合中的值。

这种合并行为同样适用于<list/>、<map/>和<set/>集合类型。在<list/>元素的特定情况下，将保留与list集合类型(即值的有序集合的概念)相关联的语义。父列表的值在所有子列表的值之前。对于Map、Set和Properties集合类型，不存在排序。因此，对于容器内部使用的关联Map、Set和Properties实现类型基础上的集合类型，没有排序语义有效。

#### 集合合并的限制

不能合并不同的集合类型(例如Map和List)。如果您尝试这样做，则会抛出一个适当的Exception。必须在较低的继承子定义上指定merge属性。在父集合定义上指定merge属性是多余的，并且不会导致所需的合并。

#### 强类型集合

随着Java 5中泛型类型的引入，您可以使用强类型集合。也就是说，可以声明一个Collection类型，使其只能包含(例如)String元素。如果使用Spring将强类型Collection依赖注入到bean中，则可以利用Spring的类型转换支持，以便在添加到Collection之前将强类型Collection实例的元素转换为适当的类型。下面的Java类和bean定义展示了如何做到这一点:


```java
public class SomeClass {

    private Map<String, Float> accounts;

    public void setAccounts(Map<String, Float> accounts) {
        this.accounts = accounts;
    }
}
```
```xml
<beans>
    <bean id="something" class="x.y.SomeClass">
        <property name="accounts">
            <map>
                <entry key="one" value="9.99"/>
                <entry key="two" value="2.75"/>
                <entry key="six" value="3.99"/>
            </map>
        </property>
    </bean>
</beans>
```
当something bean的accounts属性准备注入时，关于强类型Map<String, Float>的元素类型的泛型信息可以通过反射获得。因此，Spring的类型转换基础设施将各种值元素识别为Float类型，并将字符串值(9.99、2.75和3.99)转换为实际的Float类型。

### 空值和空字符串值

Spring将属性的空参数等作为空字符串处理。下面的基于xml的配置元数据片段将电子邮件属性设置为空字符串值("")。

```xml
<bean class="ExampleBean">
    <property name="email" value=""/>
</bean>
```

上面的例子等价于下面的Java代码:

```java
exampleBean.setEmail("");
```

`<null/>`元素处理空值。下面的清单显示了一个示例:

```xml
<bean class="ExampleBean">
    <property name="email">
        <null/>
    </property>
</bean>
```

上述配置相当于以下Java代码:

```java
exampleBean.setEmail(null);
```

### 带有p名称空间的XML快捷方式

p-名称空间允许您使用bean元素的属性(而不是嵌套的<property/>元素)来描述协作bean的属性值，或者两者都使用。

Spring支持具有名称空间的可扩展配置格式，它基于XML Schema定义。本章中讨论的bean配置格式是在XML Schema文档中定义的。但是，p名称空间并没有定义在XSD文件中，它只存在于Spring的核心中。

下面的示例显示了两个XML片段(第一个使用标准XML格式，第二个使用p-名称空间)，它们解析相同的结果:

```xml
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:p="http://www.springframework.org/schema/p"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean name="classic" class="com.example.ExampleBean">
        <property name="email" value="someone@somewhere.com"/>
    </bean>

    <bean name="p-namespace" class="com.example.ExampleBean"
        p:email="someone@somewhere.com"/>
</beans>
```
该示例显示了bean定义中p名称空间中名为email的属性。这告诉Spring包含一个属性声明。如前所述，p-名称空间没有模式定义，因此可以将属性的名称设置为属性名。

下一个例子包括另外两个bean定义，它们都引用了另一个bean:

```xml
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:p="http://www.springframework.org/schema/p"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean name="john-classic" class="com.example.Person">
        <property name="name" value="John Doe"/>
        <property name="spouse" ref="jane"/>
    </bean>

    <bean name="john-modern"
        class="com.example.Person"
        p:name="John Doe"
        p:spouse-ref="jane"/>

    <bean name="jane" class="com.example.Person">
        <property name="name" value="Jane Doe"/>
    </bean>
</beans>
```
这个例子不仅包括使用p命名空间的属性值，而且还使用一种特殊的格式来声明属性引用。第一个bean定义使用<property name="spouse" ref="jane"/>来创建一个从bean john到bean jane的引用，而第二个bean定义使用p:spouse-ref="jane"作为属性来做同样的事情。在本例中，spouse是属性名，而-ref部分表明这不是一个直接的值，而是对另一个bean的引用。

> 注意：
> p名称空间不像标准XML格式那样灵活。例如，声明属性引用的格式与以Ref结尾的属性冲突，而标准XML格式则不会。我们建议您仔细选择方法，并将其告知您的团队成员，以避免生成同时使用所有三种方法的XML文档。

### 带有c命名空间的XML快捷方式

与带有p-名称空间的XML快捷方式类似，Spring 3.1中引入的c-名称空间允许内联属性来配置构造函数参数，而不是嵌套构造函数参数元素。
下面的例子使用c:命名空间来做与基于from构造器的依赖注入相同的事情:

```xml
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:c="http://www.springframework.org/schema/c"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="beanTwo" class="x.y.ThingTwo"/>
    <bean id="beanThree" class="x.y.ThingThree"/>

    <!-- traditional declaration with optional argument names -->
    <bean id="beanOne" class="x.y.ThingOne">
        <constructor-arg name="thingTwo" ref="beanTwo"/>
        <constructor-arg name="thingThree" ref="beanThree"/>
        <constructor-arg name="email" value="something@somewhere.com"/>
    </bean>

    <!-- c-namespace declaration with argument names -->
    <bean id="beanOne" class="x.y.ThingOne" c:thingTwo-ref="beanTwo"
        c:thingThree-ref="beanThree" c:email="something@somewhere.com"/>

</beans>
```
命名空间c:使用与p: one (bean引用末尾的-ref)相同的约定，通过名称设置构造函数参数。类似地，它需要在XML文件中声明，即使它没有在XSD模式中定义(它存在于Spring核心中)。

对于构造函数参数名不可用的罕见情况(通常是在没有调试信息的情况下编译字节码)，可以使用回退参数索引，如下所示:

```xml
<!-- c-namespace index declaration -->
<bean id="beanOne" class="x.y.ThingOne" c:_0-ref="beanTwo" c:_1-ref="beanThree"
    c:_2="something@somewhere.com"/>
```
由于XML语法的原因，索引表示法要求使用前导_，因为XML属性名不能以数字开头(尽管一些ide允许)。对于<constructor-arg>元素，也可以使用相应的索引表示法，但不常用，因为一般的声明顺序就足够了。


### 复合属性名

在实践中，构造函数解析机制在匹配参数方面非常有效，所以除非真的需要，否则我们建议在整个配置中使用名称表示法。

在设置bean属性时，可以使用复合或嵌套属性名，只要路径的所有组件(最终属性名除外)不为空。考虑以下bean定义:

```xml
<bean id="something" class="things.ThingOne">
    <property name="fred.bob.sammy" value="123" />
</bean>
```

something bean有一个fred属性，fred属性有一个bob属性，bob属性有一个sammy属性，最后的sammy属性被设置为123。为了使其工作，在构造bean之后，fred的fred属性和fred的bob属性不能为空。否则，抛出NullPointerException。

## 使用 depends-on

如果一个bean是另一个bean的依赖项，这通常意味着一个bean被设置为另一个bean的属性。通常使用基于xml的配置元数据中的<ref/>元素来实现这一点。然而，有时bean之间的依赖关系不那么直接。例如，当需要触发类中的静态初始化式时，例如数据库驱动程序注册。依赖属性可以显式强制在初始化使用此元素的bean之前初始化一个或多个bean。下面的例子使用依赖属性来表示对单个bean的依赖关系:

```xml
<bean id="beanOne" class="ExampleBean" depends-on="manager"/>
<bean id="manager" class="ManagerBean" />
```

要表示对多个bean的依赖关系，提供一个bean名称列表作为依赖属性的值(逗号、空格和分号是有效的分隔符):

```xml
<bean id="beanOne" class="ExampleBean" depends-on="manager,accountDao">
    <property name="manager" ref="manager" />
</bean>

<bean id="manager" class="ManagerBean" />
<bean id="accountDao" class="x.y.jdbc.JdbcAccountDao" />
```

依赖属性既可以指定初始化时依赖关系，也可以指定对应的销毁时依赖关系(仅在单例bean中)。在销毁给定bean本身之前，首先销毁与给定bean定义依赖关系的依赖bean。因此，依赖还可以控制关机顺序。

## 懒加载Bean

默认情况下，ApplicationContext实现会作为初始化过程的一部分主动创建和配置所有单例bean。通常，这种预实例化是可取的，因为配置或周围环境中的错误是立即发现的，而不是几个小时甚至几天之后。当这种行为不可取时，您可以通过将bean定义标记为惰性初始化来防止单例bean的预实例化。延迟初始化的bean告诉IoC容器在第一次请求时创建bean实例，而不是在启动时。

在XML中，这个行为是由<bean/>元素上的lazy-init属性控制的，如下面的示例所示:

```xml
<bean id="lazy" class="com.something.ExpensiveToCreateBean" lazy-init="true"/>
<bean name="not.lazy" class="com.something.AnotherBean"/>
```

当上述配置被ApplicationContext使用时，惰性bean不会在ApplicationContext启动时立即预实例化，而`not.lazy` bean是主动预实例化的。

然而，当一个延迟初始化的bean是一个没有延迟初始化的单例bean的依赖项时，ApplicationContext在启动时创建这个延迟初始化的bean，因为它必须满足单例bean的依赖项。延迟初始化的bean被注入到没有延迟初始化的其他地方的单例bean中。

您还可以在容器级使用`<beans/>`元素上的default-lazy-init属性来控制延迟初始化，如下面的示例所示:

```xml
<beans default-lazy-init="true">
    <!-- no beans will be pre-instantiated... -->
</beans>

```

## 自动装配器

Spring容器可以自动装配协作bean之间的关系。您可以通过检查ApplicationContext的内容，让Spring自动为您的bean解析合作者(其他bean)。自动装配具有以下优点:

* 自动装配可以显著减少指定属性或构造函数参数的需要。(本章其他部分讨论的其他机制，如bean模板，在这方面也很有价值。
* 自动装配可以随着对象的发展更新配置。例如，如果您需要向类添加依赖项，则无需修改配置即可自动满足该依赖项。因此，自动连接在开发过程中特别有用，当代码库变得更加稳定时，无需放弃切换到显式连接的选项。

在使用基于xml的配置元数据时(请参阅依赖项注入)，您可以使用<bean/>元素的自动连接属性为bean定义指定自动连接模式。自动装配功能有四种模式。您可以指定每个bean的自动装配，从而可以选择要自动装配哪些bean。自动装配的四种模式如下表所示:

|Mode|Explanation|
|---|---|
|no|(默认)没有自动装配。Bean引用必须由ref元素定义。对于较大的部署，不建议更改默认设置，因为明确指定协作者可以提供更大的控制和清晰度。在某种程度上，它记录了系统的结构。|
|byName|通过属性名自动装配。Spring寻找与需要自动连接的属性同名的bean。例如，如果一个bean定义被设置为按名称自动连接，并且它包含一个主属性(也就是说，它有一个setMaster(..)方法)，Spring就会寻找一个名为master的bean定义并使用它来设置属性。|
|byType|如果容器中恰好有一个属性类型的bean，则允许自动连接属性。如果存在多个，则抛出致命异常，这表明您不能对该bean使用byType自动装配。如果没有匹配的bean，则不会发生任何事情(没有设置属性)。|
|constructor|类似于byType，但应用于构造函数参数。如果容器中没有一个构造函数参数类型的bean，则会引发致命错误。|

使用byType或构造函数自动装配模式，可以连接数组和类型化集合。在这种情况下，容器中所有匹配预期类型的自动装配候选对象都将被提供以满足依赖关系。如果期望的键类型是String，则可以自动装配强类型Map实例。自动连接的Map实例的值包含所有与期望类型匹配的bean实例，Map实例的键包含相应的bean名称。

### 自动装配的局限性和缺点
当在整个项目中始终如一地使用自动装配时，它的工作效果最好。如果自动装配通常不使用，那么只使用它来连接一个或两个bean定义可能会让开发人员感到困惑。

考虑自动装配的局限性和缺点:

* 属性和构造参数设置中的显式依赖总是覆盖自动装配。不能自动连接简单属性，如原语、字符串和类(以及这些简单属性的数组)。这种限制是由设计造成的。
* 自动布线没有显式布线精确。不过，正如前面的表中所指出的，Spring小心地避免猜测，以防出现可能产生意外结果的歧义。spring管理对象之间的关系不再显式地记录下来。
* 连接信息可能对可能从Spring容器生成文档的工具不可用。
* 容器内的多个bean定义可以与要自动连接的setter方法或构造函数参数指定的类型匹配。对于数组、集合或Map实例，这并不一定是个问题。然而，对于期望单个值的依赖项，这种模糊性是不能任意解决的。如果没有唯一的bean定义可用，则抛出异常。

在后一种情况下，您有几个选项:

* 放弃自动布线，选择显式布线。
* 通过将bean的autowire候选属性设置为false，可以避免bean定义的自动装配，如下一节所述。
* 通过将其<bean/>元素的primary属性设置为true，将单个bean定义指定为主要候选项
* 实现基于注释的配置中提供的更细粒度的控制，如基于注释的容器配置中所述。

### 从自动装配中排除Bean

在每个bean的基础上，您可以将一个bean排除在自动装配之外。在Spring的XML格式中，将<bean/>元素的autowire-candidate属性设置为false。容器使特定的bean定义对自动装配基础设施不可用(包括@Autowired这样的注释样式配置)。

autowire-candidate属性被设计为仅影响基于类型的自动装配。它不会影响按名称的显式引用，即使指定的bean没有被标记为自动连接候选对象，也会解析该引用。因此，如果名称匹配，按名称自动装配仍然会注入一个bean。

您还可以根据bean名称的模式匹配来限制自动装配候选对象。顶级元素`<beans/>`在其default-autowire-candidates属性中接受一个或多个模式。例如，要将自动连接候选状态限制为名称以Repository结尾的任何bean，可以提供*Repository的值。要提供多个模式，请在逗号分隔的列表中定义它们。bean定义的自动连接候选属性的显式值为true或false总是优先。对于这样的bean，模式匹配规则不适用。

这些技术对于那些永远不想通过自动装配将其注入到其他bean中的bean非常有用。这并不意味着被排除的bean本身不能通过使用自动装配来配置。相反，bean本身不是自动装配其他bean的候选对象。


## 方法注入

在大多数应用程序场景中，容器中的大多数bean都是单例的。当一个单例bean需要与另一个单例bean协作，或者一个非单例bean需要与另一个非单例bean协作时，通常通过将一个bean定义为另一个bean的属性来处理依赖关系。当bean的生命周期不同时，就会出现问题。假设单例bean A需要使用非单例(原型)bean B，可能是在A的每个方法调用上。容器只创建单例bean A一次，因此只有一次机会设置属性。容器不能在每次需要bean A时都向bean A提供一个新的bean B实例。

一个解决方案是放弃一些控制反转。您可以通过实现ApplicationContextAware接口，以及在bean A每次需要bean B实例时对容器进行getBean(“B”)调用，从而使bean A意识到容器。下面的例子展示了这种方法:

```java
// a class that uses a stateful Command-style class to perform some processing
package fiona.apple;

// Spring-API imports
import org.springframework.beans.BeansException;
import org.springframework.context.ApplicationContext;
import org.springframework.context.ApplicationContextAware;

public class CommandManager implements ApplicationContextAware {

    private ApplicationContext applicationContext;

    public Object process(Map commandState) {
        // grab a new instance of the appropriate Command
        Command command = createCommand();
        // set the state on the (hopefully brand new) Command instance
        command.setState(commandState);
        return command.execute();
    }

    protected Command createCommand() {
        // notice the Spring API dependency!
        return this.applicationContext.getBean("command", Command.class);
    }

    public void setApplicationContext(
            ApplicationContext applicationContext) throws BeansException {
        this.applicationContext = applicationContext;
    }
}
```
前面的操作是不可取的，因为业务代码知道Spring框架并与之耦合。方法注入是Spring IoC容器的一个稍高级的特性，它允许您整洁地处理这个用例。

### 查找方法注入
查找方法注入是容器覆盖容器管理bean上的方法并返回容器中另一个命名bean的查找结果的能力。查询通常涉及一个原型bean，如上一节描述的场景所示。Spring框架通过使用来自CGLIB库的字节码生成来动态生成覆盖该方法的子类来实现这种方法注入。

* 要使这个动态子类化工作，Spring bean容器子类的类不能是final，要覆盖的方法也不能是final。
* 对具有抽象方法的类进行单元测试，要求您自己子类化该类，并提供抽象方法的存根实现。
* 具体的方法对于组件扫描也是必要的，这需要具体的类来拾取。
* 另一个关键限制是，查找方法不能与工厂方法一起工作，特别是不能与配置类中的@Bean方法一起工作，因为在这种情况下，容器不负责创建实例，因此不能动态地创建运行时生成的子类。

对于前面代码片段中的CommandManager类，Spring容器动态地覆盖createCommand()方法的实现。CommandManager类没有任何Spring依赖项，正如重做的示例所示:

```java
package fiona.apple;

// no more Spring imports!

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
在包含要注入方法的客户端类中(本例中是CommandManager)，要注入的方法需要以下形式的签名:

```xml
<public|protected> [abstract] <return-type> theMethodName(no-arguments);
```

如果方法是抽象的，则动态生成的子类实现该方法。否则，动态生成的子类将重写在原始类中定义的具体方法。考虑下面的例子:

```xml
<!-- a stateful bean deployed as a prototype (non-singleton) -->
<bean id="myCommand" class="fiona.apple.AsyncCommand" scope="prototype">
    <!-- inject dependencies here as required -->
</bean>

<!-- commandProcessor uses statefulCommandHelper -->
<bean id="commandManager" class="fiona.apple.CommandManager">
    <lookup-method name="createCommand" bean="myCommand"/>
</bean>
```
标识为commandManager的bean在需要myCommand bean的新实例时调用它自己的createCommand()方法。如果实际需要将myCommand bean部署为原型，则必须小心。如果它是一个单例，则每次都会返回myCommand bean的相同实例。

或者，在基于注释的组件模型中，您可以通过@Lookup注释声明一个查找方法，如下面的示例所示:

```java
public abstract class CommandManager {

    public Object process(Object commandState) {
        Command command = createCommand();
        command.setState(commandState);
        return command.execute();
    }

    @Lookup("myCommand")
    protected abstract Command createCommand();
}
```

或者，更习惯地说，您可以依赖于根据声明的查找方法返回类型解析目标bean:

```java
public abstract class CommandManager {

    public Object process(Object commandState) {
        Command command = createCommand();
        command.setState(commandState);
        return command.execute();
    }

    @Lookup
    protected abstract Command createCommand();
}
```

注意，通常应该使用具体的存根实现声明这种带注释的查找方法，以便它们与Spring的组件扫描规则(默认情况下抽象类会被忽略)兼容。此限制不适用于显式注册或显式导入的bean类。
访问范围不同的目标bean的另一种方法是ObjectFactory/ Provider注入点。参见将作用域bean作为依赖项。
您可能还会发现ServiceLocatorFactoryBean(在org.springframework.beans.factory.config包中)是有用的。

### 任意替换方法
方法注入的一种不如查找方法注入有用的形式是，能够用另一种方法实现替换托管bean中的任意方法。在真正需要此功能之前，您可以安全地跳过本节的其余部分。
对于基于xml的配置元数据，可以使用replacement -method元素将已部署bean的现有方法实现替换为另一个方法实现。考虑下面的类，它有一个我们想要覆盖的名为computeValue的方法:

```java
public class MyValueCalculator {

    public String computeValue(String input) {
        // some real code...
    }

    // some other methods...
}
```
一个实现了org.springframework.bean .factory. support.methodreplacer接口的类提供了新的方法定义，如下面的示例所示:

```java
/**
 * meant to be used to override the existing computeValue(String)
 * implementation in MyValueCalculator
 */
public class ReplacementComputeValue implements MethodReplacer {

    public Object reimplement(Object o, Method m, Object[] args) throws Throwable {
        // get the input value, work with it, and return a computed result
        String input = (String) args[0];
        ...
        return ...;
    }
}
```

用于部署原始类和指定方法覆盖的bean定义类似于以下示例:  
```xml
<bean id="myValueCalculator" class="x.y.z.MyValueCalculator">
    <!-- arbitrary method replacement -->
    <replaced-method name="computeValue" replacer="replacementComputeValue">
        <arg-type>String</arg-type>
    </replaced-method>
</bean>

<bean id="replacementComputeValue" class="a.b.c.ReplacementComputeValue"/>
```
您可以在<replace -method/>元素中使用一个或多个<arg-type/>元素来指示被覆盖的方法的方法签名。只有当方法被重载且类中存在多个变量时，参数的签名才有必要。为方便起见，实参的类型字符串可以是完全限定类型名的子字符串。例如，下面的所有代码都匹配
```text  
java.lang.String
String
Str
```
因为参数的数量通常足以区分每种可能的选择，这个快捷方式允许您只输入与参数类型匹配的最短字符串，从而节省了大量的输入。


> * [使用Spring MVC服务Web内容](https://spring.io/guides/gs/serving-web-content/)
> * [Spring安全架构(参考指南)](https://spring.io/guides/topicals/spring-security-architecture/)
> * [Spring Framework官方文档](https://spring.io/projects/spring-framework)
> * [SpringIoc简介](https://github.com/fredomli/java-standard/blob/main/docs/spring/spring/core/SpringIoc容器.md)
> * [Spring容器使用](https://github.com/fredomli/java-standard/blob/main/docs/spring/spring/core/Spring容器使用.md)
> * [SpringIoc简介](https://github.com/fredomli/java-standard/blob/main/docs/spring/spring/core/SpringIoc容器.md)


*(☞ﾟヮﾟ)☞[返回首页README.md address](https://github.com/fredomli/java-standard)*

