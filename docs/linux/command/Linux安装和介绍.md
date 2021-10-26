# Linux安装和介绍

## linux 操作系统介绍
操作系统（Operating System，简称OS）是计算机程序。  
用于管理和控制计算机硬件与软件资源，是直接运行在“裸机”上的最基本的系统软件，任何其他软件都必须在操作系统的支持下才能运行

> 诞生于60年代末期的Bell实验室。UNIX与外界的首次接触是Unix的第一篇文章 “The UNIX Time Sharing System”，由Ken Thompson和Dennis Ritchie于1974年7月发表于 The Communications of the ACM。
早期各大学公司开始通过Unix源码对Unix进行了各种各样的改进和扩展。于是，Unix开始广泛流行。
20世纪70年代，AT&T公司开始注意到Unix所带来的商业价值。从1979年Unix的版本V7开始，Unix的许可证开始禁止大学使用Unix的源码，包括在授课中学习
>
> UNIX操作系统是商业版 ，需要收费，价格比Microsoft Windows正版要贵一些。
> 
> Unix开始收费之后，学校教学不能再免费使用Unix,编程爱好者在使用Unix时很容易吃上官司，于是促进了另一个类Unix操作系统——Linux的发展。
> 
> Linux的第一个内核编写于1991年10月5日，由Linus Torvlds于芬兰赫尔辛基大学发布。

### Linux 操作系统特征
* 多任务、多用户  
* 安全性：Unix提供了强大的安全保护机制，防止系统及其数据未经许可而被非法访问  
* 稳定性：Unix系统比较稳定，提供了非常强大的纠错能力，以保证系统的稳定运行  
* 强大的网络功能，是服务器的首选操作系统  
* 系统源代码用C语言编写，移植性好  
* 并行处理能力：Unix支持多处理器系统，允许多个处理器协调并行运行  
* 支持模块化结构，能适应许多的应用环境  
* 使用了层次化的文件系统，所有对象都是以文件的方式体现  
* 具有功能强大的shell，能完成许多复杂的操作  


## linux 操作系统的安装

### linux 的常见发行版本
redhat：advanced standard 5 ； Enterprise standard 5 ；workstation standard  
fedora： fedora 10  
Ubuntu：ubuntu 8.10  
OpenSUSE：opensuse 11.0  
redflag：redflag 7  
asianux: asianux 3.0  

### linux 的安装过程
1、两种安装模式，以及读取信息文件  
2、在时间选项中强调 UTC 时间和 GMT 时间  
3、root 等同 administrator  
4、定制安装包组，以及简述包之间的依赖关系  
5、安装完成之后的 gnome 和 KDE 界面  


## linux 操作系统的简单应用

### linux 的文本模式介绍

> [root@localhost ~]

* 第一列 root 代表当前用户
* 第二列 localhost 代表主机名
* 第三列~代表当前所在的目录 ~家目录 home 目录  

linux 的命令可以补全不全目录和文件名，如果不能补全双击 `tab` 键可以显示出要选择的命令

###  linux 的登陆与登出
* login 登入系统
* logout 登出系统
* exit 注销当前用户
* clear 清屏命令

###  linux 的关机

* shutdown 关机命令
* shutdown now 立即进入维护模式
* halt 直接关机
* shutdown -h now 立即关机
* shutdown -r now 立即重新启动计算机
* shutdown -h 20:00& 20:00 关闭计算机
* shutdown -r 20:00& 20:00 重新启动计算机
* shutdown -k 3 warning:system will shutdown! 只是发送消息给所以用户 3 分钟后进入维护模式
* shutdown +3 "system will shutdown after 3 minutes!" 发送消息给所以用户 3 分钟后进入系统维护模式


shutdown 命令演示：
```shell
[root@localhost ~] shutdown
Shutdown scheduled for 二 2021-10-26 16:33:28 CST, use 'shutdown -c' to cancel.
```
```shell
# 取消关机
[root@localhost ~] shutdown -c
```

###  linux 的 Init 进程
* \#0 停机(千万不能把 initdefault 设置为 0)
* \#1 单用户模式
* \#2 多用户，没有 NFS(和级别 3 相似，会停止部分服务)
* \#3 完全多用户模式
* \#4 没有用到
* \#5 x11(Xwindow)
* \#6 重新启动(千万不要把 initdefault 设置为 6)


### 查看 linux 系统信息  
* hostname 显示主机名
* hostname eduask 修改主机名为 eduask
* uname 显示系统及版本信息  
-a 显示系统及版本的所有信息  
-s 显示内核名称  
-n 显示网络节点名称（完整的计算机名称）  
-r 显示内核发行版本  
-v 显示内核版本信息  
-m 显示计算机类型  
-o 显示操作系统的类型  
--version 显示系统发行版本信息  
--help 系统命令的帮助信息和参数含义  

hostname演示：
```shell
[root@localhost ~] hostname
localhost.localdomain
```
uname演示：
```shell
[root@localhost ~] uname
Linux
```

```shell
[root@localhost ~] uname --version
uname (GNU coreutils) 8.22
Copyright (C) 2013 Free Software Foundation, Inc.
许可证：GPLv3+：GNU 通用公共许可证第3 版或更新版本<http://gnu.org/licenses/gpl.html>。
本软件是自由软件：您可以自由修改和重新发布它。
在法律范围内没有其他保证。

由David MacKenzie 编写。
```

```shell
[root@localhost ~] uname -r
3.10.0-1160.el7.x86_64
```


### linux 下查看用户信息

* whoami 显示当前用户
* who 当前系统所登陆的用户，以及所登录的控制台
* w 当前系统所登陆的用户，以及所登录的控制台的详细信息

演示：
```shell
[root@localhost ~] whoami
root
```

```shell
[root@localhost ~] who
root     pts/0        2021-10-26 16:43 (192.168.199.1)
```

```shell
[root@localhost ~] w
16:50:39 up 14 min,  1 user,  load average: 0.00, 0.01, 0.04
USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
root     pts/0    192.168.199.1    16:43    7.00s  0.06s  0.02s w
```

参阅：
> * [虚拟机安装](https://github.com/fredomli/java-standard/blob/main/docs/vm/虚拟机安装.md)
> * [Centos镜像下载](https://github.com/fredomli/java-standard/blob/main/docs/vm/Centos系统镜像下载.md)
> * [使用VMware和Centos7创建虚拟机](https://github.com/fredomli/java-standard/blob/main/docs/vm/使用VMware和Centos7创建虚拟机.md)
> * [linux下软件的安装与管理](https://github.com/fredomli/java-standard/blob/main/docs/linux/command/linux下软件的安装与管理.md)
> * [Linux命令大全](https://fredomli-oss.oss-cn-chengdu.aliyuncs.com/picture/Linux%E5%91%BD%E4%BB%A4%E5%A4%A7%E5%85%A8.pdf)

*(☞ﾟヮﾟ)☞[返回首页README.md address](https://github.com/fredomli/java-standard)*
