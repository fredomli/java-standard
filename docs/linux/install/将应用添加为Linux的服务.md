# 将应用（Httpd,Nginx等）添加为Linux的服务
通过Jar解压源码的方式安装的软件，默认情况下对软件进行操作需要进入安装目录下，例如Httpd服务需要进入`bin`目录下执行`./apachectl start`命令进行启动。

所以我们需要进行手动配置，将服务添加为系统服务，这样就可以通过一些系统命令来进行管理。

## 第一步
查看一下/etc/init.d/下是否存在httpd这个服务。

```shell
ls /etc/init.d/ | grep httpd
```

## 第二步
将自己安装目录下的apachect1复制到该目录下并改为httpd
```shell
cp /usr/local/apache2/bin/apachectl /etc/init.d/httpd
```

## 第三步
打开/etc/init.d/httpd脚本，我们在里面需要添加两行注释。
```shell
# 打开httpd
vi /etc/init.d/httpd
```
chkconfig：这一行是此脚本中默认设置的服务识别参数，35代表在系统级别3、5中启动；启动和关闭顺序分别为85和21
description：这一行没有什么具体作用，只是服务描述信息。

![https://fredomli-oss.oss-cn-chengdu.aliyuncs.com/picture/shell-httpd-add-sysconfig.png](https://fredomli-oss.oss-cn-chengdu.aliyuncs.com/picture/shell-httpd-add-sysconfig.png)

## 第四步
执行：chkconfig --add httpd命令，将httpd服务添加为系统服务。
```shell
chkconfig --add httpd
```
345代表在系统级别345中启动，通过下列命令可以启动345代表的系统级别。
```shell
chkconfig httpd on
```

## 第五步
参看服务是否添加成功。
```shell
chkconfig --list httpd
```
效果如下所示：
```shell
[root@localhost init.d]# chkconfig --list httpd
httpd          	0:关	1:关	2:关	3:开	4:开	5:开	6:关
```

## 检查是否配置成功
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

## 参阅
[apache官方文档](https://httpd.apache.org/docs/)

*(☞ﾟヮﾟ)☞[返回首页README.md address](https://github.com/fredomli/java-standard)*
