## ThreadLocal

### 什么是ThreadLocal

ThreadLocal是一个本地线程副本变量工具类，在每个线程中都创建一个ThreadLocalMap对象，简单来说ThreadLocal就是一种时间换空间的做法，每个线程可以访问自己内部ThreadLocalMap对象内的value。通过这种方式，避免资源在多线程之间共享。

### 原理

线程局部变量是局限于线程内部的变量，属于线程自身所有，不在多个线程中共享，Java提供ThreadLocal类来支持线程局部变量，是一种线程安全的方式，但是管理环境下（如web服务器）使用线程局部变量的时候要小心，在这种情况下，工作线程的生命周期比任何应用变量的生命周期都要长。任何局部变量一旦工作完成后没有释放，Java应用就会存在内存泄漏的风险。

### ThreadLocal的作用

ThreadLocal的作用主要是做数据隔离，填充的数据只属于当前线程，变量的数据对别的线程而言是相对隔离的，在多线程环境下，如何防止自己的线程变量被其他线程修改

### [使用场景](https://www.zhihu.com/question/341005993)

Spring采用ThreadLocal的方式，来保证单个线程中的数据库操作使用的是同一个数据库连接，同时，采用这种方式可以使业务层使用事务时不需要感知并管理connection对象，通过传播级别，巧妙地管理多个事务配置至今的切换、挂起和恢复

Spring框架里面就是用的ThreadLocal来实现这种隔离，主要是TransactionSynchronizationManager类中的

```java
private static final Log logger = LogFactory.getLog(TransactionSynchronizationManager.class);

 private static final ThreadLocal<Map<Object, Object>> resources =
   new NamedThreadLocal<>("Transactional resources");

 private static final ThreadLocal<Set<TransactionSynchronization>> synchronizations =
   new NamedThreadLocal<>("Transaction synchronizations");

 private static final ThreadLocal<String> currentTransactionName =
   new NamedThreadLocal<>("Current transaction name");


```

Spring的事务主要是ThreadLocal和AOP去实现的，每个线程自己的链接是靠ThreadLocal保存

### 自己的使用场景

之前我们上线后发现部分用户的日期居然不对了，排查下来是SimpleDataFormat的锅，当时我们使用SimpleDataFormat的parse()方法，内部有一个Calendar对象，调用SimpleDataFormat的parse()方法会先调用Calendar.clear（），然后调用Calendar.add()，如果一个线程先调用了add()然后另一个线程又调用了clear()，这时候parse()方法解析的时间就不对了。

其实要解决这个问题很简单，让每个线程都new 一个自己的 SimpleDataFormat就好了，但是1000个线程难道new1000个SimpleDataFormat？

所以当时我们使用了线程池加上ThreadLocal包装SimpleDataFormat，再调用initialValue让每个线程有一个SimpleDataFormat的副本，从而解决了线程安全的问题，也提高了性能。



### 内存泄漏

#### 什么是内存泄漏

> 百度百科：内存泄漏是指程序已动态分配的堆内存由于某种原因程序未释放或无法释放，造成内存的浪费，导致程序运行速度减慢甚至系统崩溃等严重后果

内存泄漏是指向系统申请分配内存进行使用，然后系统在堆内存中给这个对象申请了一块内存空间，但当我们使用完了却没有归还系统，导致这个不使用的对象一致占据内存单元，造成系统将不能再把它分配给需要的程序。

一次内存泄漏的危害可以忽略不计，但是内存泄漏堆积则后果很严重，无论多少内存，迟早会被占完，造成内存泄漏。

#### 内存泄漏的危害

程序运行崩溃：导致一旦内存不足以为某些对象分配所需要的空间，将会导致程序崩溃造成体验差。

频繁GC：系统分配给每个应用程序的内存资源是有限的，内存泄漏会导致其他组件可用的内存变少后，一方面会使得GC频率加剧，在发生GC的时候，所有进程都必须等待，GC频率越高，用户越容易感应到卡顿，另一方面内存变少，可能使得系统额外分配给该对象一些内存，而影响整个系统的运行情况。

### 内存溢出

> 百度百科：内存溢出(Out Of Memory，简称OOM)是指应用系统中存在无法回收的内存或使用的内存过多，最终使得程序运行要用到的内存大于能提供的最大内存。此时程序就运行不了，系统会提示内存溢出，有时候会自动关闭软件，重启电脑或者软件后释放掉一部分内存又可以正常运行该软件，而由系统配置、数据流、用户代码等原因而导致的内存溢出错误，即使用户重新执行任务依然无法避免。

### 什么是线程局部变量？

线程局部变量是局限于线程内部的变量，属于线程自身所有，不在多个线程间共享。Java 提供 ThreadLocal 类来支持线程局部变量，是一种实现线程安全的方式。但是在管理环境下（如 web 服务器）使用线程局部变量的时候要特别小心，在这种情况下，工作线程的生命周期比任何应用变量的生命周期都要长。任何线程局部变量一旦在工作完成后没有释放，Java 应用就存在内存泄露的风险。

### ThreadLocal造成内存泄漏的原因

ThreadLocalMap中使用的key为ThreadLocal的弱引用，而value是强引用，所以，如果ThreadLocal没有被外部强引用的情况下，在垃圾回收的时候，key会被清理掉，而value不会被清理掉。这样一来ThreadLocalMap中就会出现key为null的Entry。加入我们不做任何措施的话，value永远无法被GC回收。这个时候就可能产生内存泄露的风险。ThreadLocalMap实现中已经考虑了这种情况，在调用 `set()`、`get()`、`remove()` 方法的时候，会清理掉 key 为 null 的记录。使用完 `ThreadLocal`方法后 最好手动调用`remove()`方法

### ThreadLocal内存泄漏的解决方案

- 每次使用完ThreadLocal，都调用它的remove()方法，清除数据。
- 在使用线程池的情况下，没有及时清理ThreadLocal，不仅是内存泄漏的问题，更严重的是可能导致业务逻辑出现问题。所以，使用ThreadLocal就跟加锁完要解锁一样，用完就清理。