# linux 下软件的安装与管理

## linux 下软件介绍
### rpm 包
> 红帽子包管理器(RPM)提供了标准化方式，可以对各种实用程序和应用程序组织所要的软件。红帽子包管理器使红帽子公司很容易地把 Linux 组织成不到两千个包，而不是几万个文件。 类似于 windows 的.exe 文件

### srpm 包
srpm 包为未编译过的 rpm 包，需要以 rpm 管理的方式编译，然后以 rpm 的安装方式安装。

### tar 包
压缩包，常见的有.tar.gz 和.tar.bz2，其中 gz 为使用 gzip 压缩的 tar 包，如“linuxqq_v1.0-
preview3_i386.tar.gz”最新的 QQ 版本，前面为文件名称，后面为文件的扩展名，我们可以看出是以
gzip 压缩的 tar 包；.tar.bz2 是以 bzip 压缩的 tar 包。

## rpm包

```text
rpm 包是依赖 cpu 架构的，常见的格式：  
扩展名  
CPU  
noarch.rpm  
不依赖于 CPU， 可以在所有计算机上安装  
i386.rpm  
基于 Intel 386 CPU，这些 RPM 包可以在所有 Intel 兼容计算机上安装    
i486.r pm  
用于带 Intel 486 CPU 的计算机(随时)  
i586.rpm  
用于带 Intel 586 CPU 的计算机  
i686.rpm  
用于带 Intel 686 CPU 的计算机  
ia64.rpm  
用于带 Intel ltanium 64 位 CPU 的计算机  
alpha.rpm  
用于带 HP Alpha CPU 的计算机，最初是 DEC 公司开发的  
nthlon.rpm  
基于 AMD Athlon CPU  
ppc.rpm  
用于带 Apple Powe rPC CPU 的计算机  
s390.rpm  
用于基于 S/390 CPU 的 IBM 服务器  
sparc.rpm  
用于带 Sun 系统公司 SPARC CPU 的计算机  
```

### rpm 软件包的查询

* rpm 命令:
* -q 对已安装的包进行简单查询
* rpm -q packagename（包的名称）
* rpm -qi packagename 对已安装的包进行详细信息查询
* rpm -ql packagename 查询已安装包中包含的文件
* rpm -qa 显示已经安装的所有 rpm 包
* rpm -qa |grep linux 显示已经安装的所有包含 linux 字段的包

### rpm 包的安装
* rpm -i packagename 安装包（在包所在的目录下）
* rpm -i /media/udisk/linux/linuxqq_v1.0-preview3_i386.rpm 安装指定目录下的包
* rpm -ivh packagename 安装包并显示安装的进度和详细信息
* -v 显示安装过程的详细处理过程
* -h 显示安装进度

### rpm 包的卸载
rpm -e packagename 卸载已安装的 rpm 包  
可以以空格隔开同时删除多个包

## srpm 包的安装
> 源代码 RPM 包的结尾通常是`.src.rpm`

使用方法:  
* rpm -i rpmpackage.src.rpm  
* cd /usr/src/redhat/SPECS  
* rpmbuild -bb rpmpackage.specs  
* /usr/src/redhat/RPM/i386/目录下，有一个新的 rpm 包，这个是编译好的二进制文件。  
* rpm -i new-package.rpm 即可安装完成。  

## tar 包软件的安装和卸载
tar 包为压缩包，常见的文件类型为`.tar.gz` `.tar.bz2` `.tgz .tar.zip`,在 linux 下安装方式为：   
1、先解压缩，各种文件类型的解压缩方式不同  
`.tar.gz` `.tgz` 文件执行 
* tar -xvzf softname.tar.gz
* tar -xvzf softname.tgz  
-x 解压缩文件  
-v 显示详细过程  
-z 支持 gzip 压缩文件  
-f 指定压缩文件  
* tar -xvjf softname.tar.bz2  
-j 支持 bzip2 压缩文件  
* unzip -v softname.tar.zip  
-v 解压文件  
-d 指定解压缩目录
  
2、在软件所在目录下会生成同名的目录，里面会存放着所有文件，进入到这个目录  

3、阅读 readme 文件或是 install 文件，查找执行配置，编译，安装命令方式

4、执行配置、编译和安装命令， 通常为：
* ./configure 执行配置
* make 编译
* make install 安装
* make clean 清理临时文件

5、tar 包的卸载
可以在安装目录下执行 `make uninstall` 也可以直接删除目录，文件分散多少个目录就删除多少个目录。
参阅：
> * [虚拟机安装](https://github.com/fredomli/java-standard/blob/main/docs/vm/虚拟机安装.md)
> * [Centos镜像下载](https://github.com/fredomli/java-standard/blob/main/docs/vm/Centos系统镜像下载.md)
> * [使用VMware和Centos7创建虚拟机](https://github.com/fredomli/java-standard/blob/main/docs/vm/使用VMware和Centos7创建虚拟机.md)
> * [Linux常用命令](https://github.com/fredomli/java-standard/blob/main/docs/linux/command/Linux常用命令.md)
> * [Linux命令大全](https://fredomli-oss.oss-cn-chengdu.aliyuncs.com/picture/Linux%E5%91%BD%E4%BB%A4%E5%A4%A7%E5%85%A8.pdf)


*(☞ﾟヮﾟ)☞[返回首页README.md address](https://github.com/fredomli/java-standard)*
