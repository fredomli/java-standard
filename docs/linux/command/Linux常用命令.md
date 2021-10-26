# Linux常用命令

## 改变目录 `cd`
目录的表达方法  
* / 根目录  
* . 当前目录  
* .. 上一级目录  
* ~ 家目录  

cd命令：
* \#cd / 进入到系统根目录  
* \#cd . 进入当前目录  
* \#cd .. 进入当前目录的父目录，返回上层目录  
* \#cd /tmp 进入指定目录/tmp  
* \#cd ~ 进入当前用户的家目录  
* \#cd 进入当前用户的家目录  
* \#cd - 回到刚才所在的目录  

cd演示：  
```shell
[root@localhost ~]# cd /
[root@localhost /]#
[root@localhost /]# pwd
/
```
```shell
[root@localhost ~]# cd /home
[root@localhost home]#
[root@localhost /]# pwd
/home
```

## 显示当前所在目录 `pwd`
pwd 显示当前所在目录的路径，如下所示：  
```shell
[root@localhost ~]# cd /home
[root@localhost home]#
[root@localhost /]# pwd
/home
```

## 显示文件或目录的属性 `ls` （dir）

* \# dir 显示当前目录的内容（无颜色）  
* \# ls 显示当前目录的内容(有颜色)
* \# ls /tmp 显示指定目录/tmp 的内容
* \# ls -l 列出文件和文件夹的基本属性和详细信息
* \# ll 列出文件和文件夹的基本属性和详细信息
* \# ls -a 列出当前目录的全部内容，包括隐藏文件（在文件和文件夹前面加“.”隐藏）
* \# ls -l -a 列出当前目录的全部文件和文件夹的基本属性和详细信息
* \# ls -la 列出当前目录的全部文件和文件夹的基本属性和详细信息
* \# ll -a 列出当前目录的全部文件和文件夹的基本属性和详细信息
* \# ls -A 列出当前目录的全部内容，包括隐藏文件，不显示“.”和“..”
* \# ls --help 列出 ls 命令的帮助内容
* \# ls a2* 列出以 a2 开头的文件和文件夹
* \# ls -l a2* 列出以 a2 开头的文件和文件夹的基本属性和详细信息

ls 演示：
```shell
[root@localhost /]# ls
bin   dev  home  lib64  mnt  proc  run   srv  tmp  var
boot  etc  lib   media  opt  root  sbin  sys  usr
```
```shell
[root@localhost /]# ls -al
总用量 20
dr-xr-xr-x.  17 root root  224 10月 24 20:41 .
dr-xr-xr-x.  17 root root  224 10月 24 20:41 ..
lrwxrwxrwx.   1 root root    7 10月 24 20:36 bin -> usr/bin
dr-xr-xr-x.   5 root root 4096 10月 24 20:43 boot
drwxr-xr-x.  20 root root 3220 10月 26 16:36 dev
drwxr-xr-x.  75 root root 8192 10月 26 16:36 etc
drwxr-xr-x.   3 root root   22 10月 24 20:42 home
lrwxrwxrwx.   1 root root    7 10月 24 20:36 lib -> usr/lib
lrwxrwxrwx.   1 root root    9 10月 24 20:36 lib64 -> usr/lib64
drwxr-xr-x.   2 root root    6 4月  11 2018 media
drwxr-xr-x.   2 root root    6 4月  11 2018 mnt
drwxr-xr-x.   2 root root    6 4月  11 2018 opt
dr-xr-xr-x. 112 root root    0 10月 26 16:36 proc
dr-xr-x---.   2 root root  135 10月 24 22:20 root
drwxr-xr-x.  24 root root  720 10月 26 16:36 run
lrwxrwxrwx.   1 root root    8 10月 24 20:36 sbin -> usr/sbin
drwxr-xr-x.   2 root root    6 4月  11 2018 srv
dr-xr-xr-x.  13 root root    0 10月 26 16:36 sys
drwxrwxrwt.  15 root root 4096 10月 26 16:37 tmp
drwxr-xr-x.  13 root root  155 10月 24 20:36 usr
drwxr-xr-x.  19 root root  267 10月 24 20:44 var
```

> 文件和文件夹（蓝色代表目录， 白色代表文件，黄色代表设备文件，红色代表压缩文件，绿色代表 可执行文件，浅蓝色代表链接文件）linux 是以属性来控制文件是否能执行。

## 创建目录 `mkdir`  
* mkdir dir1 在当前目录下创建 dir 子目录
* mkdir /tmp/dir2 在指定目录/tmp 下创建 dir2 子目录
* mkdir -p dir3/dir4 在当前目录下创建 2 级目录 dir3 和其子目录 dir4
* mkdir -p /dir5/dir6 在根目录下创建 2 级目录 dir5 和其子目录 dir6
* mkdir dir7 dir8 dir9 在当前目录下创建 3 个目录 dir7 dir8 dir9，以空格隔开

mkdir 演示：  
```shell
[root@localhost /]# mkdir dir1
[root@localhost /]# ls -al
总用量 0
drwxr-xr-x. 3 root     root      18 10月 26 19:00 .
drwx------. 5 lishuang lishuang 108 10月 26 19:00 ..
drwxr-xr-x. 2 root     root       6 10月 26 19:00 dir1
```

## 创建空文本文件 `touch`

* \#touch file1 在当前目录下创建 file1 文件
* \#touch /tmp/file2 在指定目录/tmp 下创建 file2 文件

touch 演示：
```shell
[root@localhost /]# touch file1
[root@localhost /]# ls -al
drwxr-xr-x. 3 root     root      31 10月 26 19:03 .
drwxr-xr-x. 2 root     root       6 10月 26 19:00 dir1
-rw-r--r--. 1 root     root       0 10月 26 19:03 file1
```

## 复制文件命令 `cp`
* \# cp file2 /tmp 复制 file2 文件到/tmp 目录下
* \# cp /tmp/file2 /home 复制/tmp/file2 文件到/home 目录下
* \# cp /home/file2 /tmp/file3 复制/home/file2 到/tmp 目录下并改名为 file3
* \# cp -p /tmp/file3 /home 复制/tmp/file3 到/home 目录下并复制文件属性
* \# cp -r /dir5 /tmp 复制/dir5 目录到/tmp 下

## 移动文件或目录命令 `mv`
* \# mv file4 /tmp 移动 file4 文件到/tmp 目录下
* \# mv /home/file3 /tmp 移动/home/file3 文件到/tmp 目录下
* \# mv /home/file3 /tmp/file5 移动/home/file3 文件到/tmp 目录下并改名为 file5
* \# mv file3 file4 将 file3 改名为 file4
* \# mv dir10 /tmp 移动目录到/tmp 下
* \# mv dir10 dir11 讲 dir10 目录改名为 dir11

## 删除文件命令 `rm`
* \# rm file1 删除文件 file1
* \# rm -f file1 不用确认直接删除 file1
* \# rm -f file1 file2 file3 不用确认同时删除多个文件
* \# rm /tmp/file1 删除指定目录/tmp 下的文件 file1
* \# rm fi* 删除以 fi 开头的文件
* \# rmdir 删除空目录
* \# rm -r dir 递归的方式删除非空目录 dir
* \# rm -rf dir 不用确认直接删除非空目录 dir

## 查看文件内容命令 `cat`
* \# cat /etc/passwd 查看/etc/passwd 文件
* \# cat /etc/passwd |more 分屏查看文件内容
* \# cat /etc/passwd |less 分屏查看文件内容，可以上下翻页，“`q`”退出

## 查找文件命令 `find`
* \# find pass* 在当前目录下查找以 pass 开头的文件
* \# find /etc/pass* 在/etc 目录中查找以 pass 开头的文件
* \# find /etc/pass* -print 在/etc 目录中查找以 pass 开头的文件，并显示出来


##  在文件内容中查找关键字 `grep`
* \# grep “rpm” /etc/passwd 在/etc/passwd 文件中查找关键字 rpm

## `vi` 文本编辑器

### `vi` 的两种模式
1、命令模式 vi 的默认进入状态（不可以输入字符，但可以对字符进行操作，复制，移动、删除等操作）  
2、输入模式 输入字符状态（只可以输入和使用 del 和退格 backspace 键删除文字）

### `vi` 的启动和退出
* \#vi file 编辑 file 文件
* \#vi /tmp/file1 编辑指定目录/tem 下的 file1 文件
* :w 保存修改
* :q 退出 vi
* :wq 保存并退出
* :q! 强行退出 vi，不保存修改

### `vi` 命令模式下的操作
：set nu 设置行号  
：set nonu 取消设置行号

设置行号演示：  
```shell
[root@localhost test]# vi file1
      1 this is first row
      2 
      3 this is three row
      4 
INSERT
```

### 删除字符
* x 键或 del 键  
* 7x 删掉光标后面的 7 个字符  
* dw 删除一个词（剪切）  
* dd 删除行（剪切）  
* 4dd 删除 4 行（剪切）

### 复制操作
* yw 复制一个词 
* yy 复制光标所在的行
* 4yy 复制光标所在行的下面 4 行

### 粘贴操作
p 粘贴在光标所在的下一行（如果粘贴词的话，粘贴在光标字符的后面）

### 撤销操作
* u 撤销，可以撤销到最近的一次保存的状态
* ：e! 恢复到文档的初始状态

### 光标快速定位
* G 光标到达行末
* 7G 快速找到第 7 行
* /adm 简单搜索，快速定位光标到光标后的第一个 adm 单词的位置，当到行末没有的话，返回从头开始查找（类似于 word 的查找）

> 技巧  
> 让行号永久生效  
> 进入该用户的家目录，在目录下创建 1 个文件，“.vimrc”  
> 内容 :set nu  

### 替换内容
：7，12 s/:/? 把第 7-12 行中每一行的第一个：改成？
：7，12 s/:/?/g 把第 7-12 行中的：全部改成？

### 进入和退出输入模式
* i 在光标之前输入文字
* ESC 退出
* a 在光标之后输入文字
* A 在行尾插入文字
* o 光标下面插入 1 行空行
* O 在光标上面插入 1 行空行

参阅：
> * [虚拟机安装](https://github.com/fredomli/java-standard/blob/main/docs/vm/虚拟机安装.md)
> * [Centos镜像下载](https://github.com/fredomli/java-standard/blob/main/docs/vm/Centos系统镜像下载.md)
> * [使用VMware和Centos7创建虚拟机](https://github.com/fredomli/java-standard/blob/main/docs/vm/使用VMware和Centos7创建虚拟机.md)
> * [linux下软件的安装与管理](https://github.com/fredomli/java-standard/blob/main/docs/linux/command/linux下软件的安装与管理.md)
> * [Linux命令大全](https://fredomli-oss.oss-cn-chengdu.aliyuncs.com/picture/Linux%E5%91%BD%E4%BB%A4%E5%A4%A7%E5%85%A8.pdf)

*(☞ﾟヮﾟ)☞[返回首页README.md address](https://github.com/fredomli/java-standard)*
