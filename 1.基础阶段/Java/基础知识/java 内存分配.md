# 一、JVM内存划分

<img src="/Users/zyw/Library/Application Support/typora-user-images/image-20221222132131836.png" alt="image-20221222132131836" style="zoom:50%;" />



1. class文件

   class文件是.java程序编译后生成的中间代码，这些中间代码将会被JVM解释执行。

2. 类装载器子系统

   JVM 有两种类加载器，分别是启动类装载器和用户自定义类装载器。其中，启动类装载器是JVM实现的一部分；用户自定义类装载器则是java程序的一部分。必须是ClassLoader类的子类。

   >1.Bootstrap ClassLoader。这是JVM的根ClassLoader，它时用C++实现的，当JVM启动时，初始化此ClassLoader，并完成JAVA_HOEM/jre/lib/rt.jar 中所有class文件的加载，这个jar中包含了Java规范定义的所有接口以及实现。
   >
   >2.Extension ClassLoader。JVM用这个加载器加载扩展功能的一些jar包。
   >
   >3.System ClassLoader。JVM用词ClassLoader来加载启动参数中指定的ClassPath中jar包。
   >
   >4.User-Defined ClassLoader，继承ClassLoader抽象类自行实现ClassLoader，加载非classpath中的jar。

3. 方法区

   方法区与 java 堆一样，是各个线程共享的内存区域，他用于存储已经被虚拟机加载的类信息、常量、静态变量、即时编译器编译后的代码等数据。

    java 虚拟机规范中把方法区描述为堆的一个逻辑部分，但是他却有一个别名 — Non-Heap，“非堆”。 方法区与 java 堆一样，不要求使用连续内存，但在逻辑上是连续的，并且可以无需使用垃圾收集，有的实现中，会对常量池进行内存的回收，对类型进行卸载。 可以通过 -XX:PermSize 和 -XX:MaxPermSize 限制方法去的大小。 如果方法区无法满足内存分配需求，就会抛出 OutOfMemoryError 异常。

4. 堆

   Java Heap是Java虚拟机所管理的内存中最大的一块。Java堆是被所有线程共享的一块内存区域，在虚拟机启动时创建。此区域的唯一目的就是存放对象实例（Java虚拟机规范中的描述时：所有的对象实例以及数组都要在堆上分配）。

   Java堆是GC的主要区域，因此很多时候也被称为GC堆。

   **从内存分配的角度来看**，线程共享的Java堆中可能划分出多个线程私有的分配缓冲区（Thread Local Allocation Buffer, TLAB）

   **从内存回收的角度来看**，由于现在收集器基本都采用分代收集算法，所以Java堆中还可以细分为：新生代和老年代，在细致一点的有Eden空间，From Survivor空间，To Survivor空间等。

   备注：有OOM异常

5. 虚拟机栈

   栈是线程私有的区域，每当有新的线程创建时，就会给它分配一个栈空间，当线程执行结束，就会回收这个栈空间。每个方法在执行的同时都会创建一个栈帧，用于存储方法局部变量表、操作数、动态链接、方法出口等信息。

   通过虚拟机运行参数中的 -Xss 参数可以设定他的大小。 每调用一个方法，则这个方法在线程私有的 java 虚拟机栈中创建一个栈帧，方法调用结束则出栈。 虚拟机栈空间中的局部变量表以 Slot 为单位进行内存的分配，每个 Slot 32bit，他在编译期间完成内存的分配。

   如果线程请求的栈深度大于虚拟机所允许的深度，就会抛出 StackOverflowError 异常。 

   如果虚拟机可以动态扩展，在动态扩展时无法申请到足够内存，则会抛出 OutOfMemoryError 异常。

6. 程序计数器

   程序计数器是一块较小的内存空间，他存储了正在执行的虚拟机字节码指令的地址。 通过改变程序计数器来选取下一条需要执行的字节码指令。 这块内存被每个线程私有，且是唯一不会抛出 OutOfMemoryError 的内存区域。

7. 本地方法栈

   本地方法栈与虚拟机栈非常像，他只为 native 方法提供存储空间，有的实现中本地方法栈与虚拟机栈使用的是相同的内存空间。

8. 执行引擎

9. 垃圾回收器

我们首先看下JDK内存区域划分图（JDK1.6及其之前）：

<img src="/Users/zyw/Library/Application Support/typora-user-images/image-20221228105625285.png" alt="image-20221228105625285" style="zoom:50%;" />

<img src="/Users/zyw/Library/Application Support/typora-user-images/image-20221228105640780.png" alt="image-20221228105640780" style="zoom:50%;" />



# 二、运行时内存划分

<img src="/Users/zyw/Library/Application Support/typora-user-images/image-20221222111934993.png" alt="image-20221222111934993" style="zoom:50%;" />



**堆区:**

1.存储的全部是对象，每个对象都包含一个与之对应的class的信息(class的目的是得到操作指令) ；
2.jvm只有一个堆区(heap)，且被所有线程共享，堆中不存放基本类型和对象引用，只存放对象本身和数组本身；

**栈区:**
1.每个线程包含一个栈区，栈中只保存基础数据类型本身和自定义对象的引用；
2.每个栈中的数据(原始类型和对象引用)都是私有的，其他栈不能访问；
3.栈分为3个部分：基本类型变量区、执行环境上下文、操作指令区(存放操作指令)；

**方法区（静态区）:**
1.被所有的线程共享，方法区包含所有的class（class是指类的原始代码，要创建一个类的对象，首先要把该类的代码加载到方法区中，并且初始化）和static变量。
2.方法区中包含的都是在整个程序中永远唯一的元素，如class，static变量。

**在 JDK 7 版本及 JDK 7 版本之前**，堆内存被通常被分为下面三部分：

- 新生代内存
- 老年代内存
- 永久代内存

<img src="/Users/zyw/Library/Application Support/typora-user-images/image-20221228105843879.png" alt="image-20221228105843879" style="zoom:50%;" />

**在JDK8**之后，方法区被彻底移除了，取而代之的是元空间，元空间使用的是直接内存。

<img src="/Users/zyw/Library/Application Support/typora-user-images/image-20221228105901096.png" alt="image-20221228105901096" style="zoom:50%;" />

堆空间在使用时情况如下：

大部分情况下对象会直接在Eden区分配。
在经过一次新生代垃圾回收之后，如果如果对象还存活会进入s0或者s1 （其实就是From区），并且对象的年龄还会加1
当年龄增加到到一定程度之后，就会被晋升到老年代中。对象晋升到老年代的年龄阈值可以通过参数 -XX:MaxTenuringThreshold 来设置。

```
新建对象优先在Eden区分配内存，如果Eden区满了，那么在创建对象时，就会因为无法申请空间而触发minorGC操作，minnorGC主要是对年轻代垃圾回收：把Eden区中不能被回收的对象放入到空的Survior区，另一个Survivor区里不能被垃圾回收期回收的对象也会被放入到这个Survior中，这样就始终能保证一个Survior是空的。如果这个过程中发现Survior区也满了，那么久会把这写对象复制到老年代。或者Survior区没有满，但是有些对象已近存在了非常长的时间，这些对象也会被放入到老年代中。如果老年代也满了，就会触发FullGC.
```

什么情况下回触发FullGC?

1. 调用System.gc
2. 老年代空间不足
3. 永久代满

# 三、垃圾回收

**典型的垃圾回收算法**

​    \1. 标记-清除算法(Mark-Sweep)

​    \2. 复制算法(Copying)

​    \3. 标记-整理算法(Mark-Compact)

​    \4. 分代收集算法

# 四、Java平台与内存管理