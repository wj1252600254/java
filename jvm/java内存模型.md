# 内存模型
##运行时数据区域
###程序计数器
是当前线程所执行的**字节码的行号指示器**。
如果线程正在执行的是一个Java方法，计数器记录的是正在执行的虚拟机字节码指令的地址（如果正在执行的是本地方法则为空）。
线程私有，是唯一一个没有OOM的区域。
###java虚拟机栈
线程私有。
Java方法执行的内存模型：每个方法在执行的同时会创建一个栈帧，用于储存**局部变量表**（存放编译期可知的基本数据类型、对象引用、returnAdredd类型；局部变量表所需的内存大小在编译期就完成了分配，也就是说当进入一个方法时，此方法需要在栈帧中分配多大的局部变量表空间时完全确定的，运行期不会改变）、**操作数栈**、**动态链接**、**方法出口**等信息。
两种异常：线程请求栈深度大于虚拟机所允许的深度，抛出StackOverflowError异常；动态扩展时无法申请到足够内存，抛出OOM。
###本地方法栈
作用和虚拟机栈基本一样，只不过这里为Native方法服务 
JVM规范没有强制规定本地方法栈中的方法使用的语言、使用方式、数据结构，所以具体JVM不同实现。
HotSpot虚拟机直接把虚拟机栈和本地方法栈合二为一了。
异常和虚拟机栈一样。
###java堆
存放对象实例。被所有线程共享。
1. 从内存回收的角度看
分为新生代(Eden、From Survivor、To Survivor)和年老代。
2. 从内存分配角度看
可能会划分出多个线程私有的分配缓存区（TLAB）。
异常：OOM。
###方法区
被所有线程共享
用于存储已被虚拟机加载的
* 类信息(class metadata)
* 常量(包括interned Strings)
字面量包括：1.文本字符串 2.八种基本类型的值 3.被声明为final的常量等;
符号引用包括：1.类和方法的全限定名 2.字段的名称和描述符 3.方法的名称和描述符。
* 静态变量（类变量 class static variables）
* 即时编译器编译后的代码等
虽然JVM规范把方法区描述为堆的一个逻辑部分，但是它确有一个别名叫Non-Heap，目的应该是与Java堆区分开来
~~永久代~~
对于使用HotSpot VM的程序员来说，很多人把方法区称之为“永久代（Permanent Generation）”
按内存回收的角度，有新生代、老年代，所以就有了这里的“永久代”。另外，其它虚拟机是没有永久代这个概念的。
本质上两者不等价，仅仅是因为HotSpot团队把GC分代收集扩展到了方法区（或者说用永久代这种方式来实现方法区），这样的话GC就可以像管理Java堆一样管理这部分内存，省去了为方法区编写内存管理代码的工作。
1.7变化：符号引用(Symbols)转移到了native heap；字面量(interned strings)转移到了java heap；类的静态变量(class statics)转移到了java heap。
1.8变化：完全移除，变成元空间。
2.为什么移除永久代？
1、字符串存在永久代中，容易出现性能问题和内存溢出。
2、永久代大小不容易确定，PermSize指定太小容易造成永久代OOM
3、永久代会为 GC 带来不必要的复杂度，并且回收效率偏低。
4、Oracle 可能会将HotSpot 与 JRockit 合二为一。
异常：OOM。
###运行时的常量池
class文件中有一项信息是常量池表（constant_pool table），用于存放编译期生成的“字面量”和“符号引用”，这部分内容将在类加载后进入方法区的运行时常量池中（Run-Time Constant Pool）存放
也就是说：每一个class都会根据constant_pool table 来1：1创建一个此class对应的Run-Time Constant Pool。
就是运行时所需要的常量数据的容器
JVM规范对class文件的每一部分（包括constant_pool table）都有严格的规范，但是对于运行时常量池却没有做任何细节要求，不过一般来说，除了class文件中的符号引用外，直接引用也会存储在运行时常量池中
运行时常量池具备动态性，Java语言并没有要求常量一定只能编译期产生，运行期也可以将新常量放入池中。这个特性用的较多的便是String类的intern()方法。
~~字符串常量~~
String pool是用来存储被驻留的字符串的（interned string），是JVM实例全局唯一的，被所用类共享。
HotSpot中实现string pool的方式是一个哈希表：StringTable，这个StringTable的value是字符串实例的引用，也就是说某个普通的字符串实例如果被纳入StringTable的引用之后就等同于被赋予了“interned string”的身份。
字符串常量池与运行时常量池不是一个概念：
1.String Pool 是JVM 实例全局共享的全局只有一个，而Runtime Constant Pool 每个类都有一个。
2.String Pool 只记录字符串对象，而Runtime Constant Pool 记录各种对象。
3.JVM规范要求进入这里的String 实例叫“被驻留的字符串 - interned string”，各个JVM 可以有不同的实现，HotSpot 是设置了一个哈希表 - StringTable 来引用堆中的字符串实例，被引用就是被驻留。
4.字符串池在JDK 1.7 之后存在于Heap 堆中，旧版存在于方法区中。
异常：OOM。
[方法区和常量池](https://blog.csdn.net/xiao______xin/article/details/81985654)
[为什么说字符串常量池和运行时常量池不是一个概念](https://blog.csdn.net/zm13007310400/article/details/77534349)
###直接内存
JDK1.4中加入的NIO（New Input/Output）类，引入了一种基于通道（Channel）与缓冲区（Buffer）的I/O方式，这个类可以使用Native函数库直接分配堆外内存，然后通过一个存储在堆中的DirectByteBuffer对象作为这块内存的引用进行操作。这样能在一些场合显著提高性能，因为避免了在Java堆和Native堆中来回复制数据。
直接内存不会受到Java堆大小限制，但是会受到本机总内存大小，以及处理器寻址空间的限制。JVM管理员在配置JVM参数时，会根据本机实际内存设置（如-Xmx等参数），但是经常忽略了要被使用的这一份“直接内存”。最终使得各个内存总和大于物理内存限制，从而导致动态扩展时出现OutOfMemoryError异常
***
***
##对象的创建
首先检查常量池中是否存在类的符号引用，并且检查这个符号引用代表的类是否已经被加载、解析和初始化过。
在类加载检查通过，分配内存。
划分堆空间的方法有两种：“指针碰撞”、“空闲列表”，选取方式由java堆的完整性决定，因此，也可以说取决于垃圾收集器。
Serial、ParNew---指针碰撞
CMS---空闲列表
对象创建的并发安全性：1.CAS2.TLAB
###对象的内存布局
对象的储存可以分为3块区域：对象头、实例数据和对齐填充。
* 对象头
1.自身的运行时数据，hashcode,gc年龄，锁状态标志。Mark-Word 32bit    64bit
2.类型指针，指向类元数据的指针(对象头存有数组长度的数据)
* 实例数据
* 占位符
保证对象头部分正好是8字节的倍数。
###对象的访问定位
1.使用句柄
java堆中会划分出一块内存来作为句柄池，reference储存的就是对象的句柄地址，句柄中包含对象实例数据与类型数据的具体地址。
2.直接引用  Sun HotSpot
reference储存的直接就是对象地址。
##OOM
堆内存大小	
	-Xms	启动 JVM 时堆内存的大小
	-Xmx	堆内存最大限制
	-XX:+HeapDumpOnOutMemoryError可以让虚拟机出现内存异常时Dump出当前的内存堆快照。
新生代空间大小	
	-XX:NewRatio	新生代和老年代的内存比
	-XX:NewSize	新生代内存大小
	-XX:SurvivorRatio	Eden 区和 Survivor 区的内存比
方法区
	-XX:PermSize   -XX:MaxPermSize   限制永久代大小
直接内存
	--XX:MaxDirectMemorySize  调用unsafe.allocateMemory()
虚拟机栈的大小
	-Xss    设定栈内存容量
	-Xoss     设置本地方法栈大小
	
	
