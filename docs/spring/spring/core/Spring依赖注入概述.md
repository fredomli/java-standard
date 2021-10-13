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

> * [使用Spring MVC服务Web内容](https://spring.io/guides/gs/serving-web-content/)
> * [Spring安全架构(参考指南)](https://spring.io/guides/topicals/spring-security-architecture/)
> * [Spring Framework官方文档](https://spring.io/projects/spring-framework)
> * [SpringIoc简介](https://github.com/fredomli/java-standard/blob/main/docs/spring/spring/core/SpringIoc容器.md)
> * [Spring容器使用](https://github.com/fredomli/java-standard/blob/main/docs/spring/spring/core/Spring容器使用.md)
> * [SpringIoc简介](https://github.com/fredomli/java-standard/blob/main/docs/spring/spring/core/SpringIoc容器.md)


*(☞ﾟヮﾟ)☞[返回首页README.md address](https://github.com/fredomli/java-standard)*

