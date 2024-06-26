# [JVM面试题](https://thinkwon.blog.csdn.net/article/details/104390752)

## 内存区域的划分

### 运行时数据区是什么？

虚拟机在执行Java程序的过程中会把它所管理的内存划分为若干分不同的数据区，这些区域有各自的用途、创建和销毁时间

线程私有：程序计数器、Java虚拟机栈、本地方法栈

线程共享：Java堆、方法区

### 程序计数器是什么？

程序计数器是一块较小的内存空间，可以看做当前线程执行字节码的行号指示器，是唯一没有内存溢出的区域，字节码解释器工作时通过改变计数器的值选取下一条执行指令。分支、循环、跳转、线程恢复等功能都需要依赖计数器

如果线程正在执行java方法，计数器记录正在执行的虚拟机字节码指令地址。如果是本地方法，计数器值为Undefined

### Java虚拟机栈的作用

Java虚拟机栈来描述Java方法的内存模型，每当有新的线程创建时就会分配一个栈空间，线程结束后栈空间被回收的，栈和线程拥有相同的生命周期。栈中元素用于支持虚拟机进行方法调用，每个方法在执行时都会创建一个栈帧存储局部变量表、操作栈、动态链接和方法出口等信息，每个方法从调用到执行完成，就是栈帧从入栈到出栈的过程

有两类异常：

-  线程请求的栈深度大于虚拟机允许的深度抛出 StackOverflowError。
-  如果 JVM 栈容量可以动态扩展，栈扩展无法申请足够内存抛出 OutOfMemoryError（HotSpot 不可动态扩展，不存在此问题）

### 本地方法栈的作用

本地方法栈与虚拟机栈作用相似，不同的是虚拟机栈为虚拟机执行 Java 方法服务，本地方法栈为虚本地方法服务。调用本地方法时虚拟机栈保持不变，动态链接并直接调用指定本地方法

虚拟机规范对本地方法栈中方法的语言与数据结构无强制规定，虚拟机可自由实现，例如 HotSpot 将虚拟机栈和本地方法栈合二为一

### 堆的作用

堆是虚拟机所管理的内存中最大的一块，所有线程共享，在虚拟机启动时创建。堆用来存放对象实例，Java中所有的对象实例都在堆上分配内存。堆可以处于物理上不连续的内存空间，逻辑上应该连续，但对于例如数组这样的大对象，多数虚拟机实现出于简单、存储高效的考虑会要求连续的内存空间

堆既可以被实现成固定大小，也可以是可扩展的，可通过 `-Xms` 和 `-Xmx` 设置堆的最小和最大容量，当前主流 JVM 都按照可扩展实现。如果堆没有内存完成实例分配也无法扩展，抛出 OutOfMemoryError

### 运行时常量池的作用

运行时常量池是方法区的一部分，Class文件除了有类的版本、字段、方法、接口等描述信息外，还有一项信息是常量池表，用来存放编译器生成的各种字面量与符号，这部分内容在类加载后存放到运行时常量池，一般除了保存Class文件描述的符号引用外，还会把符号引用翻译的直接引用也存储在运行时常量池

运行时常量池相对于 Class 文件常量池的一个重要特征是动态性，Java 不要求常量只有编译期才能产生，运行期间也可以将新的常量放入池中，这种特性利用较多的是 String 的 `intern` 方法

## 直接内存是什么

直接内内存不属于运行时数据区，也不是虚拟机规范定义的内存区域，但这部分内存被频繁使用，而且可能导致内存溢出

JDK1.4 中新加入了 NIO 这种基于通道与缓冲区的 IO，它可以使用 Native 函数库直接分配堆外内存，通过一个堆里的 DirectByteBuffer 对象作为内存的引用进行操作，避免了在 Java 堆和 Native堆来回复制数据。

直接内存的分配不受 Java 堆大小的限制，但还是会受到本机总内存及处理器寻址空间限制，一般配置虚拟机参数时会根据实际内存设置 `-Xmx` 等参数信息，但经常忽略直接内存，使存区域总和大于物理内存限制，导致动态扩展时出现 OOM

由直接内存导致的内存溢出，一个明显的特征是在 Heap Dump 文件中不会看见明显的异常，如果发现内存溢出后产生的 Dump 文件很小，而程序中又直接或间接使用了直接内存（典型的间接使用就是 NIO），那么就可以考虑检查直接内存方面的原因

## 深拷贝和浅拷贝

- 浅拷贝（ShallowCopy）：只是增加了一个指针指向已存在内存地址
- 深拷贝（DeepCopy）：是增加了一个指针并且申请了一个新的内存，使这个增加的指针指向这个新的内存
- 使用深拷贝的情况下，释放内存的时候不会因为出现浅拷贝时释放同一个内存的错误
- 浅复制：仅仅是指向被复制的内存地址，如果原地址发生改变，那么浅复制出来的对象也会相应的改变
- 深复制：在计算机中开辟一块新的内存地址用于存放复制的对象

## 堆栈的区别

| 区别         | 堆                                                           | 栈                                                         |
| ------------ | ------------------------------------------------------------ | ---------------------------------------------------------- |
| 物理地址     | 物理地址分配对对象是不连续的，在GC的时候也要考虑到不连续的分配，所以有各种算法。比如，标记-消除，复制，标记-压缩，分代（即新生代使用复制算法，老年代使用标记——压缩） | 数据结构中的栈，新进后出的原则，物理地址分配是连续的       |
| 性能         | 慢一些                                                       | 快                                                         |
| 内存分配时间 | 运行期                                                       | 编译器                                                     |
| 存放内容     | 对象的实例数据和数组，因此该区更关注的是数据的存储           | 局部变量，操作数栈，返回结果，该区更关注的是程序方法的执行 |
| 程序可见度   | 堆对整个应用程序都是共享、可见的                             | 栈只对于线程可见，私有的                                   |

- 静态变量存放在方法区
- 静态对象还是存放在堆

## 元空间和永久代

> 永久代和元空间都是方法区的一种实现

### 永久代

永久代更规范的名字的叫做方法区，永久代是方法区的一种实现方式，方法区是java规范中定义的，只有hotspot才有永久代。之所以叫它永久代是因为垃圾回收效果很差，大部分的数据会一直存在直到程序停止运行

永久代里面一般存储类相关信息，比如类常量、字符串常量、方法代码、类的定义数据等，如果要回收永久代的空间，需要将类卸载，而类卸载的条件非常苛刻，所以空间一般回收很难。**当程序中有大量动态生成类时**，这些类信息都要存储到永久代，很容易造成方法区溢出

### 元空间

> JDK6的时候HotSpot设计团队已经开始打算放弃永久代，直到JDK8，彻底的废弃了永久代的概念

元空间替代了永久代，原来存放于永久代的类信息现在放到了元空间，我们再也不会看到**java.lang.OutOfMemoryError：PermGenSpace**（Permanent Generationspace）的异常了，本质上来说，元空间也是方法区的一种实现

元空间是用来存放class metadata的，class metadata用于记录一个Java 类在JVM中的信息，包括但不限于JVM class file format的运行时数据：

- Klass 结构，这个非常重要，把它理解为一个 Java 类在虚拟机内部的表示；
- method metadata，包括方法的字节码、局部变量表、异常表、参数信息等；
- 注解；
- 方法计数器，记录方法被执行的次数，用来辅助 JIT 决策；
- 其他

特点：

- 充分利用了Java语言规范中的好处：类及相关的元数据的生命周期与类加载器的一致。
- 每个加载器有专门的存储空间
- 只进行线性分配
- 不会单独回收某个类
- 省掉了GC扫描及压缩的时间
- 元空间里的对象的位置是固定的
- 如果GC发现某个类加载器不再存活了，会把相关的空间整个回收掉

> 元空间逻辑上存在，物理上不存在

## 如何判断两个类是否相等？

任意一个类都必须由类加载器和这个类本身共同确立其在虚拟机中的唯一性

两个类只有由同一类加载器加载才有比较意义，否则即使两个类来源于同一个 Class 文件，被同一个 JVM 加载，只要类加载器不同，这两个类就必定不相等