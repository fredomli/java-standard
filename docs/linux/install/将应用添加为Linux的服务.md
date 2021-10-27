# 将应用（Httpd,Nginx等）添加为Linux的服务
通过Jar解压源码的方式安装的软件，默认情况下对软件进行操作需要进入安装目录下，例如Httpd服务需要进入`bin`目录下执行`./apachectl start`命令进行启动。

所以我们需要进行手动配置，将服务添加为系统服务，这样就可以通过一些系统命令来进行管理。
## 将Httpd添加为Linux服务

### 第一步
查看一下/etc/init.d/下是否存在httpd这个服务。

```shell
ls /etc/init.d/ | grep httpd
```

### 第二步
将自己安装目录下的apachect1复制到该目录下并改为httpd
```shell
cp /usr/local/apache2/bin/apachectl /etc/init.d/httpd
```

### 第三步
打开/etc/init.d/httpd脚本，我们在里面需要添加两行注释。
```shell
# 打开httpd
vi /etc/init.d/httpd
```
chkconfig：这一行是此脚本中默认设置的服务识别参数，35代表在系统级别3、5中启动；启动和关闭顺序分别为85和21
description：这一行没有什么具体作用，只是服务描述信息。

![https://fredomli-oss.oss-cn-chengdu.aliyuncs.com/picture/shell-httpd-add-sysconfig.png](https://fredomli-oss.oss-cn-chengdu.aliyuncs.com/picture/shell-httpd-add-sysconfig.png)

### 第四步
执行：chkconfig --add httpd命令，将httpd服务添加为系统服务。
```shell
chkconfig --add httpd
```
345代表在系统级别345中启动，通过下列命令可以启动345代表的系统级别。
```shell
chkconfig httpd on
```

### 第五步
参看服务是否添加成功。
```shell
chkconfig --list httpd
```
效果如下所示：
```shell
[root@localhost init.d]# chkconfig --list httpd
httpd          	0:关	1:关	2:关	3:开	4:开	5:开	6:关
```

### 检查是否配置成功
执行 `systemctl status httpd` 命令查看httpd服务状态。可以看到服务添加是添加成功的。
运行 `systemctl start httpd` 命令启动服务。随后在查看状态可以发现已启动成功。
```shell
[root@localhost init.d]# systemctl status httpd
● httpd.service - SYSV: Start and stop the Apache HTTP Server
   Loaded: loaded (/etc/rc.d/init.d/httpd; bad; vendor preset: disabled)
   Active: inactive (dead)
     Docs: man:systemd-sysv-generator(8)
[root@localhost init.d]# systemctl start httpd
[root@localhost init.d]# systemctl status httpd
● httpd.service - SYSV: Start and stop the Apache HTTP Server
   Loaded: loaded (/etc/rc.d/init.d/httpd; bad; vendor preset: disabled)
   Active: active (exited) since 三 2021-10-27 20:09:33 CST; 2s ago
     Docs: man:systemd-sysv-generator(8)
  Process: 39901 ExecStart=/etc/rc.d/init.d/httpd start (code=exited, status=0/SUCCESS)

10月 27 20:09:33 localhost.localdomain systemd[1]: Starting SYSV: Start and stop the Apache HTT.....
10月 27 20:09:33 localhost.localdomain httpd[39901]: httpd (pid 39255) already running
10月 27 20:09:33 localhost.localdomain systemd[1]: Started SYSV: Start and stop the Apache HTTP...r.
Hint: Some lines were ellipsized, use -l to show in full.
```
如下图所示：

![https://fredomli-oss.oss-cn-chengdu.aliyuncs.com/picture/shell-httpd-add-sysconfig-success.png](https://fredomli-oss.oss-cn-chengdu.aliyuncs.com/picture/shell-httpd-add-sysconfig-success.png)

访问服务器：

![https://fredomli-oss.oss-cn-chengdu.aliyuncs.com/picture/install-apache-httpd-success.png](https://fredomli-oss.oss-cn-chengdu.aliyuncs.com/picture/install-apache-httpd-success.png)

停止服务：
```shell
system stop httpd
```

开机自启：
```shell
systemctl enable httpd
```

重新加载：
```shell
systemctl reload httpd
```

> 注意：对于应用程序，都具自己依赖或者开发的端口号，服务启动成功的情况下，对于开发的端口我们需要在防火墙（Firewall）配置。

## 将Nginx添加为Linux服务

### 创建nginx文件
在目录/etc/init.d 其实是 /etc/rc.d/init.d的符号链接，在/etc/init.d 目录下新建文件nginx。需要root权限。

```shell
vi /etc/init.d/nginx
```
### 脚本类容
```shell
#!/bin/bash
#
# chkconfig: - 85 15
# description: Nginx is a World Wide Web server.
# processname: nginx

nginx=/usr/local/nginx/sbin/nginx
conf=/usr/local/nginx/conf/nginx.conf
case $1 in
start)
echo -n "Starting Nginx"
$nginx -c $conf
echo " done"
;;
stop)
echo -n "Stopping Nginx"
killall -9 nginx
echo " done"
;;
test)
$nginx -t -c $conf
;;
reload)
echo -n "Reloading Nginx"
ps auxww | grep nginx | grep master | awk '{print $2}' | xargs kill -HUP
echo " done"
;;
restart)
$0 stop
$0 start
;;
show)
ps -aux|grep nginx
;;
*)
echo -n "Usage: $0 {start|restart|reload|stop|test|show}"
;;
esac
```

看下面的命令：
```shell
[root@localhost init.d]# ls -al
总用量 48
drwxr-xr-x.  2 root root    96 10月 27 20:46 .
drwxr-xr-x. 10 root root   127 10月 24 20:37 ..
-rw-r--r--.  1 root root 18281 5月  22 2020 functions
-rwxr-xr-x.  1 root root  3513 10月 27 20:05 httpd
-rwxr-xr-x.  1 root root  4569 5月  22 2020 netconsole
-rwxr-xr-x.  1 root root  7928 5月  22 2020 network
-rw-r--r--.  1 root root   576 10月 27 20:46 nginx
-rw-r--r--.  1 root root  1160 10月  2 2020 README
```
可以看出 httpd 服务的权限为 `-rwxr-xr-x.` 具有执行的权限，而nginx不具备，下面就为nginx添加可执行的权限。

### 授予该脚本可执行权限：

```shell
chmod +x /etc/init.d/nginx
```

效果如下：
```shell
[root@localhost init.d]# ls -al
总用量 48
drwxr-xr-x.  2 root root    96 10月 27 20:46 .
drwxr-xr-x. 10 root root   127 10月 24 20:37 ..
-rw-r--r--.  1 root root 18281 5月  22 2020 functions
-rwxr-xr-x.  1 root root  3513 10月 27 20:05 httpd
-rwxr-xr-x.  1 root root  4569 5月  22 2020 netconsole
-rwxr-xr-x.  1 root root  7928 5月  22 2020 network
-rwxr-xr-x.  1 root root   576 10月 27 20:46 nginx
-rw-r--r--.  1 root root  1160 10月  2 2020 README

```
将服务添加到系统服务：
```shell
chkconfig --add nginx
```
检查是否添加成功：
```shell
chkconfig --list nginx
```
设置开机启动：
```shell
chkconfig nginx on
```
### 启动 && 停止 等常用命令
```shell
# 启动命令
systemctl start nginx

# 查看状态
systemctl status nginx

# 停止命令
systemctl stop nginx

# 重启命令
systemctl restart nginx

# 重新加载配置
systemctl reload nginx

# 开机自启
systemctl enable nginx

# 关闭开机自己
systemctl disable nginx
```


## 参阅
[apache官方文档](https://httpd.apache.org/docs/)

*(☞ﾟヮﾟ)☞[返回首页README.md address](https://github.com/fredomli/java-standard)*
