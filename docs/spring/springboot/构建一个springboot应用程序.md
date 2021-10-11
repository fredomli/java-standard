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
> 参阅
> * [使用Spring MVC服务Web内容](https://spring.io/guides/gs/serving-web-content/)
> * [Spring安全架构(参考指南)](https://spring.io/guides/topicals/spring-security-architecture/)
> * [设置Spring安全 (SpringMVC整合Spring Security)](https://github.com/fredomli/java-standard/blob/main/docs/spring/security/创建一个不安全的Web应用.md)

*(☞ﾟヮﾟ)☞[返回首页README.md address](https://github.com/fredomli/java-standard)*
