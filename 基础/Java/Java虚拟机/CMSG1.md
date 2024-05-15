CMS收集器，是为了低延迟而生，通过尽可能的**并行执行垃圾回收的几个阶段**把延迟控制到最低。CMS收集器是老年代的垃圾收集器，一般情况下会有ParNew来配合执行，ParNew也是使用并行的算法来执行年轻代的回收。当然除此之外，你还可以选择使用Serial收集器来收集年轻代，不过一般很少这样使用。通过咱们说的CMS收集器是指广义上的CMS收集器，包含以下几个：ParNew（Young）GC + CMS（Old）GC ＋ Serial GC 算法（应对核心的CMS GC某些时候的不赶趟，开销很大）。

## 触发条件

 CMS垃圾收集器的触发条件有以下几个：
1、如果没有设置-XX:+UseCMSInitiatingOccupancyOnly，虚拟机会根据收集的数据决定是否触发（建议带上这个参数）。
2、老年代使用率达到阈值 CMSInitiatingOccupancyFraction，默认92%，前提是配置了第一个参数。
3、永久代的使用率达到阈值 CMSInitiatingPermOccupancyFraction，默认92%，前提是开启 CMSClassUnloadingEnabled并且配置了第一个参数。
4、新生代的晋升担保失败。

## CMS的收集阶段

CMS收集器在收集老年代的时候分为以下几个阶段：**初始标记、并发标记、预清理、可中断预清理、最终标记、并发清除、并发重置**。



![file](https://gitee.com/zhang-songyao/tripimages/raw/master/20210811145426.jpeg)

## G1

G1收集器伴随JAVA9于2017-9-21发布,G1收集器兼顾低延迟和高吞吐在服务端运行,HotSpot团队期望取代CMS收集器。也就是在满足停顿时间的情况下获取最大的吞度量。有两种收集模式Young GC和Mixed GC。G1收集器将堆内存划分成大小相等的Region,新生代,老年代也就成了逻辑概念。整体上采用的是标记-整理算法,局部采用了复制算法。

## G1和CMS的区别

G1采用标记-整理算法,CMS采用标记-清除算法,所以G1不会产生很多垃圾碎片.
G1的STW(stop the world)可控,可以使用-XX:MaxGCPauseMillis设置默认200ms
G1的Young GC模式可以工作在年轻代,而单独的CMS只能工作在老年代.

## Stop The World

简称STW，指的是GC事件发生过程中，会产生应用程序停顿，停顿产生时整个应用程序线程都会被暂停，没有任何响应，这个停顿称为STW，除了native代码可以执行，但不能和JVM交互

举例：可达性分析算法中枚举根节点（GC Roots）会导致所有Java执行线程停顿

回到最开始的问题，那个***\*停顿类型就是STW\****，至于**有\**GC\**和\**Full GC\**之分，还有\**Full GC (System)\****。个人认为主要是Full GC时STW的时间相对GC来说时间很长，因为Full GC针对整个堆以及永久代的，因此整个GC的范围大大增加；还有就是他的回收算法就是我们之前说过的“标记–清除–整理”，这里也会损耗一定的时间。**所以我们在优化JVM的时候，减少Full GC的次数也是经常用到的办法。** 