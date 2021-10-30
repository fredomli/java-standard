# Maven Wrapper

## 介绍

Maven Wrapper是一种简单的方法，可以确保Maven构建的用户拥有运行Maven构建所需的一切。

为什么这是必要的?到目前为止，Maven对用户来说非常稳定，在大多数系统上都可以使用，或者很容易获得:但是随着Maven最近的许多更改，用户将更容易获得由项目提供的完全封装的构建设置。使用Maven包装器可以很容易做到这一点，这是从Gradle借鉴来的一个很棒的想法。

为您的项目设置Maven包装器的最简单方法是使用Takari Maven Plugin及其提供的包装器目标。要在项目中添加或更新所有必需的Maven Wrapper文件，请执行以下命令:

```shell
mvn -N io.takari:maven:0.7.7:wrapper
```
> 注意:默认用法是`mvn -N io.takari:maven:wrapper`但对于一些用户来说，这似乎会导致使用旧版本的包装器，从而安装更旧的maven默认值，等等。

通常情况下，你会指导用户安装一个特定版本的Apache Maven，把它放在PATH上，然后运行mvn命令，如下所示:

```shell
mvn clean install
```

但是现在，有了Maven包装器设置，你可以指导用户运行包装器脚本:

```shell
./mvnw clean install
```

or on Windows

```shell
./mvnw clean install
```
如果用户没有`.mvn/wrapper/maven-wrapper.properties`中指定的必要的Maven版本，那么一个普通的Maven构建将会执行一个重要的更改。它将首先为用户下载，安装，然后使用。

## 没有二进制JAR的使用
默认情况下，Maven Wrapper JAR归档文件作为小二进制文件`.mvn/wrapper/maven-wrapper.jar`添加到using项目中。它用于引导从包装器shell脚本下载和调用Maven。

如果您的项目不允许包含这样的二进制文件，您可以配置您的版本控制系统，以排除包装器jar的签入/提交。

如果脚本没有找到可用的JAR，它们将尝试从`.mvn/wrapper/maven-wrapper.properties`中指定的URL下载文件。并将其放在合适的位置。下载尝试通过curl、wget以及编译。`./mvn/wrapper/MavenWrapperDownloader.java`文件并执行生成的类。

如果Maven存储库受密码保护，可以通过环境变量MVNW_USERNAME指定用户名，通过环境变量MVNW_PASSWORD指定密码。

## 使用不同版本的Maven
要切换用于构建项目的Maven版本，你可以使用以下命令初始化它:
```shell
mvn -N io.takari:maven:0.7.7:wrapper -Dmaven=3.5.4
```
它适用于任何版本，除了快照。一旦你有了包装器，你可以通过在.mvn/wrapper/maven-wrapper.properties中设置distributionUrl来更改它的版本。例如。

```properties
distributionUrl=https://repo.maven.apache.org/maven2/org/apache/maven/apache-maven/3.5.4/apache-maven-3.5.4-bin.zip
```

## 在分发下载中使用基本身份验证
要从一个需要基本身份验证的位置下载Maven，你有两个选项:
* 设置环境变量MVNW_USERNAME和MVNW_PASSWORD
* 向distributionUrl添加用户和密码:
  `distributionUrl=https://username:password@<yourserver>/maven2/org/apache/maven/apache-maven/3.2.1/apache-maven-3.2.1-bin.zip`

## 指定Maven分发基础路径
这是Maven本身的一个特性，包装器恰好考虑到了它。只需将MAVEN_USER_HOME设置为所需的路径，包装器就会使用它作为Maven发行版安装的基础。

## 要测试Maven包装器的使用情况:
* 确保您构建的Unix文件系统具有正确的可执行标志设置
* 将maven包装器构建为快照版本
* 更新版本在takari-maven-plugin
* 构建takari-maven-plugin
* 在测试项目中使用takari-maven-plugin版本
```shell
mvn -N -X io.takari:maven:0.7.7-SNAPSHOT:wrapper
```

## 更新Maven版本:
* 更新maven-wrapper/.mvn/wrapper/maven-wrapper.properties中的URL
* 更新MavenWrapperMain中的URL
* 更新了takari-maven-plugin WrapperMojo类中的DEFAULT_MAVEN_VER参数

> 使用maven命令`mvnw`如果提示没有找到文件错误，解决方法：按照项目地址手动创建目录，并将缺少的Maven版本压缩包和解压包一起放入创建的目录中。
> 
> 因为默认情况下你的网络是下载不到需要的Maven包的，除非不使用该命令，直接使用本地安装的maven。即`mvn`命令。


项目地址：[https://github.com/takari/maven-wrapper](https://github.com/takari/maven-wrapper)

*(☞ﾟヮﾟ)☞[返回首页README.md address](https://github.com/fredomli/java-standard)*
