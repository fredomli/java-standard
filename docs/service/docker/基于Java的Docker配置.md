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
> 这里构建是不能成功的，因为maven原因，这里项目使用的是本地安装的Maven，仅作参考学习。 

正确的可以运行方式，直接将项目打包。DockerFile文件改为下面内容：
```
# syntax=docker/dockerfile:1
FROM cantara/alpine-openjdk-jdk8
COPY /target/spring-boot-docker.jar .
ENTRYPOINT ["java","-jar","spring-boot-docker.jar"]
```
再次构建镜像。

> 注意：docker配置文件名为：`Dockerfile`,某个人将它创建成`DockerFile`了，导致报错说没找到DockerFile。

## 查看本地镜像
```shell
$ docker images
REPOSITORY                    TAG                 IMAGE ID            CREATED             SIZE
REPOSITORY                    TAG                 IMAGE ID            CREATED             SIZE
java-docker                   latest              3921b6cde97c        3 minutes ago       123MB
```

您至少应该看到我们刚刚构建的java-docker:latest。

## 镜像的标签
镜像名称由斜杠分隔的名称组件组成。名称组件可以包含小写字母、数字和分隔符。分隔符定义为句点、一个或两个下划线或一个或多个破折号。名称组件不能以分隔符开始或结束。

镜像由清单和层列表组成。此时，除了指向这些工件组合的“标记”之外，不要太担心清单和层。一个镜像可以有多个标记。让我们为我们创建的镜像创建第二个标签，并看看它的层。
```shell
docker tag java-docker:latest java-docker:v1.0.0
```
现在，运行docker images命令查看本地镜像的列表。
```shell
$ docker images
REPOSITORY                    TAG                 IMAGE ID            CREATED             SIZE
java-docker                   latest              3921b6cde97c        8 minutes ago       123MB
java-docker                   v1.0.0              3921b6cde97c        8 minutes ago       123MB
```
您可以看到我们有两个以java-docker开始的镜像。我们知道它们是相同的图像，因为如果你看一下image ID列，你可以看到这两个镜像的值是相同的。

## 删除镜像
让我们删除刚刚创建的标记。为此，我们将使用rmi命令。rmi命令代表“remove image”。
```shell
$ docker rmi java-docker:v1.0.0
Untagged: java-docker:v1.0.0
```

## 运行容器
容器是一个正常的操作系统进程，除了这个进程是隔离的，它有自己的文件系统、自己的网络和自己与主机隔离的进程树。

要在容器中运行映像，可以使用docker run命令。docker run命令需要一个参数，即镜像的名称。让我们启动映像并确保它正确运行。在终端上运行以下命令:
```shell
docker run java-docker
```
运行此命令后，您将注意到我们没有返回到命令提示符。这是因为我们的应用程序是一个REST服务器，在循环中运行，等待传入的请求，而不会将控制权返回给操作系统，直到我们停止容器。
让我们打开一个新的终端，然后使用curl命令向服务器发出GET请求。
```shell
curl --request GET \
--url http://localhost:8080/ \
--header 'content-type: application/json'

```
如您所见，我们的curl命令失败了，因为到服务器的连接被拒绝了。这意味着我们无法连接到端口8080上的本地主机。这是意料之中的，因为我们的容器是独立运行的，其中包括网络。让我们停止集装箱并重新启动，在我们的本地网络上发布8080端口。

要停止容器，请按ctrl-c。这将返回到终端提示符。

为了发布容器的端口，我们将在docker run命令上使用——publish标志(简称-p)。——publish命令的格式为[host port]:[container port]。因此，如果我们想要将容器内部的端口8000公开到容器外部的端口8080，我们需要将8080:8000传递给——publish标志。

启动容器并将8080端口公开到主机上的8080端口。
```shell
docker run --publish 8080:8080 java-docker
```
现在，让我们从上面重新运行curl命令。

```shell
 curl --request GET \
--url http://localhost:8080/ \
--header 'content-type: application/json'
```

## 列出本地容器
当我们在后台运行容器时，我们如何知道容器是否正在运行，或者其他什么容器正在我们的机器上运行?我们可以运行docker ps命令。就像在Linux中运行ps命令来查看机器上的进程列表一样，我们也可以运行docker ps命令来查看机器上运行的容器列表。
```shell
$ docker ps
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                    NAMES
1d93aa88224f        java-docker         "java -jar spring-bo…"   36 seconds ago      Up 34 seconds       0.0.0.0:8080->8080/tcp   nostalgic_einstein
```
docker ps命令提供了一系列关于正在运行的容器的信息。我们可以看到容器ID，容器内部运行的图像，用于启动容器的命令、创建容器时的状态、公开的端口和容器的名称。

您可能想知道我们的集装箱名称来自哪里。由于我们在启动容器时没有为它提供名称，Docker生成了一个随机名称。我们马上就会解决这个问题，但首先我们需要停止容器。要停止容器，运行docker stop命令，停止容器。我们需要传递容器的名称，或者可以使用容器ID。
```shell
$ docker stop nostalgic_einstein
nostalgic_einstein
```
## 停止、启动和命名容器
您可以启动、停止和重新启动Docker容器。当我们停止一个容器时，它不是被删除，而是状态被更改为已停止，容器内的进程也将停止。当我们在前一个模块中运行docker ps命令时，默认输出只显示正在运行的容器。当我们传递——all或-a时，我们会看到机器上的所有容器，而不管它们的启动或停止状态。

```shell
$ docker ps -a
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS                            PORTS               NAMES
1d93aa88224f        java-docker         "java -jar spring-bo…"   9 minutes ago       Exited (143) About a minute ago                       nostalgic_einstein
c4163c78e861        9709a179344f        "java -jar spring-bo…"   2 hours ago         Exited (143) About an hour ago                        boot-start
87beecf74441        ad3c7f023ed0        "docker-entrypoint.s…"   7 hours ago         Exited (0) 5 hours ago                                bold_wozniak
```
您现在应该看到列出了几个容器。这些是我们启动和停止的容器，但没有被移走。
让我们重新启动刚刚停止的容器。找到我们刚刚停止的容器的名称，并使用restart命令替换下面的容器名称。

```shell
$ docker restart nostalgic_einstein
```
现在我们的容器停止了，让我们删除它。当您删除容器时，容器内的进程将停止，容器的元数据将被删除。
要删除容器，只需运行docker rm命令传递容器名称。可以使用单个命令向命令传递多个容器名称。同样，将下面命令中的容器名称替换为系统中的容器名称。
```shell
$ docker rm trusting_beaver boot-start
trusting_beaver
boot-start
```
再次运行`docker ps --all`命令，查看所有容器都被删除了。

在，让我们来解决随机命名的问题。标准的做法是为容器命名，原因很简单，这样更容易识别容器中运行的是什么，以及它与什么应用程序或服务相关联。

要命名容器，我们只需要将`--name`标志传递给docker run命令。
```shell
$ docker run --rm -d -p 8080:8080 --name springboot-server java-docker
7de85ba064107a32be5c8ec8b66982146018c987c4f3ce808aad9494b2c24f66
```
```shell
$ docker ps -a
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS                       PORTS                    NAMES
7de85ba06410        java-docker         "java -jar spring-bo…"   8 seconds ago       Up 7 seconds                 0.0.0.0:8080->8080/tcp   springboot-server
1d93aa88224f        java-docker         "java -jar spring-bo…"   15 minutes ago      Exited (143) 7 minutes ago                            nostalgic_einstein
c4163c78e861        9709a179344f        "java -jar spring-bo…"   2 hours ago         Exited (143) 2 hours ago                              boot-start
87beecf74441        ad3c7f023ed0        "docker-entrypoint.s…"   8 hours ago         Exited (0) 5 hours ago                                bold_wozniak

```
这是更好的!现在，我们可以根据名称轻松地识别容器。

## 使用容器进行开发

## 参考
* [Docker官网地址](https://www.docker.com/)
* [Docker官网文档](https://docs.docker.com/)



*(☞ﾟヮﾟ)☞[返回首页README.md address](https://github.com/fredomli/java-standard)*
