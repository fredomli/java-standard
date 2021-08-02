# Java的基本程序设计结构
> 现在假定你已经成功地安装JDK，并且能够运行“[Java程序设计环境](https://github.com/fredomli/java-standard/blob/main/docs/java/core/basis/environment/Java程序设计环境.md) ”的示例程序，下面将开始介绍程序设计。主要介绍程序的基本概念（如数据结构、分支以及循环）在java中的实现方式。

## 一个简单的Java应用程序（你应该了解的）

#### 下面看一个最简单的Java应用程序，它只能发送一条消息到控制台窗口中：  

```java
public class FirstSample()
{
    public static void main(String [] args)
    {
        System.out.println("We will not use 'Hello World!'");
    }    
}
```

这个程序虽然很简单，但所有的Java应用程序都具有这种结构，因此还是值得花一些时间来研究的，首先，Java区分大小写。

####下面逐行地查看这段源代码。关键字`public`称为访问修饰符（access modifier），这些修饰符用于控制程序的其他部分对这段代码的访问级别。关键字class表明Java程序中的全部内容都包含在类中。

#### 关键字class后面紧跟类名。Java中定义类名的规则很宽松。名字必须以字母开头，后面可以跟字母和数字的任意组合。长度基本没有限制。但是不能使用Java保留字。

#### 标准的命名规范为：类名是以大写字母开头的名词。如果名字由多个单词组成，每个单词的第一个字母都应该大写（这种在一个单词中间使用大写字母的方式称为骆驼命名法（camel case）。以其自身为例，应该写成 Camel Case）。

源代码的文件名必须与公共类的名字相同，并用.java作为扩展名。因此，存储这段源代码的文件名必须为FirstSample.java。

如果已经正确地命名了这个文件，并且源代码中没有任何录入错误，在编译这段源代码之后就会得到一个包含这个类字节码的文件。Java编译器将字节码文件自动地命名为FirstSample.class，并存储在源文件的同一个目录下。

**使用下面命令运行这个程序：**  
```shell
java FirstSample
# 记住不要添加.class扩展名
```

**执行结果如下：**  
![https://fredomli-oss.oss-cn-chengdu.aliyuncs.com/picture/corejava-v1ch02-FirstSample.png](https://fredomli-oss.oss-cn-chengdu.aliyuncs.com/picture/corejava-v1ch02-FirstSample.png)

#### 当使用java ClassName 运行已经编译的程序时，Java虚拟机总是从指定类中的main方法的代码开始执行，因此为了代码能够执行，在类的源文件中必须包含一个main方法。

> 注释：根据Java语言规范，main方法必须声明为public（Java语言规范是描述Java语言的官方文档，参考文末）。


#### 需要注意源代码中的大括号`{}`，在Java中，像在C/C++中一样，用大括号划分程序的各个部分（通常称为块）。Java中任何代码都用 “{” 开始， “}” 结束。  
大括号的使用风格曾经引发过许多无意义的争论。我们习惯是把匹配的大括号上下对齐。不过，由于空白符会被Java编译器忽略，所以你可以选用自己喜欢的大括号风格。

**接下来研究以下代码：**  
```
{
    System.out.println("We Will not use 'Hello, World!'");
}
```
一对大括号表示方法体的开始与结束，在这个方法中只包含一条语句。与大多数程序设计语言一样，可以将Java语句看成是语言中的句子。在Java中，每个句子必须用分号结束。回车不是语句的结束标志。

在上面这个main方法体中只包含了一条语句，其功能是将一个文本行输出到控制台上。  
这里，我们使用`System.out`对象并调用了它的`println`方法。注意，点号（.）用于调用方法。

> 注释: System.out 还有一个 print 方法，它不在输出之后增加换行符。

## 其他  
* [Java注解](https://github.com/fredomli/java-standard/blob/main/docs/java/core/basis/simple/Java注解.md)
* [Java基本数据类型](https://github.com/fredomli/java-standard/blob/main/docs/java/core/basis/simple/Java基本数据类型.md)

## 参考资料
* ***《Java核心技术卷一》***
* [Java语言规范](https://docs.oracle.com/javase/specs/jls/se8/html/index.html)
* [Java 教程(Java8)](https://docs.oracle.com/javase/tutorial/)
* [Java语言更改](https://docs.oracle.com/en/java/javase/16/language/java-language-changes.html)
* [JDK发行说明](https://www.oracle.com/java/technologies/javase/jdk-relnotes-index.html)
```
注意：  
1. 项目使用的Java版本为Java8，所选用的资料为Java8版本以上。
2. 实例源码目录：`java-standard\java-source\horstmann\corejava`
```
___________
*(☞ﾟヮﾟ)☞[返回首页README.md address](https://github.com/fredomli/java-standard)*
