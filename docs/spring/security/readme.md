# Spring Security

## 1. 介绍

> Spring Security is a powerful and highly customizable authentication and access-control framework. It is the de-facto standard for securing Spring-based applications.
>
> Spring Security is a framework that focuses on providing both authentication and authorization to Java applications. Like all Spring projects, the real power of Spring Security is found in how easily it can be extended to meet custom requirements

Spring Security是一个强大的、高度可定制的身份验证和访问控制框架。它是保护基于spring的应用程序的事实上的标准。
Spring Security是一个专注于为Java应用程序提供身份验证和授权的框架。与所有Spring项目一样，Spring Security的真正强大之处在于它可以很容易地扩展以满足定制需求

### 特点：  
* 对身份验证和授权的全面和可扩展支持
* 针对会话固定、点击劫持、跨站点请求伪造等攻击的保护
* Servlet API的集成
* 与Spring Web MVC的可选集成
* Much more…

注意：更多请参考 [官方文档](https://spring.io/projects/spring-security#overview)

## 2. 入门指南
> 该指南旨在在15-30分钟内,为Spring开发任务构建入门应用程序提供快速、实用的指导。 

具体参考官方文档 [保护Web应用程序](https://spring.io/guides/gs/securing-web/)  

### 2.1 获得一个Web应用程序
> 本指南将引导您使用Spring Security保护的资源创建一个简单的web应用程序。  

### 2.2 你将构建怎样的应用
> 您将构建一个Spring MVC应用程序，该应用程序使用一个由固定用户列表支持的登录表单来保护页面。  

*准备：*
* 15分钟时间
* 最喜欢文本编辑器或IDE
* JDK1.8或更高版本
* Gradle4+ 或 Maven3.2+
* 也可以直接将代码导入IDEA中:
  * [Spring Tool Suite (STS)](Spring Tool Suite (STS))
  * [IntelliJ IDEA](https://spring.io/guides/gs/intellij-idea/)
    

### 2.3 初始化一个Spring项目
> 像大多数Spring Getting Started指南一样，您可以从头开始并完成每个步骤，或者您可以绕过您已经熟悉的基本设置步骤。无论哪种方式，最终都会得到可工作的代码。  

要从头开始，请继续阅读[开始一个Spring初始化项目](https://github.com/fredomli/java-standard/blob/main/docs/spring/security/readme.md:53) 。

需要跳过初始化项目的步骤可以执行下列操作：
* 下载并解压缩本指南的源代码库，或者使用Git克隆它:Git clone https://github.com/spring-guides/gs-securing-web.git
* cd到gs-securing-web/initial
* 跳转到后面步骤`创建一个不安全的应用程序。`


### 2.4 Spring初始化项目
* [创建一个不安全的Web应用](https://github.com/fredomli/java-standard/blob/main/docs/spring/security/创建一个不安全的Web应用.md)  

### 2.5 Spring安全配置

> 假设您希望防止未经授权的用户在`/hello`处查看欢迎页面，现在，如果访问者点击主页上的链接，他们看到的问候没有障碍阻止他们。您需要添加一个障碍，迫使访问者登录之前，他们可以看到该页面。

可以通过在应用程序中配置Spring Security来实现。如果Spring Security在类路径上，Spring Boot会自动使用“基本”身份验证保护所有HTTP端点。但是，您可以进一步自定义安全设置。您需要做的第一件事是将Spring Security添加到类路径。

使用Maven，您需要向pomo .xml中的<dependencies>元素添加两个额外的条目(一个用于应用程序，一个用于测试)，如下所示:

```xml
<dependencies>
    <!-- Spring Security Jar   -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-security</artifactId>
    </dependency>
    <!--  Spring Security Test Jar  -->
    <dependency>
        <groupId>org.springframework.security</groupId>
        <artifactId>spring-security-test</artifactId>
        <scope>test</scope>
    </dependency>
</dependencies>
```

完整的Maven依赖文件参考：[完整的Maven依赖Pom.xml文件](https://github.com/fredomli/java-standard/blob/main/docs/spring/security/一份SpringSecurity依赖文件(Maven).md)

下面的安全配置(来自src/main/java/com/example/securingweb/WebSecurityConfig.java)确保只有经过认证的用户才能看到这个秘密的问候语:
```java
package com.example.securingweb;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.config.annotation.web.configuration.EnableWebSecurity;
import org.springframework.security.config.annotation.web.configuration.WebSecurityConfigurerAdapter;
import org.springframework.security.core.userdetails.User;
import org.springframework.security.core.userdetails.UserDetails;
import org.springframework.security.core.userdetails.UserDetailsService;
import org.springframework.security.provisioning.InMemoryUserDetailsManager;

@Configuration
@EnableWebSecurity
public class WebSecurityConfig extends WebSecurityConfigurerAdapter {
	@Override
	protected void configure(HttpSecurity http) throws Exception {
		http
			.authorizeRequests()
				.antMatchers("/", "/home").permitAll()
				.anyRequest().authenticated()
				.and()
			.formLogin()
				.loginPage("/login")
				.permitAll()
				.and()
			.logout()
				.permitAll();
	}

	@Bean
	@Override
	public UserDetailsService userDetailsService() {
		UserDetails user =
			 User.withDefaultPasswordEncoder()
				.username("user")
				.password("password")
				.roles("USER")
				.build();

		return new InMemoryUserDetailsManager(user);
	}
}
```
* WebSecurityConfig类带有@EnableWebSecurity注释，以启用Spring Security的web安全支持，并提供Spring MVC集成。它还扩展了WebSecurityConfigurerAdapter，并覆盖了它的一些方法来设置web安全配置的一些细节。
* configure(HttpSecurity)方法定义了应该保护哪些URL路径，哪些不应该。具体来说，/和/home路径被配置为不需要任何身份验证。所有其他路径都必须进行身份验证。
* 当用户成功登录时，他们被重定向到先前请求的需要身份验证的页面。有一个自定义/登录页面(由loginPage()指定)，每个人都可以查看它。
* userDetailsService()方法使用单个用户设置内存中的用户存储。该用户的用户名为user，密码为password，角色为user。
* 现在您需要创建登录页面。已经有了登录视图的视图控制器，所以你只需要创建登录视图本身，如下所示:

```html
<!DOCTYPE html>
<html xmlns="http://www.w3.org/1999/xhtml" xmlns:th="https://www.thymeleaf.org"
      xmlns:sec="https://www.thymeleaf.org/thymeleaf-extras-springsecurity3">
    <head>
        <title>Spring Security Example </title>
    </head>
    <body>
        <div th:if="${param.error}">
            Invalid username and password.
        </div>
        <div th:if="${param.logout}">
            You have been logged out.
        </div>
        <form th:action="@{/login}" method="post">
            <div><label> User Name : <input type="text" name="username"/> </label></div>
            <div><label> Password: <input type="password" name="password"/> </label></div>
            <div><input type="submit" value="Sign In"/></div>
        </form>
    </body>
</html>
```
这个Thymeleaf模板提供了一个表单，该表单捕获用户名和密码并将它们发布到/login。正如所配置的，Spring Security提供了一个过滤器来拦截该请求并对用户进行身份验证。如果用户身份验证失败，页面会重定向到/login?错误，页面将显示适当的错误消息。成功签出后，您的申请被发送到/login?注销，页面将显示适当的成功消息。

最后，您需要为访问者提供一种显示当前用户名和退出的方式。为此，更新hello.html向当前用户问好，并包含一个Sign Out表单，如下所示(来自src/main/resources/templates/hello.html):

```html
<!DOCTYPE html>
<html xmlns="http://www.w3.org/1999/xhtml" xmlns:th="https://www.thymeleaf.org"
      xmlns:sec="https://www.thymeleaf.org/thymeleaf-extras-springsecurity3">
    <head>
        <title>Hello World!</title>
    </head>
    <body>
        <h1 th:inline="text">Hello [[${#httpServletRequest.remoteUser}]]!</h1>
        <form th:action="@{/logout}" method="post">
            <input type="submit" value="Sign Out"/>
        </form>
    </body>
</html>
```

我们通过使用Spring安全与HttpServletRequest#getRemoteUser()的集成来显示用户名。“注销”表单提交一个POST到/logout。成功登出后，它将用户重定向到/login?logout。

### 运行程序
Spring Initializr为您创建一个应用程序类。在这种情况下，您不需要修改类。下面的清单(来自`src/main/java/com/example/securingweb/SecuringWebApplication.java`)显示了这个应用程序类:

```java
package com.example.securingweb;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class SecuringWebApplication {

	public static void main(String[] args) throws Throwable {
		SpringApplication.run(SecuringWebApplication.class, args);
	}

}
```

### 构建可执行文件
您可以使用Gradle或Maven从命令行运行应用程序。您还可以构建一个包含所有必需的依赖项、类和资源的可执行JAR文件并运行它。构建可执行jar使得在整个开发生命周期、跨不同环境等过程中，将服务作为应用程序来发布、版本化和部署变得很容易。

如果你使用Gradle，你可以使用。`/gradlew bootRun`来运行应用程序。或者，你可以使用。`/gradlew build`构建JAR文件，然后运行JAR文件，如下所示:

```shell
java -jar build/libs/gs-securing-web-0.1.0.jar
```

如果你使用Maven，你可以使用。`/mvnw spring-boot:run`来运行应用程序。或者，您可以使用。`/mvnw clean package` 构建JAR文件，然后运行JAR文件，如下所示:
```shell
java -jar target/gs-securing-web-0.1.0.jar
```

> 这里描述的步骤创建一个可运行的JAR。您还可以构建一个经典的WAR文件。


应用程序启动后，将浏览器指向http://localhost:8080。你应该看到主页。
恭喜你!您已经开发了一个使用Spring Security保护的简单web应用程序。

## 3. 主题
> 可在一个小时或更少的时间内阅读和理解，提供比入门指南更广泛或更主观的内容。  

具体参考官方文档 [Spring Security 构建风格](https://spring.io/guides/topicals/spring-security-architecture)
## 4. 文档
每一个Spring项目都有对应的文档，它非常详细地解释了如何使用项目特性以及使用它们可以实现什么。  

![spring-security-docs](https://gitee.com/fredomli/fredomli-picture/raw/picgo/static/images/wordpress/spring-security-document.png)
### 快速开始一个项目
> Quickstart Your Project (开始你的项目，使用 Spring Initializr)  
> Bootstrap your application with [Spring Initializr](https://start.spring.io/) .


> 参阅
> * [构建一个SpringBoot应用程序](https://github.com/fredomli/java-standard/blob/main/docs/spring/springboot/构建一个springboot应用程序.md)
> * [使用Spring MVC服务Web内容](https://spring.io/guides/gs/serving-web-content/)
> * [Spring安全架构(参考指南)](https://spring.io/guides/topicals/spring-security-architecture/)
> * [设置Spring安全 (SpringMVC整合Spring Security)](https://github.com/fredomli/java-standard/blob/main/docs/spring/security/创建一个不安全的Web应用.md)
> * [源码](https://github.com/fredomli/java-standard/blob/main/java-spring-example/gs-securing-web/)

*(☞ﾟヮﾟ)☞[返回首页README.md address](https://github.com/fredomli/java-standard)*
