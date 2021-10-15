# Spring 快速开始

## Spring快速入门

> 你会建立:  
> 您将构建一个经典的“Hello World!”任何浏览器都可以连接的终端。你甚至可以告诉它你的名字，它会以更友好的方式回应你。

## 前期准备

* 集成开发人员环境(IDE)  
* 流行的选择包括IntelliJ IDEA、Spring Tools、Visual Studio Code或Eclipse等等。  
* Java™开发工具包(JDK)  
* 我们推荐使用AdoptOpenJDK 8或11版本。  

## 启动一个新的Spring Boot项目
使用start.spring.io创建一个“web”项目。在“Dependencies”对话框中搜索并添加“web”依赖项，如图所示。点击“生成”按钮，下载压缩文件，并将其解压到计算机上的一个文件夹中。

![quick-img](https://gitee.com/fredomli/fredomli-picture/raw/picgo/static/images/wordpress/quick-img-1-12bfde9c5c280b1940d85dee3d81772d.png)

由start.spring.io创建的项目包含Spring Boot，这是一个框架，可以让Spring在应用程序中工作，但不需要太多代码或配置。Spring Boot是启动Spring项目的最快和最流行的方式。

## 添加你的代码

在IDE中打开项目，在src/main/java/com/example/demo文件夹中找到DemoApplication.java文件。现在，通过添加下面代码中所示的额外方法和注释来更改文件的内容。您可以复制并粘贴代码，或者直接键入代码。

```java
package com.example.demo;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestParam;
import org.springframework.web.bind.annotation.RestController;

@SpringBootApplication
@RestController
public class DemoApplication {

public static void main(String[] args) {
SpringApplication.run(DemoApplication.class, args);
}

@GetMapping("/hello")
public String hello(@RequestParam(value = "name", defaultValue = "World") String name) {
return String.format("Hello %s!", name);
}
}
```

我们添加的hello()方法被设计为接受一个名为name的String参数，然后将该参数与代码中的单词“hello”组合在一起。这意味着如果您在请求中设置您的名字为“Amy”，则响应将是“Hello Amy”。

@RestController注释告诉Spring，这段代码描述了一个应该在web上可用的端点。@GetMapping(“/hello”)告诉Spring使用我们的hello()方法来回答发送到http://localhost:8080/hello地址的请求。最后，@RequestParam告诉Spring在请求中期望一个名称值，但如果它不存在，它将默认使用单词“World”。

## 调试

让我们构建并运行该程序。打开命令行(或终端)，导航到项目文件所在的文件夹。我们可以通过发出以下命令来构建和运行应用程序:  

MacOS/Linux:

```xml
./mvnw spring-boot:run
```

Windows:

```xml
mvnw spring-boot:run
```
你应该会看到一些类似的输出:

![quick-img2](https://gitee.com/fredomli/fredomli-picture/raw/picgo/static/images/wordpress/quick-img2-ac5ae88c60ffaa062234a580f9f1abc3.png)
这里的最后几行告诉我们，春天已经开始了。Spring Boot的嵌入式Apache Tomcat服务器充当web服务器，并在本地主机端口8080上侦听请求。打开浏览器，在顶部的地址栏中输入http://localhost:8080/hello。你会得到这样友好的回应:

## 参考
* [Spring 官网](https://spring.io/)

*(☞ﾟヮﾟ)☞[返回首页README.md address](https://github.com/fredomli/java-standard)*

