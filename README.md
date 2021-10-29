# Java Standard
Java是目前用户最多、使用范围最广的软件开发技术，Java的技术体系主要由支撑程序运行的虚拟机、提供各开发领域接口支持的Java类库、Java编程语言及许许多多的第三方框架（Spring、Mybatis等等）构成。在国内，有关Java 类库API、Java语言语法以及第三方框架技术的技术资料都是很丰富的，看了很多的技术书籍资料等，对于Java语言和相关的技术和实现原理也有一个系统全面的了解，但是这些都是建立在理论上书本上的，自己或许有做过一些案例、实验甚至于一些项目，用到了很多相关的技术，但是没有系统的总结过所学习的技术和知识，永远只是注重实用性和目的性，忽略了很多技术上底层原理上的细节，对于我们的成长有害无利，让我们只会依赖技术实现功能，而不会创造技术。

接下来系统的总结自己的所学，深入了解技术的底层实现和背后的故事吧。  

## Java基础部分
#### 简介 
Java 编程语言是一种相对高级的语言，因为无法通过该语言获得机器表示的细节。它包括自动存储管理，通常使用垃圾收集器，以避免显式释放的安全问题（如在 Cfree 或 C++ 中delete）。高性能垃圾收集实现可以有有限的暂停来支持系统编程和实时应用程序。该语言不包含任何不安全的构造，例如没有索引检查的数组访问，因为这种不安全的构造会导致程序以未指定的方式运行。

关于项目结构查看(☞ﾟヮﾟ)☞[Java部分项目结构说明](https://github.com/fredomli/java-standard/blob/main/docs/java/core/readme.md)

### Java 基础知识
#### 1. [Java程序设计概述](https://github.com/fredomli/java-standard/blob/main/docs/java/core/basis/Java程序设计概述.md)
#### 2. [Java程序设计环境](https://github.com/fredomli/java-standard/blob/main/docs/java/core/basis/environment/Java程序设计环境.md)   
* [安装库源文件和文档](https://github.com/fredomli/java-standard/blob/main/docs/java/core/basis/environment/安装库源文件和文档.md)
* [使用命令行工具](https://github.com/fredomli/java-standard/blob/main/docs/java/core/basis/environment/使用命令行工具.md)
* [使用集群开发环境](https://github.com/fredomli/java-standard/blob/main/docs/java/core/basis/environment/使用集成开发环境.md)
* [JShell](https://github.com/fredomli/java-standard/blob/main/docs/java/core/basis/environment/JShell.md)
#### 3. [Java的基本程序设计结构](https://github.com/fredomli/java-standard/blob/main/docs/java/core/basis/simple/Java的基本程序设计结构.md)

#### 4. [Java对象与类](https://github.com/fredomli/java-standard/blob/main/docs/java/core/basis/object/对象与类.md)

#### 5. [Java继承](https://github.com/fredomli/java-standard/blob/main/docs/java/core/basis/extends/Java继承.md)
### Java 核心技术（高级特性）

### Java 虚拟机
``` 
注释： 以下内容摘自Java虚拟机规范  

Java 虚拟机是 Java 平台的基石。它是技术的组成部分，负责其硬件和操作系统的独立性、编译代码的
小尺寸以及保护用户免受恶意程序侵害的能力。

Java 虚拟机是一个抽象的计算机器。像真正的计算机器一样，它有一个指令集并在运行时操纵各种内存区域。
使用虚拟机实现编程语言是相当普遍的。最著名的虚拟机可能是 UCSD Pascal 的 P-Code 机器。

Java 虚拟机的第一个原型实现是在 Sun Microsystems, Inc. 完成的，它模拟了由类似于当代个人数字
助理 (PDA) 的手持设备托管的软件中的Java 虚拟机指令集。Oracle 当前的实现在移动、桌面和服务器设备
上模拟 Java 虚拟机，但 Java 虚拟机不假定任何特定的实现技术、主机硬件或主机操作系统。它不是固有的解释，
但也可以通过将其指令集编译为硅 CPU 的指令集来实现。它也可以在微代码中或直接在硅中实现。

Java 虚拟机对 Java 编程语言一无所知，只知道一种特定的二进制格式，即class文件格式。一个class文件
包含的Java虚拟机指令（或字节码）和符号表，以及其它辅助信息。

为了安全起见，Java 虚拟机对class文件中的代码施加了很强的语法和结构约束。但是，任何具有可以用有效
class文件表示的功能的语言都可以由Java虚拟机托管。被普遍可用的、独立于机器的平台所吸引，其他语言的
实现者可以将 Java 虚拟机作为他们语言的交付工具。
```

*更多详细信息可以参考* [虚拟机规范](https://docs.oracle.com/javase/specs/jvms/se8/html/jvms-1.html#jvms-1.1) 

* [深入理解Java虚拟机](https://github.com/fredomli/java-standard/blob/main/docs/java/jvm/readme.md)
* [Java内存区域与内存溢出异常](https://github.com/fredomli/java-standard/blob/main/docs/java/jvm/memory/Java内存区域与内存溢出异常.md)

### Java 并发编程

### Java 面向对象编程

### Java 设计模式

### Java版数据结构与算法
> **算法和数据结构**的学习是所有计算机学科教学计划的基础，但它并不只是对程序员和计算机系的学生有用， 
> 任何计算机使用者都希望计算机运行得更快一些或是能解决更大规模的问题。参考
(☞ﾟヮﾟ)☞ [数据结构和算法](https://github.com/fredomli/java-standard/blob/main/docs/java/algorithm/readme.md)  

* [数据结构的基本概念](https://github.com/fredomli/java-standard/blob/main/docs/java/structure/description/数据结构的基本概念.md)


算法是问题的解决方案，这个解决方案本身并不是问题的答案，而是能获得答案的指令序列。对于许多实际的问题，写出一个正确的算法还不够，如果这个算法在规模较大的数据集上运行，那么运行效率就成为一个重要的问题。 在选择和设计算法时要有效率的概念，这一点比提高计算机本身的速度更为重要（比如排序问题）。
(☞ﾟヮﾟ)☞ [算法实例](https://github.com/fredomli/java-standard/blob/main/docs/java/algorithm/readme.md)
______

## Java Web
### 简介
Java Web，是用Java技术来解决相关Web互联网领域的技术栈。Web包括: web服务器和web客户端两部分。Java在客户端还有
Java Applet,不过现在已经很少使用，JavaScript完全取代了它的作用。Java在服务器端的应用非常丰富，比如Servlet，JSP、等第三方框架等等。


## Java 企业开发（Java EE）
### 简介
Java EE 是 J2EE的一个新的名称，之所以改名，目的还是让大家清楚J2EE只是Java企业应用。在2004年底中国软件技术大会Ioc微容器（也就是Jdon框架的实现原理）演讲中指出：我们需要一个跨J2SE/WEB/EJB的微容器，保护我们的业务核心组件（中间件），以延续它的生命力，而不是依赖J2SE/J2EE版本。

这是比较官方的解释，可以从不同Java应用方向相互比较，Java SE > Java Web > Java EE。 
### Spring
* [使用SpringInitializr创建项目.md](https://github.com/fredomli/java-standard/blob/main/docs/spring/spring/使用SpringInitializr创建项目.md)
* [Spring框架简介](https://github.com/fredomli/java-standard/blob/main/docs/spring/spring/readme.md)
* [Spring Framework概述](https://github.com/fredomli/java-standard/blob/main/docs/spring/spring/readme.md)
* [Spring核心技术](https://github.com/fredomli/java-standard/blob/main/docs/spring/spring/core/SpringIoc容器.md)
* [SpringIoc简介](https://github.com/fredomli/java-standard/blob/main/docs/spring/spring/core/SpringIoc容器.md)
* [Spring容器使用](https://github.com/fredomli/java-standard/blob/main/docs/spring/spring/core/Spring容器使用.md)
* [Spring依赖注入](https://github.com/fredomli/java-standard/blob/main/docs/spring/spring/core/Spring依赖注入概述.md)
* [SpringBean作用域](https://github.com/fredomli/java-standard/blob/main/docs/spring/spring/core/SpringBean作用域.md)
* [SpringBean-继承](https://github.com/fredomli/java-standard/blob/main/docs/spring/spring/core/SpringBean定义继承.md)
* [SpringBean自定义属性](https://github.com/fredomli/java-standard/blob/main/docs/spring/spring/core/SpringBean自定义性质.md)
* [Spring基于注解的容器配置](https://github.com/fredomli/java-standard/blob/main/docs/spring/spring/core/Spring基于注解的容器配置.md)
* [Spring基于Java的容器配置](https://github.com/fredomli/java-standard/blob/main/docs/spring/spring/core/Spring基于java的容器配置.md)
* [Spring Environment Abstraction](https://github.com/fredomli/java-standard/blob/main/docs/spring/spring/core/Spring环境抽象.md)


### Spring Security 
* [Spring Security的使用](https://github.com/fredomli/java-standard/blob/main/docs/spring/security/readme.md)


### Mybatis
#### Mybatis Generator
* [Mybatis Generator的使用](https://github.com/fredomli/java-standard/blob/main/docs/mybatis/generator/readme.md)

### Mybatis-Plus  
#### Mybatis-Plus AutoGenerator  
* [Mybatis-Plus AutoGenerator的使用](https://github.com/fredomli/java-standard/blob/main/docs/mybatis/mybatisplus/generator/readme.md)

## 技术栈

* Java
* Git
* Markdown
* IDEA

## 参考资料
* [Markdown语法基本使用](https://github.com/fredomli/java-standard/blob/main/docs/markdown/markdown.md)
* [Git-分布式版本控制](https://git-scm.com/doc)
* [集成开发环境-IntelliJ IDEA](https://www.jetbrains.com/)
* [Java语言和虚拟机规范](https://docs.oracle.com/javase/specs/index.html)  
* [如何搭建一个简单的图床工具](https://github.com/fredomli/java-standard/blob/main/docs/utils/使用gitee和picgo搭建图床.md)  
* [如何选择自己需要版本的资源](https://github.com/fredomli/java-standard/blob/main/docs/utils/关于版本术语解释.md)  
* [Spring参考资料](https://github.com/fredomli/java-standard/blob/main/docs/spring/readme.md)  
* [Spring官网](https://spring.io/)
* [Git官方文档](https://git-scm.com/doc)
* [VM-虚拟机安装](https://github.com/fredomli/java-standard/blob/main/docs/vm/虚拟机安装.md)
* [Centos镜像下载](https://github.com/fredomli/java-standard/blob/main/docs/vm/Centos系统镜像下载.md)
* [使用VMware和Centos7创建虚拟机](https://github.com/fredomli/java-standard/blob/main/docs/vm/使用VMware和Centos7创建虚拟机.md)
* [XShell工具非商业版本](https://github.com/fredomli/java-standard/blob/main/docs/utils/shell/XShell工具学生版.md)
* [Linux安装和介绍](https://github.com/fredomli/java-standard/blob/main/docs/linux/command/Linux安装和介绍.md)
* [Linux常用命令](https://github.com/fredomli/java-standard/blob/main/docs/linux/command/Linux常用命令.md)
* [linux下软件的安装与管理](https://github.com/fredomli/java-standard/blob/main/docs/linux/command/linux下软件的安装与管理.md) 
* [Linux命令大全](https://fredomli-oss.oss-cn-chengdu.aliyuncs.com/picture/Linux%E5%91%BD%E4%BB%A4%E5%A4%A7%E5%85%A8.pdf)
* [Docker基础学习](https://github.com/fredomli/java-standard/blob/main/docs/service/docker/学习Docker基础.md)
* [基于Java的Docker配置](https://github.com/fredomli/java-standard/blob/main/docs/service/docker/基于Java的Docker配置.md)
* [基于IDEA和Docker的SpringBoot项目容器化部署](https://github.com/fredomli/java-standard/blob/main/docs/service/docker/基于IDEA和Docker的SpringBoot项目容器化.md)
* [centos7安装nginx](https://github.com/fredomli/java-standard/blob/main/docs/linux/install/centos7安装nginx.md)
* [centos7安装apache](https://github.com/fredomli/java-standard/blob/main/docs/linux/install/centos7安装apache.md)
* [centos7安装docker](https://github.com/fredomli/java-standard/blob/main/docs/linux/install/centos7安装docker.md)
* [centos7安装jdk](https://github.com/fredomli/java-standard/blob/main/docs/linux/install/centos7安装jdk.md)
* [centos7安装maven](https://github.com/fredomli/java-standard/blob/main/docs/linux/install/centos7安装maven.md)
* [centos7安装mysql](https://github.com/fredomli/java-standard/blob/main/docs/linux/install/centos7安装mysql.md)
* [centos7安装redis](https://github.com/fredomli/java-standard/blob/main/docs/linux/install/centos7安装redis.md)
* [centos7安装tomcat](https://github.com/fredomli/java-standard/blob/main/docs/linux/install/centos7安装tomcat.md)
* [将应用添加为Linux的服务](https://github.com/fredomli/java-standard/blob/main/docs/linux/install/将应用添加为Linux的服务.md)



## 参考书籍
```markdown
 ・Java编程思想
 ・Java核心技术卷一
 ・Java核心技术卷二
 ・深入理解Java虚拟机
 ・大话设计模式
 ・计算机网络（自顶向下方法）
 ・现代操作系统（第四版）
 ・Java数据结构与算法分析（Java语言描述）
 ・算法（第四版）
 ・Java并发编程实战
```
