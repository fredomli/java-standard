# Centos7 安装 Elasticsearch

### 下载 elasticsearch
在官网下载源码安装包：
* elasticsearch官网地址：[https://www.elastic.co/cn/](https://www.elastic.co/cn/)
* elasticsearch下载地址：[https://www.elastic.co/cn/downloads/elasticsearch](https://www.elastic.co/cn/downloads/elasticsearch)

使用wget在线下载：
```shell
wget https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-7.15.1-linux-x86_64.tar.gz
```

### 解压安装文件

```shell
tar -zxvf elasticsearch-7.15.1-linux-x86_64.tar.gz
```
将解压缩的文件剪切到/usr/local/目录下:

```shell
mv elasticsearch-7.15.1 /usr/local/
```

### 配置
进入config文件夹下编辑elasticsearch.yml，如下列所示：

* 集群模式下,放开cluster.name注释,单机模式下,放开node.name。
* 数据存储和日志存储路径放开注释
* 网络设置 设置ip限制,端口设置,跨越设置i

### 启动
配置完成后进入bin目录执行启动脚本elasticsearch

前台启动：
```shell
./elasticsearch
```

后台启动：
```shell
./elasticsearch  -d
```

## 参阅
wget安装：
```shell
yum -y install wget
```

*(☞ﾟヮﾟ)☞[返回首页README.md address](https://github.com/fredomli/java-standard)*
