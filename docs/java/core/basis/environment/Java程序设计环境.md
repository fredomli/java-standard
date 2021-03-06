# Java程序设计环境

#### 本节主要介绍如何安装Java开发工具包JDK以及如何编译和运行不同类型的程序：控制台程序、图形化应用程序以及applet。可以在终端窗口中键入命令来运行JDK工具。然而，很多程序员更喜欢使用集成开发环境。为此，稍后将介绍如何使用一个免费的开发环境编译和运行Java程序。  

## 安装Java开发工具包
Oracle公司为Linux、Mac OS、Solaris和Windows提供了最新、最完备的Java开发工具包版本。对于很多其他平台，也有处于不同开发阶段的JDK版本，不过，这些版本要由相应平台的开发商授权和分发。  

### 下载JDK
想要下载Java开发工具包，可以访问Oracle公司的网站: [https://www.oracle.com/java/technologies/javase-downloads.html](https://www.oracle.com/java/technologies/javase-downloads.html) 。在得到所需要的软件之前，必须弄清楚大量的专业术语。  

*JDK下载页面如下图所示：*  

![jdk download tille](https://fredomli-oss.oss-cn-chengdu.aliyuncs.com/picture/jdk-download-pic.png "JDK Download")

*Java术语：*  

[comment]: <>  (记得换行：这是一个注释)  

|术语名|缩写|解释|  
|----|------|------|
|Java Development Kit(Java 开发工具包)|JDK|编写Java程序的程序员使用的软件|  
|Java Runtime Environment(java运行时环境)|JRE|运行Java程序的用户使用的软件|  
|Server JRE(服务器 JRE)|——|在服务器上运行Java程序的软件|  
|Standard Edition(标准版)|SE|用于桌面或简单服务器应用的Java平台|  
|Enterprise(企业版)|EE|用于复杂服务器应用的Java平台|  
|Micro Edition(微型版)|ME|用于小型设备的Java平台|  
|Java FX|——|用于图形化用户界面的一个备选工具包，在Java11之前的某些Java SE发布版本中提供|  
|OpenJDK|——|Java SE的一个免费开源实现|  
|Java2|J2|一个过时的术语，用于描述1998~2006年之间的Java版本|  
|Software Development Kit(软件开发工具包)|SDK|一个过时的术语，用于描述1998~2006年之间的JDK|  
|Update|u|Oracle公司的术语，表示Java8之前的bug修正版本|  
|NetBeans|——|Oracle公司的集成开发环境|  


#### NetBeans集成开发环境
> [https://netbeans.apache.org/](https://netbeans.apache.org/)  
> netbeans 开发环境、工具平台和应用程序框架。

**一些无关紧要的话**
> 
> 你已经看到，JDK是Java Development Kit 的缩写。有点混乱的是：这个工具包的版本1.2~版本1.4被称为Java SDK。在某些场合下，还可以看到这个过时的术语。在开发Java 10 之前，还有一个术语是Java运行时环境（JRE），它只包含虚拟机。这并不是开发人员想要的。  
> 
> 在一些资料中，你会看到大量的Java SE，相对于Java EE 和 Java ME， Java SE是Java 标准版。  
> 
> Java  2 这种提法始于1998年，当时Sun公司的销售人员感觉以增加小数点后面数值的方式改变版本号并没有反映出JDK1.2的重大改进，但是，由于在发布之后才意识到这个问题，所以他们决定开发工具包的版本任然沿用1.2，接下来的版本是1.3、1.4、和5.0.不过，Java平台被重新命名为Java 2。因此，就有了Java 2 Standard Edition Software Development Kit(Java 2标准版软件开发包)5.0版，即J2SE SDK 5.0。  
> 
> 不过，“内部”版本号却分别是1.6.0、1.7.0和1.8.0。到了Java SE9，这种混乱终于终结，版本号变为9，以及后来的9.0.1。  
> 
>  *注意*： 在后面将省去缩写“SE”。如果你看到“Java 9”，这就表示 “Java SE 9”。  
> 
> 在Java 9 之前，有32位和64位两个版本的Java开发工具包。现在Oracle 公司不再开发32位版本。要使用Oracle JDK，你需要有一个64位的操作系统。  
> 
> 对于Linux，还可以在RPM文件和.tar.gz文件之间做出选择。我们建议使用后者，这样可以在你希望的任何位置直接解压缩这个压缩包。  

**现在已经了解了如何选择适当的JDK。**

* 你需要的是JDK (Java SE 开发工具包)，而不是JRE。
* 对于Linux，选择 .tar.gz 版本。

### 设置JDK  

#### 下载JDK之后，需要安装这个开发工具包并明确要安装在哪里，后面还会需要这个信息。  

* 在Windows上，启动安装程序，会询问你要在哪里安装JDK。最好不要接受路径名中包含 <span style="color:red;">空格</span> 的默认位置，如`c:\Program Files\Java\Jdk-11.0.x.x`。取出路径名中的Program Files部分就可以了。
* 在Mac上，运行安装程序。这会把软件安装到`/Library/Java/JavaVirtuaLMachines/jdk-11.0.x.jdk/Contents/Home`。用查找功能找到这个目录。
* 在Linux上，只需要把.tar.gz文件解压缩到你选择的某个位置，如你的主目录，或者/opt。如果从RPM文件安装，则要反复检查是否安装在/usr/java/jdk-11.0.x上。

> 在windows或Linux上安装JDK时，还需要另外完成一个步骤：将jdk/bin目录添加到可执行路径中——可执行路径是操作系统查找可执行文件时所遍历的目录列表。  

* 在Linux中，需要在~/.bashrc或~/.bash_profile文件的最后增加这样一行：
> export PATH=jdk/bin:$PATH  

* 在Windows 10中，在Windows Settings的搜索栏中键入<span style="color:green;">environment（环境）</span>选择Edit environment variables for your account(<span style="color:green;">编辑系统环境变量</span>)。会出现一个Environment Variables（环境变量）对话框。在User Variables（用户变量）列表中找到并选择一个名为Path的变量。点击Edit（编辑）按钮，再点击New按钮，增加一个变量，值为jdk\bin目录。  

保存所做的设置。之后新打开的所有命令提示窗口都会有正确的路径。  

**测试设置是否正确：**  
```shell
javac --version | javac -version
```
然后按回车键。应该能看到显示以下信息：  
```shell
javac 1.8.x_xxx
```
如果得到诸如"javac:command not found" (javac: 命令未找到)或 “The name specified is not recognized as an internal or external command, operable program or batch file” 
（指定名不是一个内部或外部命令、可执行的程序或批文件）的信息，就需要退回去反复检查你的安装。  


![https://fredomli-oss.oss-cn-chengdu.aliyuncs.com/picture/environment-java-jdk.png](https://fredomli-oss.oss-cn-chengdu.aliyuncs.com/picture/environment-java-jdk.png)
![https://fredomli-oss.oss-cn-chengdu.aliyuncs.com/picture/environment-config.png](https://fredomli-oss.oss-cn-chengdu.aliyuncs.com/picture/environment-config.png)
<span style="color:red;"></span>

## 扩展  
* [安装库源文件和文档](https://github.com/fredomli/java-standard/blob/main/docs/java/core/basis/environment/安装库源文件和文档.md)
* [使用命令行工具](https://github.com/fredomli/java-standard/blob/main/docs/java/core/basis/environment/使用命令行工具.md)
* [使用集群开发环境](https://github.com/fredomli/java-standard/blob/main/docs/java/core/basis/environment/使用集成开发环境.md)
* [JShell](https://github.com/fredomli/java-standard/blob/main/docs/java/core/basis/environment/JShell.md)


## 参考资料

* ***《Java核心技术卷一》***  
* [Java语言规范](https://docs.oracle.com/javase/specs/jls/se8/html/index.html)  
```
注意：  
1. 项目使用的Java版本为Java8，所选用的资料为Java8版本以上。
2. 实例源码目录：`java-standard\java-source\horstmann\corejava`
```


___________
*(☞ﾟヮﾟ)☞[返回首页README.md address](https://github.com/fredomli/java-standard)*
