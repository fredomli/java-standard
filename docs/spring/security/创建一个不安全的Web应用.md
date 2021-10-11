# 创建一个不安全的Web应用

> 在将安全性应用到web应用程序之前，您需要确保web应用程序的安全性。本节将引导您创建一个简单的web应用程序。然后在下一节中使用Spring Security保护它。

web应用程序包括两个简单的视图:主页和“Hello, World”页面。主页在以下Thymeleaf模板中定义(来自src/main/resources/templates/home.html):


```html
<!DOCTYPE html>
<html xmlns="http://www.w3.org/1999/xhtml" xmlns:th="https://www.thymeleaf.org" xmlns:sec="https://www.thymeleaf.org/thymeleaf-extras-springsecurity3">
    <head>
        <title>Spring Security Example</title>
    </head>
    <body>
        <h1>Welcome!</h1>
        
        <p>Click <a th:href="@{/hello}">here</a> to see a greeting.</p>
    </body>
</html>
```
>这个简单的视图包含一个到/hello页面的链接，它在以下Thymeleaf模板中定义(来自src/main/resources/templates/hello.html):

```html
<!DOCTYPE html>
<html xmlns="http://www.w3.org/1999/xhtml" xmlns:th="https://www.thymeleaf.org"
      xmlns:sec="https://www.thymeleaf.org/thymeleaf-extras-springsecurity3">
    <head>
        <title>Hello World!</title>
    </head>
    <body>
        <h1>Hello world!</h1>
    </body>
</html>
```
> web应用程序是基于Spring MVC的。因此，您需要配置Spring MVC并设置视图控制器来公开这些模板。下面的清单(来自src/main/java/com/example/securingweb/MvcConfig.java)显示了一个在应用程序中配置Spring MVC的类:

```java
package com.example.securingweb;
import org.springframework.context.annotation.Configuration;
import org.springframework.web.servlet.config.annotation.ViewControllerRegistry;
import org.springframework.web.servlet.config.annotation.WebMvcConfigurer;

@Configuration
public class MvcConfig implements WebMvcConfigurer {
	public void addViewControllers(ViewControllerRegistry registry) {
		registry.addViewController("/home").setViewName("home");
		registry.addViewController("/").setViewName("home");
		registry.addViewController("/hello").setViewName("hello");
		registry.addViewController("/login").setViewName("login");
	}
}
```

`addViewControllers()`方法(它覆盖了`WebMvcConfigurer`中同名的方法)添加了四个视图控制器。其中两个视图控制器引用名为`home`的视图(在`home.html`中定义)，另一个视图引用名为`hello`的视图(在`hello.html`中定义)。第四个视图控制器引用另一个名为`login`的视图。您将在下一节中创建该视图。
此时，您可以跳转到“运行应用程序”，无需登录任何东西就可以运行应用程序。
现在你有了一个不安全的web应用程序，你可以给它添加安全性。

> 扩展  
> [设置Spring安全 (SpringMVC整合Spring Security)](https://github.com/fredomli/java-standard/blob/main/docs/spring/security/Spring安全配置.md)  

*(☞ﾟヮﾟ)☞[返回首页README.md address](https://github.com/fredomli/java-standard)*
