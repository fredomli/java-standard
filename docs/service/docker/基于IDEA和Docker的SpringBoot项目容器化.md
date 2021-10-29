# 基于IDEA和Docker的SpringBoot容器化部署

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


## IDEA中Docker插件安装

![pc2](https://gitee.com/fredomli/fredomli-picture/raw/picgo/static/images/wordpress/install-docker-plugin.png)


## IDEA中Docker配置
选择Docker服务，这里我们Docker是直接在Window系统上安装的，Docker服务在本地，所以选择Window的选项就可以了。

选择之后，去焦状态下，会自动检查，如下所示：

![pc3](https://gitee.com/fredomli/fredomli-picture/raw/picgo/static/images/wordpress/idea-docker-config.png)

这时候退出应该就可以看见Docker服务窗口了，如果没有在View -> Tool Windows中打开。

## Add Configuration
配置一个Docker服务，如下图所示：

![pc4](https://gitee.com/fredomli/fredomli-picture/raw/picgo/static/images/wordpress/docker-add-configration.png)


## 创建DockerFile

```docker
# syntax=docker/dockerfile:1
FROM cantara/alpine-openjdk-jdk8
COPY /target/spring-boot-docker.jar .
ENTRYPOINT ["java","-jar","spring-boot-docker.jar"]
```

## 编译程序
使用maven package 命令对项目进行打包，打包完成后就可以在项目目录下查看打包后的文件，文件都保存在target目录下，包括我们需要的`.jar`文件.

和上面的DockerFile文件相互对应：
* FROM 是依赖 这里是依赖的jdk8
* COPY 这是复制命令，将我们的程序包，赋值到我们的镜像文件中
* ENTRYPOINT 启动命令。

## 启动程序
完成以上功能我们就可以，使用我们配置的`Add configuration`服务直接启动。

```text
Deploying 'boot-start Dockerfile: DockerFile'...
Building image...
Preparing build context archive...
[==================================================>]97/97 files
Done

Sending build context to Docker daemon...
[==================================================>] 31.05MB
Done

Step 1/3 : FROM cantara/alpine-openjdk-jdk8
 ---> 25a1ec607129
Step 2/3 : COPY /target/spring-boot-docker.jar .
 ---> c1cde3e51688
Step 3/3 : ENTRYPOINT ["java","-jar","spring-boot-docker.jar"]
 ---> Running in 7016d00a05bb
Removing intermediate container 7016d00a05bb
 ---> 9709a179344f

Successfully built 9709a179344f
Successfully tagged boot:latest
Creating container...
Container Id: c4163c78e861c4804a835e6b9c6840ac0c2ab561770da76f12e5a240affff47a
Container name: 'boot-start'
Starting container 'boot-start'
'boot-start Dockerfile: DockerFile' has been deployed successfully.
```

![pc5](https://gitee.com/fredomli/fredomli-picture/raw/picgo/static/images/wordpress/idea-config-docker-run-success.png)

## 推送到Docker镜像仓库
在Docker服务窗口，选中需要提交的镜像，右键push,配置镜像仓库地址，仓库名称，分支名称（远程可以没有，会创建新分支）。如下图所示：

![pc6](https://gitee.com/fredomli/fredomli-picture/raw/picgo/static/images/wordpress/idea-config-docker-push-reps.png)

在docker hub中查看：

![pc7](https://gitee.com/fredomli/fredomli-picture/raw/picgo/static/images/wordpress/idea-config-push-success.png)


## 在linux中运行
```shell
# 拉取镜像
docker pull fredomli/fredomli-reps:spring-boot

# 运行镜像，会自动下载镜像，前面一句可有可无 
docker run -dp 8080:8080 fredomli/fredomli-reps:spring-boot
```
记得要开放端口呀：
```shell
查看防火墙状态
firewall-cmd --state

重新加载防火墙配置
firewall-cmd --reload

查看开放的端口信息
firewall-cmd [-zone=public] -- list-all

添加开放端口
firewall-cmd [-zone=public] --add-port=8080/tcp --permanent

删除开发端口
firewall-cmd [-zone=public] --remove-port=8080/tcp --permanent
```

## 参考
* [Docker官网地址](https://www.docker.com/)
* [Docker官网文档](https://docs.docker.com/)
* [Docker基于Java的配置](https://docs.docker.com/language/java/build-images/)


*(☞ﾟヮﾟ)☞[返回首页README.md address](https://github.com/fredomli/java-standard)*
