# 编写Dockerfiles的最佳实践

本文介绍了构建高效映像的推荐最佳实践和方法。

Docker通过从Dockerfile中读取指令来自动构建映像——Dockerfile是一个文本文件，其中包含构建给定映像所需的所有命令。Dockerfile遵循特定的格式和一组说明，你可以在Dockerfile参考中找到这些说明。

一个Docker映像由只读层组成，每个层代表一个Dockerfile指令。这些层是堆叠的，每一层都是前一层变化的增量。考虑一下这个Dockerfile:

```docker
# syntax=docker/dockerfile:1
FROM ubuntu:18.04
COPY . /app
RUN make /app
CMD python /app/app.py
```
每条指令创建一个层:
* 从ubuntu:18.04 Docker映像创建一个镜像层。
* COPY从Docker客户端当前目录添加文件。
* RUN使用make构建应用程序。
* CMD指定在容器中运行什么命令。

当您运行一个镜像并生成一个容器时，您将在底层之上添加一个新的可写层(“容器层”)。对正在运行的容器所做的所有更改，如写入新文件、修改现有文件和删除文件，都被写入这个可写容器层。

## 一般指引及建议
Dockerfile定义的映像应该生成尽可能短暂的容器。通过“短暂”，我们的意思是容器可以停止和破坏，然后重建和替换绝对最小的设置和配置。
参考The Twelve-factor App方法论中的Processes，了解以这种无状态方式运行容器的动机。

## 理解构建语义
当您发出docker构建命令时，当前工作目录称为构建上下文。默认情况下，假定Dockerfile位于这里，但是您可以使用文件标志(-f)指定一个不同的位置。无论Dockerfile实际位于何处，当前目录中所有文件和目录的递归内容都会作为构建上下文发送给Docker守护进程。

> 构建环境的例子:  
> 
>     mkdir myproject && cd myproject
>     echo "hello" > hello
>     echo -e "FROM busybox\nCOPY /hello /\nRUN cat /hello" > Dockerfile
>     docker build -t helloapp:v1 .
> 将Dockerfile和hello移到单独的目录中，构建映像的第二个版本(不依赖于上次构建的缓存)。使用-f指向Dockerfile并指定构建上下文的目录:
> 
>     mkdir -p dockerfiles context
>     mv Dockerfile dockerfiles && mv hello context
>     docker build --no-cache -t helloapp:v2 -f dockerfiles/Dockerfile context

无意中包含构建图像不需要的文件会导致更大的构建上下文和更大的图像大小。这可能会增加构建映像的时间、拉出和推送映像的时间，以及容器运行时大小。要想知道你的构建上下文有多大，在构建Dockerfile时可以看到这样一条信息:

```docker
Sending build context to Docker daemon  187.8MB
```
下面的示例使用通过stdin传递的Dockerfile构建映像。不会将任何文件作为构建上下文发送到守护进程。

```shell
docker build -t myimage:latest -<<EOF
FROM busybox
RUN echo "hello world"
EOF
```
在Dockerfile不需要将文件复制到映像的情况下，省略构建上下文可能很有用，并且可以提高构建速度，因为不会向守护进程发送任何文件。
如果你想通过排除构建上下文中的一些文件来提高构建速度，请使用.dockerignore引用exclude

注意:如果使用此语法，尝试构建使用COPY或ADD的Dockerfile将会失败。下面的例子说明了这一点:

```shell
# create a directory to work in
mkdir example
cd example

# create an example file
touch somefile.txt

docker build -t myimage:latest -<<EOF
FROM busybox
COPY somefile.txt ./
RUN cat /somefile.txt
EOF

# observe that the build fails
...
Step 2/3 : COPY somefile.txt ./
COPY failed: stat /var/lib/docker/tmp/docker-builder249218248/somefile.txt: no such file or directory
```

下面的示例使用当前目录(.)作为构建上下文，并使用Dockerfile构建映像，该Dockerfile使用here文档通过stdin传递。

```shell
# create a directory to work in
mkdir example
cd example

# create an example file
touch somefile.txt

# build an image using the current directory as context, and a Dockerfile passed through stdin
docker build -t myimage:latest -f- . <<EOF
FROM busybox
COPY somefile.txt ./
RUN cat /somefile.txt
EOF
```



## 参考
* [Docker官网地址](https://www.docker.com/)
* [Docker官网文档](https://docs.docker.com/)

*(☞ﾟヮﾟ)☞[返回首页README.md address](https://github.com/fredomli/java-standard)*

