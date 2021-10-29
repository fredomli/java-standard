# Centos7 安装 MySQL

### 下载maven
下载源码包并上传：
* maven官网地址：[http://maven.apache.org/](http://maven.apache.org/)
* maven下载地址：[https://maven.apache.org/download.cgi](https://maven.apache.org/download.cgi)

使用wget命令直接下载：
```shell
wget https://archive.apache.org/dist/maven/maven-3/3.6.3/binaries/apache-maven-3.6.3-bin.tar.gz
```
### 解压maven安装文件
```shell
tar -zxvf apache-maven-3.6.3-bin.tar.gz
```
移动到指定位置并重命名：
```shell
mv apache-maven-3.6.3 /usr/local/maven-3.6
```
### 配置环境变量
打开配置文件：
```shell
vi /etc/profile
```
追加下列内容：
```text
export MAVEN_HOME=/usr/local/maven-3.6/
export PATH=${PATH}:${MAVEN_HOME}/bin
```

使配置文件生效：
```shell
source /etc/profile
```

### 查看是否安装成功
```shell
mvn -v
```

看到你的maven版本信息就代表安装成功了,如果没成功,很大原因是你的环境变量写的不对,要细心一点

### 扩展
设置阿里私服，进入配置文件 `/usr/local/maven-3.6/conf/settings.xml`
```xml
<mirror>
    <id>alimaven</id>
    <name>aliyun maven</name>
    <url>http://maven.aliyun.com/nexus/content/repositories/central/</url>
    <mirrorOf>central</mirrorOf>
</mirror>
```


## 参阅
wget安装：
```shell
yum -y install wget
```

*(☞ﾟヮﾟ)☞[返回首页README.md address](https://github.com/fredomli/java-standard)*
