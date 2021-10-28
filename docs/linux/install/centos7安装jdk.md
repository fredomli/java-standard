# Centos7 安装 JDK

检测当前系统是否已经安装JDK。
```shell
java -version
```

## 下载JDK
JDK下载地址：[https://www.oracle.com/java/technologies/downloads/](https://www.oracle.com/java/technologies/downloads/)

## 使用Jar命令解压JDK源码
将JDK源码包上传到指定的系统中。使用jar命令进行解压，在移动到指定的位置。
```shell
jar -zxvf jdk-8u291-linux-x64.tar.gz
```

## 配置环境变量
配置环境变量可以设置系统环境变量或者用户环境变量，下面依次使用两种方式：
### 在当前非root用户下的.bashrc文件中配置（用户环境变量）
打开指定用户下的.bashrc文件。
```shell
vi /home/xxx/.bashrc
```

> xxx 为用户目录

在`.bashrc`中追加下列内容：
```shell
export JAVA_HOME=/home/xxx/java/jdk1.8.0_291

export CLASSPATH=.:$JAVA_HOME/jre/lib/rt.jar:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar

export PATH=$PATH:$JAVA_HOME/bin

```

编辑完成后保存退出，并使用source命令使文件生效。

```shell
source /home/xxx/.bashrc
```

检测是否安装成功：
```shell
javac

java

java -version
```

### 配置全局环境变量。

运行下列命令：
```shell
# /etc/profile 为全局变量文件
vi /etc/profile
```
在文件中追加下列类容，JAVA_HOME为安装目录地址。
```shell
export JAVA_HOME=/home/xxx/java/jdk1.8.0_291

export CLASSPATH=.:$JAVA_HOME/jre/lib/rt.jar:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar

export PATH=$PATH:$JAVA_HOME/bin

```

检测是否安装成功：
```shell
javac

java

java -version
```

## 参阅

*(☞ﾟヮﾟ)☞[返回首页README.md address](https://github.com/fredomli/java-standard)*
