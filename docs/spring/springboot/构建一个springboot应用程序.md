# 构建一个Spring Boot 应用程序

本指南提供了Spring Boot如何帮助您加速应用程序开发的示例。随着您阅读更多Spring Getting Started指南，您将看到更多Spring Boot用例。本指南旨在让您快速体验Spring Boot。如果您想创建自己的基于Spring boot的项目，请访问[Spring Initializr](https://start.spring.io/) ，填写项目详细信息，选择选项，并下载打包的项目作为zip文件。

### 你将做什么
您将使用Spring Boot构建一个简单的web应用程序，并向其中添加一些有用的服务。

*准备：*
* 15分钟时间
* 最喜欢文本编辑器或IDE
* JDK1.8或更高版本
* Gradle4+ 或 Maven3.2+
* 也可以直接将代码导入IDEA中:
    * [Spring Tool Suite (STS)](Spring Tool Suite (STS))
    * [IntelliJ IDEA](https://spring.io/guides/gs/intellij-idea/)

### 如何完成本指南

需要跳过初始化项目的步骤可以执行下列操作：
* 下载并解压缩本指南的源代码库，或者使用Git克隆它:git clone https://github.com/spring-guides/gs-spring-boot.git
* cd到gs-spring-boot/initial
* 跳转到后面步骤`创建一个不安全的应用程序。`    

完成后，可以根据gs-spring-boot/complete中的代码检查结果。

### 你可以用Spring Boot做什么
Spring Boot提供了一种构建应用程序的快速方法。它查看您的类路径和您配置的bean，对您缺少的内容做出合理的假设，然后添加那些项。使用Spring Boot，您可以更多地关注业务特性，而较少地关注基础设施。

下面的例子展示了Spring Boot可以为您做什么:
* Spring MVC在类路径上吗?您几乎总是需要一些特定的bean, Spring Boot会自动添加它们。Spring MVC应用程序还需要servlet容器，因此Spring Boot自动配置嵌入式Tomcat。
* Jetty在类路径上吗?如果是这样，您可能不想要Tomcat，而是想要嵌入式Jetty。Spring Boot为您处理。
* Thymeleaf在类路径上吗?如果是这样，则必须始终将一些bean添加到应用程序上下文中。Spring Boot为您添加了它们。

这些只是Spring Boot提供的自动配置的几个示例。与此同时，Spring Boot不会妨碍您。例如，如果Thymeleaf在您的路径上，Spring Boot会自动将SpringTemplateEngine添加到应用程序上下文。但是，如果您使用自己的设置定义自己的SpringTemplateEngine, Spring Boot不会添加一个。这让你不用付出多少努力就能控制局面。

```
注意：
Spring Boot不会生成代码或对文件进行编辑。相反，在启动应用程序时，Spring Boot会动态连接bean和设置，并将它们应用到应用程序上下文。
```
### 通过官网创建项目

> 您可以使用这个 [预先初始化的项目](https://start.spring.io/) 并单击Generate下载ZIP文件。此项目已配置为适合本教程中的示例。

### 手动初始化项目步骤
* 导航到 https://start.spring.io。这个服务会引入应用程序所需的所有依赖项，并为您完成大部分设置。
* 选择Gradle或Maven和你想使用的语言。本指南假设您选择了Java。
* 单击Dependencies并选择Spring Web和Thymeleaf。
* 点击构建
* 下载得到的ZIP文件，这是一个web应用程序的存档文件，它配置了您的选择。

> 如果您的IDE具有Spring Initializr集成，您可以从您的IDE完成这个过程。
>
> 您还可以从Github中派生项目，并在IDE或其他编辑器中打开它。

### 创建一个简单的Web应用程序
现在你可以为一个简单的web应用程序创建一个web控制器，如下所示:

```java
package com.example.springboot;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class HelloController {

  @GetMapping("/")
  public String index() {
    return "Greetings from Spring Boot!";
  }

}
```

这个类被标记为@RestController，这意味着它可以被Spring MVC使用来处理web请求。@GetMapping将/映射到index()方法。当从浏览器调用或在命令行上使用curl时，该方法将返回纯文本。这是因为@RestController结合了@Controller和@ResponseBody，这两个注解会导致web请求返回数据而不是视图。

### 创建一个Application类
Spring Initializr为您创建了一个简单的应用程序类。然而，在这种情况下，它太简单了。你需要修改应用程序类以匹配以下清单(来自src/main/java/com/example/springboot/Application.java):

```java
package com.example.springboot;

import java.util.Arrays;

import org.springframework.boot.CommandLineRunner;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.context.ApplicationContext;
import org.springframework.context.annotation.Bean;

@SpringBootApplication
public class Application {

	public static void main(String[] args) {
		SpringApplication.run(Application.class, args);
	}

	@Bean
	public CommandLineRunner commandLineRunner(ApplicationContext ctx) {
		return args -> {

			System.out.println("Let's inspect the beans provided by Spring Boot:");

			String[] beanNames = ctx.getBeanDefinitionNames();
			Arrays.sort(beanNames);
			for (String beanName : beanNames) {
				System.out.println(beanName);
			}

		};
	}

}
```

@SpringBootApplication是一个方便的注释，它添加了以下所有内容:
* @Configuration:将类标记为应用程序上下文的bean定义的源。
* @EnableAutoConfiguration:告诉Spring Boot开始添加基于类路径设置、其他bean和各种属性设置的bean。例如，如果spring-webmvc在类路径上，这个注释将把应用程序标记为web应用程序，并激活关键行为，例如设置DispatcherServlet。
* @ComponentScan:告诉Spring在com/example包中寻找其他组件、配置和服务，让它找到控制器。

main()方法使用Spring Boot的SpringApplication.run()方法来启动应用程序。您注意到没有任何一行XML吗?也没有web.xml文件。这个web应用程序是100%纯Java的，你不需要配置任何管道或基础设施。

还有一个标记为@Bean的CommandLineRunner方法，它在启动时运行。它检索由应用程序创建的或由Spring Boot自动添加的所有bean。它把它们分类并打印出来。  

### 构建与运行应用程序
要运行该应用程序，请在终端窗口(完整)目录下运行以下命令:  

```text
./gradlew bootRun
```

如果您使用Maven，请在终端窗口(完整)目录中运行以下命令:  
```text
./mvnw spring-boot:run
```
你应该看到类似如下的输出:  

```text
Let's inspect the beans provided by Spring Boot:
application
beanNameHandlerMapping
defaultServletHandlerMapping
dispatcherServlet
embeddedServletContainerCustomizerBeanPostProcessor
handlerExceptionResolver
helloController
httpRequestHandlerAdapter
messageSource
mvcContentNegotiationManager
mvcConversionService
mvcValidator
org.springframework.boot.autoconfigure.MessageSourceAutoConfiguration
org.springframework.boot.autoconfigure.PropertyPlaceholderAutoConfiguration
org.springframework.boot.autoconfigure.web.EmbeddedServletContainerAutoConfiguration
org.springframework.boot.autoconfigure.web.EmbeddedServletContainerAutoConfiguration$DispatcherServletConfiguration
org.springframework.boot.autoconfigure.web.EmbeddedServletContainerAutoConfiguration$EmbeddedTomcat
org.springframework.boot.autoconfigure.web.ServerPropertiesAutoConfiguration
org.springframework.boot.context.embedded.properties.ServerProperties
org.springframework.context.annotation.ConfigurationClassPostProcessor.enhancedConfigurationProcessor
org.springframework.context.annotation.ConfigurationClassPostProcessor.importAwareProcessor
org.springframework.context.annotation.internalAutowiredAnnotationProcessor
org.springframework.context.annotation.internalCommonAnnotationProcessor
org.springframework.context.annotation.internalConfigurationAnnotationProcessor
org.springframework.context.annotation.internalRequiredAnnotationProcessor
org.springframework.web.servlet.config.annotation.DelegatingWebMvcConfiguration
propertySourcesBinder
propertySourcesPlaceholderConfigurer
requestMappingHandlerAdapter
requestMappingHandlerMapping
resourceHandlerMapping
simpleControllerHandlerAdapter
tomcatEmbeddedServletContainerFactory
viewControllerHandlerMapping
```
您可以清楚地看到`org.springframework.boot.autoconfigure bean`。还有一个`tomcatEmbeddedServletContainerFactory`。

现在使用curl运行服务(在一个单独的终端窗口中)，运行以下命令(显示其输出):

```text
$ curl localhost:8080
Greetings from Spring Boot!
```
### 添加单元测试
您可能希望为添加的端点添加一个测试，而Spring test为此提供了一些机制。

如果你使用的是Gradle，请在你的构建中添加以下依赖项。gradle文件:

```text
testImplementation('org.springframework.boot:spring-boot-starter-test')
```

如果你使用Maven，在你的pom.xml文件中添加以下内容:

```xml
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-test</artifactId>
	<scope>test</scope>
</dependency>
```
现在编写一个简单的单元测试，通过端点模拟servlet请求和响应，如下所示(来自src/test/java/com/example/springboot/HelloControllerTest.java):

```java
package com.example.springboot;

import static org.hamcrest.Matchers.equalTo;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.content;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.status;

import org.junit.jupiter.api.Test;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.autoconfigure.web.servlet.AutoConfigureMockMvc;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.http.MediaType;
import org.springframework.test.web.servlet.MockMvc;
import org.springframework.test.web.servlet.request.MockMvcRequestBuilders;

@SpringBootTest
@AutoConfigureMockMvc
public class HelloControllerTest {

	@Autowired
	private MockMvc mvc;

	@Test
	public void getHello() throws Exception {
		mvc.perform(MockMvcRequestBuilders.get("/").accept(MediaType.APPLICATION_JSON))
				.andExpect(status().isOk())
				.andExpect(content().string(equalTo("Greetings from Spring Boot!")));
	}
}
```
MockMvc来自Spring Test，它允许您通过一组方便的构建器类将HTTP请求发送到DispatcherServlet，并对结果进行断言。注意使用@AutoConfigureMockMvc和@SpringBootTest来注入MockMvc实例。使用@SpringBootTest,我们要求创建整个应用程序上下文。另一种选择是要求Spring Boot使用@WebMvcTest只创建上下文的web层。在这两种情况下，Spring Boot都会自动尝试定位应用程序的主应用程序类，但是如果您想构建一些不同的东西，可以覆盖它或缩小它的范围。

除了模拟HTTP请求周期外，您还可以使用Spring Boot编写一个简单的全栈集成测试。例如，我们可以创建以下测试(来自src/test/java/com/example/springboot/HelloControllerIT.java)，而不是前面显示的模拟测试:

```java
package com.example.springboot;

import org.junit.jupiter.api.Test;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.boot.test.web.client.TestRestTemplate;
import org.springframework.http.ResponseEntity;

import static org.assertj.core.api.Assertions.assertThat;

@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
public class HelloControllerIT {

	@Autowired
	private TestRestTemplate template;

    @Test
    public void getHello() throws Exception {
        ResponseEntity<String> response = template.getForEntity("/", String.class);
        assertThat(response.getBody()).isEqualTo("Greetings from Spring Boot!");
    }
}
```
由于webEnvironment = SpringBootTest.WebEnvironment，嵌入式服务器在一个随机端口上启动。RANDOM_PORT，实际的端口在teststtemplate的基础URL中自动配置。

### 添加产品级服务

如果你正在为你的企业建立一个网站，你可能需要添加一些管理服务。Spring Boot通过其执行器模块提供了几个这样的服务(如运行状况、审计、bean等)。

如果你使用的是Gradle，请在你的构建中添加以下依赖项。gradle文件:

```text
implementation 'org.springframework.boot:spring-boot-starter-actuator'
```

如果你使用Maven，请将以下依赖项添加到pomo .xml文件中:  

```xml
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```
然后重新启动应用程序。如果你使用Gradle，在终端窗口中(在完整目录中)运行以下命令:
```text
./gradlew bootRun
```
如果您使用Maven，请在终端窗口中(在完整目录中)运行以下命令:
```xml
./mvnw spring-boot:run
```
您应该会看到，应用程序中添加了一组新的RESTful端点。这些是由Spring Boot提供的管理服务。下面的清单显示了典型的输出:
```text
management.endpoint.configprops-org.springframework.boot.actuate.autoconfigure.context.properties.ConfigurationPropertiesReportEndpointProperties
management.endpoint.env-org.springframework.boot.actuate.autoconfigure.env.EnvironmentEndpointProperties
management.endpoint.health-org.springframework.boot.actuate.autoconfigure.health.HealthEndpointProperties
management.endpoint.logfile-org.springframework.boot.actuate.autoconfigure.logging.LogFileWebEndpointProperties
management.endpoints.jmx-org.springframework.boot.actuate.autoconfigure.endpoint.jmx.JmxEndpointProperties
management.endpoints.web-org.springframework.boot.actuate.autoconfigure.endpoint.web.WebEndpointProperties
management.endpoints.web.cors-org.springframework.boot.actuate.autoconfigure.endpoint.web.CorsEndpointProperties
management.health.diskspace-org.springframework.boot.actuate.autoconfigure.system.DiskSpaceHealthIndicatorProperties
management.info-org.springframework.boot.actuate.autoconfigure.info.InfoContributorProperties
management.metrics-org.springframework.boot.actuate.autoconfigure.metrics.MetricsProperties
management.metrics.export.simple-org.springframework.boot.actuate.autoconfigure.metrics.export.simple.SimpleProperties
management.server-org.springframework.boot.actuate.autoconfigure.web.server.ManagementServerProperties
```
#### 执行器公开以下内容:
* actuator/health(http://localhost:8080/actuator/health)
* actuator

还有一个/actuator/shutdown端点，但默认情况下，它仅通过JMX可见。要将其启用为HTTP端点，请添加management.endpoint.shutdown=true。属性文件并使用management.endpoints.web.exposure.include=health,info,shutdown。但是，您可能不应该为公开可用的应用程序启用关闭端点。  

可以运行以下命令检查应用程序的运行状况:
```shell
$ curl localhost:8080/actuator/health
{"status":"UP"}
```

您还可以尝试通过curl调用shutdown，以查看当您没有向application.properties添加必要的行(如前面的注释所示)时会发生什么:

```shell
$ curl -X POST localhost:8080/actuator/shutdown
{"timestamp":1401820343710,"error":"Not Found","status":404,"message":"","path":"/actuator/shutdown"}
```
因为我们没有启用它，所以请求的端点不可用(因为端点不存在)。
有关每个REST端点的详细信息，以及如何使用应用程序调优它们的设置。属性文件(在src/main/resources中)，请参阅关于端点的文档。

### 查看Spring Boot的启动器

您已经看到了Spring Boot的一些“启动器”。你可以在源代码中看到它们。

### JAR支持和Groovy支持
最后一个示例展示了Spring Boot如何让您连接您可能不知道自己需要的bean。它还展示了如何开启便捷的管理服务。
然而，Spring Boot做的不止这些。由于Spring Boot的加载器模块，它不仅支持传统的WAR文件部署，还允许将可执行jar放在一起。各种指南通过spring-boot-gradle-plugin和spring-boot-maven-plugin演示了这种双重支持。


除此之外，Spring Boot还支持Groovy，让您可以用一个简单的文件构建Spring MVC web应用程序。

创建一个名为app.groovy的新文件，并放入以下代码:

```groovy
@RestController
class ThisWillActuallyRun {

    @GetMapping("/")
    String home() {
        return "Hello, World!"
    }

}
```

文件在哪里并不重要。你甚至可以在一条tweet中安装这么小的应用程序!

接下来，安装Spring Boot的CLI。  
运行以下命令运行Groovy应用程序:

> Spring Boot CLI(命令行接口)是一个命令行工具，您可以使用它来快速创建Spring的原型。它允许您运行Groovy脚本，这意味着您有一个熟悉的类似java的语法，而没有太多的样板代码。
您不需要使用CLI来使用Spring Boot，但这是在没有IDE的情况下启动Spring应用程序的一种快速方法。

```shell
$ spring run app.groovy
```
关闭之前的应用程序，以避免端口冲突。

从一个不同的终端窗口，运行以下curl命令(显示其输出):
```shell
$ curl localhost:8080
Hello, World!
```
Spring Boot通过在代码中动态添加关键注释，并使用Groovy Grape下拉运行应用程序所需的库来实现这一点。
#### 总结
恭喜你!您使用Spring Boot构建了一个简单的web应用程序，并了解了它如何提高您的开发速度。您还打开了一些方便的生产服务。这只是Spring Boot功能的一小部分示例。更多信息请参见[Spring Boot的在线文档](https://docs.spring.io/spring-boot/docs/current/reference/htmlsingle/) 。
> 参阅
> * [使用Spring MVC服务Web内容](https://spring.io/guides/gs/serving-web-content/)
> * [Spring安全架构(参考指南)](https://spring.io/guides/topicals/spring-security-architecture/)
> * [设置Spring安全 (SpringMVC整合Spring Security)](https://github.com/fredomli/java-standard/blob/main/docs/spring/security/创建一个不安全的Web应用.md)

*(☞ﾟヮﾟ)☞[返回首页README.md address](https://github.com/fredomli/java-standard)*
