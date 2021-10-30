# 安装Jenkins

### 目录
* Docker
* Kubernetes
* Linux
* macOS
* WAR files
* Windows
* Other Systems
* Offline Installations
* Initial Settings

### 介绍
Jenkins通常作为一个独立的应用程序运行在它自己的进程中，并使用内置的Java servlet容器/应用服务器(Jetty)。

Jenkins也可以作为servlet在不同的Java servlet容器(如Apache Tomcat或GlassFish)中运行。然而，设置这些类型的安装的说明超出了本章的范围。


## Docker 安装方式
Docker是一个在被称为“容器”(或Docker容器)的隔离环境中运行应用程序的平台。像Jenkins这样的应用程序可以下载为只读的“镜像”(或Docker镜像)，每个镜像都作为容器运行在Docker中。一个Docker容器实际上是一个Docker镜像的“运行实例”。从这个角度来看，图像或多或少是永久存储的(即在图像更新发布的范围内)，而容器是临时存储的。

Docker的基础平台和容器设计意味着一个Docker映像(对于任何给定的应用程序，如Jenkins)可以在任何支持的操作系统(macOS, Linux和Windows)或云服务(AWS和Azure)上运行Docker。

### 安装Docker
第一种方式，在您的操作系统上安装Docker，参考 [centos7安装docker](https://github.com/fredomli/java-standard/blob/main/docs/linux/install/centos7安装docker.md)

另一种解决方案是，您可以访问Dockerhub并选择适合您的操作系统或云服务的Docker社区版。按照他们网站上的安装说明安装。

如果您在基于linux的操作系统上安装Docker，请确保您配置了Docker，使其可以作为非root用户进行管理。更多信息请参阅Docker文档中的Linux安装后步骤页面。这个页面还包含了关于如何配置Docker在引导时启动的信息。

### 前提条件

最低系统要求：
* 256 MB 内存
* 1 GB的存储空间(如果将Jenkins作为Docker容器运行，建议最小容量为10 GB)

小型团队的推荐硬件配置:
* 4 GB+ 内存
* 50 GB+ 储存空间 
  
全面的硬件建议:
* 参见：[硬件推荐](https://www.jenkins.io/doc/book/scaling/hardware-recommendations/)

软件需求：
* Java:请参见[Java需求页面](https://www.jenkins.io/doc/administration/requirements/java/)
* Web浏览器:请参见[“Web浏览器兼容性”页面](https://www.jenkins.io/doc/administration/requirements/web-browsers/)
* 对于Windows操作系统:[Windows 支持政策](https://www.jenkins.io/doc/administration/requirements/windows/)

### 下载并运行Docker中的Jenkins

有几个Jenkins的Docker镜像可用。推荐使用的Docker镜像是官方jenkins/jenkins镜像(来自Docker Hub存储库)。此镜像包含Jenkins的当前长期支持(LTS)版本(已准备用于生产)。但是这个镜像里面没有docker CLI，也没有和常用的Blue Ocean插件和特性绑定。这意味着，如果你想使用Jenkins和Docker的全部功能，你可能需要通过下面描述的安装过程。

每次jenkins Docker的新版本发布时，都会发布一个新的jenkins/jenkins图像。你可以在标签页上看到先前发布的jenkins/jenkins图像版本的列表。

### 在macOS和Linux上
* 打开终端窗口。
* 使用下面的Docker network Create命令在Docker中创建一个桥接网络:  
  ```
  docker network create jenkins
  ```
* 为了在Jenkins节点中执行Docker命令，下载并运行`Docker:dind` Docker映像，使用下面的Docker run命令:
    ```shell
    docker run \
    --name jenkins-docker \
    --rm \
    --detach \
    --privileged \
    --network jenkins \
    --network-alias docker \
    --env DOCKER_TLS_CERTDIR=/certs \
    --volume jenkins-docker-certs:/certs/client \
    --volume jenkins-data:/var/jenkins_home \
    --publish 2376:2376 \
    docker:dind \
    --storage-driver overlay2
    ```
命令解释：
* `--name` (可选)指定要用于运行映像的Docker容器名称。默认情况下，Docker将为容器生成一个唯一的名称。
* `--rm` (可选)当Docker容器关闭时，会自动删除它(Docker映像的实例)。
* `--detach` (可选)在后台运行Docker容器。这个实例可以稍后通过运行docker stop jenkins-docker来停止。
* `--privileged` 目前在Docker中运行Docker需要特权访问才能正常运行。新的Linux内核版本可能会放宽这一要求。
* `--network` 这与前面步骤中创建的网络相对应。
* `--network-alias` 使Docker容器中的Docker在jenkins网络中作为主机名Docker可用。
* `--env` 启用Docker服务器中TLS的使用。由于使用了特权容器，所以建议这样做，尽管这需要使用下面描述的共享卷。这个环境变量控制管理Docker TLS证书的根目录。
* `--volume jenkins-docker-certs:/certs/client` 将容器中的/certs/client目录映射到上面创建的名为jenkins-docker-certs的Docker卷。
* `--volume jenkins-data:/var/jenkins_home` 将容器内的/var/jenkins_home目录映射到名为jenkins-data的Docker卷。这将允许由该Docker容器的Docker守护进程控制的其他Docker容器装入来自Jenkins的数据。
* `--publish` (可选)在主机上公开Docker守护进程端口。这对于在主机上执行docker命令来控制这个docker内部守护进程非常有用。
* `docker:dind` docker:查找图像本身。这个镜像可以在运行之前通过命令:docker image pull docker:dind下载。
* `--storage-driver` Docker卷的存储驱动程序。有关支持的选项，请参阅“Docker存储驱动程序”。

注意:如果复制和粘贴上面的命令片段不起作用，请在这里尝试复制和粘贴这个没有注释的版本:

```shell
docker run --name jenkins-docker --rm --detach \
  --privileged --network jenkins --network-alias docker \
  --env DOCKER_TLS_CERTDIR=/certs \
  --volume jenkins-docker-certs:/certs/client \
  --volume jenkins-data:/var/jenkins_home \
  --publish 2376:2376 docker:dind --storage-driver overlay2
```

### 定制官方Jenkins Docker镜像
执行以下两个步骤:
1.创建Dockerfile，内容如下:
```shell
FROM jenkins/jenkins:2.303.2-jdk11
USER root
RUN apt-get update && apt-get install -y apt-transport-https \
       ca-certificates curl gnupg2 \
       software-properties-common
RUN curl -fsSL https://download.docker.com/linux/debian/gpg | apt-key add -
RUN apt-key fingerprint 0EBFCD88
RUN add-apt-repository \
       "deb [arch=amd64] https://download.docker.com/linux/debian \
       $(lsb_release -cs) stable"
RUN apt-get update && apt-get install -y docker-ce-cli
USER jenkins
RUN jenkins-plugin-cli --plugins "blueocean:1.25.0 docker-workflow:1.26"
```

2. 从这个Dockerfile构建一个新的docker映像，并为映像指定一个有意义的名称，例如:“myjenkins-blueocean: 1.1”:
```shell
docker build -t myjenkins-blueocean:1.1 .
```
请记住，如果之前没有这样做过，上面描述的过程将自动下载官方Jenkins Docker图像。

### 使用下面的Docker Run命令在Docker中运行你自己的myjenkins-blueocean:1.1映像作为容器:
```shell
docker run \
  --name jenkins-blueocean \
  --rm \
  --detach \
  --network jenkins \
  --env DOCKER_HOST=tcp://docker:2376 \
  --env DOCKER_CERT_PATH=/certs/client \
  --env DOCKER_TLS_VERIFY=1 \
  --publish 8080:8080 \
  --publish 50000:50000 \
  --volume jenkins-data:/var/jenkins_home \
  --volume jenkins-docker-certs:/certs/client:ro \
  myjenkins-blueocean:1.1
```
命令解释：
* (可选)指定Docker映像实例的Docker容器名。
* (可选)当Docker容器关闭时，自动移除该容器。
* (可选)在后台运行当前容器。"detached"模式)并输出容器ID。如果不指定此选项，则在终端窗口中输出该容器的运行Docker日志。
* 将此容器连接到前面步骤中定义的jenkins网络。这使得上一步中的Docker守护进程可以通过主机名Docker对Jenkins容器使用。
* 指定docker、docker-compose和其他docker工具在上一步中连接到docker守护进程时使用的环境变量。
* (即地图。"发布")当前容器的8080端口到主机上的8080端口。第一个数字表示主机上的端口，最后一个数字表示容器的端口。因此，如果您为该选项指定了-p 49000:8080，那么您将通过端口49000访问主机上的Jenkins。
* 可选)将当前容器50000端口映射到主机50000端口。只有当您在其他机器上设置了一个或多个入站Jenkins代理，这些代理又与您的Jenkins -blueocean容器(Jenkins“控制器”)交互时，才有必要这样做。缺省情况下，入站Jenkins代理通过TCP端口50000与Jenkins控制器通信。您可以通过配置全局安全页面在Jenkins控制器上更改此端口号。如果您想将Jenkins控制器的入站Jenkins代理的TCP端口更改为51000(例如)，那么您将需要重新运行Jenkins(通过docker run…命令)，并使用类似于——publish 52000:51000的方式指定这个“publish”选项，其中，最后一个值与Jenkins控制器上的更改值匹配，第一个值是托管Jenkins控制器的机器上的端口号。
  入站Jenkins代理与该端口上的Jenkins控制器通信(本例中为52000)。注意，WebSocket代理不需要这个配置。
* 将容器中的/var/jenkins_home目录映射到名为jenkins-data的Docker卷。除了将/var/jenkins_home目录映射到Docker卷，您还可以将此目录映射到您的机器本地文件系统上的一个卷。例如，指定选项,--volume $HOME/jenkins:/var/jenkins_home将容器的/var/jenkins_home目录映射到本地机器的$HOME目录中的jenkins子目录，通常是/Users/<your-username>/jenkins或/ HOME/ <your-username>/jenkins。请注意，如果为此更改源卷或目录，则需要更新上面docker:dind容器中的卷以与之匹配。
* 将/certs/client目录映射到先前创建的jenkins-docker-certs卷。这使得连接Docker守护进程所需的客户端TLS证书在DOCKER_CERT_PATH环境变量指定的路径中可用。
* 在上一步中构建的Docker映像的名称。

注意:如果复制和粘贴上面的命令片段不起作用，请在这里尝试复制和粘贴这个没有注释的版本:

```shell
docker run --name jenkins-blueocean --rm --detach \
  --network jenkins --env DOCKER_HOST=tcp://docker:2376 \
  --env DOCKER_CERT_PATH=/certs/client --env DOCKER_TLS_VERIFY=1 \
  --publish 8080:8080 --publish 50000:50000 \
  --volume jenkins-data:/var/jenkins_home \
  --volume jenkins-docker-certs:/certs/client:ro \
  myjenkins-blueocean:1.1
```
## Window 安装方式

## 安装后的设置向导
下载、安装和运行Jenkins后，使用上面的一个过程，安装后安装向导就开始了。

这个设置向导带你通过几个快速的“一次性”步骤来解锁Jenkins，用插件定制它，并创建第一个管理员用户，通过它你可以继续访问Jenkins。

### 解锁 Jenkins
当您第一次访问一个新的Jenkins实例时，系统会要求您使用一个自动生成的密码来解锁它。

浏览器定位到 `http://localhost:8080` (或安装时为Jenkins配置的任何端口)，并等待直到解锁Jenkins页面出现。






## 参考
* [官网地址](https://www.jenkins.io/)
* [Jenkins文档](https://www.jenkins.io/doc/)
* [Docker官网地址](https://www.docker.com/)
* [Docker官网文档](https://docs.docker.com/)

*(☞ﾟヮﾟ)☞[返回首页README.md address](https://github.com/fredomli/java-standard)*
