# Centos7 安装 Nginx

## 概述
nginx 是一个HTTP和反向代理服务器，一个邮件代理服务器，和一个通用的TCP/UDP代理服务器，最初由Igor Sysoev编写。很长一段时间以来，它一直在许多重载的俄罗斯网站上运行，包括Yandex, Mail。Ru, VK和漫步者。根据Netcraft的数据，nginx在2021年10月服务或代理了22.50%最繁忙的站点。以下是一些成功的例子:Dropbox、Netflix、Wordpress.com、FastMail.FM。

## 安装依赖
#### 1.下载gcc-c++
gcc是linux下的编译器。nginx是使用C语言开发的高性能的http服务/反向代理服务器。安装命令如下： 
```shell
yum -y install gcc-c++
```

#### 2.下载安装pcre(Perl Compatible Regular Expressions)是一个Perl库，包括perl兼容的正则表达式库，nginx的http模块使用pcre来解析正则表达式。

安装命令： 
```shell
yum install -y pcre pcre-devel
```
#### 3.zlib该库提供了很多种压缩和解压缩的方式，nginx使用zlib对http包的内容进行gzip。
安装命令：
```shell
yum install -y zlib zlib-devel
```
#### 4.openssl一个强大的安全套接字层密码库，囊括主要的密码算法、常用的密钥和证书封装管理功能及SSL协议，并提供丰富的应用程序供测试或其它目的使用。nginx不仅支持http协议，还支持https（即在ssl协议上传输http）。
安装命令：
```shell
yum install -y openssl openssl-devel
```

## 下载nginx源码包
通过下列地址了解nginx并下载相关版本的nginx安装包。  
* nginx官网地址：[http://nginx.org/](http://nginx.org/)
* nginx下载地址：[http://nginx.org/en/download.html](http://nginx.org/en/download.html)

选择自己需要的安装包版本，复制源码包地址即可，如下图所示：
![pc1](https://gitee.com/fredomli/fredomli-picture/raw/picgo/static/images/wordpress/nginx-download-pc.png)

下载命令：
```shell
wget http://nginx.org/download/nginx-1.12.0.tar.gz
```

## 解压源码包
解压命令如下：  

```shell
tar -zxvf nginx-1.12.0.tar.gz
```
使用`cd`命令进入解压后的目录，可以使用`mv`命令移动下载后的源码包。


## 配置编译参数(可以使用./configure --help查询详细参数)
```text
./configure \
　　　　--prefix=/usr/local/nginx \
　　　　--pid-path=/var/run/nginx/nginx.pid \
　　　　--lock-path=/var/lock/nginx.lock \
　　　　--error-log-path=/var/log/nginx/error.log \
　　　　--http-log-path=/var/log/nginx/access.log \
　　　　--with-http_gzip_static_module \
　　　　--http-client-body-temp-path=/var/temp/nginx/client \
　　　　--http-proxy-temp-path=/var/temp/nginx/proxy \
　　　　--http-fastcgi-temp-path=/var/temp/nginx/fastcgi \
　　　　--http-uwsgi-temp-path=/var/temp/nginx/uwsgi \
　　　　--http-scgi-temp-path=/var/temp/nginx/scgi
```
> 注意：  
> 这是手动配置方式，编译操作的参数值需要提前创建好。

直接默认参数创建，如下所示：
```shell
 ./configure

make && make install
```
可以进入/usr/local/nginx查看文件是否存在conf、sbin、html文件夹，若存在则安装成功。

## 运行
进入运行命令目录：
```shell
cd /usr/local/nginx/sbin/
```
启动：
```shell
 ./nginx
```

查看是否启动：
```shell
ps -ef | grep nginx
```
如果有master和worker两个进程证明启动成功.

启动指定配置文件：
```shell
 ./nginx -c 配置文件路径
 
# 如果不指定-c，Nginx在启动时默认加载conf/nginx.conf文件，此文件的地址也可以在编译安装nginx时指定./configure的参数(--conf-path= 指向配置文件(nginx.conf))
```

优雅停止Nginx
```shell
 ./nginx -s quit
```

强制停止
```shell
 ./nginx -s stop
```

重新加载配置文件
```shell
 ./nginx -s reload
```

## 停止
暴力停止：
```shell
kill -9 processId
```
快速停止：
```shell
cd /usr/local/nginx/sbin && ./nginx -s stop
```
> 此方式相当于先查出nginx进程id再使用kill命令强制杀掉进程

完整停止(建议使用):
```shell
cd /usr/local/nginx/sbin && ./nginx -s quit
```
此方式停止步骤是待nginx进程处理任务完毕进行停止

## 重启及重新加载配置
先停止再启动（建议使用）:
```shell
./nginx -s quit && ./nginx
```
重新加载配置文件:
```shell
./nginx -s reload
```
## 防火墙配置
防火墙配置（Centos7使用firewalld代替了原来的iptables。nginx的默认端口为80，可通过nginx.conf来配置）。

开启端口
```shell
firewall-cmd --zone=public --add-port=80/tcp --permanent
```

重启防火墙
```shell
firewall-cmd --reload
```

查询80端口是否打开
```shell
firewall-cmd --query-port=80/tcp
```

nginx安装成功，启动nginx，即可通过ip地址来访问nginx：
![pc2](https://gitee.com/fredomli/fredomli-picture/raw/picgo/static/images/wordpress/nginx-welcome-page.png)

## 参阅
wget安装：
```shell
yum -y install wget
```

*(☞ﾟヮﾟ)☞[返回首页README.md address](https://github.com/fredomli/java-standard)*
