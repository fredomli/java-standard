# 使用RESTful Web服务

本指南将带领您完成创建使用RESTful web服务的应用程序的过程。

### 目标
您将构建一个应用程序，该应用程序使用Spring的`RestTemplate`在`https://quoters.apps.pcfone.io/api/random` 上检索一个随机的`Spring Boot`报价。

### 准备
* 一个集成开发环境（IDE）
* Java JDK环境（1.8）
* 项目管理工具（Maven 或 grable）
* 创建一个基本项目，比如 Spring Initializr 项目

### 创建一个项目
创建一个Spring Boot项目(其他项目也是可以的，只要你能够学习到相关技术)，可以使用 `Spring Initializr`，`IDEA`，`Spring STS`等来创建你的项目。

源码：```git clone https://github.com/spring-guides/gs-consuming-rest.git```

### 获取REST资源

项目设置完成后，您可以创建一个使用RESTful服务的简单应用程序。

一个RESTful服务已经在`https://quoters.apps.pcfone.io/api/random` 上上线了。它随机获取关于Spring Boot的报价，并将它们作为JSON文档返回。

如果你通过web浏览器或curl请求这个URL，你会收到一个JSON文档，看起来像这样:

```json
{
   type: "success",
   value: {
      id: 10,
      quote: "Really loving Spring Boot, makes stand alone Spring apps easy."
   }
}
```
这很简单，但在通过浏览器或curl获取时用处不大。

使用REST web服务的一种更有用的方法是编程。为了帮助您完成这项任务，Spring提供了一个方便的模板类，名为`RestTemplate`。`RestTemplate`使与大多数RESTful服务的交互成为一行咒语。它甚至可以将数据绑定到自定义域类型。

首先，您需要创建一个域类来包含您需要的数据。下面的清单显示了`Quote`类，您可以使用它作为域类:

```src/main/java/com/example/consumingrest/Quote.java```

```java
package com.example.consumingrest;

import com.fasterxml.jackson.annotation.JsonIgnoreProperties;

@JsonIgnoreProperties(ignoreUnknown = true)
public class Quote {

  private String type;
  private Value value;

  public Quote() {
  }

  public String getType() {
    return type;
  }

  public void setType(String type) {
    this.type = type;
  }

  public Value getValue() {
    return value;
  }

  public void setValue(Value value) {
    this.value = value;
  }

  @Override
  public String toString() {
    return "Quote{" +
        "type='" + type + '\'' +
        ", value=" + value +
        '}';
  }
}
```
这个简单的Java类有一些属性和匹配的getter方法。它使用来自Jackson JSON处理库的`@JsonIgnoreProperties`进行注释，以指示应该忽略任何未绑定在此类型中的属性。

要将数据直接绑定到自定义类型，需要指定变量名与从API返回的JSON文档中的键完全相同。如果JSON文档中的变量名和键不匹配，可以使用@JsonProperty注释指定JSON文档的确切键。(本例将每个变量名匹配到一个JSON键，所以这里不需要注释。)

您还需要一个额外的类来嵌入内部引号本身。Value类填充了需要的内容，如下所示(`在src/main/java/com/example/consumingrest/Value.java`):

```java
package com.example.consumingrest;

import com.fasterxml.jackson.annotation.JsonIgnoreProperties;

@JsonIgnoreProperties(ignoreUnknown = true)
public class Value {

  private Long id;
  private String quote;

  public Value() {
  }

  public Long getId() {
    return this.id;
  }

  public String getQuote() {
    return this.quote;
  }

  public void setId(Long id) {
    this.id = id;
  }

  public void setQuote(String quote) {
    this.quote = quote;
  }

  @Override
  public String toString() {
    return "Value{" +
        "id=" + id +
        ", quote='" + quote + '\'' +
        '}';
  }
}
```

它使用相同的注释，但映射到其他数据字段。

### 完成应用程序
`initizr`创建一个带有`main()`方法的类。下面的清单显示了`Initializr`创建的类(在`src/main/java/com/example/consumingrest/ConsumingRestApplication.java`):

```java
package com.example.consumingrest;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class ConsumingRestApplication {

	public static void main(String[] args) {
		SpringApplication.run(ConsumingRestApplication.class, args);
	}

}
```

现在需要向`ConsumingRestApplication`类添加其他一些东西，以使它显示来自我们的`RESTful`源的引用。您需要添加:
* 一个记录器，用于将输出发送到日志(在本例中是控制台)。
* `RestTemplate`，它使用Jackson JSON处理库来处理传入的数据。
* `CommandLineRunner`在启动时运行RestTemplate(因此，获取我们的报价)。

下面的清单显示了完成的`ConsumingRestApplication`类(位于`src/main/java/com/example/consumingrest/ConsumingRestApplication.java`):

```java
package com.example.consumingrest;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.boot.CommandLineRunner;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.boot.web.client.RestTemplateBuilder;
import org.springframework.context.annotation.Bean;
import org.springframework.web.client.RestTemplate;

@SpringBootApplication
public class ConsumingRestApplication {

	private static final Logger log = LoggerFactory.getLogger(ConsumingRestApplication.class);

	public static void main(String[] args) {
		SpringApplication.run(ConsumingRestApplication.class, args);
	}

	@Bean
	public RestTemplate restTemplate(RestTemplateBuilder builder) {
		return builder.build();
	}

	@Bean
	public CommandLineRunner run(RestTemplate restTemplate) throws Exception {
		return args -> {
			Quote quote = restTemplate.getForObject(
					"https://quoters.apps.pcfone.io/api/random", Quote.class);
			log.info(quote.toString());
		};
	}
}
```
### 运行应用程序

您可以使用Gradle或Maven从命令行运行应用程序。您还可以构建一个包含所有必需的依赖项、类和资源的可执行JAR文件并运行它。构建可执行jar使得在整个开发生命周期、跨不同环境等过程中，将服务作为应用程序来发布、版本化和部署变得很容易。

如果你使用Gradle，你可以使用`./gradlew bootRun`来运行应用程序。或者，你可以使用`./gradlew build`构建JAR文件，然后运行JAR文件，如下所示:

```shell
java -jar build/libs/gs-consuming-rest-0.1.0.jar
```
如果你使用Maven，你可以使用`./mvnw spring-boot:run`来运行应用程序。或者，您可以使用`./mvnw clean package`包构建JAR文件，然后运行JAR文件，如下所示:

```shell
java -jar target/gs-consuming-rest-0.1.0.jar
```

> `./`不是必须的，参照自己的系统来，如果你是在windows系统上直接运行 `mvnw spring-boot:run` 或 `gradlew bootRun`，配置文件必须存在当前项目中，下载的资源包中已经存在，除此之外还可以使用本地的包管理。

> 这里描述的步骤创建一个可运行的JAR。您还可以构建一个经典的WAR文件。

```如果你看到一个错误，Could not extract response: no suitable HttpMessageConverter found for response type [class com.example.consumingrest.Quote],您可能处于无法连接到后端服务的环境中(如果您能够到达它，它将发送JSON)。也许你是公司的代理人。尝试设置http。proxyHost和http。proxyPort系统属性为适合您的环境的值。```

效果如下：
```text
E:\coding\IntellijWorkSpace\IdeaWorkSpack\spring-example\gs-consuming-rest)
2021-11-05 18:01:07.420  INFO 8740 --- [           main] c.e.c.GsConsumingRestApplication         : No active profile set, falling back to default profiles: default
2021-11-05 18:01:08.482  INFO 8740 --- [           main] o.s.b.w.embedded.tomcat.TomcatWebServer  : Tomcat initialized with port(s): 8080 (http)
2021-11-05 18:01:08.491  INFO 8740 --- [           main] o.apache.catalina.core.StandardService   : Starting service [Tomcat]
2021-11-05 18:01:08.491  INFO 8740 --- [           main] org.apache.catalina.core.StandardEngine  : Starting Servlet engine: [Apache Tomcat/9.0.54]
2021-11-05 18:01:08.574  INFO 8740 --- [           main] o.a.c.c.C.[Tomcat].[localhost].[/]       : Initializing Spring embedded WebApplicationContext
2021-11-05 18:01:08.574  INFO 8740 --- [           main] w.s.c.ServletWebServerApplicationContext : Root WebApplicationContext: initialization completed in 1081 ms
2021-11-05 18:01:08.727  INFO 8740 --- [           main] c.e.c.GsConsumingRestApplication         : bean is successful run.
2021-11-05 18:01:08.968  INFO 8740 --- [           main] o.s.b.w.embedded.tomcat.TomcatWebServer  : Tomcat started on port(s): 8080 (http) with context path ''
2021-11-05 18:01:08.978  INFO 8740 --- [           main] c.e.c.GsConsumingRestApplication         : Started GsConsumingRestApplication in 2.137 seconds (JVM running for 3.517)
2021-11-05 18:01:10.752  INFO 8740 --- [           main] c.e.c.GsConsumingRestApplication         : Quote{type='success', value=Value{id=1, quote='Working with Spring Boot is like pair-programming with the Spring developers.'}}
```
## 参考
* [Spring 官网](https://spring.io/)
* [https://start.spring.io](https://start.spring.io)

*(☞ﾟヮﾟ)☞[返回首页README.md address](https://github.com/fredomli/java-standard)*
