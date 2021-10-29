# Centos7安装 Docker

### 卸载旧版本
```shell
yum remove docker \
docker-client \
docker-client-latest \
docker-common \
docker-latest \
docker-latest-logrotate \
docker-logrotate \
docker-selinux \
docker-engine-selinux \
docker-engine
```
如果 yum 报告未安装任何这些软件包，这表示情况正常。如下所示：
```text
已加载插件：fastestmirror
参数 docker 没有匹配
参数 docker-client 没有匹配
参数 docker-client-latest 没有匹配
参数 docker-common 没有匹配
参数 docker-latest 没有匹配
参数 docker-latest-logrotate 没有匹配
参数 docker-logrotate 没有匹配
参数 docker-selinux 没有匹配
参数 docker-engine-selinux 没有匹配
参数 docker-engine 没有匹配
不删除任何软件包

```

### 安装依赖包
```shell
yum install -y yum-utils device-mapper-persistent-data lvm2
```

### 添加阿里云地址为docker源地址
```shell
yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
```

### 安装docker
```shell
#从高到低列出Docker-ce的版本
yum list docker-ce.x86_64  --showduplicates | sort -r
```
指定安装版本：
```shell
# 例子
yum install docker-ce-18.09.9 docker-ce-cli-18.09.9 containerd.io
```
安装最新版本：
```shell
yum -y install docker-ce
```
### 启动docker
```shell
systemctl start docker
```

### 测试docker:

```shell
docker version
```

### 设置docker为开机自启动
```shell
systemctl enable docker
```

*(☞ﾟヮﾟ)☞[返回首页README.md address](https://github.com/fredomli/java-standard)*
