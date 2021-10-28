# centos7 安装 Tomcat

## 下载Tomcat

* Tomcat官网地址：[http://tomcat.apache.org/](http://tomcat.apache.org/)
* Tomcat下载地址：[https://tomcat.apache.org/download-80.cgi](http://tomcat.apache.org/)

> 这里下载地址中Tomcat版本为8.5.x，需要指定版本在官网选择。

## 安装步骤
将源码包上传到指定的系统，并进行解压：

```shell
jar -zxvf apache-tomcat-8.5.72.tar.gz
```

解压的同时可以将解压文件移动到指定的位置，如下所示：

```shell
jar -zxvf apache-tomcat-8.5.72.tar.gz /usr/local/tomcat
```

在tomcat/bin目录下执行 startup.sh（注意防火墙）`./startup.sh`停止服务`./shutdown.sh`

> 注意在防火墙允许Tomcat的端口号通过。

开发8080端口，如下所示：
```shell
# 永久开放8080端口
firewall-cmd --zone=public --add-port=8080/tcp --permanent

# 重启防火墙
firewall-cmd --reload

# 参看指定端口状态
firewall-cmd --query-port=8080/tcp
```

效果如下：

![pc1](https://gitee.com/fredomli/fredomli-picture/raw/picgo/static/images/wordpress/install-tomcat-success.png)

## 参阅
wget安装：
```shell
yum -y install wget
```

*(☞ﾟヮﾟ)☞[返回首页README.md address](https://github.com/fredomli/java-standard)*
