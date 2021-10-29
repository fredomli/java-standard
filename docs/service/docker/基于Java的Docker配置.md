# 基于win10+Docker+idea的SpringBoot项目容器化部署

## 工具 & 环境
* maven
* jdk
* idea
* docker
* win10
* Git

## win10上安装Docker
在[Docker官网地址](https://www.docker.com/) 下载对应的Window版本Docker安装包安装即可。
在不同系统上使用不同的安装方式，在window 或者 mac 上直接下载可执行安装包运行安装，各种配置都已经默认配置完成，使用Linux系统的就需要手动配置一些参数，包括远程访问，buildkit等等。


在任务管理器中，允许虚拟化操作：

![pc1](https://gitee.com/fredomli/fredomli-picture/raw/picgo/static/images/wordpress/window10-install-docker.png)

docker 安装完默认的docker hub 网址是 hub.docker.com，在官网去注册，docker hub是保存镜像的地方，后面会用到。

## 创建一个Springboot项目并且整合docker
参考 [使用SpringInitializr创建项目](https://github.com/fredomli/java-standard/blob/main/docs/spring/spring/使用SpringInitializr创建项目.md)

## 先决条件
### 使BuildKit
在开始构建映像之前，确保您已经在机器上启用了BuildKit。BuildKit允许你高效地构建Docker映像。有关更多信息，请参见使用BuildKit构建图像。

在Docker Desktop上，所有用户默认都启用了BuildKit。如果你已经安装了Docker Desktop，你就不需要手动启用BuildKit。如果你在Linux上运行Docker，你可以通过使用环境变量或将BuildKit设置为默认设置来启用BuildKit。

要在运行docker build命令时设置BuildKit环境变量，请运行:
```shell
DOCKER_BUILDKIT=1 docker build 
```

要默认启用docker BuildKit，请在/etc/docker/daemon.中设置daemon.Json特性设置为true并重启守护进程。如果守护进程。Json文件不存在，创建新的文件称为daemon.Json，然后将以下内容添加到文件中。

```json
{
  "features":{"buildkit" : true}
}
```
重新启动Docker守护进程。

## 为Java创建一个Dockerfile
Dockerfile是一个文本文档，包含组装Docker映像的指令。当我们告诉Docker通过执行Docker build命令来构建我们的映像时，Docker会读取这些指令并执行它们，最终创建一个Docker映像。

让我们来看看为应用程序创建Dockerfile的过程。在项目的根目录下，创建一个名为Dockerfile的文件，并在文本编辑器中打开该文件。

> 如何命名Dockerfile?  
> 
> Dockerfile使用的默认文件名是`Dockerfile`(没有文件扩展名)。使用默认名称允许您运行`docker build`命令，而不必指定额外的命令标志。
> 
> 有些项目可能需要不同的dockerfile用于特定目的。一个常见的约定是将这些文件命名为`Dockerfile.<something>`或`<something>.Dockerfile`。这样的Dockerfiles可以通过`docker build`命令的-file(或-f简写)选项使用。参考`docker build`中的“指定Dockerfile”一节来了解`-file`选项。
> 
> 我们建议对项目的主Dockerfile使用默认值(Dockerfile)

要添加到Dockerfile中的第一行是`#syntax`语法解析器指令。虽然这个指令是可选的，但它指示Docker构建器在解析Dockerfile时使用什么语法，并允许启用了BuildKit的旧Docker版本在开始构建之前升级解析器。解析器指令必须出现在Dockerfile中的任何其他注释、空格或Dockerfile指令之前，并且应该是Dockerfiles中的第一行。
```docker
# syntax=docker/dockerfile:1
```

我们建议使用`docker/dockerfile:1`，它总是指向版本1语法的最新版本。在构建之前，BuildKit会自动检查语法的更新，确保你使用的是最新的版本。

接下来，我们需要在Dockerfile中添加一行代码，告诉Docker我们想为我们的应用程序使用什么基础映像。

```docker
# syntax=docker/dockerfile:1

FROM openjdk:16-alpine3.13
```

Docker映像可以从其他映像继承。在本指南中，我们使用了来自Docker Hub的官方openjdk映像和Java JDK，其中已经包含了运行Java应用程序所需的所有工具和包。

为了在运行其余命令时更容易一些，让我们设置映像的工作目录。这将指示Docker使用此路径作为所有后续命令的默认位置。通过这样做，我们不必键入完整的文件路径，但可以使用基于工作目录的相对路径。

```docker
WORKDIR /app
```

通常，当您下载了一个用Java编写的项目(使用Maven进行项目管理)后，您要做的第一件事就是安装依赖项。

在运行`mvnw dependency`项之前，我们需要将Maven包装器和`pom.xml`文件放入镜像中。我们将使用`COPY`命令来完成此操作。COPY命令接受两个参数。第一个参数告诉Docker要将哪些文件复制到映像中。第二个参数告诉Docker您希望将该文件复制到何处。我们将所有这些文件和目录复制到我们的工作目录- `/app`中。

```docker
COPY .mvn/ .mvn

COPY mvnw pom.xml ./
```
一旦在映像中有了`pom.xml`文件，我们就可以使用RUN命令来执行命令`mvnw dependency:go-offline`。这与我们在机器上本地运行mvnw(或mvn)依赖项的工作方式完全相同，但这次依赖项将安装到映像中。

```docker
RUN ./mvnw dependency:go-offline
```

现在，我们有了一个基于OpenJDK版本16的Alpine版本3.13映像，并且我们还安装了依赖项。我们需要做的下一件事是将源代码添加到图像中。我们将使用COPY命令，就像我们对上面的pom.xml文件所做的那样。

```docker
COPY src ./src
```

COPY命令获取当前目录中的所有文件并将它们复制到映像中。现在，我们要做的就是告诉Docker，当我们的映像在容器中执行时，我们想要运行什么命令。我们使用CMD命令来做这个

```docker
CMD ["./mvnw", "spring-boot:run"]
```

下面是完整的Dockerfile。

```docker
# syntax=docker/dockerfile:1

FROM openjdk:16-alpine3.13

WORKDIR /app

COPY .mvn/ .mvn
COPY mvnw pom.xml ./
RUN ./mvnw dependency:go-offline

COPY src ./src

CMD ["./mvnw", "spring-boot:run"]
```
## 创建一个.dockerignore文件

要在构建上下文中使用文件，Dockerfile引用指令中指定的文件，例如COPY指令。为了提高构建的性能，并排除文件和目录，我们建议您在上下文目录中创建一个.dockerignore文件。为了提高上下文加载时间，可以在.dockerignore文件中添加一个目标目录。

## 构建一个镜像

现在我们已经创建了Dockerfile，让我们来构建映像。为此，我们使用docker构建命令。docker build命令通过一个Dockerfile和一个“context”来构建docker映像。构建的上下文是位于指定PATH或URL中的一组文件。Docker构建进程可以访问位于此上下文中的任何文件。

build命令可以选择使用——tag标志。标签用于设置图像的名称和格式为name:tag的可选标签。为了简化问题，我们暂时不使用可选标签。如果我们不传递标签，Docker将使用“latest”作为它的默认标签。您可以在构建输出的最后一行中看到这一点。

第一次构建镜像
```docker
docker build --tag java-docker .
```

## 参考
* [Docker官网地址](https://www.docker.com/)
* [Docker官网文档](https://docs.docker.com/)



*(☞ﾟヮﾟ)☞[返回首页README.md address](https://github.com/fredomli/java-standard)*
