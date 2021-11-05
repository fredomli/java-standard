# 用Maven构建Java项目

本指南指导您如何使用Maven构建一个简单的Java项目。

### 目标  
您将创建一个提供时间的应用程序，然后使用Maven构建它。

### 准备
* 一个你喜欢的编辑器
* JKD8 或者 以上

### 创建一个项目
创建一个Spring Boot项目(其他项目也是可以的，只要你能够学习到相关技术)，可以使用 `Spring Initializr`，`IDEA`，`Spring STS`等来创建你的项目。这里我们手动创建我们的项目，按照下面的目录结构。

源码：```git clone https://github.com/spring-guides/gs-maven.git```

### 建立项目

首先，您需要设置Maven要构建的Java项目。为了将重点放在Maven上，现在让项目尽可能简单。在您选择的项目文件夹中创建此结构。

#### 创建目录结构
在你选择的项目目录中，创建以下子目录结构;例如，在*nix系统上使用`mkdir -p src/main/java/hello`:

```text
└── src
    └── main
        └── java
            └── hello
```

在src/main/java/hello目录中，你可以创建任何你想要的java类。为了与本指南的其余部分保持一致，创建这两个类:`HelloWorld.java`和`Greter.java`。

```src/main/java/hello/HelloWorld.java```

```java
package hello;

public class HelloWorld {
  public static void main(String[] args) {
    Greeter greeter = new Greeter();
    System.out.println(greeter.sayHello());
  }
}
```

```src/main/java/hello/Greeter.java```
```java
package hello;

public class Greeter {
  public String sayHello() {
    return "Hello world!";
  }
}
```

现在您已经有了一个可以用Maven构建的项目，下一步是安装Maven。

Maven可以在https://maven.apache.org/download.cgi上下载zip文件。只需要二进制文件，所以寻找到apache-maven-{version}-bin.zip或apache-maven-{version}-bin.tar.gz的链接。

下载zip文件后，将其解压缩到计算机中。然后将bin文件夹添加到路径中。

要测试Maven安装，请从命令行运行mvn:
```shell
mvn -v
```

如果一切顺利，您将看到有关Maven安装的一些信息。它看起来类似于(尽管可能略有不同)以下:

```shell
Apache Maven 3.6.3 (cecedd343002696d0abdw42k2b541b5yuba2883f)
Maven home: D:\maven\apache-maven-3.6.3\bin\..
Java version: 1.8.0_291, vendor: Oracle Corporation, runtime: E:\Development\JavaEnvironment\jdk1.8.0_291\jre
Default locale: zh_CN, platform encoding: GBK
OS name: "windows 10", version: "10.0", arch: "amd64", family: "windows"
```

恭喜你!现在您已经安装了Maven。

你可能会考虑使用Maven包装器来隔离你的开发人员使用正确的Maven版本，或者不得不安装它。从Spring Initializr下载的项目包含包装器。它显示为一个脚本`mvnw`在你的项目的顶层，你运行的地方`mvn`。

### 定义一个简单的Maven构建
现在Maven已经安装好了，您需要创建一个Maven项目定义。Maven项目是用一个名为`pom.XML`的XML文件定义的。除此之外，该文件提供了项目的名称、版本和它在外部库上的依赖项。

在项目的根目录下创建一个名为pom.xml的文件(也就是说，把它放在src文件夹的旁边)，并赋予它以下内容:

```pom.xml```
```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>org.springframework</groupId>
    <artifactId>gs-maven</artifactId>
    <packaging>jar</packaging>
    <version>0.1.0</version>

    <properties>
        <maven.compiler.source>1.8</maven.compiler.source>
        <maven.compiler.target>1.8</maven.compiler.target>
    </properties>

    <build>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-shade-plugin</artifactId>
                <version>3.2.4</version>
                <executions>
                    <execution>
                        <phase>package</phase>
                        <goals>
                            <goal>shade</goal>
                        </goals>
                        <configuration>
                            <transformers>
                                <transformer
                                    implementation="org.apache.maven.plugins.shade.resource.ManifestResourceTransformer">
                                    <mainClass>hello.HelloWorld</mainClass>
                                </transformer>
                            </transformers>
                        </configuration>
                    </execution>
                </executions>
            </plugin>
        </plugins>
    </build>
</project>
```
除了可选的`<packaging>`元素外，这可能是构建Java项目所需的最简单的`pom.xml`文件。它包括以下详细的项目配置:


### 构建Java代码
Maven现在可以构建项目了。现在，您可以使用Maven执行几个构建生命周期目标，包括编译项目代码、创建库包(比如JAR文件)以及将库安装到本地Maven依赖库中。

要尝试构建，请在命令行中发出以下命令:

```shell
mvn compile
```
这将运行Maven，告诉它执行编译目标。当它完成时，您应该在`target/classes`目录中找到已编译的`.class`文件。

因为你不太可能想要直接分发或使用.class文件，你可能想要运行包目标:

```shell
mvn package
```
包目标将编译您的Java代码，运行任何测试，并将代码打包到目标目录中的JAR文件中。JAR文件的名称将基于项目的`<artifactId>`和`<version>`。例如，给定以前的最小的`pom.xml`文件，JAR文件将被命名为`gs-maven-0.1.0.jar`。

要执行JAR文件，请运行:

```shell
java -jar target/gs-maven-0.1.0.jar
```

> 如果您将<packaging>的值从“jar”更改为“war”，那么结果将是目标目录中的war文件，而不是jar文件。

Maven还在本地机器上维护一个依赖项存储库(通常在主目录的.m2/repository目录中)，以便快速访问项目依赖项。如果您想将项目的JAR文件安装到本地存储库中，那么您应该调用安装目标:

```shell
mvn install
```
安装目标将编译、测试和打包项目代码，然后将其复制到本地依赖项存储库中，以便另一个项目将其作为依赖项引用。

说到依赖关系，现在是时候在Maven构建中声明依赖关系了。

### 声明依赖性
简单的Hello World示例是完全自包含的，不依赖于任何额外的库。然而，大多数应用程序依赖于外部库来处理常见和复杂的功能。

例如，假设除了说“Hello World!”之外，您还希望应用程序打印当前日期和时间。虽然可以使用本机Java库中的日期和时间工具，但可以通过使用Joda time库使事情变得更有趣。

首先，将HelloWorld.java改为如下所示:


```src/main/java/hello/HelloWorld.java```

```java
package hello;

import org.joda.time.LocalTime;

public class HelloWorld {
  public static void main(String[] args) {
    LocalTime currentTime = new LocalTime();
    System.out.println("The current local time is: " + currentTime);
    Greeter greeter = new Greeter();
    System.out.println(greeter.sayHello());
  }
}
```
这里`HelloWorld`使用Joda Time的`LocalTime`类来获取和打印当前时间。

如果您现在要运行`mvn compile`来构建项目，那么构建将会失败，因为您没有在构建中声明Joda Time作为编译依赖项。你可以通过在`pom.xml`(在`<project>`元素中)中添加以下几行来解决这个问题:

```xml
<dependencies>
    <dependency>
        <groupId>joda-time</groupId>
        <artifactId>joda-time</artifactId>
        <version>2.9.2</version>
    </dependency>
</dependencies>
```

这个XML块声明了项目的依赖项列表。具体来说，它为Joda Time库声明了一个依赖项。在`<dependency>`元素中，依赖坐标由三个子元素定义:

* `<groupId>` -依赖项所属的组或组织。
* `<artifactId>` -所需的库。
* `<version>` -所需的库的特定版本。

默认情况下，所有依赖项的作用域都是编译依赖项。也就是说，它们应该在编译时可用(如果您正在构建WAR文件，包括在`WAR的/WEB-INF/libs`文件夹中)。此外，您可以指定`<scope>`元素来指定以下作用域之一:

* `provided `-编译项目代码所需要的依赖项，但将由运行代码的容器在运行时提供(例如，Java Servlet API)。
* `test`—用于编译和运行测试，但不是生成或运行项目运行时代码所必需的依赖项。

现在，如果您运行mvn compile或mvn包，Maven应该会从Maven中央存储库解析Joda Time依赖关系，构建将会成功。
  
### 编写一个测试
首先在测试范围内将JUnit作为依赖项添加到pom.xml中:
```xml
<dependency>
	<groupId>junit</groupId>
	<artifactId>junit</artifactId>
	<version>4.12</version>
	<scope>test</scope>
</dependency>
```

然后像这样创建一个测试用例:
```src/test/java/hello/GreeterTest.java```
```java
package hello;

import static org.hamcrest.CoreMatchers.containsString;
import static org.junit.Assert.*;

import org.junit.Test;

public class GreeterTest {
  
  private Greeter greeter = new Greeter();

  @Test
  public void greeterSaysHello() {
    assertThat(greeter.sayHello(), containsString("Hello"));
  }

}
```
Maven使用一个名为“surefire”的插件来运行单元测试。这个插件的默认配置编译和运行`src/test/java`中的所有类，并使用匹配`*test`的名称。您可以像这样在命令行上运行测试

```shell
mvn test
```
```pom.xml```
```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
	<modelVersion>4.0.0</modelVersion>
	
	<groupId>org.springframework</groupId>
	<artifactId>gs-maven</artifactId>
	<packaging>jar</packaging>
	<version>0.1.0</version>

	<properties>
		<maven.compiler.source>1.8</maven.compiler.source>
		<maven.compiler.target>1.8</maven.compiler.target>
	</properties>

	<dependencies>
		<!-- tag::joda[] -->
		<dependency>
			<groupId>joda-time</groupId>
			<artifactId>joda-time</artifactId>
			<version>2.9.2</version>
		</dependency>
		<!-- end::joda[] -->
		<!-- tag::junit[] -->
		<dependency>
			<groupId>junit</groupId>
			<artifactId>junit</artifactId>
			<version>4.12</version>
			<scope>test</scope>
		</dependency>
		<!-- end::junit[] -->
	</dependencies>

	<build>
		<plugins>
			<plugin>
				<groupId>org.apache.maven.plugins</groupId>
				<artifactId>maven-shade-plugin</artifactId>
				<version>3.2.4</version>
				<executions>
					<execution>
						<phase>package</phase>
						<goals>
							<goal>shade</goal>
						</goals>
						<configuration>
							<transformers>
								<transformer implementation="org.apache.maven.plugins.shade.resource.ManifestResourceTransformer">
									<mainClass>hello.HelloWorld</mainClass>
								</transformer>
							</transformers>
						</configuration>
					</execution>
				</executions>
			</plugin>
		</plugins>
	</build>

</project>
```
完整的`pom.xml`文件使用`Maven Shade Plugin`，以方便使JAR文件可执行。本指南的重点是开始使用Maven，而不是使用这个特定的插件。

## 参考
* [Spring 官网](https://spring.io/)
* [https://start.spring.io](https://start.spring.io)
* [Maven 官网](https://maven.apache.org/)

*(☞ﾟヮﾟ)☞[返回首页README.md address](https://github.com/fredomli/java-standard)*
