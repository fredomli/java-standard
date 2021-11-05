# 构建RESTful Web服务（Building a RESTful Web Service）

本指南将带领您完成使用Spring创建“Hello, World”RESTful web服务的过程。
### 目标
您将在 `http://localhost:8080/greeting` 上构建一个接受HTTP GET请求的服务。 它将响应一个JSON表示的问候，如下所示:

```shell
{"id":1,"content":"Hello, World!"}
```
您可以在查询字符串中使用可选的`name`参数定制问候语，如下所示:
```shell
http://localhost:8080/greeting?name=User
```

`name`参数值覆盖World的默认值，并反映在响应中，如下所示:
```shell
{"id":1,"content":"Hello, User!"}
```

### 准备
* 一个集成开发环境（IDE）
* Java JDK环境（1.8）
* 项目管理工具（Maven 或 grable）
* 创建一个基本项目，比如 Spring Initializr 项目

### 创建项目
这里我们使用IDEA内置的Spring Initializr创建项目，创建项目依赖于网站`https://start.spring.io`，创建过程中我们添加`Spring Web`依赖。一般情况下对于应用的存档文件，及一些配置依赖的资源。需要添加进项目中。

> 如果您的IDE具有Spring Initializr集成，您可以从您的IDE完成这个过程。
> 
> 您还可以从Github中派生项目，并在IDE或其他编辑器中打开它。

### 创建资源表示类

既然已经设置好了项目和构建系统，就可以创建web服务了。

通过考虑服务交互来开始流程。

该服务将处理`/greeting`的`GET`请求，可以在查询字符串中使用`name`参数。`GET`请求应该返回一个`200 OK`的响应，其中包含表示欢迎的JSON。它应该类似于下面的输出:

```json
{
    "id": 1,
    "content": "Hello, World!"
}
```
`id`字段是欢迎语的唯一标识符，`content`是欢迎语的文本表示形式。

要对欢迎表示建模，请创建一个资源表示类。为此，提供一个带有字段、构造函数和id和内容数据访问器的普通Java对象，如下所示(来自`src/main/java/com/example/restservice/welcome.java`):

```java
package com.example.gsrestservice.restservice;

public class Greeting {
    private final long id;
    private final String content;

    public Greeting(long id, String content) {
        this.id = id;
        this.content = content;
    }

    public long getId() {
        return id;
    }

    public String getContent() {
        return content;
    }
}
```
该应用程序使用`Jackson JSON`库自动将类型`Greeting`的实例编组为JSON。`Jackson` 是包含在 `web starter`中。

![pc1](https://gitee.com/fredomli/fredomli-picture/raw/picgo/static/images/wordpress/get-rest-service-pc1.png)


### 创建控制器
在`Spring`构建`RESTful web`服务的方法中，HTTP请求由控制器处理。这些组件由`@RestController`注释标识，下面的清单(来自`src/main/java/com/example/restservice/GreetingController.java`)中显示的`GreetingController`通过返回`greeting`类的一个新实例来处理`/greeting`的`GET`请求:

```java
package com.example.restservice;

import java.util.concurrent.atomic.AtomicLong;

import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestParam;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class GreetingController {

	private static final String template = "Hello, %s!";
	private final AtomicLong counter = new AtomicLong();

	@GetMapping("/greeting")
	public Greeting greeting(@RequestParam(value = "name", defaultValue = "World") String name) {
		return new Greeting(counter.incrementAndGet(), String.format(template, name));
	}
}
```
这个控制器简洁而简单，但在引擎盖下还有很多东西。我们一步一步地分解它。

`@GetMapping`注释确保对`/greeting`的HTTP GET请求被映射到`greeting()`方法。

> 其他HTTP动词也有相应的注解(例如POST的`@PostMapping`)。还有一个`@RequestMapping`注释，它们都是从这个注释派生出来的，并且可以作为同义词(例如`@RequestMapping(method=GET)`)。

`@RequestParam`将查询字符串参数名称的值绑定到`greeting()`方法的`name`参数中。如果请求中没有`name`参数，则使用World的`defaultValue`。

方法体的实现根据计数器的下一个值创建并返回一个新的Greeting对象，该对象具有id和内容属性，并使用Greeting模板格式化给定的名称。

传统MVC控制器和前面介绍的`RESTful web`服务控制器之间的一个关键区别是创建HTTP响应体的方式。这个`RESTful web`服务控制器填充并返回一个`greeting`对象，而不是依赖视图技术将欢迎数据执行到HTML的服务器端呈现。对象数据将以`JSON`的形式直接写入HTTP响应。

这段代码使用了Spring `@RestController`注释，它将类标记为一个控制器，其中每个方法返回一个域对象而不是视图。它是同时包含`@Controller`和`@ResponseBody`的简写。

必须将`Greeting`对象转换为`JSON`。由于`Spring`的`HTTP`消息转换器支持，您不需要手动进行这种转换。因为Jackson2在类路径上，所以会自动选择Spring的`MappingJackson2HttpMessageConverter`来将`Greeting`实例转换为JSON。

`@SpringBootApplication`是一个方便的注释，它添加了以下所有内容:

* `@Configuration`:将类标记为应用程序上下文的bean定义的源。
* `@EnableAutoConfiguration`:告诉Spring Boot开始添加基于类路径设置、其他bean和各种属性设置的bean。例如，如果spring-webmvc在类路径上，这个注释将把应用程序标记为web应用程序，并激活关键行为，例如设置DispatcherServlet。
* `@ComponentScan`:告诉Spring查找其他组件、配置和服务

`main()`方法使用`Spring Boot`的`SpringApplication.run()`方法来启动应用程序。您注意到没有任何一行XML吗?也没有web.xml文件。这个web应用程序是100%纯Java的，你不需要配置任何管道或基础设施。

### 构建可执行JAR

您可以使用Gradle或Maven从命令行运行应用程序。您还可以构建一个包含所有必需的依赖项、类和资源的可执行JAR文件并运行它。构建可执行jar使得在整个开发生命周期、跨不同环境等过程中，将服务作为应用程序来发布、版本化和部署变得很容易。

如果你使用Gradle，你可以使用`./gradlew bootRun`来运行应用程序。或者，你可以使用`./gradlew build`构建JAR文件，然后运行JAR文件，如下所示:
```shell
java -jar build/libs/gs-rest-service-0.0.1-SNAPSHOT.jar
```
如果你使用Maven，你可以使用`./mvnw spring-boot:run`来运行应用程序。或者，您可以使用`./mvnw`干净包构建JAR文件，然后运行JAR文件，如下所示:
```shell
java -jar target/gs-rest-service-0.0.1-SNAPSHOT.jar
```

这里描述的步骤创建一个可运行的JAR。您还可以构建一个经典的WAR文件。

### 测试服务

现在服务已经开通，访问`http://localhost:8080/greeting` ，在那里你会看到:
```json
{"id":1,"content":"Hello, World!"}
```
通过访问`http://localhost:8080/greeting?name=User` 提供名称查询字符串参数。注意内容属性的值如何从`Hello, World!` 到 `Hello, User!`，如下所示:

```js
{"id":2,"content":"Hello, User!"}
```

## 参考
* [Spring 官网](https://spring.io/)
* [https://start.spring.io](https://start.spring.io)

*(☞ﾟヮﾟ)☞[返回首页README.md address](https://github.com/fredomli/java-standard)*
