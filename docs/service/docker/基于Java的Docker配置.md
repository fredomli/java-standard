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
在本模块中，我们将逐步为前面模块中构建的应用程序设置本地开发环境。我们将使用Docker来构建我们的图像和Docker Compose，使一切变得更容易。

## 应用示例
```shell
# 创建目录
cd /path/to/working/directory
# 使用git clone 拉取项目
git clone https://github.com/spring-projects/spring-petclinic.git
# 进入目录
cd spring-petclinic
```

> 这里我们可以将克隆下来的项目使用集成环境打开，例如IDEA，还能够使用IDEA的Docker插件。
> 
> mvnw 命令不能正常使用参考： [MavenWrapper介绍](https://github.com/fredomli/java-standard/blob/main/docs/maven/wrapper/MavenWrapper介绍.md)

创建容器：
```shell
# syntax=docker/dockerfile:1

FROM cantara/alpine-openjdk-jdk8

WORKDIR /app

COPY .mvn/ .mvn
COPY mvnw pom.xml ./
RUN ./mvnw dependency:go-offline

COPY src ./src

CMD ["./mvnw", "spring-boot:run"]
```
```shell
docker build --tag java-docker .
```

### 在容器中运行数据库
首先，我们将了解如何在容器中运行数据库，以及如何使用卷和网络来持久化数据，并允许应用程序与数据库对话。然后，我们将把所有内容放到一个Compose文件中，该文件允许我们使用一个命令设置和运行本地开发环境。最后，我们将看看如何将调试器连接到运行在容器中的应用程序。

而不是下载MySQL，安装，配置，然后运行MySQL数据库作为服务，我们可以使用Docker官方映像MySQL并在容器中运行它。

在容器中运行MySQL之前，我们将创建几个卷，Docker可以管理这些卷来存储持久性数据和配置。让我们使用Docker提供的托管卷特性，而不是使用绑定挂载。您可以在我们的文档中阅读有关使用卷的所有内容。

现在我们来创建卷。我们将为MySQL的数据和配置创建一个。
```shell
docker volume create mysql_data
docker volume create mysql_config
```
现在，我们将创建一个网络，应用程序和数据库将使用该网络彼此通信。这个网络被称为用户定义的桥接网络，它为我们提供了一个很好的DNS查找服务，我们可以在创建连接字符串时使用它。
```shell
docker network create mysqlnet
```
现在，让我们在容器中运行MySQL，并将其附加到上面创建的卷和网络上。Docker从Hub提取映像并在本地运行它。
```shell
$ docker run -it --rm -d -v mysql_data:/var/lib/mysql \
-v mysql_config:/etc/mysql/conf.d \
--network mysqlnet \
--name mysqlserver \
-e MYSQL_USER=petclinic -e MYSQL_PASSWORD=petclinic \
-e MYSQL_ROOT_PASSWORD=root -e MYSQL_DATABASE=petclinic \
-p 3306:3306 mysql:8.0.23
```
好的，现在我们有了一个运行中的MySQL，让我们更新Dockerfile来激活应用程序中定义的MySQL Spring配置文件，并从内存中的H2数据库切换到我们刚刚创建的MySQL服务器。
我们只需要将MySQL配置文件作为参数添加到`CMD`定义中。
```docker
CMD ["./mvnw", "spring-boot:run", "-Dspring-boot.run.profiles=mysql"]
```
让我们建立我们的镜像。
```shell
docker build --tag java-docker .
```
现在，让我们运行容器。这一次，我们需要设置MYSQL_URL环境变量，以便我们的应用程序知道使用什么连接字符串来访问数据库。我们将使用docker run命令来实现这一点。

```shell
$ docker run --rm -d \
--name springboot-server \
--network mysqlnet \
-e MYSQL_URL=jdbc:mysql://mysqlserver/petclinic \
-p 8080:8080 java-docker
```
让我们测试一下应用程序是否连接到数据库并能够列出兽医。
```shell
$  curl  --request GET \
  --url http://localhost:8080/vets \
  --header 'content-type: application/json'
```

### 使用Compose进行本地开发

在本节中，我们将创建一个Compose文件，使用一个命令启动java-docker和MySQL数据库。我们还将设置Compose文件以调试模式启动Java -docker应用程序，以便将调试器连接到正在运行的Java进程。

在IDE或文本编辑器中打开petclinic，并创建一个名为docker-compose.dev.yml的新文件。复制并粘贴以下命令到文件中。

```yaml
version: '3.8'
services:
  petclinic:
    build:
      context: .
    ports:
      - 8000:8000
      - 8080:8080
    environment:
      - SERVER_PORT=8080
      - MYSQL_URL=jdbc:mysql://mysqlserver/petclinic
    volumes:
      - ./:/app
    command: ./mvnw spring-boot:run -Dspring-boot.run.profiles=mysql -Dspring-boot.run.jvmArguments="-agentlib:jdwp=transport=dt_socket,server=y,suspend=n,address=*:8000"

  mysqlserver:
    image: mysql:8.0.23
    ports:
      - 3306:3306
    environment:
      - MYSQL_ROOT_PASSWORD=
      - MYSQL_ALLOW_EMPTY_PASSWORD=true
      - MYSQL_USER=petclinic
      - MYSQL_PASSWORD=petclinic
      - MYSQL_DATABASE=petclinic
    volumes:
      - mysql_data:/var/lib/mysql
      - mysql_config:/etc/mysql/conf.d
volumes:
  mysql_data:
  mysql_config:
```
这个Compose文件非常方便，因为我们不需要输入传递给docker run命令的所有参数。我们可以使用Compose文件进行声明。

我们公开端口8000并声明JVM的调试配置，以便可以附加调试器。

使用Compose文件的另一个很酷的特性是，我们设置了服务解析来使用服务名称。因此，我们现在可以在连接字符串中使用mysqlserver了。我们使用mysqlserver的原因是我们在撰写文件中已经将MySQL服务命名为mysqlserver。

现在，启动我们的应用程序并确认它正在正常运行。

```shell
docker-compose -f docker-compose.dev.yml up --build
```

我们传递——build标志，这样Docker将编译我们的映像，然后启动容器。如果运行成功，你应该会看到类似的输出:

现在让我们测试API端点。运行curl命令:
```shell
$ curl  --request GET \
  --url http://localhost:8080/vets \
  --header 'content-type: application/json'
```
您将收到以下回复:
```text
{"vetList":[{"id":1,"firstName":"James","lastName":"Carter","specialties":[],"nrOfSpecialties":0,"new":false},{"id":2,"firstName":"Helen","lastName":"Leary","specialties":[{"id":1,"name":"radiology","new":false}],"nrOfSpecialties":1,"new":false},{"id":3,"firstName":"Linda","lastName":"Douglas","specialties":[{"id":3,"name":"dentistry","new":false},{"id":2,"name":"surgery","new":false}],"nrOfSpecialties":2,"new":false},{"id":4,"firstName":"Rafael","lastName":"Ortega","specialties":[{"id":2,"name":"surgery","new":false}],"nrOfSpecialties":1,"new":false},{"id":5,"firstName":"Henry","lastName":"Stevens","specialties":[{"id":1,"name":"radiology","new":false}],"nrOfSpecialties":1,"new":false},{"id":6,"firstName":"Sharon","lastName":"Jenkins","specialties":[],"nrOfSpecialties":0,"new":false}]}
```
### Connect 连接调试器
我们将使用IntelliJ IDEA附带的调试器。您可以使用这个IDE的社区版本。在IntelliJ IDEA中打开项目，然后进入运行菜单>编辑配置。添加一个新的远程JVM调试配置，类似如下:

打开以下文件src/main/java/org/springframework/samples/ petclinics /vet/ vetcontroller .java，并在showResourcesVetList函数中添加一个断点，例如第54行。

启动您的调试会话，运行菜单，然后调试您的配置名称


发送请求调用服务器端点。：
```shell
curl --request GET --url http://localhost:8080/vets
```
您应该已经看到代码在第54行中断，现在您可以像平常一样使用调试器了。你还可以检查和监视变量，设置条件断点，查看堆栈跟踪和其他一些操作。

您还可以激活由SpringBoot Dev Tools提供的实时重新加载选项。有关如何连接到远程应用程序的信息，请参阅SpringBoot文档。

## Docker 测试
测试是现代软件开发的重要组成部分。对于不同的开发团队来说，测试意味着很多东西。有单元测试、集成测试和端到端测试。在本指南中，我们将看看如何在Docker中运行单元测试。

### 重构Dockerfile以运行测试
Spring宠物诊所的源代码已经在测试目录src/test/java/org/springframework/samples/petclinic中定义了测试。您只需要在pomo.xml中更新JaCoCo版本，以确保您的测试可以使用<jacoco.version>0.8.6</jacoco.version>，所以我们可以使用以下Docker命令来启动容器并运行测试:

```shell
docker run -it --rm --name springboot-test java-docker ./mvnw test
```

### 多阶段Dockerfile测试
让我们看看如何将测试命令拖放到Dockerfile中。下面是一个多阶段的Dockerfile，我们将使用它来构建我们的生产映像和测试映像。将突出显示的行添加到Dockerfile中

```docker
# syntax=docker/dockerfile:1

FROM openjdk:16-alpine3.13 as base

WORKDIR /app

COPY .mvn/ .mvn
COPY mvnw pom.xml ./
RUN ./mvnw dependency:go-offline
COPY src ./src

FROM base as test
CMD ["./mvnw", "test"]

FROM base as development
CMD ["./mvnw", "spring-boot:run", "-Dspring-boot.run.profiles=mysql", "-Dspring-boot.run.jvmArguments='-agentlib:jdwp=transport=dt_socket,server=y,suspend=n,address=*:8000'"]

FROM base as build
RUN ./mvnw package

FROM openjdk:11-jre-slim as production
EXPOSE 8080

COPY --from=build /app/target/spring-petclinic-*.jar /spring-petclinic.jar

CMD ["java", "-Djava.security.egd=file:/dev/./urandom", "-jar", "/spring-petclinic.jar"]
```
我们首先在FROM openjdk:16-alpine3.13语句中添加一个标签。这允许我们在其他构建阶段中引用这个构建阶段。接下来，我们添加了一个新的构建阶段，标记为test。我们将使用这个阶段运行我们的测试。

现在让我们重新构建映像并运行测试。我们将像上面一样运行docker构建命令，但这次我们将添加`--target`测试标志，以便我们专门运行测试构建阶段。

```shell
docker build -t java-docker --target test .
```
现在我们已经构建了测试映像，我们可以将其作为容器运行，看看我们的测试是否通过。
```shell
docker run -it --rm --name springboot-test java-docker
```
构建输出被截断，但是您可以看到Maven测试运行器是成功的，我们的所有测试都通过了。

这是伟大的。但是，我们必须运行两个Docker命令来构建和运行我们的测试。我们可以通过在测试阶段使用RUN语句而不是CMD语句来稍微改进这一点。CMD语句不会在映像构建过程中执行，而是在容器中运行映像时执行。当使用RUN语句时，我们的测试在构建映像时运行，在失败时停止构建。

```docker
# syntax=docker/dockerfile:1

FROM openjdk:16-alpine3.13 as base

WORKDIR /app

COPY .mvn/ .mvn
COPY mvnw pom.xml ./
RUN ./mvnw dependency:go-offline
COPY src ./src

FROM base as test
RUN ["./mvnw", "test"]

FROM base as development
CMD ["./mvnw", "spring-boot:run", "-Dspring-boot.run.profiles=mysql", "-Dspring-boot.run.jvmArguments='-agentlib:jdwp=transport=dt_socket,server=y,suspend=n,address=*:8000'"]

FROM base as build
RUN ./mvnw package

FROM openjdk:11-jre-slim as production
EXPOSE 8080

COPY --from=build /app/target/spring-petclinic-*.jar /spring-petclinic.jar

CMD ["java", "-Djava.security.egd=file:/dev/./urandom", "-jar", "/spring-petclinic.jar"]
```

现在，要运行我们的测试，我们只需要像上面一样运行docker构建命令。
```shell
docker build -t java-docker --target test .
```
为了简单起见，构建输出被截断，但是您可以看到我们的测试成功运行并通过了。让我们中断其中一个测试，并在测试失败时观察输出。

打开src/test/java/org/springframework/samples/ petclinics /model/ validatortests .java文件，将第57行更改为如下内容。

```json
55   ConstraintViolation<Person> violation = constraintViolations.iterator().next()
56   assertThat(violation.getPropertyPath().toString()).isEqualTo("firstName")
57   assertThat(violation.getMessage()).isEqualTo("must be empty")
58 }
```
现在，从上面运行docker构建命令，观察构建失败，失败的测试信息被打印到控制台。

```shell
$ docker build -t java-docker --target test .
 [base 6/6] COPY src ./src
 ERROR [test 1/1] RUN ["./mvnw", "test"]
```
### 多阶段Dockerfile开发
Dockerfile的新版本生成了一个最终映像，该映像已准备好用于生产，但是您可以注意到，您还需要一个专门的步骤来生成开发容器。

```docker
FROM base as development
CMD ["./mvnw", "spring-boot:run", "-Dspring-boot.run.profiles=mysql", "-Dspring-boot.run.jvmArguments='-agentlib:jdwp=transport=dt_socket,server=y,suspend=n,address=*:8000'"]
```
现在我们可以更新docker-compose.dev.yml，使用这个特定的目标来构建petclinic服务，并删除命令定义，如下所示:

```yaml
services:
 petclinic:
   build:
     context: .
     target: development
   ports:
     - 8000:8000
     - 8080:8080
   environment:
     - SERVER_PORT=8080
     - MYSQL_URL=jdbc:mysql://mysqlserver/petclinic
   volumes:
     - ./:/app
```
现在，让我们运行Compose应用程序。您现在应该看到应用程序的行为与以前一样，并且您仍然可以调试它。
```shell
docker-compose -f docker-compose.dev.yml up --build
```

## Docker CI/CD(持续化集成，持续化部署) 配置

## 部署你的项目
现在，我们已经配置了CI/CD管道，让我们看看如何部署应用程序。Docker支持在Azure ACI和AWS ECS上部署容器。如果你已经在Docker Desktop中启用了Kubernetes，你也可以将你的应用部署到Kubernetes。

## 参考
* [Docker官网地址](https://www.docker.com/)
* [Docker官网文档](https://docs.docker.com/)



*(☞ﾟヮﾟ)☞[返回首页README.md address](https://github.com/fredomli/java-standard)*
