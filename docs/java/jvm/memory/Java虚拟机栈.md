# Java虚拟机栈

与程序计数器一样，Java虚拟机栈（Java Virtual Machine Stack）也是线程私有的，它的生命周期与线程相同。虚拟机栈描述的是Java方法执行的线程内存模型；每个方法被执行的时候，Java虚拟机都会同步创建一个*栈帧*（Stack Frame）用于存储局部变量表、操作数栈、动态连接、方法出口等信息。每个方法被调用直至执行完毕的过程，就对应着一个栈帧在虚拟机栈中从入栈到出栈的过程。


可以参考[Java虚拟机规范-虚拟机堆栈](https://docs.oracle.com/javase/specs/jvms/se8/html/jvms-2.html#jvms-2.5.2)

经常有人把Java内存区域笼统地划分为堆内存(Heap)和栈内存(Stack)，这种划分方式直接继承自传统的C、C++程序的内存布局结构，在Java语言里就显得有些粗糙了，实际内存划分要比这更复杂。不过这种划分方式的流行也间接说明了程序员最关注的与对象内存分配关系最密切的区域是“堆”和“栈”两块，其中，“堆”在其他地方介绍，而“栈”通常就是指这里讲的虚拟机栈，或者更多的情况下只是指虚拟机栈中局部变量表部分。  

局部变量表存放了编译期可知的各种Java虚拟机基本数据类型（boolean、byte、char、short、int、float、long、double）、对象引用（reference类型，它并不等同于对象本身，可能是个指向对象起始地址的引用指针，也可能是指向一个代表对象的句柄或者其他与此对象相关的位置）和returnAddress类型(指向了一条字节码指令的地址)。





*(☞ﾟヮﾟ)☞[返回首页README.md address](https://github.com/fredomli/java-standard)*
