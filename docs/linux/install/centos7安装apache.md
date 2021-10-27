# Centos7 安装 Apache

## tar 安装
先检查linux中是否已经安装了Apache，如下所示：
```shell
ps -ef | grep httpd
```
查看linux系统中有没有httpd服务。
```shell
chkconfig --list
```

### 下载apache
在官网下载自己需要的安装包。
* apache官网地址：[http://httpd.apache.org/](http://httpd.apache.org/)
* apache下载地址：[http://httpd.apache.org/download.cgi](http://httpd.apache.org/download.cgi)

如下图所示：

![pc1](https://fredomli-oss.oss-cn-chengdu.aliyuncs.com/picture/linux-install-apache-homepage.png)
![pc2](https://fredomli-oss.oss-cn-chengdu.aliyuncs.com/picture/linux-install-apache-homepage.png)

下载地址：[https://dlcdn.apache.org//httpd/httpd-2.4.51.tar.gz](https://dlcdn.apache.org//httpd/httpd-2.4.51.tar.gz)

### 解压文件

```shell
[root@localhost download]# tar -zxvf httpd-2.4.51.tar.gz
[root@localhost download]# ls -al
drwxr-xr-x. 12      504 games    4096 10月  7 21:12 httpd-2.4.51
```
使用cd命令进入解压后的目录，可以使用mv命令移动下载后的源码包。将文件移动到自己需要安装的位置，这个文件只是源文件，并不是安装文件位置。
```shell
[root@localhost download]# mv httpd-2.4.51 /xxx/xxx
```


### 编译安装

编译并指定安装目录：  
```shell
#./configure是用来检测你的安装平台的目标特征的
# /usr/local/apache2是安装目录
./configure --prefix=/usr/local/apache2 

# make是用来编译的，它从Makefile中读取指令，然后编译
# make install是用来安装的，它也从Makefile中读取指令，安装到指定的位置。
make && make install
```

> 注意：下面的错误，表示缺少APR依赖
> ```shell
> [root@localhost httpd-2.4.51]# ./configure --prefix=/usr/local/apache2
> checking for chosen layout... Apache
> checking for working mkdir -p... yes
> checking for grep that handles long lines and -e... /usr/bin/grep
> checking for egrep... /usr/bin/grep -E
> checking build system type... x86_64-pc-linux-gnu
> checking host system type... x86_64-pc-linux-gnu
> checking target system type... x86_64-pc-linux-gnu
> configure:
> configure: Configuring Apache Portable Runtime library...
> configure:
> checking for APR... no
> configure: error: APR not found.  Please read the documentation.
>
> ```

### apr (Apache Portable Runtime) && apr-util 安装
apr下载地址：[http://apr.apache.org/download.cgi](http://apr.apache.org/download.cgi)

apr-util下载地址同上，如下图所示。  

![pc3](https://fredomli-oss.oss-cn-chengdu.aliyuncs.com/picture/apr-and-aprutil-download.png)

将下载好的源码包上传到需要安装的指定系统。

### apr && apr-util 安装
使用tar命令进行解压并进入解压目录：

```shell
[root@localhost download]# tar -zxvf apr-1.7.0.tar.gz

[root@localhost download]# mv mv apr-1.7.0 /home/xxx/dvelopment/

[root@localhost download]# tar -zxvf apr-util-1.6.1.tar.gz

[root@localhost download]# mv apr-util-1.6.1 /home/xxx/dvelopment/
```

安装演示：
```shell
#./configure是用来检测你的安装平台的目标特征的
# /usr/local/apr是安装目录
./configure --prefix=/usr/local/apr 

# make是用来编译的，它从Makefile中读取指令，然后编译
# make install是用来安装的，它也从Makefile中读取指令，安装到指定的位置。
make && make install
```
```shell
#./configure是用来检测你的安装平台的目标特征的
# /usr/local/apr-util是安装目录
./configure --prefix=/usr/local/apr-util --with-apr=/usr/local/apr/bin/apr-1-config

# make是用来编译的，它从Makefile中读取指令，然后编译
# make install是用来安装的，它也从Makefile中读取指令，安装到指定的位置。
make && make install
```

如果在安装apr-util时出现下列错误：

![pc](https://fredomli-oss.oss-cn-chengdu.aliyuncs.com/picture/install-apche-httpd-apruitl-error.png)

这里的错误时缺少包造成的，这里我们安装这个expat依赖。解决方式如下：
```shell
yum install expat-devel -y
```

接下来继续执行安装apache的命令，注意在这个过程中出现任何缺少依赖的错误，根据提示安装即可。

第二次安装：
```shell
#./configure是用来检测你的安装平台的目标特征的
# /usr/local/apache2是安装目录
./configure --prefix=/usr/local/apache2 --with-apr=/usr/local/apr --with-apr-util=/usr/local/apr-util

# make是用来编译的，它从Makefile中读取指令，然后编译
# make install是用来安装的，它也从Makefile中读取指令，安装到指定的位置。
make && make install
```
在/usr/local/ 下可以看见安装目录如下：
```shell
[root@localhost local]# ls
apache2  apr  apr-util  bin  etc  games  include  lib  lib64  libexec  nginx  sbin  share  src
```
### 启动 && 关闭 && 重启
```shell
[root@localhost apache2]# cd /usr/local/apache2/bin/
[root@localhost bin]# ls
ab         apxs      dbmmanage  envvars-std  htcacheclean  htdigest  httpd      logresolve
apachectl  checkgid  envvars    fcgistarter  htdbm         htpasswd  httxt2dbm  rotatelogs
[root@localhost bin]# ./apachectl start
[root@localhost bin]# ./apachectl stop
[root@localhost bin]# ./apachectl restart
```

> 启动发生错误：
> ```shell
> AH00558: httpd: Could not reliably determine the server's fully qualified domain name, using localhost.localdomain. Set the 'ServerName' directive globally to suppress this message
> httpd (pid 39255) already running
> ```
> 解决方案如下，修改配置

### 修改配置
```shell
cd /usr/local/apache2/conf/
vi httpd.conf
```
```text
    188 #
    189 # ServerName gives the name and port that the server uses to identify itself.
    190 # This can often be determined automatically, but we recommend you specify
    191 # it explicitly to prevent problems during startup.
    192 #
    193 # If your host doesn't have a registered DNS name, enter its IP address here.
    194 #
    195 #ServerName www.example.com:80
    
```
将serverName 改为如下值:

```text
    188 #
    189 # ServerName gives the name and port that the server uses to identify itself.
    190 # This can often be determined automatically, but we recommend you specify
    191 # it explicitly to prevent problems during startup.
    192 #
    193 # If your host doesn't have a registered DNS name, enter its IP address here.
    194 #
    195 ServerName localhost:80
```
> 完成后重新启动服务，`./apachectl start`。

通过httpd.conf配置文件中主机号和端口号访问即可。默认端口80，效果如下图。

![https://fredomli-oss.oss-cn-chengdu.aliyuncs.com/picture/install-apache-httpd-success.png](https://fredomli-oss.oss-cn-chengdu.aliyuncs.com/picture/install-apache-httpd-success.png)

## yum 安装

使用yum安装Apache要简单得多，Apache在默认的CentOS仓库中可用，安装非常简单。 在CentOS和RHEL上，Apache软件包和服务称为httpd。 要安装软件包，请运行以下命令：
```shell
sudo yum -y install httpd
```
### 启用并启动httpd服务
```shell
sudo systemctl enable httpd
sudo systemctl start httpd
```
其他命令：
```shell
# 停止服务
sudo systemctl stop httpd

# 重启服务
sudo systemctl restart httpd

# 重新加载
sudo systemctl reload httpd

# 自动启动服务
sudo systemctl enable httpd

# 禁用自动启动服务
sudo systemctl disable httpd
```

### 打开防火墙
如果您正在运行防火墙，则还需要打开HTTP和HTTPS端口80和443：

```shell
sudo firewall-cmd --permanent --zone=public --add-service=http
sudo firewall-cmd --permanent --zone=public --add-service=https
sudo firewall-cmd --reload
```
### 查看服务状态
```shell
sudo systemctl status httpd
```
### 查看版本
```shell
httpd -v
```

## 参阅
[apache官方文档](https://httpd.apache.org/docs/)  
[将应用添加为Linux的服务](https://github.com/fredomli/java-standard/blob/main/docs/linux/install/将应用添加为Linux的服务.md)

*(☞ﾟヮﾟ)☞[返回首页README.md address](https://github.com/fredomli/java-standard)*
