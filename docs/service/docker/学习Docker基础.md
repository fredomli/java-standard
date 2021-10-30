# Docker基础学习

## 概述

> Docker是最受欢迎和最受欢迎的开发工具，帮助数百万开发人员构建、共享和运行任何应用程序，无论在云上还是在任何地方。

Docker使开发变得高效和可预测
Docker去掉了重复的、平凡的配置任务，并在整个开发生命周期中用于快速、简单和可移植的应用程序开发——桌面和云。Docker全面的端到端平台包括ui、cli、api和安全，它们被设计成在整个应用程序交付生命周期中协同工作。


### 构建
* 利用Docker镜像来有效地在Windows和Mac上开发自己独特的应用程序，从而在编码方面领先一步。使用Docker Compose创建多容器应用程序。
* 在整个开发管道中集成你最喜欢的工具——Docker可以与你使用的所有开发工具一起工作，包括VS Code, CircleCI和GitHub。
* 将应用程序打包为可移植容器镜像，以在任何环境中一致地运行，从本地Kubernetes到AWS ECS、Azure ACI、谷歌GKE等等。

### 分享
* 利用Docker可信内容，包括Docker官方镜像和来自Docker Hub库的Docker验证发布者的图像。
* 通过与团队成员和其他开发人员合作进行创新，并轻松地将镜像发布到Docker Hub。
* 使用基于角色的访问控制个性化开发人员对镜像的访问，并使用Docker Hub审计日志深入了解活动历史。

### 运行
* 免费提供多个应用程序，并让它们以相同的方式在所有环境中运行，包括设计、测试、登台和生产环境——桌面或云本地环境。
* 在不同的容器中使用不同的语言独立地部署应用程序。降低语言、库或框架之间冲突的风险。
* 通过Docker Compose CLI的简单性加快开发速度，只需一个命令，就可以用AWS ECS和Azure ACI在本地和云上启动应用程序。

### 开始说明
这个页面包含了关于如何开始使用Docker的一步一步的说明。在本教程中，您将学习如何
* 构建并运行作为容器的镜像
* 使用Docker Hub共享镜像
* 使用带有数据库的多个容器部署Docker应用程序
* 使用Docker Compose运行应用程序

### 下载并安装Docker
在[官网下载](https://www.docker.com/) 需要的Docker，本教程假设您的机器上安装了当前版本的Docker。如果您没有安装Docker，请选择您喜欢的操作系统下载Docker，下面为Linux安装Docker的示例：
[centos7安装docker](https://github.com/fredomli/java-standard/blob/main/docs/linux/install/centos7安装docker.md)

## 开始教程
如果您已经运行了该命令以开始学习本教程，那么恭喜您!如果没有，打开命令提示符或bash窗口，运行命令:

```shell
docker run -d -p 80:80 docker/getting-started
```
您会注意到使用了一些标志。这里有更多关于他们的信息:
-d -以分离模式运行容器(在后台)
-p 80:80 -映射主机的80端口到容器docker/getting-started中的80端口-要使用的映像

> 您可以组合单个字符标志来缩短完整的命令。例如，上面的命令可以写成:
> 
>     docker run -dp 80:80 docker/getting-started

### Docker仪表盘
在深入讨论之前，我们想突出显示Docker Dashboard，它可以让您快速查看在您的机器上运行的容器。Docker Dashboard可用于Mac和Windows。它可以让您快速访问容器日志，让您在容器中获得一个shell，并让您轻松地管理容器的生命周期(停止、删除等)。

仪表盘就是一个桌面管理工具，是Docker的可视化操作界面。如下图所示，

![pc1](https://gitee.com/fredomli/fredomli-picture/raw/picgo/static/images/wordpress/docker-desktop-getstart.png)


### 什么是容器
现在在您已经运行了容器（getting started），那么什么是容器呢?简单地说，容器是您机器上的一个沙箱进程，它与主机上的所有其他进程隔离。这种隔离利用了内核名称空间和cgroups，这些特性在Linux中已经存在很长时间了。Docker努力使这些功能易于使用。

### 什么是镜像
当运行容器时，它使用一个隔离的文件系统。这个自定义文件系统由容器镜像提供。由于镜像包含容器的文件系统，它必须包含运行应用程序所需的所有东西——所有依赖项、配置、脚本、二进制文件等。映像还包含容器的其他配置，如环境变量、要运行的默认命令和其他元数据。

## 简单应用程序

在本教程的剩余部分，我们将使用Node.js中运行的一个简单的todo列表管理器。如果您不熟悉Node.js，请不要担心。不需要真正的JavaScript经验。

此时，你的开发团队非常小，你只是在开发一款能够证明你的MVP(最小可行产品)的应用。你想要展示它是如何工作的，以及它能够做什么，而不需要考虑它在一个大型团队、多个开发人员中是如何工作的。

### 获取资源包
在运行应用程序之前，我们需要将应用程序源代码放到机器上。对于真正的项目，您通常会克隆回购。但是，对于本教程，我们已经创建了一个包含应用程序的ZIP文件。

* [下载App内容](https://github.com/docker/getting-started/tree/master/app) 你可以把整个项目拉出来，也可以下载一个压缩包，然后把app文件夹解压出来。
* 提取后，使用您喜欢的代码编辑器打开项目。如果你需要编辑器，你可以使用Visual Studio Code。您应该看到这个包。Json和两个子目录(src和spec)。
  
使用VSCode打开，如下图所示：

![pc2](https://gitee.com/fredomli/fredomli-picture/raw/picgo/static/images/wordpress/docker-test-source-show.png)



### 构建应用程序的容器镜像
为了构建应用程序，我们需要使用Dockerfile。Dockerfile是一个简单的基于文本的指令脚本，用于创建容器映像。如果您以前创建过Dockerfiles，您可能会在下面的Dockerfile中看到一些缺陷。但是,别担心。我们会复习的。

在与文件package.json 相同的文件夹中创建一个名为Dockerfile的文件。包含以下内容:
```text
# syntax=docker/dockerfile:1
FROM node:12-alpine
RUN apk add --no-cache python g++ make
WORKDIR /app
COPY . .
RUN yarn install --production
CMD ["node", "src/index.js"]
```
> 请检查Dockerfile文件是否没有.txt之类的文件扩展名。一些编辑器可能会自动附加此文件扩展名，这将导致在下一步中出现错误。

如下图所示：  

![pc3](https://gitee.com/fredomli/fredomli-picture/raw/picgo/static/images/wordpress/docker-test-source-dockerfile.png)

如果你还没有这样做，打开一个终端，用Dockerfile进入app目录。现在使用`docker build`命令构建容器映像。

```shell
docker build -t getting-started .
```
这个命令使用Dockerfile来构建一个新的容器映像。您可能已经注意到许多“层”被下载。这是因为我们告诉构建者我们想要从node:12-alpine图像开始。但是，由于我们的机器上没有这个，这个映像需要下载。

在下载映像之后，我们在应用程序中复制并使用yarn安装应用程序的依赖项。CMD指令指定从这个映像启动容器时要运行的默认命令。

最后，-t标记我们的图像。将其简单地看作是最终图像的一个人类可读的名称。因为我们将图像命名为getting-started，所以我们可以在运行容器时引用该图像。

`.`在docker构建命令的末尾告诉docker应该在当前目录中查找Dockerfile。

如下图所示：

![pc4](https://gitee.com/fredomli/fredomli-picture/raw/picgo/static/images/wordpress/docker-sample-application-build.png)


### 启动一个应用程序容器
现在我们有了映像，让我们运行应用程序。为此，我们将使用docker run命令(还记得之前讲过的吗?)

使用`docker run`命令启动容器，并指定我们刚刚创建的映像的名称:
```shell
docker run -dp 3000:3000 getting-started
```

如下图所示：

![pc5](https://gitee.com/fredomli/fredomli-picture/raw/picgo/static/images/wordpress/docker-run-success-img.png)

> 还记得-d和-p标志吗?我们以“分离”模式(在后台)运行新容器，并在主机端口3000和容器端口3000之间创建映射。如果没有端口映射，我们就无法访问应用程序。

几秒钟后，打开浏览器到`http://localhost:3000`。你应该看看我们的应用。

![pc6](https://gitee.com/fredomli/fredomli-picture/raw/picgo/static/images/wordpress/docker-tudolist-run-success.png)

继续添加一两个项目，看看它是否如您所期望的那样工作。您可以将项目标记为完整，并删除项目。您的前端成功地将项目存储在后端。又快又简单，是吧?


如果你快速浏览一下Docker Dashboard，你应该会看到两个容器正在运行(本教程和刚刚启动的app容器)。

> 在这个简短的小节中，我们学习了关于构建容器映像的基本知识，并创建了一个Dockerfile来完成此任务。一旦我们构建了映像，我们就启动了容器并看到了正在运行的应用程序。
> 
> 接下来，我们将对应用程序进行修改，并学习如何使用新镜像更新正在运行的应用程序。在此过程中，我们将学习其他一些有用的命令。

## 更新应用程序
作为一个小功能请求，产品团队要求我们在没有任何待办事项时更改“空文本”。他们希望将其过渡到以下内容:

```text
You have no todo items yet! Add one above!
```
### 更新源代码

#### 在`src/static/js/app.js`文件中，更新第56行以使用新的空文本。

```text
 -     <p className="text-center">No items yet! Add one above!</p>
 +     <p className="text-center">You have no todo items yet! Add one above!</p>
```

#### 让我们使用与前面相同的命令构建映像的更新版本。
```shell
docker build -t getting-started .
```

> 你可能看到了这样的错误(id将是不同的):
>       
>       docker: Error response from daemon: driver failed programming external connectivity on endpoint crazy_napier (7b7e0b077e1d26a799e799c95a198a6d654658d3bbe2d5f689e6ed2334031b2c): Bind for 0.0.0.0:3000 failed: port is already allocated.


所以,发生了什么事?我们无法启动新的容器，因为旧的容器还在运行。这是一个问题，因为容器使用主机的端口3000，并且机器上只有一个进程(包括容器)可以侦听特定的端口。为了解决这个问题，我们需要移除旧的容器。

### 移除旧的容器
要移除一个容器，首先需要停止它。一旦它停止了，它就可以被移除。我们有两种方法可以移除旧容器。自由地选择你最舒服的道路。

#### 通过CLI移除容器
##### 使用docker ps命令获取容器的ID。
```shell
docker ps
```
##### 使用docker stop命令停止容器。
```shell
# 用来自docker ps的ID交换出<容器ID>
docker stop <容器ID>
```
##### 一旦容器停止，就可以使用docker rm命令删除它。
```shell
docker rm <容器ID>
```
> 你可以通过在docker rm命令中添加" force "标志来停止和删除一个容器。例如:docker rm -f <the-container-id>

### 使用Docker仪表板移除容器
如果您打开Docker仪表板，您可以删除一个容器，只需单击两下!这当然比查找容器ID并删除它要容易得多。
* 打开仪表板后，将鼠标悬停在app容器上，你会看到右侧出现一组操作按钮。
* 单击垃圾桶图标删除容器。
* 确认删除，你就完成了!

![pc7](https://gitee.com/fredomli/fredomli-picture/raw/picgo/static/images/wordpress/docker-run-success-img.png)


#### 启动更新后的应用程序容器
```shell
docker run -dp 3000:3000 getting-started
```
在http://localhost:3000上刷新你的浏览器，你应该会看到你更新的帮助文本!

## 共享应用程序
现在我们已经创建了一个图像，让我们分享它吧!要共享Docker映像，必须使用Docker注册表。默认的注册表是Docker Hub，它是我们使用的所有图像的来源。

> Docker ID  
> Docker ID允许您访问Docker Hub，这是世界上最大的容器映像库和社区。如果没有的话，请免费创建一个Docker ID。

### 创建仓库
推送映像，我们首先需要在Docker Hub上创建一个存储库。
* [注册](http://hub.docker.com/sso/start) 或登录到[Docker Hub](https://hub.docker.com/) 。
* 单击Create Repository按钮。
* 对于仓库名称，使用getting-started。确保能见度是公开的。

> 私人仓库  
> 
> 您知道Docker提供了私有存储库，允许您将内容限制到特定的用户或团队吗?查看Docker定价页面的详细信息。

### 点击创建按钮!
如果您查看页面的右侧，您将看到一个名为Docker命令的部分。这给出了一个示例命令，您将需要运行该命令来推送到这个repo。
```shell
docker push docker/getting-started:tagname
```

### 推送镜像
在命令行中，尝试运行您在Docker Hub上看到的push命令。注意，您的命令将使用您的名称空间，而不是“docker”。
```shell
docker push docker/getting-started
The push refers to repository [docker.io/docker/getting-started]
An image does not exist locally with the tag: docker/getting-started
```
为什么会失败呢?push命令正在寻找一个名为docker/getting-started的图像，但是没有找到。如果您运行docker image ls，您也不会看到一个。
为了解决这个问题，我们需要“标记”我们已经构建的现有图像，以赋予它另一个名称。

#### 使用命令`docker Login -u YOUR-USER-NAME`登录到Docker Hub。
```shell
docker Login -u YOUR-USER-NAME
Login Succeeded
```
#### 使用docker标记命令给开始映像一个新名称。请确保用Docker ID交换your - user - name。
```shell
docker tag getting-started YOUR-USER-NAME/getting-started
```

[comment]: <> (docker tag getting-started fredomli/getting-started)

现在再试一次push命令。如果从Docker Hub复制值，可以删除tagname部分，因为我们没有向图像名称添加标记。如果没有指定标记，Docker将使用名为latest的标记。
```shell
docker push YOUR-USER-NAME/getting-started
```

[comment]: <> (docker push fredomli/getting-started)

### 在新实例上运行映像
现在，我们已经构建了映像并将其推入注册表，让我们尝试在一个从未见过这个容器映像的全新实例上运行应用程序!为此，我们将使用Play with Docker。
* 打开你的浏览器来玩Docker。
* 单击Login，然后从下拉列表中选择docker。
* 连接您的Docker Hub帐户。
* 登录后，单击左侧栏上的ADD NEW INSTANCE选项。如果你看不到，把你的浏览器调宽一点。几秒钟后，在浏览器中打开一个终端窗口。

在终端中，启动你刚推送的应用程序。

```shell
docker run -dp 3000:3000 YOUR-USER-NAME/getting-started
```
您应该看到图像被拉下并最终启动!

点击3000徽章时，它出现，你应该看到应用程序与您的修改!万岁!如果3000的徽章没有出现，你可以点击“打开端口”按钮，输入3000。

[comment]: <> (docker run -dp 3000:3000 fredomli/getting-started)
使用虚拟机直接运行docker镜像：

![pc8](https://gitee.com/fredomli/fredomli-picture/raw/picgo/static/images/wordpress/docker-run-fredomli-getting-started-new-os.png)

```shell
查看防火墙状态
firewall-cmd --state

重新加载防火墙配置
firewall-cmd --reload

查看开放的端口信息
firewall-cmd [-zone=public] -- list-all

添加开放端口
firewall-cmd [-zone=public] --add-port=3000/tcp --permanent

删除开发端口
firewall-cmd [-zone=public] --remove-port=3000/tcp --permanent
```

[comment]: <> (firewall-cmd --zone=public --add-port=3000/tcp --permanent)

## 参考
* [Docker官网地址](https://www.docker.com/)
* [Docker官网文档](https://docs.docker.com/)
* [基于Java的Docker配置](https://github.com/fredomli/java-standard/blob/main/docs/service/docker/基于Java的Docker配置.md)
* [基于IDEA和Docker的SpringBoot项目容器化部署](https://github.com/fredomli/java-standard/blob/main/docs/service/docker/基于IDEA和Docker的SpringBoot项目容器化.md)


*(☞ﾟヮﾟ)☞[返回首页README.md address](https://github.com/fredomli/java-standard)*
