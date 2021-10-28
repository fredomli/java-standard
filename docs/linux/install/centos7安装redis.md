# Centos7 安装 Redis

### 下载redis

下载Redis的安装包：
* Redis官网地址：[https://redis.io/](https://redis.io/)
* Redis下载地址：[https://redis.io/download](https://redis.io/download)

通过 linux 系统的 wget 命令直接下载：

```shell
wget https://download.redis.io/releases/redis-6.2.6.tar.gz
```

### 解压安装包：
```shell
jar -zxvf redis-6.2.6.tar.gz
```

将解压后的文件，放到 /usr/local/ 目录下：
```shell
# 由于 redis 路径不存在，故为重命名
mv ~/download/redis-6.2.6 /usr/local/redis/
```

### 进入 redis 目录，对解压后的文件进行编译
```shell
cd /usr/local/redis/

# 对解压后的文件进行编译操作
sudo make
```
> 注意：如果解压文件不能进行编译和安装，则需要安装gcc环境，原因：由于redis是由C语言编写的，它的运行需要C环境，因此我们需要先安装gcc。
> 
>       yum install gcc-c++

执行完 make 命令后，redis 的 src 目录下会出现编译后的 redis 服务程序 redis-server，还有用于测试的客户端程序 redis-cli：

### 下面启动 redis 服务

```shell
cd /usr/local/redis/src
./redis-server
```
> 注意这种方式启动 redis 使用的是默认配置。也可以通过启动参数告诉 redis 使用指定配置文件使用下面命令启动。

```shell
cd src
./redis-server ../redis.conf
```
redis.conf 是一个默认的配置文件。我们可以根据需要使用自己的配置文件。

启动 redis 服务进程后，就可以使用测试客户端程序 redis-cli 和 redis 服务交互了。 比如：

```shell
# cd src
# ./redis-cli

redis> set foo bar
OK
redis> get foo
"bar"
```
## 参阅
wget安装：
```shell
yum -y install wget
```

*(☞ﾟヮﾟ)☞[返回首页README.md address](https://github.com/fredomli/java-standard)*
