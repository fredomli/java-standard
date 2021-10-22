# Bean定义继承

bean定义可以包含大量配置信息，包括构造函数参数、属性值和特定于容器的信息，比如初始化方法、静态工厂方法名，等等。子bean定义从父定义继承配置数据。子定义可以根据需要覆盖一些值或添加其他值。使用父bean和子bean定义可以节省大量输入。实际上，这是模板的一种形式。

如果您以编程方式使用ApplicationContext接口，子bean定义将由ChildBeanDefinition类表示。大多数用户不会在这个级别上使用它们。相反，它们在类(如ClassPathXmlApplicationContext)中以声明方式配置bean定义。当您使用基于xml的配置元数据时，您可以通过使用父属性来指示子bean定义，并将父bean指定为该属性的值。下面的例子展示了如何做到这一点:

```xml
<bean id="inheritedTestBean" abstract="true"
        class="org.springframework.beans.TestBean">
    <property name="name" value="parent"/>
    <property name="age" value="1"/>
</bean>

<bean id="inheritsWithDifferentClass"
        class="org.springframework.beans.DerivedTestBean"
        parent="inheritedTestBean" init-method="initialize">  
    <property name="name" value="override"/>
    <!-- the age property value of 1 will be inherited from parent -->
</bean>
```

如果没有指定，子bean定义将使用来自父定义的bean类，但也可以覆盖它。在后一种情况下，子bean类必须与父bean兼容(也就是说，它必须接受父bean的属性值)。

子bean定义从父bean继承范围、构造函数参数值、属性值和方法覆盖，并可选择添加新值。您指定的任何范围、初始化方法、销毁方法或静态工厂方法设置都会覆盖相应的父方法设置。

其余的设置总是取自子定义:依赖、自动连接模式、依赖项检查、单例和延迟初始化。

前面的示例通过使用抽象属性显式地将父bean定义标记为抽象。如果父bean定义没有指定类，则需要显式地将父bean定义标记为抽象，如下例所示:
```xml
<bean id="inheritedTestBeanWithoutClass" abstract="true">
    <property name="name" value="parent"/>
    <property name="age" value="1"/>
</bean>

<bean id="inheritsWithClass" class="org.springframework.beans.DerivedTestBean"
        parent="inheritedTestBeanWithoutClass" init-method="initialize">
    <property name="name" value="override"/>
    <!-- age will inherit the value of 1 from the parent bean definition-->
</bean>
```
父bean不能单独实例化，因为它是不完整的，而且它也显式地被标记为抽象的。当定义是抽象的时候，它只能作为一个纯模板bean定义使用，作为子定义的父定义。如果试图单独使用这样一个抽象的父bean，将其作为另一个bean的ref属性引用，或者使用父bean ID执行显式的getBean()调用，将会返回错误。类似地，容器的内部preinstantiatesingleton()方法会忽略定义为抽象的bean定义。


默认情况下，ApplicationContext预实例化所有单例。因此,它是重要的(至少对单例bean),如果你有一个(父)bean定义你只打算使用作为模板,这个定义指定了一个类,您必须确保设置抽象属性为true,否则应用程序上下文会(试图)实例化抽象的bean。




> * [使用Spring MVC服务Web内容](https://spring.io/guides/gs/serving-web-content/)
> * [Spring安全架构(参考指南)](https://spring.io/guides/topicals/spring-security-architecture/)
> * [Spring Framework官方文档](https://spring.io/projects/spring-framework)
> * [SpringIoc简介](https://github.com/fredomli/java-standard/blob/main/docs/spring/spring/core/SpringIoc容器.md)
> * [Spring容器使用](https://github.com/fredomli/java-standard/blob/main/docs/spring/spring/core/Spring容器使用.md)
> * [SpringIoc简介](https://github.com/fredomli/java-standard/blob/main/docs/spring/spring/core/SpringIoc容器.md)
> * [SpringBean作用域](https://github.com/fredomli/java-standard/blob/main/docs/spring/spring/core/SpringBean作用域.md)


*(☞ﾟヮﾟ)☞[返回首页README.md address](https://github.com/fredomli/java-standard)*

