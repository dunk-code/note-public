[八股](https://xie.infoq.cn/article/ac87ac5f9e8def9f91b817bf9) 

[八股](https://juejin.cn/post/7103456883800801311)

https://github.com/eddycjy/blog

https://github.com/eddycjy/go-developer-roadmap

[面试宝典](https://golang.design/go-questions/map/range/)

GC

Go采用标记清除算法，标记清除法会产生大量内存碎片，但是由于Golang内存管理采用tcmalloc，本身内存就是碎片化的。所以清除后只需要把分片的内存挂到空闲链表即可

Golang GC历史



- v1.1串行标记&清除，整个过程需要STW,停顿可能是秒级别
- v1.3清除阶段可并行，标记阶段仍需STW,停顿上百ms
- 1.5支持并发标记、清除，实现非分代的、非移动的、并发的、三色
- 标记算法，停顿在10-40ms之间，生产环境已不是问题
- v1.8引入混合写屏障，STW时间在1ms以下
- 1.8以后，基本的GC原理保持不变，针对部分细节进行不断优化



注意：屏障机制只作用于堆，由于效率原因**栈不开启屏障**。

[三色标记法图解](https://www.kandaoni.com/news/22431.html)

[golang里gc相关的write barrier(写屏障）是个什么样的过程或者概念？](https://www.zhihu.com/question/62000722)

###### 混合屏障的具体操作

**1.GC开始时将栈上全部对象标记为黑色（之后不再进行第二次重复扫描，无需STW）**

**2.GC期间，任何在栈上创建的对象，均为黑色。**

**3.被删的对象标记为灰色。**

**4.被添加的对象标记为灰色。**

满足弱三色不变式。

[G1](https://zhuanlan.zhihu.com/p/431406707#:~:text=%E4%B8%89%E8%89%B2%E6%A0%87%E8%AE%B0%E6%B3%95%E6%98%AF,%E4%B8%BA%E4%B8%89%E8%89%B2%E6%A0%87%E8%AE%B0%E6%B3%95%E3%80%82)

GMP调度

GMP调度流程大致如下：

- 线程M想运行任务就需得获取 P，即与P关联。
- 然从 P 的本地队列(LRQ)获取 G
- 若LRQ中没有可运行的G，M 会尝试从全局队列(GRQ)拿一批G放到P的本地队列，
- 若全局队列也未找到可运行的G时候，M会随机从其他 P 的本地队列偷一半放到自己 P 的本地队列。
- 拿到可运行的G之后，M 运行 G，G 执行之后，M 会从 P 获取下一个 G，不断重复下去。

[GMP模型](http://go.cyub.vip/gmp/gmp-model.html#)

线程是被分割的 CPU 资源，协程是组织好的代码流程

**M 与P 的数量没有绝对关系，一个M 阻塞，P 就会去创建或者切换另一个M**，所以，即使P 的默认数量是1，也有可能会创建很多个M 出来。 M 与P 何时被创建？ P 在运行时系统会根据事先设定的数量而被创建。 N:1: N 个协程绑定1个线程，优点是协程在用户态线程内完成切换，不会陷入到内核态，切换快速。

G 的数量：

理论上没有数量上限限制的。查看当前G的数量可以使用`runtime. NumGoroutine()`

P 的数量：

由启动时环境变量 `$GOMAXPROCS` 或者是由`runtime.GOMAXPROCS()` 决定。这意味着在程序执行的任意时刻都只有 $GOMAXPROCS 个 goroutine 在同时运行。

M 的数量:

go 语言本身的限制：go 程序启动时，会设置 M 的最大数量，默认 10000. 但是内核很难支持这么多的线程数，所以这个限制可以忽略。 runtime/debug 中的 SetMaxThreads 函数，设置 M 的最大数量 一个 M 阻塞了，会创建新的 M。M 与 P 的数量没有绝对关系，一个 M 阻塞，P 就会去创建或者切换另一个 M，所以，即使 P 的默认数量是 1，也有可能会创建很多个 M 出来。

![https://static.cyub.vip/images/202008/golang_schedule_status.jpeg](https://gitee.com/zhang-songyao/blog-images/raw/master/202306211512257.jpeg)