# Java注解

#### 与大多数程序设计语言一样，Java中的注释也不会出现在可执行程序中。因此，可以在源程序中根据需要添加任意多的注释，而不必担心可执行代码会膨胀。在Java中，有3种标记注释的方式。最常用的方式是使用//，其注释内容从//开始到本行结尾。  
```text
System.out.println("We will not use 'Hello, World'");// this is comment
```

当需要更长的注释时，既可以在每行的注释前标记 `//`，也可以使用 `/*` 和 `*/` 注释界定符将一段比较长的注释括起来。  

最后，第3种注释可以用来自动地生成文档。这种注释以 `/**` 开始，以`*/` 结束。

**实例程序:**  
```java
/**
 * This is the first sample program in Core Java Chapter 3
 * @version 1.01 1997-03-22
 * @author Gary Cornell
 */
public class FirstSample
{
   public static void main(String[] args)
   {
      System.out.println("We will not use 'Hello, World!'");
   }
}

```
> 警告: 在Java中， `/*` `*/ `注释不能嵌套，也就是说，不能简单地把代码用`/*` 和 `*/`括起来作为注释，因为这段代码本身也可能包含一个`*/`定界符。 


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
