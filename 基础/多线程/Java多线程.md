# 日拱一卒，不期速成，厚积薄发

[TOC]



# 线程介绍

## 程序、进程、线程基本概念

- 程序（program）是为完成特定任务，用某种编程语言编写的一组指令的集合。即指一段静态的代码，静态对象。

- 进程（process）一个在内存中运行的应用程序。每个进程都有自己的独立的一块内存空间，一个进程可以有多个线程，比如Windows系统中，一个运行的*.exe就是一个进程。是一个动态的过程：有它自身的产生、存在和消亡的过程 -----生命周期

  - 如：运行中的QQ，运行中的MP3播放器程序是静态的，进程是动态的。
  - 进程作为资源分配的单位，系统在运行时会为每个进程分配不同的内存区域。

- 线程（thread）进程中的一个执行单元（控制单元），负责当前进程中程序的执行，是一个程序内部的一条执行路径。一个进程至少有一个线程，一个进程可以运行多个线程，多个线程之间共享数据

  - 若一个进程同一时间并行执行多个线程，就是支持多线程的
  - 线程作为调度和执行的单元，每个线程拥有独立的运行栈和程序计数器（PC），线程切换的开销小。

  - 一个进程中的多个线程共享相同的内存单元/内存地址空间，它们从同一个堆中分配对象，可以访问相同的变量和对象，这就使得线程间通信更简便、高效。但是多个线程操作共享的系统资源可能就会带来安全隐患。

  ![进程与线程](https://gitee.com/zhang-songyao/blog-images/raw/master/20210420162320.png)

- 单核CPU和多核CPU的理解

  - 单核CPU，其实是一种假的多线程，因为在一个时间单元内，也只能执行一个线程的任务。例如：虽然有多车道，但是收费站只有一个工作人员在收费，只有收了费才能通过，那么CPU就好比收费人员。如果有某个人不想交钱，那么收费人员就可以把它“挂起”（晾着他，等他想通了，准备好了钱，再去收费）。但是因为CPU时间单元特别短，因此感觉不出来。
  - 如果是多核的话，才能更好地发挥多线程的效率。（现在的服务器都是多核的）。
  - 一个Java应用程序java.exe，其实至少有三个线程：main()主线程、gc()垃圾回收线程，异常处理线程。当然如果发生异常，会影响主线程。

### 进程与线程的区别

线程具有许多传统进程所具有的特征，故又称为轻型进程（Light---Weight Process）或进程元；而把传统的进程称为重新进程（Heavy---Weight Process），它相当于只有一个线程的任务。在引入线程的操作系统中，通常一个进程有若干个线程，至少包含一个线程。

|   区别   |                             进程                             |                             线程                             |
| :------: | :----------------------------------------------------------: | :----------------------------------------------------------: |
| 根本区别 |               进程是操作系统资源分配的基本单元               |             线程是处理器任务调度和执行的基本单元             |
| 资源开销 | 每个进程都有独立的代码和数据空间（程序上下文），程序之间切换会有较大的开销 | 线程可以看做轻量级的进程，同一线程共享代码和数据空间，每个线程都有自己独立的运行栈和程序计数器（PC），线程之间切换的开销小 |
| 包含关系 | 如果一个进程中有多个线程，则执行过程不是一条线的，而是多条线共同完成的 |  线程是进程的一部分，所以线程也被称为轻权进程或者轻量级进程  |
| 影响关系 |      一个进程崩溃后，在保护模式下不会对其他进程产生影响      |     一个线程崩溃整个进程都会死掉，所以多进程比多线程健壮     |
| 执行过程 |    每个独立的进程有程序运行的入口、顺序执行序列和程序出口    | 线程不能独立执行，必须依存在应用程序中，由应用程序提供多个线程执行控制，两者均可并发执行 |

### 什么是上下文切换

多线程编程中一般线程的个数大于CPU的核心的个数，而一个CPU核心在任意时刻只能被一个线程使用，为了让这些线程能得到有效的执行，CPU采取的策略是为每个线程分配时间片并轮转的形式。当一个线程的时间片用完的时候就会重新处于就绪状态让给其他线程使用，这个过程就属于依次上下文交换。

概括来说就是：当前任务执行完CPU时间片切换到另一个任务之前会保存自己的状态，以便下次在切换回这个任务时，可以再加载这个任务的状态，**任务从保存到再加载的过程就是一次上下文切换**

上下文切换通常是计算密集型的，也就是说，他需要相当可观的处理器时间，在每秒几十上百次的切换中，每次切换都需要纳秒量级的时间。所以，上下文切换对系统来说意味着消耗大量的 CPU 时间，事实上，可能是操作系统中时间消耗最大的操作。

>Linux 相比与其他操作系统（包括其他类 Unix 系统）有很多的优点，其中有一项就是，其上下文切换和模式切换的时间消耗非常少。

### 串行、并行和并发有什么区别

并行：时间单位内，多个CPU同时执行多个任务，真正意义上的同时进行，比如多个人呢同时做不同的事情。

并发：一个CPU（采用时间片）按细分的时间片轮流交替执行，同时执行多个任务，从逻辑上看来这些任务是同时执行的。

串行：有n个任务，由一个线程按顺序执行，由于任务、方法都在一个线程执行所以不存在线程不安全情况，也就不存在临界区的问题。

做一个形象的比喻：

并发 = 两个队列和一台咖啡机。

并行 = 两个队列和两台咖啡机。

串行 = 一个队列和一台咖啡机。

### 使用多线程的优缺点

背景：以单核CPU为例，只使用单个线程先后完成多个任务（调用多个方法），肯定比用多个线程来完成用的时间更短，为何仍需要多线程呢？

多线程程序的优点：

- 提高应用程序的响应，对图形化界面更有意义，可增强用户体验。
- 提高计算机系统CPU的利用率。
- 改善程序结构。将既长又复杂的进程分为多个线程，独立运行，便于理解和修改。

并发编程的缺点

并发编程的目的就是为了能提高程序的执行效率，提高程序运行速度，但是并发编程并不总是能提高程序运行速度的，而且并发编程可能会遇到很多问题，比如**：内存泄漏、上下文切换、线程安全、死锁**等问题

### 何时需要多线程

- 程序需要同时执行两个或多个任务。
- 程序需要实现一些需要等待的任务时，如：用户输入、文件读写操作、网络操作、搜索等。
- 需要一些后台运行程序时。

### 并发编程的三要素

- 原子性：原子，即一个不可再分割的颗粒，原子性指的是一个或者多个操作（复合操作）要么全部执行成功要么全部执行失败。
- 可见性：一个线程对共享变量的修改，另一个线程能够立刻看到。（由于CPU和内存之间有缓存的存在）
- 有序性：程序执行的顺序按照代码的先后顺序执行。（处理器可能会对指令进行重排序）

出现线程安全问题的原因：

- 线程切换带了的原子性问题
- 缓存导致可见性问题
- 编译优化带来的有序性问题

解决办法

- 原子性：Atomic开头的原子类、synchronized、Lock
- 可见性：CAS、synchronized、Lock
- 有序性：Happens-Before规则

## 进程间的通信方式及适用场景

### 共享内存通信

共享内存是指多个进程共享一块内存，是专门来解决不同进程之间的通信问题的，由于直接对内存进行数据传输操作，所以速度最快的IPC（Inter-process communication）方式，因为是共享内存，所以需要配合信号量机制实现同步

### 管道通信

1. 无名管道：半双工，只能在有亲缘关系的进程间使用。（进程的亲缘关系是指父子进程关系）
2. 具名管道：半双工，允许无亲缘关系的进程的通信
3. 消息队列：消息队列是消息的链表，克服了信号传递消息少，管道只能承载无格式字节流以及缓冲区大小受限等缺点。
4. 套接字通信：可用于不同机器间的进程通信。

## 协程

协程是一种比线程更加轻量级的存在，一个线程可以拥有多个携程。携程不是被操作系统内核所管理，而是完全有程序所控制（也就是用户态执行），好处是性能得到了很大的提升，不会向线程切换那样消耗资源

协程在子程序内部是可中断的，然后转而执行别的子程序，在适当的时候再返回来接着执行

### 协程和多线程相比优势

- 极高的执行效率：因为子程序切换不是线程切换，而是程序自身控制，因此没有线程切换的开销，和多线程相比，线程数量越多，协程的性能优势就越明显
- 不需要多线程的锁机制：因为只有一个线程，也不存在同时写变量冲突，在协程中控制共享资源不加锁，只需要判断状态就好了，所以执行效率比线程高

# 线程实现

## 线程的创建和使用

### 线程的创建和启动

- Java语言的JVM允许程序运行多个线程，他通过java.lang.Thread类来体现
- Thread类的特性
- - 每个线程都是通过某个特定的Thread对象的run()方法来完成操作的，经常把run()方法的主体称为线程体
  - 通过该Thread()对象的start()方法来启动这个线程，而非直接调用run()

### 扩展问题

#### run()和start()有什么区别

<img src="https://gitee.com/zhang-songyao/blog-images/raw/master/20210420162338.png" alt="start和run"/>

- 每个线程都是通过某个特定的Thread对象所对应的方法run()来完成其操作的，run()称为方法的线程体。通过调用Thread类的start()方法来启动一个线程。
- start()方法用于启动一个线程，run()方法用于执行线程的代码，run()可以重复调用，而start()只能调用一次。
- start()方法来启动一个线程，真正实现了多线程运行。调用start()方法无需等待run方法体代码执行完毕，可以直接继续执行其他的代码； 此时线程是处于就绪状态，并没有运行。 然后通过此Thread类调用方法run()来完成其运行状态， run()方法运行结束， 此线程终止。然后CPU再调度其它线程。
- run()方法是在本线程里的，只是线程里的一个函数，而不是多线程的。 如果直接调用run()，其实就相当于是调用了一个普通函数而已，直接待用run()方法必须等待run()方法执行完毕才能执行下面的代码，所以执行路径还是只有一条，根本就没有线程的特征，所以在多线程执行时要使用start()方法而不是run()方法。

#### 为什么我们调用 start() 方法时会执行 run() 方法，为什么我们不能直接调用 run() 方法？

- 这是另一个非常经典的 java 多线程面试问题，而且在面试中会经常被问到。很简单，但是很多人都会答不上来！
- new 一个 Thread，线程进入了新建状态。调用 start() 方法，会启动一个线程并使线程进入了就绪状态，当分配到时间片后就可以开始运行了。 start() 会执行线程的相应准备工作，然后自动执行 run() 方法的内容，这是真正的多线程工作。
- 而直接执行 run() 方法，会把 run 方法当成一个 main 线程下的普通方法去执行，并不会在某个线程中执行它，所以这并不是多线程工作。
- 总结： 调用 start 方法方可启动线程并使线程进入就绪状态，而 run 方法只是 thread 的一个普通方法调用，还是在主线程里执行。

### Thread类

构造器

- Thread()：创建新的Thread对象
- Thread(String threadName)：创建线程并指定线程实例名
- Thread(Runnable target)：指定创建线程的目标对象，它实现了Runnable接口中的run方法
- Thread(Runnable target, String name)：创建新的Thread对象

### API中创建线程的三种方式

JDK1.5之前创建新执行的线程有两种方式：

- 继承Thread类的方式
- 实现Runnable接口的方式
- 实现Callable接口

#### 继承Thread类

步骤

1. 自定义线程类继承Thread类
2. 重写run()方法，编写线程执行体
3. 创建线程对象，调用start()方法启动线程

```java
/**
 * @author ：zsy
 * @date ：Created 2021/4/12 10:11
 * @description：继承Thread类创建线程
 */
//线程开启不一定立即执行，由CPU调度执行
public class TestThread1 extends Thread {
    @Override
    public void run() {
        //run方法线程体
        super.run();
        for (int i = 0; i < 20; i++) {
            System.out.println("testThread1---->" + i);
        }
    }

    public static void main(String[] args) {
        //main线程，主线程

        //创建一个线程对象
        TestThread1 thread1 = new TestThread1();

        //调用start()方法开启线程
        thread1.start();

        //调用run()方法开启线程
        //thread1.run();
        for (int i = 0; i < 20; i++) {
            System.out.println("main---->" + i);
        }
    }
}
```

#### 实例

实现多线程同步下载图片

```java
/**
 * @author ：zsy
 * @date ：Created 2021/4/12 17:27
 * @description：练习Thread，实现多线程同步下载图片
 */
public class TestThread2 extends Thread{

    private String url;//网图地址
    private String name;//保存文件名

    public TestThread2(String url, String name) {
        this.name = name;
        this.url = url;
    }

    @Override
    public void run() {
        super.run();
        WebDownloader webDownloader = new WebDownloader();
        webDownloader.downloader(url, name);
        System.out.println("下载的图片名称--->" + name);
    }

    public static void main(String[] args) {
        //创建线程对象
        TestThread2 t1 = new TestThread2("https://picsum.photos/id/237/300/200", "237.jpg");
        TestThread2 t2 = new TestThread2("https://picsum.photos/id/337/300/200", "337.jpg");
        TestThread2 t3 = new TestThread2("https://picsum.photos/id/437/300/200", "437.jpg");

        //启动线程
        t1.start();
        t2.start();
        t3.start();
    }
}

//下载器
class WebDownloader {

    //下载方法
    public void downloader(String url, String name) {
        try {
            FileUtils.copyURLToFile(new URL(url), new File(name));
        } catch (IOException e) {
            e.printStackTrace();
            System.out.println("IO异常，downloader方法出现异常");
        }
    }
}

执行结果
下载的图片名称--->237.jpg
下载的图片名称--->437.jpg
下载的图片名称--->337.jpg
```

#### 实现Runnable接口

步骤

1. 定义MyRunnable类实现Runnable接口
2. 实现run()方法，编写线程执行体
3. 创建线程对象，调用start()方法启动线程

*推荐使用Runnable对象，因为Java单继承的局限性*

```java
/**
 * @author ：zsy
 * @date ：Created 2021/4/12 18:05
 * @description：实现Runnable接口创建多线程
 */
public class TestThread3 implements Runnable{
    @Override
    public void run() {
        //run方法线程体
        for (int i = 0; i < 20; i++) {
            System.out.println("testThread1---->" + i);
        }
    }

    public static void main(String[] args) {
        //创建runnable接口的实例对象
        TestThread3 thread3 = new TestThread3();
        
        /*
        //创建线程对象,通过线程对象来开启我们的线程，代理
        Thread thread = new Thread(thread3);
        //调用start()方法开启线程
        thread.start();
        */
        
        new Thread(thread3).start();

        //调用run()方法开启线程
        //thread1.run();
        for (int i = 0; i < 20; i++) {
            System.out.println("main---->" + i);
        }
    }
}
```

#### 静态代理&动态代理

##### -静态代理

- 代理类需要自己手动实现，自己创建一个java类，表示代理类。
- 代理的目标类是确定的
- 特点
  - 实现简单
  - 容易理解
- 缺点
  - 当目标类和代理类很多的时候，代理类也需要成倍的增加。
  - 当你的接口中功能增加了，或者修改了，会影响众多实现类，代理类都需要修改

```java
/**
 * @author ：zsy
 * @date ：Created 2021/4/12 21:45
 * @description：卖优盘接口
 */
public interface UsbSell {
    float sell();
}


/**
 * @author ：zsy
 * @date ：Created 2021/4/12 21:47
 * @description：金士顿厂商
 */
public class UsbKingFactory implements UsbSell {
    //定义厂家出厂价格
    @Override
    public float sell() {
        return 85.0f;
    }
}

/**
 * @author ：zsy
 * @date ：Created 2021/4/12 21:48
 * @description：淘宝代理商
 */
public class Taobao implements UsbSell {
    //明确代理的目标对象
    private UsbKingFactory factory = new UsbKingFactory();
    @Override
    public float sell() {
        float price = factory.sell();
        //增加价格
        price += 15f;
        return price;
    }
}

/**
 * @author ：zsy
 * @date ：Created 2021/4/12 21:50
 * @description：用户类
 */
public class User {
    //通过淘宝购买金士顿U盘
    public float buyUsb() {
        Object factory = new UsbKingFactory();
        InvocationHandler handler = new MyHandler(factory);
        UsbSell proxy = (UsbSell) Proxy.newProxyInstance(factory.getClass().getClassLoader(),
                factory.getClass().getInterfaces(), handler);
        return proxy.sell();
    }
}

/**
 * @author ：zsy
 * @date ：Created 2021/4/12 21:51
 * @description：测试
 */
public class Test {
    public static void main(String[] args) {
        //创建用户
        User user = new User();

        //购买U盘
        System.out.println(user.buyUsb());
    }
}
```

##### -动态代理

###### JDK动态代理

- 在程序执行过程中，使用JDK反射机制，创建代理对象，并动态的指定代理目标类
- 动态代理是一种创建java对象的能力，让你不用创建TaoBao类，就能创建代理类对象
- 实现步骤
  - 创建接口，定义目标类要完成的功能
  - 创建目标类实现接口
  - 创建InvocationHandler接口的实现类，在invoke方法中完成代理类的功能
    - 调用目标方法
    - 增强功能
  - 使用Proxy类的静态方法，创建代理对象。 并把返回值转为接口类型。

```java
/**
 * @author ：zsy
 * @date ：Created 2021/4/12 21:45
 * @description：卖优盘接口
 */
public interface UsbSell {
    float sell();
}


/**
 * @author ：zsy
 * @date ：Created 2021/4/12 21:47
 * @description：金士顿厂商
 */
public class UsbKingFactory implements UsbSell {
    //定义厂家出厂价格
    @Override
    public float sell() {
        return 85.0f;
    }
}

/**
 * @author ：zsy
 * @date ：Created 2021/4/12 21:53
 * @description：动态代理类
 */
public class MyHandler implements InvocationHandler {

    //代理类(不确定)
    private Object target = null;

    public MyHandler(Object target) {
        this.target = target;
    }

    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {

        Object res = method.invoke(target, args);

        //获取到金士顿厂家的U盘价格
        if (res != null) {
            float price = (float) res;
            System.out.println("从厂家拿到的U盘价格" + price);
            price += 15f;
            res = price;
        }
        return res;
    }
}


/**
 * @author ：zsy
 * @date ：Created 2021/4/12 21:50
 * @description：用户类
 */
public class User {
    //通过淘宝购买金士顿U盘
    public float buyUsb() {
        Object factory = new UsbKingFactory();
        InvocationHandler handler = new MyHandler(factory);
        UsbSell proxy = (UsbSell) Proxy.newProxyInstance(factory.getClass().getClassLoader(),
                factory.getClass().getInterfaces(), handler);
        return proxy.sell();
    }
}


/**
 * @author ：zsy
 * @date ：Created 2021/4/12 21:57
 * @description：测试动态代理
 */
public class Test {
    public static void main(String[] args) {
        //创建用户
        User user = new User();
        System.out.println("用户最终购买U盘的价格" + user.buyUsb());
    }
}
```

###### CGILIB动态代理

- 原理
  - CGILIB通过继承目标类，创建它的子类，在子类中重写父类中同名的方法，实现功能的增强
- 因为CGILIB是继承，重写方法，所以要求目标类不能是final的， 方法也不能是final的。cglib的要求目标类比较宽松， 只要能继承就可以了。

#### 实现Callable接口

步骤

1. 实现Callable接口，需要返回值类型
2. 重写call方法，需要抛出异常
3. 创建目标对象
4. 创建执行服务 ExectorService service = Exectors.newFixedThreadPoll(1);
5. 提交执行 Future<> r1 = service.submit(thread);
6. 获取解过 boolean res1 = r1.get();
7. 关闭服务 service.shutdownNow();

#### 实例

实现多线程同步下载图片

```java
/**
 * @author ：zsy
 * @date ：Created 2021/4/12 21:01
 * @description：实现Callable接口
 */
public class TestCallable implements Callable<Boolean> {

    private String url;//网图地址
    private String name;//保存文件名

    public TestCallable(String url, String name) {
        this.name = name;
        this.url = url;
    }

    @Override
    public Boolean call() {
        WebDownloader webDownloader = new WebDownloader();
        webDownloader.downloader(url, name);
        System.out.println("下载的图片名称--->" + name);
        return true;
    }

    public static void main(String[] args) throws ExecutionException, InterruptedException {
        //创建线程对象
        TestCallable t1 = new TestCallable("https://upload-images.jianshu.io/upload_images/20068213-872f9137d7826e87.png?imageMogr2/auto-orient/strip|imageView2/1/w/360/h/240", "1.jpg");
        TestCallable t2 = new TestCallable("https://profile.csdnimg.cn/C/2/6/2_qq_45796208", "2.jpg");
        TestCallable t3 = new TestCallable("https://img-home.csdnimg.cn/images/20210412010711.png", "3.jpg");

        //创建执行服务
        ExecutorService service = Executors.newFixedThreadPool(3);

        //执行提交
        Future<Boolean> r1 = service.submit(t1);
        Future<Boolean> r2 = service.submit(t2);
        Future<Boolean> r3 = service.submit(t3);

        //获取结果
        boolean re1 = r1.get();
        boolean re2 = r2.get();
        boolean re3 = r3.get();

        //关闭服务
        service.shutdownNow();
    }
}

//下载器
class WebDownloader {

    //下载方法
    public void downloader(String url, String name) {
        try {
            FileUtils.copyURLToFile(new URL(url), new File(name));
        } catch (IOException e) {
            e.printStackTrace();
            System.out.println("IO异常，downloader方法出现异常");
        }
    }
}
```

1. 有返回值
2. 可以抛出异常
3. 方法不同 run()\ call()

#### Callable和Runnable关系

![FutureTask继承图](https://gitee.com/zhang-songyao/blog-images/raw/master/20210422110752.png)

```java
public FutureTask(Callable<V> callable) {
    if (callable == null)
        throw new NullPointerException();
    this.callable = callable;
    this.state = NEW;       // ensure visibility of callable
}
```

因此可以通过Runnable的方式创建实现了Callable接口的线程

>创建线程实现Callable接口

```java
/**
 * @author ：zsy
 * @date ：Created 2021/4/19 22:03
 * @description：通过FutureTast创建Callable线程
 */
public class CallableTest {

    public static void main(String[] args) throws ExecutionException, InterruptedException {
        MyThread myThread = new MyThread();
        //使用FutureTask对Callable对象进行包装
        FutureTask task = new FutureTask(myThread);

        new Thread(task, "A").start();
        new Thread(task, "B").start();
        boolean res = (boolean) task.get();
        System.out.println(res);
    }
}

class MyThread implements Callable<Boolean> {

    @Override
    public Boolean call() throws Exception {
        System.out.println("call");
        return true;
    }
}
call
true
```

>启动两个线程，存在缓存，会产生阻塞，结果可能需要等待

#### Callable和Runnable区别

- Runnable 接口 run 方法无返回值；Callable 接口 call 方法有返回值，是个泛型，和Future、FutureTask配合可以用来获取异步执行的结果
- Runnable 接口 run 方法只能抛出运行时异常，且无法捕获处理；Callable 接口 call 方法允许抛出异常，可以获取异常信息

>Callalbe接口支持返回执行结果，需要调用FutureTask.get()得到，此方法会阻塞主进程的继续往下执行，如果不调用不会阻塞

### 小结

- 继承Thread类
  - 子类继承Thread类具备多线程能力
  - 启动线程：子类对象.start()
  - 不建议使用：避免OOP单继承局限性
- 实现Runnable接口
  - 实现接口Runnable具有多线程能力
  - 启动线程：传入目标对象+Thread对象.start()
  - 推荐使用：避免单继承的局限性，灵活方便，方便同一个对象被多个线程使用

### 初始并发问题

```java
/**
 * @author ：zsy
 * @date ：Created 2021/4/12 20:10
 * @description：多线程操作同一个对象，模拟买火车票的例子
 */

//发现问题：多个线程操作同一个对象情况下，线程不安全
public class TestThread4 implements Runnable{

    private int ticketNums = 10;

    @Override
    public void run() {
        while(true) {

            //模拟延时
            try {
                Thread.sleep(200);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            if (ticketNums <= 0) break;
            System.out.println(Thread.currentThread().getName() + "拿到了第" + ticketNums-- + "张票");
        }
    }

    public static void main(String[] args) {
        TestThread4 ticket = new TestThread4();

        //统一个对象被多个线程使用
        new Thread(ticket, "小明").start();
        new Thread(ticket, "小华").start();
        new Thread(ticket, "黄牛党").start();
    }
}


小华拿到了第10张票
小明拿到了第9张票
黄牛党拿到了第10张票
黄牛党拿到了第8张票
小明拿到了第7张票
小华拿到了第7张票
黄牛党拿到了第6张票
小华拿到了第6张票
小明拿到了第6张票
黄牛党拿到了第5张票
小明拿到了第5张票
小华拿到了第5张票
小华拿到了第4张票
黄牛党拿到了第3张票
小明拿到了第4张票
黄牛党拿到了第2张票
小明拿到了第1张票
```

*发现问题：多个线程操作同一个对象情况下，线程不安全，数据紊乱*

### 龟兔赛跑模拟多线程

```java
/**
 * @author ：zsy
 * @date ：Created 2021/4/12 20:32
 * @description：模拟龟兔赛跑
 */
public class Race implements Runnable{

    //胜利者
    private static String winner;

    @Override
    public void run() {
        //模拟跑步
        for (int i = 0; i <= 100; i++) {
            //模拟兔子休息
            //每走10步休息1ms
            if (Thread.currentThread().getName().equals("兔子") && i % 10 == 0) {
                try {
                    Thread.sleep(1);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
            //判断是否已经有了胜利者
            if (hasWinner(i)) break;

            System.out.println(Thread.currentThread().getName() + "------->跑了" + i + "步");
        }
    }

    //判断是否有胜利者
    public boolean hasWinner(int step) {
        if (winner != null) {
            return true;
        }
        if (step >= 100) {
            winner = Thread.currentThread().getName();
            System.out.println("比赛结束，胜利者是" + winner);
            return true;
        }
        return false;
    }

    public static void main(String[] args) {
        Race race = new Race();

        new Thread(race, "兔子").start();
        new Thread(race, "乌龟").start();
    }

}

```

## 扩展问题

### Java中用到的线程调度算法

计算机只有一个CPU，在任意时刻只能执行一条机器指令，每个线程只有获得CPU的使用权才能执行指令，所谓的多线程并发运行，其实是指从宏观上看，各个线程轮流获得CPU的使用权，分别执行各自的任务。在运行池中，会有多个处于就绪状态的线程在等待CPU的调度，Java虚拟机的一项任务就是负责线程的调度，线程的调度是指按照特定机制为线程分配CPU的使用权

- 分时调度模型

让所有的线程轮流获得CPU的使用权，并且平均分配给每个线程占用CPU的时间片。

- 抢占式调度模型

Java虚拟机采用的是抢占式调度模型，是指优先让可运行池中优先级高的线程占有CPU，如果运行池中的线程优先级相同，难么就随机选择一个线程，使其占有CPU。处于运行状态的线程会一直运行，直到他不得不放弃CPU。

### 什么是线程调度器（Thread Scheduler）和时间分片（Time Slicing）

线程调度器是一个操作系统服务，他负责为Runnable状态的线程分配CPU时间，一旦我们创建一个线程并启动它，它的执行便依赖于线程调度器的实现。

时间分片是指将可用的CPU时间分配给可用的Runnable线程的过程，分配CPU时间可以基于线程优先级或者线程的等待时间

线程调度不受Java虚拟机的控制，所以由应用程序来控制它是更好的选择（也就是说不要让你的程序依赖于线程的优先级别）



# 线程状态

## 线程的五种状态

![线程的生命周期](https://gitee.com/zhang-songyao/blog-images/raw/master/20210420162339.png)

### 线程的状态转换

![线程状态转换图](https://gitee.com/zhang-songyao/blog-images/raw/master/20210420162340.png)



![线程的基本状态](https://gitee.com/zhang-songyao/blog-images/raw/master/20210604000358.png)

**阻塞(block)**：处于运行状态中的线程由于某种原因，暂时放弃对 CPU的使用权，停止执行，此时进入阻塞状态，直到其进入到就绪状态，才 有机会再次被 CPU 调用以进入到运行状态。

阻塞的情况分三种：
(一). 等待阻塞：运行状态中的线程执行 wait()方法，JVM会把该线程放入等待队列(waitting queue)中，使本线程进入到等待阻塞状态；
(二). 同步阻塞：线程在获取 synchronized 同步锁失败(因为锁被其它线程所占用)，，则JVM会把该线程放入锁池(lock pool)中，线程会进入同步阻塞状态；
(三). 其他阻塞: 通过调用线程的 sleep()或 join()或发出了 I/O 请求时，线程会进入到阻塞状态。当 sleep()状态超时、join()等待线程终止或者超时、或者 I/O 处理完毕时，线程重新转入就绪状态。

### 线程的方法

|               方法               |                    说明                    |
| :------------------------------: | :----------------------------------------: |
|  setPriority( int newPriority )  |               更改线程优先级               |
| static void sleep( long millis ) |  在指定的毫秒数内让当前正在执行的线程休眠  |
|           void join()            |               等待该线程终止               |
|       static void yield()        | 暂停当前正在执行的线程对象，并执行其他线程 |
|          void interrupt          |            中断线程，不用此方法            |
|        boolean isAlive()         |          测试线程是否处于活动状态          |

### ~~stop~~

- 不推荐使用JDK提供的stop()、destroy()方法
- 推荐线程自己停下来
- 建议使用一个标志位进行终止变量，当flag = false，则终止线程运行

```java
/**
 * @author ：zsy
 * @date ：Created 2021/4/13 14:31
 * @description：测试停止线程
 */
public class TestStop implements Runnable{

    //定义标志
    private boolean flag = true;

    @Override
    public void run() {
        int i = 0;
        while(flag) {
            System.out.println(Thread.currentThread().getName() + "run....." + i++);
        }
        if (!flag){
            System.out.println(Thread.currentThread().getName() + "线程停止");
        }
    }

    public void stop() {
        this.flag = false;
    }

    public static void main(String[] args) {
        TestStop testStop = new TestStop();
        new Thread(testStop,"T1").start();
        for (int i = 0; i < 300; i++) {
            System.out.println("main------>" + i);
            if (i == 200) {
                testStop.stop();
            }
        }
    }
}

```

### sleep

- sleep(时间)指定当前线程阻塞的毫秒数
- sleep存在异常InterruptedException
- sleep时间达到后线程进入就绪状态
- sleep可以模拟网络延时，倒计时等
- 每一个对象都有一把锁，sleep不会释放锁

#### sleep的作用

##### 模拟网络延迟

##### 模拟倒计时

```java
/**
 * @author ：zsy
 * @date ：Created 2021/4/13 14:55
 * @description：模拟倒计时
 */
public class TestSleep1 {

    //打印当前系统时间
    public static void main(String[] args) {
        /*try {
            tenDown();
        } catch (InterruptedException e) {
            e.printStackTrace();
        }*/
        Date nowTime = new Date();
        while(true) {
            nowTime = new Date();
            System.out.println(new SimpleDateFormat("HH:mm:ss SSS").format(nowTime));
            try {
                Thread.sleep(1000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }

    }


    //模拟倒计时
    public static void tenDown() throws InterruptedException {
        int num = 10;
        while(true) {
            if (num <= 0) break;
            Thread.sleep(1000);
            System.out.println(num--);
        }
    }
}

```

### yield

- 礼让线程，让当前正在执行的线程暂停，但不阻塞
- 将线程从运行状态转为就绪状态
- 让CPU重新调度，礼让不一定成功

```java
/**
 * @author ：zsy
 * @date ：Created 2021/4/13 15:09
 * @description：yiled方法
 */
public class TestYield implements Runnable{

    @Override
    public void run() {
        System.out.println(Thread.currentThread().getName() + "-----> Start");
        Thread.yield();
        System.out.println(Thread.currentThread().getName() + "------>End");
    }

    public static void main(String[] args) {
        TestYield testYield = new TestYield();
        new Thread(testYield, "A").start();
        new Thread(testYield, "B").start();

    }
}

B-----> Start
A-----> Start
B------>End
A------>End
```

### 为什么Thread类的sleep()和yield()的方法是静态的

Thread类的sleep()和yield()方法将在当前正在运行的线程上运行，所以在其他处于等待状态的线程上调用这些方法是没有意义的，这就是为什么这些方法是静态的，它们可一在当前正在执行的线程中工作，并避免程序员错误的认为可以在其他非运行的线程调用这些方法

### 线程的 sleep()方法和 yield()方法有什么区别？

1） sleep()方法给其他线程运行机会时不考虑线程的优先级，因此会给低优先级的线程以运行的机会；yield()方法只会给相同优先级或更高优先级的线程以运行的机会；

（2） 线程执行 sleep()方法后转入阻塞（blocked）状态，而执行 yield()方法后转入就绪（ready）状态；

（3）sleep()方法声明抛出 InterruptedException，而 yield()方法没有声明任何异常；

（4）sleep()方法比 yield()方法（跟操作系统 CPU 调度相关）具有更好的可移植性，通常不建议使用yield()方法来控制并发线程的执行。

### join

- Join合并线程，待此线程执行完成之后，在执行其他线程，其他线程阻塞

```java
/**
 * @author ：zsy
 * @date ：Created 2021/4/13 15:21
 * @description：join
 */
public class TestJoin implements Runnable {
    @Override
    public void run() {
        for (int i = 0; i < 200; i++) {
            System.out.println("线程vip------>" + i);
        }
    }

    public static void main(String[] args) {
        TestJoin testJoin = new TestJoin();
        Thread thread = new Thread(testJoin);
        thread.start();
        for (int i = 0; i < 300; i++) {
            if(i == 0) {
                try {
                    thread.join();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
            System.out.println("main-------->" + i);
        }
    }
}

```

### state

- NEW

  尚未启动的线程处于此状态

- RUNNABLE

  在Java虚拟机中执行的线程处于此状态

- BLOCKED

  被阻塞等待监视器锁定的线程处于此状态。

- WAITING

  正在等待另一个线程执行特定动作的线程处于此状态。 

- TIMED_WAITING

  正在等待另一个线程执行动作达到指定等待时间的线程处于此状态。 

- TERMINATED

  已退出的线程处于此状态

```java
/**
 * @author ：zsy
 * @date ：Created 2021/4/13 15:40
 * @description：测试线程状态
 */
public class TestState {
    public static void main(String[] args) {
        Thread thread = new Thread(() -> {
            for (int i = 0; i < 5; i++) {
                try {
                    Thread.sleep(100);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
            System.out.println("END");
        });
        Thread.State state = thread.getState();
        System.out.println(state);//NEW
        thread.start();

        state = thread.getState();
        System.out.println(state);//RUNNABLE

        while(thread.getState() != Thread.State.TERMINATED) {
            try {
                Thread.sleep(10);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            state = thread.getState();
            System.out.println(state);
        }
    }
}

NEW
RUNNABLE
TIMED_WAITING
TIMED_WAITING
TIMED_WAITING
TIMED_WAITING
TIMED_WAITING
TIMED_WAITING
TIMED_WAITING
TIMED_WAITING
TIMED_WAITING
TIMED_WAITING
TIMED_WAITING
TIMED_WAITING
TIMED_WAITING
TIMED_WAITING
TIMED_WAITING
TIMED_WAITING
TIMED_WAITING
TIMED_WAITING
TIMED_WAITING
TIMED_WAITING
TIMED_WAITING
TIMED_WAITING
TIMED_WAITING
TIMED_WAITING
TIMED_WAITING
TIMED_WAITING
TIMED_WAITING
TIMED_WAITING
TIMED_WAITING
TIMED_WAITING
TIMED_WAITING
TIMED_WAITING
TIMED_WAITING
TIMED_WAITING
TIMED_WAITING
TIMED_WAITING
TIMED_WAITING
TIMED_WAITING
TIMED_WAITING
TIMED_WAITING
TIMED_WAITING
TIMED_WAITING
TIMED_WAITING
TIMED_WAITING
TIMED_WAITING
TIMED_WAITING
END
TERMINATED
```

### priority

- Java提供一个线程调度器来监控程序中启动后进入就绪状态的所有线程，线程调度器按照优先级决定应该调度那个线程来执行
- 线程优先级用数字表示，范围从1~10
  - Thread.MIN_PRIORITY = 1
  - Thread.MAX_PRIORITY = 10
  - Thread.NORM_PRIORITY = 5
- 使用以下方法改变和获取优先级
  - getPriority() 
  - setPriority()

### 守护线程

- 线程分为**守护线程**和**用户线程**

  ```java
  thread.setDaemon(true); //默认是false 表示是用户线程，正常都是用户线程。。
  ```

- 虚拟机必须确保用户线程执行完毕

- 虚拟机不用等待守护线程执行完毕

- 如：后台记录操作日志，监控内存，垃圾回收等…

### 守护线程和用户线程有什么区别

守护线程和用户线程

- 用户 (User) 线程：运行在前台，执行具体的任务，如程序的主线程、连接网络的子线程等都是用户线程
- 守护 (Daemon) 线程：运行在后台，为其他前台线程服务。也可以说守护线程是 JVM 中非守护线程的 “佣人”。一旦所有用户线程都结束运行，守护线程会随 JVM 一起结束工作

> 比较明显的区别之一是用户线程结束，JVM 退出，不管这个时候有没有守护线程运行。而守护线程不会影响 JVM 的退出。

注意事项：

1. setDaemon(true)必须在start()方法前执行，否则会抛出 IllegalThreadStateException 异常
2. 在守护线程中产生的新线程也是守护线程
3. 不是所有的任务都可以分配给守护线程来执行，比如读写操作或者计算逻辑
4. 守护 (Daemon) 线程中不能依靠 finally 块的内容来确保执行关闭或清理资源的逻辑。因为我们上面也说过了一旦所有用户线程都结束运行，守护线程会随 JVM 一起结束工作，所以守护 (Daemon) 线程中的 finally 语句块可能无法被执行。

## 中断线程

- 线程执行完毕会自行结束
- 在线程处于阻塞，期限等待或无限期等待时，调用线程的interrupt()会抛出interruptedException异常从而提前终止线程
- 若线程没有处于阻塞状态，调用interrupte()将线程标记为中断，此时调用interrupted()判断线程是否处于中断状态，来提前终止线程
- 线程池调用shutdown（）等待所有线程执行完毕之后关闭

### Java中interrupted和isInterruped方法的区别

interrupted：用于中断线程。调用该方法的线程的状态会被置为“中断”状态

> 线程中断仅仅是置线程的中断状态位，不会停止线程。需要用户自己去监视线程的状态为并做处理。支持线程中断的方法（也就是线程中断后会抛出interruptedException 的方法）就是在监视线程的中断状态，一旦线程的中断状态被置为“中断状态”，就会抛出中断异常。

interrupted：是静态方法，查看当前中断信号是true还是false并且清楚中断信号。如果一个线程被中断了，第一次调用interrupted则返回true，第二次和后面返回的就是false了。

isInterrupted：查看当前中断信号是true还是false。

## 什么是阻塞式方法？

阻塞式方法是指程序会一直等待该方法完成期间不做其他事情，ServerSocket 的accept()方法就是一直等待客户端连接。这里的阻塞是指调用结果返回之前，当前线程会被挂起，直到得到结果之后才会返回。此外，还有异步和非阻塞式方法在任务完成前就返回。

# 线程同步

- 问题的提出：
  - 多个线程执行的不确定性引起执行结果的不稳定性
  - 多个线程对账本的共享，会造成操作的不完整性，会破坏数据。
- 由于同一个进程的多个线程共享同一块存储空间，在带来方便的同时，也带来了访问冲突问题，为了保证数据在方法中被访问时的正确性，在访问时加入**锁机制synchronized**，当一个线程获得对象的排它锁，独占资源，其他线程必须等待，使用后释放锁即可，存在问题：
  - 一个线程持有锁会导致其他所需要此锁的线程挂起
  - 在多个线程竞争下，加锁，释放锁会导致比较多的上下文切换和调度延时，引起性能问题
  - 如果一个优先级高的线程等待一个优先级的线程释放锁，会导致优先级倒置，引起性能问题

## 线程不安全案例

```java
/**
 * @author ：zsy
 * @date ：Created 2021/4/13 16:51
 * @description：模拟线程不安全案例
 */
public class UnsafeBank {
    public static void main(String[] args) {
        Account account = new Account("慈善基金", 100.0f);
        Drawing ming = new Drawing(account, 50.0f, "小明");
        Drawing hua = new Drawing(account,  100.0f, "小华");
        ming.start();
        hua.start();
    }
}

//账户类
class Account {
    public String name;
    public float balance;

    public Account(String name, float balance) {
        this.name = name;
        this.balance = balance;
    }


}

//模拟取款
class Drawing extends Thread{
    private Account account;
    private float drawingMoney;

    public Drawing(Account account, float drawingMoney, String name) {
        super(name);
        this.account = account;
        this.drawingMoney = drawingMoney;
    }

    @Override
    public void run() {
        super.run();
        if (account.balance < drawingMoney) {
            System.out.println("余额不足");
            return;
        }

        //模拟网络延时
        try {
            Thread.sleep(1000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        this.account.balance = this.account.balance - drawingMoney;
        System.out.println(this.getName() + "取出了" + drawingMoney);
        System.out.println(this.account.name + "余额" + this.account.balance);
    }
}

小明取出了50.0
慈善基金余额-50.0
小华取出了100.0
慈善基金余额-50.0
```

## 同步方法

- synchronized 方法 和 synchronized 块

```java
同步方法： public  synchronized void method(int args){}
```

- synchronized 方法控制“对象”的访问，每个对象对应一把锁，每个 synchronized 方法都必须获得调用该方法的对象的锁才能够执行，否则线程会阻塞，方法一旦执行，就独占该锁，知道方法返回才能释放锁，后面被阻塞的线程才能获得这个锁，继续执行

缺陷： 若将一个大的方法申明为  synchronized 将会影响效率

### 同步方法

```java
// synchronized 同步方法，锁的是this
    private synchronized void buy() throws InterruptedException {
        //判断是否有票
        if(ticketNums<=0){
            flag = false;
            return;
        }
        //模拟延时
        Thread.sleep(100);
        //买票
        System.out.println(Thread.currentThread().getName()+"拿到"+ticketNums--);
    }
```

### 同步块

- 同步块：synchronized(Obj) {}
- Obj称为同步监视器
  - Obj可以是任何对象，但是推荐使用共享资源作为同步监视器
  - 同步方法中无需指定同步监视器，因为同步方法的同步监视器就是this，就是这个对象本身，或者是class[反射]
- 同步监视器的执行过程
  1. 第一个线程访问，锁定同步监视器，执行其中代码
  2. 第二个线程访问，发现同步监视器被锁定，无法访问
  3. 第一个线程访问完毕，解锁同步监视器
  4. 第二个线程访问，发现同步监视器没有锁，然后锁定并访问

```java
//模拟取款
class Drawing extends Thread{
    private Account account;
    private float drawingMoney;

    public Drawing(Account account, float drawingMoney, String name) {
        super(name);
        this.account = account;
        this.drawingMoney = drawingMoney;
    }

    @Override
    public void run() {
        super.run();
        synchronized (account) {//锁的是用户
            if (account.balance < drawingMoney) {
                System.out.println("余额不足");
                return;
            }

            //模拟网络延时
            try {
                Thread.sleep(1000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            this.account.balance = this.account.balance - drawingMoney;
            System.out.println(this.getName() + "取出了" + drawingMoney);
            System.out.println(this.account.name + "余额" + this.account.balance);
        }
    }
}

```

## 死锁

多个线程各自占有一些共享资源，并且互相等待其他线程占有的资源才能运行，而导致两个或者多个线程都在等待对方释放资源，都停止运行的情形，某一个同步块同时拥有“两个以上对象的锁”时，可能会发生死锁问题

### 死锁的案例

```java
/**
 * @author ：zsy
 * @date ：Created 2021/4/13 19:54
 * @description：死锁
 */
public class DeadLock {

    public static void main(String[] args) {
        Makeup makeup1 = new Makeup(0, "灰姑娘");
        Makeup makeup2 = new Makeup(1, "白雪公主");
        makeup1.start();
        makeup2.start();
    }
}

class Lipstick {

}

class Mirror {

}

class Makeup extends Thread {

    //只有一份使用static关键字
    static Lipstick lipstick = new Lipstick();
    static Mirror mirror = new Mirror();

    int choice;
    String name;

    public Makeup (int choice, String name){
        this.choice = choice;
        this.name = name;
    }

    @Override
    public void run() {
        super.run();
        try {
            makeup();
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }

    //化妆
    public void makeup() throws InterruptedException {
        if (choice == 0) {
            synchronized (lipstick) {
                System.out.println(this.getName() + "拿到了口红");
                Thread.sleep(1000);
                synchronized (mirror) {
                    System.out.println(this.getName() + "拿到了镜子");
                }
            }
        } else {
            synchronized (mirror) {
                System.out.println(this.getName() + "拿到了镜子");
                Thread.sleep(2000);
                synchronized (lipstick) {
                    System.out.println(this.getName() + "拿到了口红");
                }
            }
        }
    }
}

```

分析：造成原因：某一个同步块同时拿到了两个对象的锁

解决方法：同一个同步块中只允许拿到一个对象的锁

![死锁](https://gitee.com/zhang-songyao/blog-images/raw/master/20210420234800.png)

```java
public void makeup() throws InterruptedException {
        if (choice == 0) {
            synchronized (lipstick) {
                System.out.println(this.getName() + "拿到了口红");
                Thread.sleep(1000);
            }
            synchronized (mirror) {
                System.out.println(this.getName() + "拿到了镜子");
            }
        } else {
            synchronized (mirror) {
                System.out.println(this.getName() + "拿到了镜子");
                Thread.sleep(2000);
            }
            synchronized (lipstick) {
                System.out.println(this.getName() + "拿到了口红");
            }
        }
    }
```

### 产生死锁的条件

- 互斥条件：一个资源每次只能被一个线程使用
- 请求与保持条件：一个线程因请求资源而阻塞时，对已获得的资源保持不放
- 不剥夺条件：线程已获得的资源，在未使用完之前，不能强行剥夺
- 循环等待条件：若干个线程之间形成一种头尾相接的循环等待资源关系

上面四个产生死锁的必要条件，只要想办法破其中的任意一个或者多个条件就可以避免死锁的发生。

### 如何避免线程死锁

- 破坏互斥条件：这个条件没有办法破坏，因为我们用锁本来就是想让他们互斥（临界资源需要互斥访问）
- 破坏请求与保持条件：一次性申请所有请求
- 破坏不剥夺条件：占用部分资源的线程进一步申请其他资源时，如果申请不到，可以主动释放它占有的资源
- 破坏循环等待条件：靠按序申请资源来预防。按某一顺序申请资源，释放资源则反序释放。

### 排查死锁

> jps : JavaVirtual Machine Process Status Tool

jps -l  定位进程号

jps –v :输出jvm参数

jps –q ：仅仅显示java进程号

jstack 进程号：查看死锁信息

## 锁

- 从JDK5.0开始，Java提供了更强大的线程同步机制-----通过显示定义同步锁对象来实现同步。同步锁使用Lock对象充当
- java.util.concurrent.locks.Lock接口是控制多个线程对共享资源进行访问的工具。锁提供了对共享资源的独占访问，每次只能有一个线程对Lock对象加锁，线程开始访问共享资源之前应先获得Lock对象
- ReentrantLock类实现了Lock，它拥有与synchronized相同的并发性和内存语义，在实现线程安全的控制中，比较常用的是ReentrantLock，可以显示加锁、释放锁。

### 案例

```java
/**
 * @author ：zsy
 * @date ：Created 2021/4/13 20:23
 * @description：锁
 */
public class TestLock {
    public static void main(String[] args) {
        TestLock2 testLock2 = new TestLock2();
        new Thread(testLock2).start();
        new Thread(testLock2).start();
        new Thread(testLock2).start();
    }
}

class TestLock2 implements Runnable{
    int ticketNum = 10;

    private final Lock lock = new ReentrantLock();

    @Override
    public void run() {
        lock.lock();
        try {
            while (true) {
                if (ticketNum <= 0) break;
                try {
                    Thread.sleep(1000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                System.out.println(ticketNum--);
            }
        }finally {
            lock.unlock();
        }

    }
}

```

## AQS

(AbstartQueueSynchronizer)抽象队列同步器，它的定位是为Java中几乎所有的锁和同步器提供一个基础的框架，AQS是基于FIFO的队列实现的，并且内部维护了一个状态变量state，通过原子更新这个状态变量state既可以实现加锁解锁操作。

### 属性

>volatile int waitStatus;

Node节点另外一个重要的成员是waitStatus，它表示节点等待在队列中的状态：

- CANCELLED：表示线程取消了等待。如果取得锁的过程中发生了一些异常，则可能出现取消的情况，比如等待过程中出现了中断异常或者出现了timeout。进入该状态的节点将不会在变化
- SIGNAL：表示后续节点需要被唤醒。
- CONDITION：线程等待在条件变量队列中。当其他线程调用了Condition的signal()方法后，CONDITION状态的结点将**从等待队列转移到同步队列中**，等待获取同步锁。
- PROPAGATE：在共享模式下，无条件传播releaseShared状态。前继结点不仅会唤醒其后继结点，同时也可能会唤醒后继的后继结点。
  0： 初始状态
- 其中CANCELLED=1，SIGNAL=-1，CONDITION=-2，PROPAGATE=-3 。在具体的实现中，就可以简单的通过waitStatus释放小于等于0，来判断是否是CANCELLED状态。

### 内部结构

在AbstractQueuedSynchronizer内部，有一个队列，我们把它叫做同步等待队列。它的作用是保存等待在这个锁上的线程(由于lock()操作引起的等待）。此外，为了维护等待在条件变量上的等待线程，AbstractQueuedSynchronizer又需要再维护一个条件变量等待队列，也就是那些由Condition.await()引起阻塞的线程。

下面的类图展示了代码层面的具体实现：

![img](https://gitee.com/zhang-songyao/blog-images/raw/master/20210421220510.png)

可以看到，无论是同步等待队列，还是条件变量等待队列，都使用同一个Node类作为链表的节点。对于同步等待队列，Node中包括链表的上一个元素prev，下一个元素next和线程对象thread。对于条件变量等待队列，还使用nextWaiter表示下一个等待在条件变量队列中的节点。

#### 同步队列

##### 结构图

![v2-237a75dd55c58af0080e76d1e2025da8_b.jpg](https://gitee.com/zhang-songyao/blog-images/raw/master/20210604000431.png)

它维护了一个volatile int state（代表共享资源）和一个FIFO线程等待队列（多线程争用资源被阻塞时会进入此队列）。这里volatile是核心关键词。

```java
static final class Node {
    /** Marker to indicate a node is waiting in shared mode */
    static final Node SHARED = new Node();
    /** Marker to indicate a node is waiting in exclusive mode */
    static final Node EXCLUSIVE = null;
    /** waitStatus value to indicate thread has cancelled */
    static final int CANCELLED =  1;
    /** waitStatus value to indicate successor's thread needs unparking */
    static final int SIGNAL    = -1;
    /** waitStatus value to indicate thread is waiting on condition */
    static final int CONDITION = -2;

    static final int PROPAGATE = -3;
    volatile Node next;
    volatile Thread thread;
    Node nextWaiter;
    final boolean isShared() {
        return nextWaiter == SHARED;
    }
    final Node predecessor() throws NullPointerException {
        Node p = prev;
        if (p == null)
            throw new NullPointerException();
        else
            return p;
    }
    Node() {    // Used to establish initial head or SHARED marker
    }
    Node(Thread thread, Node mode) {     // Used by addWaiter
        this.nextWaiter = mode;
        this.thread = thread;
    }
    Node(Thread thread, int waitStatus) { // Used by Condition
        this.waitStatus = waitStatus;
        this.thread = thread;
    }
}
```

#### 条件队列

由于一个重入锁可以生成多个条件变量对象，因此，一个重入锁就可能有多个条件变量等待队列。实际上，每个条件变量对象内部都维护了一个等待列表。其逻辑结构如下所示：

##### 结构图

![img](https://gitee.com/zhang-songyao/blog-images/raw/master/20210421220509.png)

> 参考：[Java 并发高频面试题：聊聊你对 AQS 的理解？](https://blog.csdn.net/qq_35190492/article/details/115339297?ops_request_misc=%257B%2522request%255Fid%2522%253A%2522161907923316780366571275%2522%252C%2522scm%2522%253A%252220140713.130102334.pc%255Fblog.%2522%257D&request_id=161907923316780366571275&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~blog~first_rank_v2~rank_v29-2-115339297.nonecase&utm_term=aqs)

![image-20210421215948408](https://gitee.com/zhang-songyao/blog-images/raw/master/20210421220511.png)

### 条件等待队列处理过程

![img](https://gitee.com/zhang-songyao/blog-images/raw/master/20210422161436.png)

Condition对象的signal()通知
signal()通知的时候，是在条件等待队列中，按照FIFO进行，首先从第一个节点下手：

![img](https://gitee.com/zhang-songyao/blog-images/raw/master/20210422161437.png)

### LockSupport

从上面内容我们注意到，线程的阻塞和唤醒都使用到了LockSupport的方法，LockSupport是用来创建锁和其他同步类的基本线程阻塞原语。

LockSupport定义了一组以park开头的方法用来阻塞线程，以及unpark来唤醒一个被阻塞的线程。

LockSupport提供的方法：

- public static void park() : 如果没有可用许可，则挂起当前线程
- public static void unpark(Thread thread)：给thread一个可用的许可，让它得以继续执行
- 和语句的执行顺序无关

![这里写图片描述](https://gitee.com/zhang-songyao/blog-images/raw/master/20210604000452.png)

>内部都是通过Unsafe实现的		

## 共享锁与排它锁

> 参考：[Java并发面试问题，谈谈你对AQS的理解](https://blog.csdn.net/qq_20499001/article/details/103489988?utm_medium=distribute.pc_relevant.none-task-blog-baidujs_title-1&spm=1001.2101.3001.4242)

### 排它锁

#### lock

[具体的加锁过程](https://blog.csdn.net/qq_45770532/article/details/108111160?utm_medium=distribute.pc_relevant.none-task-blog-baidujs_utm_term-0&spm=1001.2101.3001.4242)

> Tip：简述一下lock的过程，默认为非公平锁（排他锁）每次只获取1，首先尝试修改state修改成功，获取当前执行资格，开始执行。修改失败,acquire(1)去获取资源，首先尝试获取。tryAcquire(),被四个不同的类重写，尝试获取失败，将当前线程封装成节点，放入同步等待队列，并且寻找到一个前驱节点为SINGAL的节点后挂起，这里可以说是整个加锁的核心了，因为第一个节点是正在运行的线程，所以从第二个节点才能去尝试获取锁，获取锁的过程中，难免一些线程会出现异常中断，自旋的过程中，删除这些已经取消等待的节点，并且将其余节点的waitStatus置为SINGAL等待唤醒，如果循环过程中，第二个节点获取到了资源，则跳出循环。

```java
final void lock() {
    //直接CAS交换state和自己的acquires
    //交换成功，那到当前运行时间片
    if (compareAndSetState(0, 1))
        setExclusiveOwnerThread(Thread.currentThread());
    else
        //排他锁获取资源
        acquire(1);
}
```

#### acquire

> 请求锁

```java
public final void acquire(int arg) {
    //尝试获得许可， arg为许可的个数。对于重入锁来说，每次请求1个。
    if (!tryAcquire(arg) &&
    // 如果tryAcquire 失败，则先使用addWaiter()将当前线程加入同步等待队列
    // 然后继续尝试获得锁
    acquireQueued(addWaiter(Node.EXCLUSIVE), arg))
    selfInterrupt();
}
```

进入一步看一下tryAcquire()函数。该函数的作用是尝试获得一个许可。对于AbstractQueuedSynchronizer来说，这是一个未实现的抽象函数。

![image-20210422140755270](https://gitee.com/zhang-songyao/blog-images/raw/master/20210423172233.png)

```java
protected boolean tryAcquire(int arg) {
	throw new UnsupportedOperationException();
}
```

具体实现在子类中。在重入锁，读写锁，信号量等实现中， 都有各自的实现。

如果tryAcquire()成功，则acquire()直接返回成功。如果失败，就用addWaiter()将当前线程加入同步等待队列。

#### addWaiter

```java
private Node addWaiter(Node mode) {
    //Node中维护当前线程对象
    Node node = new Node(Thread.currentThread(), mode);
    // Try the fast path of enq; backup to full enq on failure
    //将节点加到队列尾端，这是一个快速的办法，可能失败
    //复杂的尝试，获取性能
    Node pred = tail;
    if (pred != null) {
        node.prev = pred;
        if (compareAndSetTail(pred, node)) {
            pred.next = node;
            return node;
        }
    }
    //如果快速加入失败，将node加到队列末尾
    enq(node);
    return node;
}
```

接着， 对已经在队列中的线程请求锁，使用acquireQueued()函数，从函数名字上可以看到，其参数node，必须是一个已经在队列中等待的节点。它的功能就是为已经在队列中的Node请求一个许可。

这个函数大家要好好看看，因为无论是普通的lock()方法，还是条件变量的await()都会使用这个方法。

#### acquireQueued

```java
final boolean acquireQueued(final Node node, int arg) {
    boolean failed = true;//标记是否成功拿到资源
    try {
        boolean interrupted = false;//标记等待过程中是否被中断过
        
        //又是一个“自旋”！
        for (;;) {
            final Node p = node.predecessor();//拿到前驱
            //如果前驱是head，即该结点已成老二，那么便有资格去尝试获取资源（可能是老大释放完资源唤醒自己的，当然也可能被interrupt了）。
            if (p == head && tryAcquire(arg)) {
                setHead(node);//拿到资源后，将head指向该结点。所以head所指的标杆结点，就是当前获取到资源的那个结点或null。
                p.next = null; // setHead中node.prev已置为null，此处再将head.next置为null，就是为了方便GC回收以前的head结点。也就意味着之前拿完资源的结点出队了！
                failed = false; // 成功获取资源
                return interrupted;//返回等待过程中是否被中断过
            }
            
            //如果自己可以休息了，就通过park()进入waiting状态，直到被unpark()。如果不可中断的情况下被中断了，那么会从park()中醒过来，发现拿不到资源，从而继续进入park()等待。
            if (shouldParkAfterFailedAcquire(p, node) &&
                parkAndCheckInterrupt())
                interrupted = true;//如果等待过程中被中断过，哪怕只有那么一次，就将interrupted标记为true
        }
    } finally {
        if (failed) // 如果等待过程中没有成功获取资源（如timeout，或者可中断的情况下被中断了），那么取消结点在队列中的等待。
            cancelAcquire(node);
    }
}
```

#### CAS自旋volatile变量

> 什么是自旋

当一个线程拿不到锁的时候，可以不放弃CPU，空转，不断重试，也就是所谓的自旋，对于多CPU或者多核，自旋就很有用了，因为没有线程切换的开销。

AtomicInteger类的getAndIncrement()的源代码：

```java
public final int getAndIncrement() {
    for (;;) {
		int current = get();  // 取得AtomicInteger里存储的数值
		int next = current + 1;  // 加1
		if (compareAndSet(current, next))   // 调用compareAndSet执行原子更新操作
			return current;
	}
}
```

- compareAndSet()方法首先判断当前值是否等于current
- 如果当前值等于current，说明AtomicInteger的值没有被修改
- 如果不等于，说明被修改，这时会再次进入循环进行等待

 compareAndSwapInt 基于的是CPU 的 CAS指令来实现的。所以基于 CAS 的操作可认为是无阻塞的，一个线程的失败或挂起不会引起其它线程也失败或挂起。并且由于 CAS 操作是 CPU 原语，所以性能比较好。

他所利用的是基于冲突检测的乐观并发策略。 可以想象，这种乐观在线程数目非常多的情况下，失败的概率会指数型增加。

>predecessor拿到队列的头节点

```java
final Node predecessor() throws NullPointerException {
    Node p = prev;
    if (p == null)
        throw new NullPointerException();
    else
        return p;
}
```

#### shouldParkAfterFailedAcquire

> 此方法主要用于检查状态，看看自己是否真的可以去休息了，万一队列前边的线程都放弃了,自己直接可以执行

```java
private static boolean shouldParkAfterFailedAcquire(Node pred, Node node) {
    int ws = pred.waitStatus;//拿到前驱的状态
    if (ws == Node.SIGNAL)
        //如果已经告诉前驱拿完号后通知自己一下，那就可以安心休息了
        return true;
    if (ws > 0) {
        /*
         * 如果前驱放弃了，那就一直往前找，直到找到最近一个正常等待的状态，并排在它的后边。
         * 注意：那些放弃的结点，由于被自己“加塞”到它们前边，它们相当于形成一个无引用链，稍后就会被保安大叔赶走了(GC回收)！
         */
        do {
            node.prev = pred = pred.prev;
        } while (pred.waitStatus > 0);
        pred.next = node;
    } else {
         //如果前驱正常，那就把前驱的状态设置成SIGNAL，告诉它拿完号后通知自己一下。有可能失败，人家说不定刚刚释放完呢！
        compareAndSetWaitStatus(pred, ws, Node.SIGNAL);
    }
    return false;
}      
```

#### parkAndCheckInterrupt

>如果线程找好安全休息点后，那就可以安心去休息了。此方法就是让线程去休息，真正进入等待状态。

```java
private final boolean parkAndCheckInterrupt() {
    //线程会停在这里
     LockSupport.park(this);//调用park()使线程进入waiting状态
     return Thread.interrupted();//如果被唤醒，查看自己是不是被中断的。
 }
```

park()会让当前线程进入waiting状态。在此状态下，有两种途径可以唤醒该线程：1）被unpark()；2）被interrupt()。需要注意的是，Thread.interrupted()会清除当前线程的中断标记位。

#### 总结

##### acquireQueued的具体流程

1. 节点进入队尾，检查状态，找到安全的休息点；
2. 调用park()进入wait，等待unpark()或interrupt()唤醒
3. 被唤醒后，看自己是不是有资格拿到资源，如果拿到，head指向当前节点，并返回从入队到拿到号整个过程是否被中断过。如果没有拿到，继续流程1

##### acquire的具体流程

1. 调用自定义同步器的tryAcquire()获取资源，如果成功则直接返回
2. 没成功，则addWaiter()将该线程加入等待队列的尾部，并标记为独占模式
3. acquireQueued()使线程在等待队列中休息，有机会时（轮到自己，会被unpark()）会去尝试获取资源。获取到资源后才返回。如果在整个等待过程中被中断过，则返回true，否则返回false。
4. 如果线程在等待过程中被中断过，他是不响应的，只是在获取资源后再进行自我中断selfInterrupt()，将中断补上

![img](https://gitee.com/zhang-songyao/blog-images/raw/master/20210604000514.png)

#### release(int)

> 此方法是独占模式下线程释放共享资源的顶层入口。他会释放指定量的资源，如果彻底释放了(state = 0)，他会唤醒等待队列里的其他线程来获取资源，这也正是unlock()的语义

```java
public final boolean release(int arg) {
    if (tryRelease(arg)) {
        Node h = head;//找到头结点
        if (h != null && h.waitStatus != 0)
            unparkSuccessor(h);//唤醒等待队列里的下一个线程
        return true;
    }
    return false;
} 
```

逻辑并不复杂。它调用tryRelease()来释放资源。有一点需要注意的是，**它是根据tryRelease()的返回值来判断该线程是否已经完成释放掉资源了！所以自定义同步器在设计tryRelease()的时候要明确这一点！！**

#### tryRelease(int)

此方法尝试去释放指定量的资源。如果已经彻底释放资源(state=0)，要返回true，否则返回false。下面是tryRelease()的源码：

```java
protected boolean tryRelease(int arg) {
     throw new UnsupportedOperationException();
}
```

####  unparkSuccessor(Node)

```java        
private void unparkSuccessor(Node node) {
    //这里，node一般为当前线程所在的结点。
    int ws = node.waitStatus;
    if (ws < 0)//置零当前线程所在的结点状态，允许失败。
        compareAndSetWaitStatus(node, ws, 0);
 
    Node s = node.next;//找到下一个需要唤醒的结点s
    if (s == null || s.waitStatus > 0) {//如果为空或已取消
        s = null;
        for (Node t = tail; t != null && t != node; t = t.prev) // 从后向前找。
            if (t.waitStatus <= 0)//从这里可以看出，<=0的结点，都是还有效的结点。
                s = t;
    }
    if (s != null)
        LockSupport.unpark(s.thread);//唤醒
}
```

> 用unpark()唤醒等待队列中最前面的那个未放弃的线程再和acquireQueued()联系起来，s被唤醒后，进入if (p == head && tryAcquire(arg))的判断（即使p!=head也没关系，它会再进入shouldParkAfterFailedAcquire()寻找一个安全点。这里既然s已经是等待队列中最前边的那个未放弃线程了，那么通过shouldParkAfterFailedAcquire()的调整，s也必然会跑到head的next结点，下一次自旋p==head就成立啦），然后s把自己设置成head标杆结点，表示自己已经获取到资源了，acquire()也返回了

#### 扩展问题

如果获取锁的线程在release时异常了，没有unpark队列中的其他结点，这时队列中的其他结点会怎么办？是不是没法再被唤醒了？

答案是YES这时，队列中等待锁的线程将永远处于park状态，无法再被唤醒！！！但是我们再回头想想，获取锁的线程在什么情形下会release抛出异常呢？？

1. 线程突然死掉了？可以通过thread.stop来停止线程的执行，但该函数的执行条件要严苛的多，而且函数注明是非线程安全的，已经标明Deprecated；
2. 线程被interupt了？线程在运行态是不响应中断的，所以也不会抛出异常；
3. release代码有bug，抛出异常了？目前来看，Doug Lea的release方法还是比较健壮的，没有看出能引发异常的情形（如果有，恐怕早被用户吐槽了）。除非自己写的tryRelease()有bug，那就没啥说的，自己写的bug只能自己含着泪去承受了。

### 共享锁	

ReentrantLock.ReadLock、Semaohore、CountDownLatch

与排他锁相比，共享锁的实现略微复杂一点。这也很好理解。因为排他锁的场景很简单，单进单出，而共享锁就不一样了。可能是N进M出，处理起来要麻烦一些。但是，他们的核心思想还是一致的。共享锁的几个典型应用有：信号量，读写锁中的写锁。

#### 共享锁执行原理

![img](https://gitee.com/zhang-songyao/blog-images/raw/master/202208242305661.png)

#### acquireShared

```java
public final void acquireShared(int arg) {
    //tryAcquireShared需要在子类中实现
    //他表示尝试获取arg个共享许可，如果tryAcquireShared返回负数，表示失败
    //返回0表示成功，但是没有多余的许可
    if (tryAcquireShared(arg) < 0)
        //申请失败，进入通同步等待队列
        doAcquireShared(arg);
}

```

#### doAcquireShared

```java
private void doAcquireShared(int arg) {
    //加入同步等待队列，指定为SHARED类型
    final Node node = addWaiter(Node.SHARED);
    boolean failed = true;
    try {
        boolean interrupted = false;
        for (;;) {
            final Node p = node.predecessor();
            //只有第二个节点有资格申请，因为是一个FIFO的队列，而第一个节点已经获得了许可
            //因此第二个节点就是第一个需要申请而的节点
            if (p == head) {
                //尝试申请许可
                int r = tryAcquireShared(arg);
                if (r >= 0) {
                    //申请成功，把自己设置为头部
                    //根据条件，判断是否需要唤醒后续线程
                    //如果条件允许，就会尝试传播这个唤醒到后续节点
                    setHeadAndPropagate(node, r);
                    p.next = null; // help GC
                    if (interrupted)
                        selfInterrupt();
                    failed = false;
                    return;
                }
            }
            //没有获取成功，只能park等待
            //将来如果被唤醒，从这里开始执行，又会回到上面的tryAcquireShared
            //和setHeadAndPropagate，去尝试共享许可，并且传播
            if (shouldParkAfterFailedAcquire(p, node) &&
                parkAndCheckInterrupt())
                interrupted = true;
        }
    } finally {
        if (failed)
            cancelAcquire(node);
    }
}
```

其实流程根acquireQueued并没有太大区别。只不过这里将补中断的selfInterrupt()放到doAcquireShared()里了，而独占模式是放到acquireQueued()之外，其实都一样。

跟独占模式比，还有一点需要注意的是，这里只有线程是head.next时（“老二”），才会去尝试获取资源，有剩余的话还会唤醒之后的队友。

那么问题就来了，假如老大用完后释放了5个资源，而老二需要6个，老三需要1个，老四需要2个。老大先唤醒老二，老二一看资源不够，他是把资源让给老三呢，还是不让？答案是否定的！老二会继续park()等待其他线程释放资源，也更不会去唤醒老三和老四了。（保证公平，但降低了并发）。

#### setHeadAndPropagate

```java
//设置标志位，唤醒后续节点
private void setHeadAndPropagate(Node node, int propagate) {
    Node h = head; // Record old head for check below
    //这里取得了许可成功，将当前节点放在头部
    setHead(node);
    //propagate>0表示后续节点的许可申请释放可以成功
    //由propagate和waitStatus来判断能否唤醒后续线程，如果只有propagate来判断
    //在并发环境中，可能出现线程不能被唤醒的情况
    if (propagate > 0 || h == null || h.waitStatus < 0 ||
        (h = head) == null || h.waitStatus < 0) {
        Node s = node.next;
        if (s == null || s.isShared())
            //唤醒下一个线程，设置传播状态
            //被唤醒的线程 又会尝试tryAcquireShared 和 setHeadAndPropagate
            doReleaseShared();
    }
}
```

此方法在setHead()的基础上多了一步，就是自己苏醒的同时，如果条件符合（比如还有剩余资源），还会去唤醒后继结点，毕竟是共享模式！

#### 总结

acquiredShared()流程

1. tryAcquiredShared()尝试成功获取资源，成功则直接返回
2. 失败通过doAcquireShared()进入等待队列，直到被unpark()/interrupt()并成功获取到资源才返回。整个等待过程也是忽略中断的。

#### releaseShared

此方法是共享模式下线程释放共享资源的顶层入口。它会释放指定量的资源，如果成功释放且允许唤醒等待线程，它会唤醒等待队列里的其他线程来获取资源。

```java
public final boolean releaseShared(int arg) {
    //尝试释放许可，需要在子类中实现
    if (tryReleaseShared(arg)) {
        doReleaseShared();
        return true;
    }
    return false;
}
```

释放掉资源后，唤醒后继。跟独占模式下的release()相似，但有一点稍微需要注意：独占模式下的tryRelease()在完全释放掉资源（state=0）后，才会返回true去唤醒其他线程，这主要是基于独占下可重入的考量；而共享模式下的releaseShared()则没有这种要求，共享模式实质就是控制一定量的线程并发执行，那么拥有资源的线程在释放掉部分资源时就可以唤醒后继等待结点。

例如，资源总量是13，A（5）和B（7）分别获取到资源并发运行，C（4）来时只剩1个资源就需要等待。

A在运行过程中释放掉2个资源量，然后tryReleaseShared(2)返回true唤醒C，C一看只有3个仍不够继续等待；随后B又释放2个，tryReleaseShared(2)返回true唤醒C，C一看有5个够自己用了，然后C就可以跟A和B一起运行。

而ReentrantReadWriteLock读锁的tryReleaseShared()只有在完全释放掉资源（state=0）才返回true，所以自定义同步器可以根据需要决定tryReleaseShared()的返回值。

#### doReleaseShared

```java
//唤醒下一个线程，或者设置传播状态
private void doReleaseShared() {
    for (;;) {
        Node h = head;
        if (h != null && h != tail) {
            int ws = h.waitStatus;
            if (ws == Node.SIGNAL) {
                //如果需要唤醒后续线程，那么就唤醒，同时设置状态为0
                if (!compareAndSetWaitStatus(h, Node.SIGNAL, 0))
                    continue;            // loop to recheck cases
                unparkSuccessor(h);
            }
            //设置PROPAGATE状态，保证线程唤醒可以传播下去
            else if (ws == 0 &&
                     !compareAndSetWaitStatus(h, 0, Node.PROPAGATE))
                continue;                // loop on failed CAS
        }
        //如果被打扰，重新再来一次
        //否则直接退出
        if (h == head)                   // loop if head changed
            break;
    }
}
```

#### Mutex（互斥锁）

Mutex是一个不可重入的互斥锁实现。锁资源（AQS里的state）只有两种状态：0表示未锁定，1表示锁定。下边是Mutex的核心源码：

```java
class Mutex implements Lock, java.io.Serializable {
    // 自定义同步器
    private static class Sync extends AbstractQueuedSynchronizer {
        // 判断是否锁定状态
        protected boolean isHeldExclusively() {
            return getState() == 1;
        }
 
        // 尝试获取资源，立即返回。成功则返回true，否则false。
        public boolean tryAcquire(int acquires) {
            assert acquires == 1; // 这里限定只能为1个量
            if (compareAndSetState(0, 1)) {//state为0才设置为1，不可重入！
                setExclusiveOwnerThread(Thread.currentThread());//设置为当前线程独占资源
                return true;
            }
            return false;
        }
 
        // 尝试释放资源，立即返回。成功则为true，否则false。
        protected boolean tryRelease(int releases) {
            assert releases == 1; // 限定为1个量
            if (getState() == 0)//既然来释放，那肯定就是已占有状态了。只是为了保险，多层判断！
                throw new IllegalMonitorStateException();
            setExclusiveOwnerThread(null);
            setState(0);//释放资源，放弃占有状态
            return true;
        }
    }
 
    // 真正同步类的实现都依赖继承于AQS的自定义同步器！
    private final Sync sync = new Sync();
 
    //lock<-->acquire。两者语义一样：获取资源，即便等待，直到成功才返回。
    public void lock() {
        sync.acquire(1);
    }
 
    //tryLock<-->tryAcquire。两者语义一样：尝试获取资源，要求立即返回。成功则为true，失败则为false。
    public boolean tryLock() {
        return sync.tryAcquire(1);
    }
 
    //unlock<-->release。两者语文一样：释放资源。
    public void unlock() {
        sync.release(1);
    }
 
    //锁是否占有状态
    public boolean isLocked() {
        return sync.isHeldExclusively();
    }
}
```

同步类在实现时一般都将自定义同步器（sync）定义为内部类，供自己使用；而同步类自己（Mutex）则实现某个接口，对外服务。当然，接口的实现要直接依赖sync，它们在语义上也存在某种对应关系！！而sync只用实现资源state的获取-释放方式tryAcquire-tryRelelase，至于线程的排队、等待、唤醒等，上层的AQS都已经实现好了，我们不用关心。

除了Mutex，ReentrantLock/CountDownLatch/Semphore这些同步类的实现方式都差不多，不同的地方就在获取-释放资源的方式tryAcquire-tryRelelase。掌握了这点，AQS的核心便被攻破了！

## 公平锁和非公平锁

```java
abstract static class Sync extends AbstractQueuedSynchronizer 
```

Sync呢又分别有两个子类：FairSync和NofairSync

![img](https://gitee.com/zhang-songyao/blog-images/raw/master/20210421150327.jpg)

### 公平锁和非公平锁在代码层面怎么体现呢

![image-20210421090136839](https://gitee.com/zhang-songyao/blog-images/raw/master/20210421091021.png)

### 非公平锁图解

A线程准备进去获取锁，首先判断了一下state状态，发现是0，所以可以CAS成功，并且修改了当前持有锁的线程为自己。

![img](https://gitee.com/zhang-songyao/blog-images/raw/master/20210421091022.jpg)

这个时候B线程也过来了，也是一上来先去判断了一下state状态，发现是1，那就CAS失败了，真晦气，只能乖乖去等待队列，等着唤醒了，先去睡一觉吧。

![img](https://gitee.com/zhang-songyao/blog-images/raw/master/20210421091023.jpg)

A持有久了，也有点腻了，准备释放掉锁，给别的仔一个机会，所以改了state状态，抹掉了持有锁线程的痕迹，准备去叫醒B。

![img](https://gitee.com/zhang-songyao/blog-images/raw/master/20210421091024.jpg)

这个时候有个带绿帽子的仔C过来了，发现state怎么是0啊，果断CAS修改为1，还修改了当前持有锁的线程为自己。

B线程被A叫醒准备去获取锁，发现state居然是1，CAS就失败了，只能失落的继续回去等待队列，路线还不忘骂A渣男，怎么骗自己，欺骗我的感情。

![img](https://gitee.com/zhang-songyao/blog-images/raw/master/20210421091025.jpg)

### 非公平锁源码

#### nonfairTryAcquire

```java
final boolean nonfairTryAcquire(int acquires) {
    //获取当前线程
    final Thread current = Thread.currentThread();
    //获取当前的状态
    int c = getState();
    //0代表空闲
    if (c == 0) {
        //直接CAS交换state和自己的acquires
        if (compareAndSetState(0, acquires)) {
            //设置当前资源的拥有者为当前线程(排它锁)
            setExclusiveOwnerThread(current);
            return true;
        }
    }
    //不为0代表当前有线程正在执行
    //而且正在执行的线程是当前线程
    else if (current == getExclusiveOwnerThread()) {
        //重新设置state的值，不能为负数
        int nextc = c + acquires;
        if (nextc < 0) // overflow
            throw new Error("Maximum lock count exceeded")
        setState(nextc);
        return true;
    }
    return false;
}
```

> state：当前线程的状态

```java
/**
* The synchronization state.
*/
private volatile int state;
```

#### tryRelease

```java
protected final boolean tryRelease(int releases) {
    //修改state状态
    int c = getState() - releases;
    if (Thread.currentThread() != getExclusiveOwnerThread())
        throw new IllegalMonitorStateException();
    boolean free = false;
    if (c == 0) {
        free = true;
        //设置当前资源拥有者为空
        setExclusiveOwnerThread(null);
    }
    //修改state
    setState(c);
    return free;
}
```



### 公平锁图解

线A现在想要获得锁，先去判断下state，发现也是0，去看了看队列，自己居然是第一位，果断修改了持有线程为自己。

![img](https://gitee.com/zhang-songyao/blog-images/raw/master/20210421091026.jpg)

线程b过来了，去判断一下state，嗯哼？居然是state=1，那cas就失败了呀，所以只能乖乖去排队了。

![未命名文件 (https://tva1.sinaimg.cn/large/00831rSTly1gcxaojuen2j30oa0jxgmh.jpg)](https://gitee.com/zhang-songyao/blog-images/raw/master/20210421091027.jpg)

线程A暖男来了，持有没多久就释放了，改掉了所有的状态就去唤醒线程B了，这个时候线程C进来了，但是他先判断了下state发现是0，以为有戏，然后去看了看队列，发现前面有人了，作为新时代的良好市民，果断排队去了。

![img](https://gitee.com/zhang-songyao/blog-images/raw/master/20210421091028.jpg)

线程B得到A的召唤，去判断state了，发现值为0，自己也是队列的第一位，那很香呀，可以得到了。

![img](https://gitee.com/zhang-songyao/blog-images/raw/master/20210421091029.jpg)

### 公平锁源码

#### tryAcquire

```java
protected final boolean tryAcquire(int acquires) {
    final Thread current = Thread.currentThread();
    int c = getState();
    if (c == 0) {
        //判断当前等待队列是否为空
        //如果为空并且交换state和acquires成功
        //占有当前资源
        if (!hasQueuedPredecessors() &&
            compareAndSetState(0, acquires)) {
            setExclusiveOwnerThread(current);
            return true;
        }
    }	
    else if (current == getExclusiveOwnerThread()) {
        int nextc = c + acquires;
        if (nextc < 0)
            throw new Error("Maximum lock count exceeded");
        setState(nextc);
        return true;
    }
    return false;
}
```

#### hasQueuedPredecessors

> 代码的大概意思也是判断当前的线程是不是位于同步队列的首位，是就是返回true，否就返回false。

```java
public final boolean hasQueuedPredecessors() {
    // The correctness of this depends on head being initialized
    // before tail and on head.next being accurate if the current
    // thread is first in queue.
    Node t = tail; // Read fields in reverse initialization order
    Node h = head;
    Node s;
    return h != t &&
        ((s = h.next) == null || s.thread != Thread.currentThread());
}
```

### 非公平锁和公平锁的区别

| 区别 |                            公平锁                            |                           非公平锁                           |
| :--: | :----------------------------------------------------------: | :----------------------------------------------------------: |
| 概念 | 多个线程按照申请锁额顺序去获取锁，线程会直接进入队列，永远都是队列的第一位才能获得锁 | 多个线程去获取锁的时候，会直接去获取锁，获取不到，在进入等待队列，如果能获取，就直接获取了 |
| 优点 |            所有线程都能得到资源，不会饿死在队列中            | 可以减少CPU去唤醒线程的开销，整体的吞吐量会高点，CPU不必唤醒所有线程，会减少唤起线程数量 |
| 缺点 | 吞吐量会下降很多，队列除了第一个线程，其他的线程都会阻塞，等待CPU唤醒，CPU唤醒阻塞线程的开销会很大 | 这样可能导致队列中间的线程一直获取不到锁，或者长时间获取不到锁，导致饿死 |

## 乐观锁和悲观锁

### 乐观锁

CAS（Compare And Swap 比较并且替换）是乐观锁的一种体现，是一种轻量级锁，JUC工具类的实现就是基于CAS

> CAS怎么实现线程安全

线程在读取数据时不进行加锁，在准备写会数据时，先查询原值，操作的时候比较原值是否修改，若未被其他线程修改则写回，若已被修改，则重新执行读取流程

比较 + 更新 整体是一个原子操作，当然这个流程还是有问题的

![img](https://gitee.com/zhang-songyao/blog-images/raw/master/20210421234832.jpg)

#### 乐观锁的问题

> ABA问题

<img src="https://gitee.com/zhang-songyao/blog-images/raw/master/20210421234833.jpg" alt="img" style="zoom:80%;" />

看到问题所在没，我说一下顺序：ƒ

线程1读取了数据A
线程2读取了数据A
线程2通过CAS比较，发现值是A没错，可以把数据A改成数据B
线程3读取了数据B
线程3通过CAS比较，发现数据是B没错，可以把数据B改成了数据A
线程1通过CAS比较，发现数据还是A没变，就写成了自己要改的值
懂了么，我尽可能的幼儿园化了，在这个过程中任何线程都没做错什么，但是值被改变了，线程1却没有办法发现，其实这样的情况出现对结果本身是没有什么影响的，但是我们还是要防范。

**加标志、时间戳，值可能相同，但是版本号一定不一样**

> **循环时间长开销大的问题**：

是因为CAS操作长时间不成功的话，会导致一直自旋，相当于死循环了，CPU的压力会很大。

> 只能保证一个共享变量的原子操作

CAS操作单个共享变量的时候可以保证原子的操作，多个变量就不行了，JDK 5之后 AtomicReference可以用来保证对象之间的原子性，就可以把多个对象放入CAS中操作。

那我就拿AtomicInteger举例，他的自增函数incrementAndGet（）就是这样实现的，其中就有大量循环判断的过程，直到符合条件才成功。

### 悲观锁

synchronized，代表这个方法加锁，相当于不管哪一个线程（例如线程A），运行到这个方法时,都要检查有没有其它线程B（或者C、 D等）正在用这个方法(或者该类的其他同步方法)，有的话要等正在使用synchronized方法的线程B（或者C 、D）运行完这个方法后再运行此线程A，没有的话，锁定调用者，然后直接运行。

#### synchronized对对象进行加锁

在JVM中对象在内存中分为三块区域：对象头（Header）、实例数据（InstanceDate）和对齐填充（Padding）。

对象头

以Hotspot虚拟机为例，Hotspot的对象头主要包括两部分数据：Mark Word（标记字段）、Kass Pointer（类型指针）

- Mark Word：默认存储对象的HashCode，分代年龄和锁标志位信息。它会根据对象的状态复用自己的存储空间，也就是说在运行期间Mark Word里存储的数据会随着锁标志位的变化而变化。
- Klass Point：对象指向它的类元数据的指针，虚拟机通过这个指针来确定这个对象是哪个类的实例。

你可以看到在对象头中保存了锁标志位和指向 monitor 对象的起始地址，如下图所示，右侧就是对象对应的 Monitor 对象。

![img](https://gitee.com/zhang-songyao/blog-images/raw/master/20210421234834.jpg)

当 Monitor 被某个线程持有后，就会处于锁定状态，如图中的 Owner 部分，会指向持有 Monitor 对象的线程。

另外 Monitor 中还有两个队列分别是EntryList和WaitList，主要是用来存放进入及等待获取锁的线程。

如果线程进入，则得到当前对象锁，那么别的线程在该类所有对象上的任何操作都不能进行。

#### synchronized对方法进行加锁

在字节码中是通过方法的 ACC_SYNCHRONIZED 标志来实现的。

#### synchronized 应用在同步块上

每个对象都会与一个monitor相关联，当某个monitor被拥有之后就会被锁住，当线程执行到monitorenter指令时，就会去尝试获得对应的monitor。

#### 小结

同步方法和同步代码块底层都是通过monitor来实现同步的。

两者的区别：同步方式是通过方法中的access_flags中设置ACC_SYNCHRONIZED标志来实现，同步代码块是通过monitorenter和monitorexit来实现。

我们知道了每个对象都与一个monitor相关联，而monitor可以被线程拥有或释放。

### CAS和synchronized的使用场景

- 对于资源竞争较少（线程冲突较轻）的情况下，使用synchronized同步锁进行线程阻塞和唤醒切换以及用户内核态间的切换操作额外浪费消耗CPU资源，而CAS基于硬件实现，不需要进入内核，不需要切换线程，操作自旋的几率比较小，因此可以获得更高的性能
- 对于资源竞争严重（线程冲突严重）的情况，CAS自旋的概率比较大，从而浪费更多的CPU资源，效率低于synchronized

## 可重入锁和不可重入锁

### 可重入锁

广义上的可重入锁指的是可重复可递归调用的锁，在外层使用锁之后，在内层仍然可以使用，并且不发生死锁（前提得是同一个对象或者class），这样的锁就叫做可重入锁。ReentrantLock和synchronized都是可重入锁

### 不可重入锁

不可重入锁，与可重入锁相反，不可递归调用，递归调用就发生死锁。看到一个经典的讲解，使用自旋锁来模拟一个不可重入锁，代码如下

```java
import java.util.concurrent.atomic.AtomicReference;
 
public class UnreentrantLock {
 
   private AtomicReference<Thread> owner = new AtomicReference<Thread>();
 
   public void lock() {
       Thread current = Thread.currentThread();
       //这句是很经典的“自旋”语法，AtomicInteger中也有
       for (;;) {
           if (!owner.compareAndSet(null, current)) {
               return;
           }
       }
   }
 
   public void unlock() {
       Thread current = Thread.currentThread();
       owner.compareAndSet(current, null);
   }
}
```

```java
/**
 * @author ：zsy
 * @date ：Created 2021/4/24 23:45
 * @description：测试锁
 */
public class Test {
    private static final UnReentrantLock lock = new UnReentrantLock();

    public static void main(String[] args) {
        new Thread(() -> {
            lock.lock();
            try {
                System.out.println(Thread.currentThread().getName() + "获取到锁");
                TimeUnit.SECONDS.sleep(2);
                System.out.println(Thread.currentThread().getName() + "释放锁");
            } catch (Exception e) {
                e.printStackTrace();
            } finally {
                lock.unlock();
            }
        }, "A").start();

        new Thread(() -> {
            lock.lock();
            try {
                System.out.println(Thread.currentThread().getName() + "获取到锁");
                TimeUnit.SECONDS.sleep(1);
                System.out.println(Thread.currentThread().getName() + "释放锁");
            } catch (Exception e) {
                e.printStackTrace();
            } finally {
                lock.unlock();
            }
        }, "B").start();
    }
}
A获取到锁
A释放锁
B获取到锁
B释放锁
```

使用原子引用来存放线程，同一线程两次调用lock()方法，如果不执行unlock()释放锁的话，第二次调用自旋的时候就会产生死锁，这个锁就不是可重入的，而实际上同一个线程不必每次都去释放锁再来获取锁，这样的调度切换是很耗资源的。

```java
import java.util.concurrent.atomic.AtomicReference;
 
public class UnreentrantLock {
 
   private AtomicReference<Thread> owner = new AtomicReference<Thread>();
   private int state = 0;
 
   public void lock() {
       Thread current = Thread.currentThread();
       if (current == owner.get()) {
           state++;
           return;
       }
       //这句是很经典的“自旋”式语法，AtomicInteger中也有
       for (;;) {
           if (!owner.compareAndSet(null, current)) {
               return;
           }
       }
   }
 
   public void unlock() {
       Thread current = Thread.currentThread();
       if (current == owner.get()) {
           if (state != 0) {
               state--;
           } else {
               owner.compareAndSet(current, null);
           }
       }
   }
}
```

在执行每次操作之前，判断当前锁持有者是否是当前对象，采用state计数，不用每次去释放锁。

## [transient](https://baijiahao.baidu.com/s?id=1636557218432721275&wfr=spider&for=pc)

- ​	transient底层实现原理
  - Java的serialization提供了一个非常棒的存储对象状态的机制，说白了serialization就是把对象的状态存储到硬盘上 去，等需要的时候就可以再把它读出来使用。有些时候像银行卡号这些字段是不希望在网络上传输的，transient的作用就是把这个字段的生命周期仅存于调用者的内存中而不会写到磁盘里持久化，意思是transient修饰的age字段，他的生命周期仅仅在内存中，不会被写到磁盘中。

## 竞态条件和数据竞争

- 竞态条件：某个计算的正确性取决于多个线程的交替执行时序时，那么就会发生竞态条件。（正确的结果取决于运气）最常见的竞态条件的类型就是"先检查后执行"。大多数竞态条件的本质就是基于一种可能失效的观察结果来做出判断或者执行某个计算
- 数据竞争：当一个线程写入一个变量而另一个线程接下来读取这个变量，或者读取一个之前由另一个线程写入的变量的时，并且这两个线程之间没有使用同步，那么就可能出现数据竞争（数据访问的错误）
- 并非所有的竞态条件都是数据竞争，同样并非所有的数据竞争都是竞态条件

# 线程协作

##  线程通信

- 应用场景：生产者和消费者问题	
  - 对于生产者，没有生产产品之前，要通知消费者等待，而生产了产品之后，又需要马上通知消费者消费
  - 对于消费者，在消费之前，要通知消费者已经结束消费，需要产生新的产品以供消费
  - 在生产者消费者问题中，仅有synchronized是不够的的
    - synchronized可阻止并发更新同一个共享资源，实现了同步
    - synchronized不能用来实现不同线程之间的消息传递（通信）
- 这是一个线程同步问题，生产者和消费者共享同一个资源，并且生产者和消费者之间相互依赖，互为条件。
- Java提供了几个方法解决线程之间的通讯问题

|       方法名       |                             作用                             |
| :----------------: | :----------------------------------------------------------: |
|       wait()       |  表示线程一直等待，直到其他线程通知，与sleep不同，会释放锁   |
| wait(long timeout) |                       指定等待的毫秒数                       |
|      notify()      |                  唤醒一个处于等待状态的线程                  |
|    notifyAll()     | 唤醒同一个对象上所有调用wait()方法的线程，优先级别高的线程优先调度 |

- **注意**：均为Object类的方法，都只能在同步方法或者同步代码块中使用，三个方法的调用者必须是同步代码块或同步方法中的同步监视器。否则会抛出异常’IIIegalMonitorStateException‘

### wait()和sleep()方法的区别

- 相同点：两者都可以使当前线程进入阻塞状态； 
-  不同点：
  - 两者所属的类不同，sleep()是Thread类中的方法，wait()是Object类中的方法； 
  - 两者的使用范围不一样，sleep()可以使用在任何有需要的场景下，而wait()方法只能使用在同步代码块和同步方法中； 
  - 是否释放锁：sleep()方法不会释放锁，而wait()方法会释放锁。 
  - 用法不同：使用sleep()方法，睡眠时间到了以后线程会自动苏醒；而使用wait()方法必须通过调用对象的notify()或notifyAll()方法唤醒； 
  - 作用不同：sleep()方法是用来暂停线程的，而wait()方法是用来进行线程间交互的。

### 解决方法

#### **方式1**：生产者消费者(管程法)

**并发协作模型“生产者/消费者模式” ——>管程法**

- 生产者：负责生产数据的模块（可能是方法、对象、线程、进程）
- 消费者：负责处理数据的模块（可能是方法、对象、线程、进程）
- 缓冲区：消费者不能直接使用生产者数据，他们之间有缓冲区

【生产者将生产好的数据放入缓冲区，消费者从缓冲区拿出数据】

```java
/**
 * @author ：zsy
 * @date ：Created 2021/4/13 22:02
 * @description：生产者消费者模式
 */
public class TestPc {

    public static void main(String[] args) {
        SynContainer container = new SynContainer();
        Thread p = new Productor(container);
        Thread c = new Consumer(container);
        p.start();
        c.start();
    }
}

//生产者
class Productor extends Thread{
    SynContainer container;

    public Productor(SynContainer synContainer) {
        this.container = synContainer;
    }

    @Override
    public void run() {
        super.run();
        for (int i = 0; i < 100; i++) {
            container.push(new Product(i));
            System.out.println("生产了------>" + i + "件产品");
        }
    }
}

//消费者
class Consumer extends Thread {
    SynContainer container;

    public Consumer(SynContainer synContainer) {
        this.container = synContainer;
    }

    @Override
    public void run() {
        super.run();
        for (int i = 0; i < 100; i++) {
            System.out.println("消费了" + container.pop().id + "件产品");
        }
    }
}

//产品
class Product {
    public int id;

    public Product(int id) {
        this.id = id;
    }
}

//缓冲区
class SynContainer {
    Product[] products = new Product[10];
    int count = 0;

    public synchronized void push(Product product) {
        if (count == products.length) {
            try {
                this.wait();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
        products[count] = product;
        count++;
        //通知消费者可以消费了
        this.notifyAll();
        
    }

    public synchronized Product pop() {
        if (count == 0) {
            try {
                this.wait();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
        count--;
        Product product = products[count];
        //通知生产者开始生产
        this.notifyAll();

        return product;
    }
}
```

#### **方式2**：(信号灯法)

并发协作模型“生产者/消费者模式” ——>信号灯法

```java
/**
 * @author ：zsy
 * @date ：Created 2021/4/13 23:37
 * @description：信号灯法
 */
public class TestPc2 {

    public static void main(String[] args) {
        Tv tv = new Tv();
        Player player = new Player(tv);
        Watcher watcher = new Watcher(tv);
        player.start();
        watcher.start();
    }
}

//演员类
class Player extends Thread {
    Tv tv;

    public Player(Tv tv) {
        this.tv = tv;
    }

    @Override
    public void run() {
        for (int i = 0; i < 20; i++) {
            if (i % 2 == 0) {
                tv.play("王牌对王牌");
            } else {
                tv.play("青春环游记");
            }
        }
    }
}

//观众类
class Watcher extends Thread{
    Tv tv;
    public Watcher(Tv tv) {
        this.tv = tv;
    }

    @Override
    public void run() {
        for (int i = 0; i < 20; i++) {
            tv.watch();
        }
    }
}

//节目
class Tv {
    String program;
    /**
     * 演员表演节目 T
     * 观众观看节目 F
     */
    boolean flag = true;

    public synchronized void play(String program) {
        if (!flag) {
            try {
                this.wait();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
        this.program = program;
        System.out.println("演员表演了----->" + program + "节目");
        this.flag = ! this.flag;
        this.notifyAll();
    }

    public synchronized void watch() {
        if(flag) {
            try {
                this.wait();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
        System.out.println("观看了" + program + "节目");
        this.flag = !this.flag;
        this.notifyAll();
    }
}

```

## 线程池

- 背景：经常创建和销毁、使用量特别大的资源，比如并发情况下的线程，对性能影响很大
- 思路：提前创建好多个线程，放入线程池中，使用时直接获取。使用完放回池中。可以避免频繁的创建和销毁、实现重复利用
- 好处
  - 提高响应速度（减少了创建新线程的时间）
  - 降低资源消耗（重复利用线程池中的线程，不需要每次都创建）
  - 便于线程管理
    - corePoolSize：核心池大小
    - maximumPoolSize：最大线程数
    - keepAliveTime：线程没有任务时最多保持多长时间后会终止

### 创建线程池

- JDK 5.0起提供了线程池相关API：ExecutorService 和 Executors
- ExecutorService：真正的线程池接口。常见子类ThreadPoolExecutor

```java
/**
 * @author ：zsy
 * @date ：Created 2021/4/14 15:35
 * @description：创建线程池
 */
public class TestPool {
    public static void main(String[] args) {
        //创建线程池
        ExecutorService service = Executors.newFixedThreadPool(10);
        //执行线程
        service.execute(new MyThread());
        service.execute(new MyThread());
        service.execute(new MyThread());
        service.execute(new MyThread());

        //关闭线程池
        service.shutdown();
    }
}

class MyThread implements Runnable {

    @Override
    public void run() {
        System.out.println(Thread.currentThread().getName());
    }
}
```

# JUC并发编程

- Java真的可以开启线程吗？
  - 不能，调用本地方法start0()，底层是C++，Java无法直接操作硬件
- 线程的状态

```java
public enum State {
        //新生
        NEW,
        //运行
        RUNNABLE,
        //阻塞
        BLOCKED,
        //等待，一直等待
        WAITING,
        //超时等待
        TIMED_WAITING,
    	//终止
        TERMINATED;
    }
```

## 传统synchronized方法

```java
/**
 * @author ：zsy
 * @date ：Created 2021/4/14 17:24
 * @description：买票
 */
public class SaleTicketDemo01 {
    
    /*
     * 真正的多线程开发，公司中的开发，降低耦合性
     * 线程就是一个单独的资源类，没有任何附属的操作！
     */
    public static void main(String[] args) {
        //创建窗口对象
        Ticket ticket = new Ticket();

        new Thread(() -> {
            for (int i = 0; i < 40; i++) {
                ticket.saleTicket();
            }
        }, "A").start();
        new Thread(() -> {
            for (int i = 0; i < 40; i++) {
                ticket.saleTicket();
            }
        }, "B").start();
    }
}

//买票窗口
class Ticket {
    //票数
    private int tickNum = 50;

    //买票
    public synchronized void saleTicket() {
        if (tickNum > 0) {
            try {
                TimeUnit.MILLISECONDS.sleep(5);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println(Thread.currentThread().getName() + "买了第" + tickNum-- + "张票，剩余" + tickNum + "张票");
        }
    }
}
```

## 传统Lock方法

```java
//买票
    public void saleTicket() {
        if (tickNum > 0) {
            lock.lock();
            try {
                try {
                    TimeUnit.MILLISECONDS.sleep(5);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                System.out.println(Thread.currentThread().getName() + "买了第" + tickNum-- + "张票，剩余" + tickNum + "张票");
            }finally {
                lock.unlock();
            }
        }
    }
```

>Lock锁

**公平锁** ：先来后到，十分公平

**非公平锁** ：可以插队，不公平

```java
public ReentrantLock() {
    sync = new NonfairSync();
}

public ReentrantLock(boolean fair) {
    sync = fair ? new FairSync() : new NonfairSync();
}
```

### tryLock()和lock()的区别

>tryLock()和lock()的区别

- tryLock()：如果获取到锁返回true，否则返回false，不阻塞
- lock()：没有获取到锁，阻塞直到获得锁

## 线程之间通信问题

### 生产者和消费者问题

重要步骤：等待，业务，唤醒

#### synchronized版

```java
/**
 * @author ：zsy
 * @date ：Created 2021/4/14 20:07
 * @description：生产者消费者模式，线程通信   
 */
public class A {

    public static void main(String[] args) {
        Data data = new Data();
        new Thread(() -> { for (int i = 0; i < 20; i++) data.increment(); },  "A").start();
        new Thread(() -> { for (int i = 0; i < 20; i++) data.decrement(); },  "B").start();
    }
}

class Data {
    int num = 0;

    public synchronized void increment() {
        if (num != 0) {
            //等待
            try {
                this.wait();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
        //业务
        System.out.println(Thread.currentThread().getName() + "--->" + ++num);

        //唤醒
        this.notifyAll();
    }

    public synchronized void decrement() {
        if(num == 0) {
            //等待
            try {
                this.wait();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }

        //业务
        System.out.println(Thread.currentThread().getName() + "--->" + --num);

        //唤醒
        this.notifyAll();
    }
}
```

在两个线程的时候可以正常运行，当线程的数量加到四个的时候，出现了**虚假唤醒**的问题

```java
new Thread(() -> { for (int i = 0; i < 20; i++) data.increment(); },  "C").start();
new Thread(() -> { for (int i = 0; i < 20; i++) data.decrement(); },  "D").start();
```

##### 虚假唤醒

- 产生原因：线程处于wait状态的时候，线程也可以被唤醒，而不会被通知，中断或超时。notifyAll()会唤醒所有等待这个同步监听器的线程。所有等待的线程都会处于被唤醒的状态，都会尝试去尝试获取锁然后运行，即使if()中的条件成立（因为if只判断一次），也不会处于等待状态
- 解决方法：将if换成while即使线程被虚假唤醒，也会再次判断条件是否成立，不会在进入等待状态

```java
 while (num != 0) {
```

#### JUC版

```java
/**
 * @author ：zsy
 * @date ：Created 2021/4/14 20:45
 * @description：Lock锁生产者消费者
 */
public class B {

    public static void main(String[] args) {
        Data data = new Data();
        new Thread(() -> { for (int i = 0; i < 20; i++) data.increment(); },  "A").start();
        new Thread(() -> { for (int i = 0; i < 20; i++) data.decrement(); },  "B").start();
        new Thread(() -> { for (int i = 0; i < 20; i++) data.increment(); },  "C").start();
        new Thread(() -> { for (int i = 0; i < 20; i++) data.decrement(); },  "D").start();
    }

    static class Data {
        int num = 0;
        private Lock lock = new ReentrantLock();
        private Condition condition = lock.newCondition();

        public void increment() {
            lock.lock();
            try {
                while (num != 0) {
                    //等待
                    try {
                        condition.await();
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                }
                //业务
                num++;
                System.out.println(Thread.currentThread().getName() + "--->" + num);

                //唤醒
                condition.signalAll();
            } catch (Exception e) {
                e.printStackTrace();
            } finally {
                lock.unlock();
            }

        }

        public void decrement() {
            lock.lock();
            try {
                while(num == 0) {
                    //等待
                    condition.await();
                }

                //业务
                num--;
                System.out.println(Thread.currentThread().getName() + "--->" + num);

                //唤醒
                condition.signalAll();
            } catch (Exception e) {
                e.printStackTrace();
            } finally {
                lock.unlock();
            }

        }
    }

}	
```

##### Condition实现精准唤醒

```java
/**
 * @author ：zsy
 * @date ：Created 2021/4/14 21:12
 * @description：多个线程按顺序打印
 */
public class C {

    public static void main(String[] args) {
        Data data = new Data();
        new Thread(() -> { for (int i = 0; i < 10; i++) data.printA();}, "A").start();
        new Thread(() -> { for (int i = 0; i < 10; i++) data.printB();}, "B").start();
        new Thread(() -> { for (int i = 0; i < 10; i++) data.printC();}, "C").start();
    }

    static class Data {
        private int num = 1;
        private Lock lock = new ReentrantLock();
        private Condition condition1 = lock.newCondition();
        private Condition condition2 = lock.newCondition();
        private Condition condition3 = lock.newCondition();

        public void printA() {
            lock.lock();
            try {
                if (num != 1) {
                    condition1.await();
                }
                System.out.println(Thread.currentThread().getName() + "---->" + num);
                num = 2;
                condition2.signal();
            } catch (Exception e) {
                e.printStackTrace();
            }finally {
                lock.unlock();
            }
        }

        public void printB() {
            lock.lock();
            try {
                if (num != 2) {
                    condition2.await();
                }
                System.out.println(Thread.currentThread().getName() + "---->" + num);
                num = 3;
                condition3.signal();
            } catch (Exception e) {
                e.printStackTrace();
            }finally {
                lock.unlock();
            }
        }

        public void printC() {
            lock.lock();
            try {
                if (num != 3) {
                    condition3.await();
                }
                System.out.println(Thread.currentThread().getName() + "---->" + num);
                num = 1;
                condition1.signal();
            } catch (Exception e) {
                e.printStackTrace();
            }finally {
                lock.unlock();
            }
        }

    }
}

```

## synchronized

### 实现原理

在编译的字节码中加入了两条指令来进行代码的同步

- monitorenter

  每个对象有一个监视器锁（monitor）。当monitor被占用时，就会处于锁定状态，线程执行monitorenter指令时尝试获取monitor的所有权，过程如下：

  1. 如果monitor的进入数为0，则该线程进入monitor，然后将进入数设置为1，该线程即为monitor的所有制。
  2. 如果线程已经占有了该monitor，只是重新进入，则进入monitor的进入数加1
  3. 如果其他线程已经占有了monitor，则该线程进入阻塞状态，指导monitor的进入数为0，再次重新尝试获取monitor所有权

- monitorexit

  指令执行时，monitorexit的线程必须是objectref所对应的monitor的所有者。指令执行时，monitor的进入数减1，如果减1后进入数为0，那线程退出monitor，不再是这个monitor的所有者。其他被这个monitor阻塞的线程可以尝试去获取这个 monitor 的所有权。

通过这两段描述，我们应该能很清楚的看出Synchronized的实现原理，Synchronized的语义底层是通过一个monitor的对象来完成，其实wait/notify等方法也依赖于monitor对象，这就是为什么只有在同步的块或者方法中才能调用wait/notify等方法，否则会抛出java.lang.IllegalMonitorStateException的异常的原因。

### 锁的对象问题

- 同步代码块包括两个部分
  - 一个作为锁的对象引用
  - 一个作为由这个锁保护的代码块 
- 以synchronized来修饰的方法就是一种横跨整个方法体的同步代码块，其中该同步代码块的锁就是方法所在的对象
- 静态的synchronized方法以Class为锁

synchronized可以保证方法或者代码块在运行时，**同一时刻只有一个方法可以进入到临界区**，同时它还可以保证共享变量的内存可见性。 

 Java中每一个对象都可以作为锁，这是synchronized实现同步的基础： 

-  普通同步方法，锁是当前实例对象 
-  静态同步方法，锁是当前类的class对象 
-  同步方法块，锁是括号里面的对象

### [synchronized优化](https://blog.csdn.net/ryo1060732496/article/details/88848267?utm_medium=distribute.pc_relevant.none-task-blog-2%7Edefault%7EBlogCommendFromBaidu%7Edefault-6.control&dist_request_id=1332042.22877.16193364780573571&depth_1-utm_source=distribute.pc_relevant.none-task-blog-2%7Edefault%7EBlogCommendFromBaidu%7Edefault-6.control)

#### 锁升级

在多线程并发编程中 synchronized 一直是元老级角色，很多人都会称呼它为重量级锁。

但是，随着 Java SE 1.6 对 synchronized 进行了各种优化之后，有些情况下它就并不那么重，Java SE 1.6 中为了减少获得锁和释放锁带来的性能消耗而引入的偏向锁和轻量级锁。

针对 synchronized 获取锁的方式，JVM 使用了锁升级的优化方式，就是先使用偏向锁（）优先同一线程然后再次获取锁，如果失败，就升级为 CAS 轻量级锁，如果失败就会短暂自旋，防止线程被系统挂起。最后如果以上都失败就升级为重量级锁。
![img](https://gitee.com/zhang-songyao/blog-images/raw/master/20210421234835.jpg)

锁只能升级，不能降级。

![img](https://gitee.com/zhang-songyao/blog-images/raw/master/20210421234836.jpg)

#### 自旋锁

自旋锁就是一个线程自转，空转，什么都不操作，但也不挂起，空循环的作用就是等待一把锁，自旋锁是明确的会产生竞争的情况下使用的。

当竞争存在时，如果线程可以很快获得锁，那么就没有必要在（操作系统）OS层面挂起线程（因为在操作系统层面去挂起，他的性能消耗是非常严重的，因此如果我们能假定他能很快获取锁，就不需要让线程挂起），而是让线程做几个空操作（称为自旋）

互斥同步对性能最大的影响是阻塞的实现，挂起线程和恢复线程的操作都需要操作系统从用户态转入内核态中完成，这些操作给系统的并发性能带来了很大的压力。

**为了让线程等待，我们只须让线程执行一个忙循环（自旋），这项技术就是自旋锁。**

如果锁长时间被占用，则浪费处理器资源，因此自旋等待的时间必须要有一定的限度，如果自旋超过了限定的次数仍然没有成功获得锁，就应当使用传统的方式去挂起线程了（默认10次）。

#### 自旋适应锁

JDK1.6引入自适应的自旋锁：自旋时间不再固定，由前一次在同一个锁上的自旋时间及锁的拥有者的状态来决定。

如果在同一个锁对象上，自旋等待刚刚成功获得过锁，并且持有锁的线程正在运行中，那么虚拟机就会认为这次自旋也很有可能再次成功，进而它将允许自旋等待持续相对更长的时间。

#### 偏向锁

大部分情况是没有竞争的，所以可以通过偏向来提高性能

所谓的偏向，就是偏心，即锁会偏向于当前已经占有锁的线程（也就是说，这个线程已经占有这个锁，当他在次试图去获取这个锁的时候，他会已最快的方式去拿到这个锁，而不需要在进行一些monitor操作，因此这方面他是会对性能有所提升的，因为在大部分情况下是没有竞争的，所以锁此时是没用的，所以使用偏向锁是可以提高性能的）

在使用偏向锁的时候会将对象头Mark的标记设置为偏向，并将拿到锁的线程的ID写入对象头Mark，这样就可以很快识别出这个线程是否拿到的锁。

在竞争激烈的场合，偏向锁会增加系统负担。

#### 锁优化

主要分为JVM层面和代码编程层面

- JVM
  - 锁粗化
  - 锁消除

- 编程
  - 减少锁持有时间
  - 锁分离
  - 减小锁的粒度

#### 锁粗化

如果一系列的连续操作都对同一个对象反复加锁和解锁，甚至加锁操作是出现在循环体中的，那即使没有线程竞争，频繁地进行互斥同步操作也会导致不比要的开销。

如果虚拟机探测到有这样一串零碎的操作都对同一个对象加锁，将会把加锁同步的范围扩展（粗化）到整个操作序列的外部（由多次加锁编程只加锁一次）。

**锁粗化就是告诉我们任何事情都有个度，有些情况下我们反而希望把很多次锁的请求合并成一个请求，以降低短时间内大量锁请求、同步、释放带来的性能损耗。**

锁削除的主要判定依据来源于逃逸分析的数据支持，如果判断到一段代码中，在堆上的所有数据都不会逃逸出去被其他线程访问到，那就可以把它们当作栈上数据对待，认为它们是线程私有的，同步加锁自然就无须进行。

#### 锁消除

锁消除是指虚拟机即时编译器在运行时，对一些代码要求做同步，但是被检测到不可能存在共享数据竞争的做进行消除比如，StringBuffer类的append操作

### synchronized和Lock对比

|         类别         |                         synchronized                         |                             Lock                             |
| :------------------: | :----------------------------------------------------------: | :----------------------------------------------------------: |
|       存在层次       |                   Java的关键字，在JVM层面                    |                           是Java类                           |
|       锁的释放       | 1、获取锁的线程执行完毕，释放锁 2、线程执行出现异常，JVM会让线程释放锁 3、抛出异常后会自动释放锁 |             finally必须释放锁，不然容易造成死锁              |
|      锁获取方式      | 用synchronized关键字的两个线程1和线程2，如果当前线程1获得锁，线程2线程等待。如果线程1阻塞，线程2则会一直等待下去 | Lock锁就不一定会等待下去，如果尝试获取不到锁，线程可以不用一直等待就结束了 **lock.tryLock();** |
|        锁类型        |                   可重入、不可中断、非公平                   |              可重入、可中断、可公平（两者皆可）              |
|       使用区别       |             synchronized锁适合代码少量的同步问题             |              Lock锁适合大量同步的代码的同步问题              |
| 能否判断获取锁的状态 |                              否                              |                   Lock可以判断是否获取到锁                   |
|       锁的对象       |                       代码块、方法、类                       |                            代码块                            |

- 使用Lock锁，JVM将花费较少的时间来调度线程，性能更好。并且具有更好的扩展性（提供更多的子类）。
- 优先使用顺序
  - Lock -> 同步代码块（已经进入了方法体，分配了相应资源） -> 同步方法（在方法体外）

### synchronized 和 ReentrantLock 区别是什么？

synchronized是和if、else、for、while一样的关键字，ReentrantLock是类，这是二者的本质区别。既然ReentrantLock是类，那么它就提供了比synchronized更多更灵活的特性，可以被继承、可以有方法、可以有各种各样的类变量，ReentrantLock比synchronized的扩展性体现在几点上： 

-  ReentrantLock可以对获取锁的等待时间进行设置，这样就避免了死锁 
-  ReentrantLock可以获取各种锁的信息 
-  ReentrantLock可以灵活地实现多路通知 

 另外，二者的锁机制其实也是不一样的:ReentrantLock底层调用的是Unsafe的park方法加锁，synchronized操作的应该是对象头中markword。

## 集合不安全类

### ArrayList

#### 测试ArrayList的线程不安全性

```java
/**
 * @author ：zsy
 * @date ：Created 2021/4/15 15:03
 * @description：CopyOnWriteList
 */
public class ListTest {

    public static void main(String[] args) {
        List<String> list = new ArrayList<>();
        for (int i = 0; i < 10; i++) {
            new Thread(() -> {
                list.add(UUID.randomUUID().toString().substring(0, 4));
                System.out.println(Thread.currentThread().getName() + "---->" + list);
            }, String.valueOf(i)).start();
        }
    }
}

报异常：ConcurrentModificationException（并发修改异常）
```

#### 解决方案

- 使用Vector代理ArrayList

```java
 List<String> list = new Vector<>();
```

- 使用Collections工具类的synchronizedList()方法

```java
List<String> list = Collections.synchronizedList(new ArrayList<>());
```

- 使用JUC包下的线程安全类CopyOnWriteArrayList

```java
List<String> list = new CopyOnWriteArrayList<>();
```

#### CopyOnWriteArrayList原理

- CopyOnWriteArrayList的实现原理就是读写分离，它对所有的写操作都使用ReentrantLock来加锁，对所有的读操作都不进行加锁。
- CopyOnWriteArrayList在写操作的时候，都会将list中的数组copy一份作为缓存，然后对该缓存中的数组进行操作，（此时若有其他线程过来读的话，那么该线程读的还是原先没有被修改过的数据，若有其他线程过来写的话，那么该线程会因为ReentrantLock的原因被锁在外面）。操作完毕后在将list中的数组地址引用指向修改后的新数组地址
- 由CopyOnWriteArrayList的原理我们可以看出，我们每次往list里面写数据的时候，数组都需要重新copy一份，所以CopyOnWriteArrayList不需要实现像ArrayList一样的扩容机制，初始创建时让list中的数组长度为0，我们每次add元素的时候，只需要对新数组长度进行加1操作即可，所以CopyOnWriteArrayList实现起来相对还是比ArrayList简单的。

>​		所以我们可以看出，如果是读操作十分频繁的话，那么多线程下使用CopyOnWriteArrayList的性能基本上跟ArrayList差不多了。但如果是写操作十分频繁的话，建议还是不要使用CopyOnWriteArrayList了，因为它会造成数组的不断扩容及复制，十分耗性能。这其实就跟我们数据库读写分离的原理是一样的，如果写操作很多的话，那么主从库就会不断的执行复制操作，消耗性能。但如果是读操作多的话，由于该库只用于读，所以不会发生数据库事务锁，效率就会比一般的单库查询快很多。

### Set

#### 测试Set的线程不安全性

```java
/**
 * @author ：zsy
 * @date ：Created 2021/4/15 17:02
 * @description：Set集合
 */
public class SetTest {
    public static void main(String[] args) {
        Set<String> set = new HashSet<>();
        for (int i = 0; i < 30; i++) {
            new Thread(() -> {
                try {
                    TimeUnit.MILLISECONDS.sleep(10);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                set.add(UUID.randomUUID().toString().substring(0, 4));
                System.out.println(set);
            }, String.valueOf(i)).start();
        }
    }
}
ConcurrentModificationException
```

#### 解决方案

- 使用Collections工具类的synchronizedList()方法

```java
Set<String> set = Collections.synchronizedList(new HashSet<>());
```

- 使用JUC包下的线程安全类CopyOnWriteArraySet

```java
Set<String> set = new CopyOnWriteArraySet<>();
```

>CopyOnWriteArraySet底层是一个CopyOnWriteArrayList

### Map

#### 测试HashMap的线程不安全性

```java
/**
 * @author ：zsy
 * @date ：Created 2021/4/15 17:50
 * @description：HashMap
 */
public class MapTest {
    public static void main(String[] args) {
        Map<String, String>map = new HashMap<>();
        for (int i = 0; i < 30; i++) {
            new Thread(() -> {
                map.put(Thread.currentThread().getName(), UUID.randomUUID().toString().substring(0, 4));
                System.out.println(map);
            }, String.valueOf(i)).start();
        }
    }
}
ConcurrentModificationException
```

#### 解决方案

- 使用Collections工具类的synchronizedMap()方法

```java
Map<String, String>map = Collections.synchronizedMap(new HashMap<>());
```

- 使用JUC包下的线程安全类ConcurrentHashMap

```java
Map<String, String>map = new ConcurrentHashMap<>();
```

#### Hashtable

#### Hashtable和HashMap对比

- 线程安全：HashMap是线程不安全的类，多线程下会造成并发冲突，但单线程下运行效率较高；Hashtable是线程安全的类，很多方法都使用了synchronized修饰，但同时因为加锁导致并发小效率低下，单线程环境效率也十分低（当多个线程同时对Hashtable进行put操作是，尽管插入的不是同一个槽位，但是由于多个线程拿的是一把锁，所以必须按顺序执行put操作）
- HashMap需要重新计算hash值,而Hashtable直接使用对象的hashCode
- 插入null：HashMap允许有一个键为null，允许多个值为null；但Hashtable不允许键或值为null

```java
public synchronized V put(K key, V value) {
    // Make sure the value is not null
    if (value == null) {
        throw new NullPointerException();
    }
    // Makes sure the key is not already in the hashtable.
    Entry<?,?> tab[] = table;
    int hash = key.hashCode();//这里如果key == null 会抛出异常
    int index = (hash & 0x7FFFFFFF) % tab.length;
    @SuppressWarnings("unchecked")
    Entry<K,V> entry = (Entry<K,V>)tab[index];
    for(; entry != null ; entry = entry.next) {
        if ((entry.hash == hash) && entry.key.equals(key)) {
            V old = entry.value;
            entry.value = value;
            return old;
        }
    }
    addEntry(hash, key, value, index);
    return null;
}
```

- 容量：HashMap底层数组长度必须为2的幂，这样做是为了hash做准备，默认为16；而Hashtable底层数组长度可以为任意值，这就造成了hash算法散射不均匀，容易造成hash冲突，默认值为11,前者扩容时,扩大两倍,后者扩大两倍+1

>rehash 扩容

```java
protected void rehash() {
    int oldCapacity = table.length;
    Entry<?,?>[] oldMap = table;
    // overflow-conscious code
    int newCapacity = (oldCapacity << 1) + 1;
    if (newCapacity - MAX_ARRAY_SIZE > 0) {
        if (oldCapacity == MAX_ARRAY_SIZE)
            // Keep running with MAX_ARRAY_SIZE buckets
            return;
        newCapacity = MAX_ARRAY_SIZE;
    }
    Entry<?,?>[] newMap = new Entry<?,?>[newCapacity];
    modCount++;
    threshold = (int)Math.min(newCapacity * loadFactor, MAX_ARRAY_SIZE + 1);
    table = newMap;
    for (int i = oldCapacity ; i-- > 0 ;) {
        for (Entry<K,V> old = (Entry<K,V>)oldMap[i] ; old != null ; ) {
            Entry<K,V> e = old;
            old = old.next;
            int index = (e.hash & 0x7FFFFFFF) % newCapacity;
            e.next = (Entry<K,V>)newMap[index];
            newMap[index] = e;
        }
    }
}
```

>Hashtable扩容，原容量 * 2 +1 ，插入采用头插法

- Hash映射：HashMap的hash算法通过常规设计，将底层table长度设计为2的幂，使用位与运算代替取模运算，减少运算消耗；而Hashtable的hash算法首先使得hash值小于整型数最大值，再通过取模进行散射运算

> 因为hashSeed ^ k.hashCode(); 的值可能是负数，如果直接index = hash % tab.length; 这样就会导致index可能为负数

```java
int index = (hash & 0x7FFFFFFF) % tab.length;
```

## 同步容器和并发容器

> 以下内容引自《Java并发编程实战第五章》

### 同步容器

同步容器类包括Vector和Hashtable，二者是早期JDK的一部分，此外还包括在JDK1.2中添加的一些功能相似的类，这些同步的封装器类由Collections.synchronizedXxx工厂方法创建，这些类实现线程安全的方式是：将他们封装起来，并对每个共有方法都有同步，每次只有一个线程能访问容器状态。

#### 同步器的问题

同步容器类都是线程安全的，但是在某些情况下可能需要额外的客户端加锁来保护复合操作。容器上常见的符合操作包括：迭代（反复访问元素，直到遍历完容器的所有元素）、跳转（根据指定顺序找到当前元素的下一个元素）以及条件运算，例如：若没有则添加（在Map中是否存在键值K，如果没有，则加入二元组(K,V)）。在同步容器类中，这些复和操作在没有客户端加锁的情况下仍是线程安全的，但当其他线程并发地修改容器时，他们可能会表现出意料之外的行为。

```java
/**
 * @author ：zsy
 * @date ：Created 2021/4/24 16:48
 * @description：测试Vector的线程安全性
 */
public class TestVector {
    public static void main(String[] args) {
        Vector vector = new Vector();
        vector.add(1);
        vector.add(2);
        new Thread(() -> System.out.println(getLast(vector)),
        new Thread(() -> deleteLast(vector), "B").start();
    }
    public static Object getLast(Vector list) {
        int lastIndex = list.size() - 1;
        try {
            Thread.sleep(10);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        return list.get(lastIndex);
    }
    public static void deleteLast(Vector list) {
        int lastIndex = list.size() - 1;
        list.remove(lastIndex);
    }
}
```

> 运行上面程序会抛出Exception in thread "A" java.lang.ArrayIndexOutOfBoundsException: Array index out of range: 1异常

分析抛出异常的原因

如果线程A在包含2个元素的Vector调用getLast，同时线程B在同一个Vector上调用deleteLast，这些操作的交替执行如图，getLast会抛出ArrayIndexOutOfBoundsException异常。

<img src="https://gitee.com/zhang-songyao/blog-images/raw/master/20210424170033.png" alt="image-20210424170021378" style="zoom: 80%;" />

由于并发容器类要遵守同步策略，即支持客户端加锁，因此可能创建一些新的操作，只要我们知道应该使用哪一个锁，那么新操作就与容器的其他操作一样都是原子操作。同步容器类会保护它的每个方法。

```java
public static Object getLast(Vector list) {
    synchronized (list) {
        int lastIndex = list.size() - 1;
        try {
            Thread.sleep(10);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        return list.get(lastIndex);
    }
}
public static void deleteLast(Vector list) {
    synchronized (list) {
        int lastIndex = list.size() - 1;
        list.remove(lastIndex);
    }
}
```

> 同理所有对共享容器进行迭代的地方都需要加锁。实际情况要更加复杂，因为在某些情况，迭代器会隐藏起来，操作不放会抛出ConcurrentModificationException

### 并发容器

同步容器将所有对容器的访问都串行化，以实现他们的线程安全性，这种方法的代价是严重降低并发性，当多个线程竞争访问容器的锁时，吞吐量将会严重减低。

针对多个线程设计的，用并发容器来代替同步容器，可以极大地提高伸缩性并降低风险。 如ConcurrentHashMap，CopyOnWriteArrayList等。并发容器使用了与同步容器完全不同的加锁策略来提供更高的并发性和伸缩性，例如在ConcurrentHashMap中采用了一种粒度更细的加锁机制，可以称为分段锁，在这种锁机制下，允许任意数量的读线程并发地访问map，并且执行读操作的线程和写操作的线程也可以并发的访问map，同时允许一定数量的写操作线程并发地修改map，所以它可以在并发环境下实现更高的吞吐量。

## ThreadLocal

### 什么是ThreadLocal

ThreadLocal是一个本地线程副本变量工具类，在每个线程中都创建一个ThreadLocalMap对象，简单来说ThreadLocal就是一种时间换空间的做法，每个线程可以访问自己内部ThreadLocalMap对象内的value。通过这种方式，避免资源在多线程之间共享。

### 原理

线程局部变量是局限于线程内部的变量，属于线程自身所有，不在多个线程中共享，Java提供ThreadLocal类来支持线程局部变量，是一种线程安全的方式，但是管理环境下（如web服务器）使用线程局部变量的时候要小心，在这种情况下，工作线程的生命周期比任何应用变量的生命周期都要长。任何局部变量一旦工作完成后没有释放，Java应用就会存在内存泄漏的风险。

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

## 常用辅助类

### CountDownLatch

>允许一个或多个线程等待直到在其他线程中执行的一组操作完成的同步辅助
>
>A CountDownLatch用给定的计数初始化。 await方法阻塞，直到由于countDown()方法的调用而导致当前计数达到零，之后所有等待线程被释放，并且任何后续的await 调用立即返回。 这是一个一次性的现象 - 计数无法重置。 如果您需要重置计数的版本，请考虑使用CyclicBarrier

```java
/**
 * @author ：zsy
 * @date ：Created 2021/4/20 19:50
 * @description：减法计数器
 */
public class CountDownLatchDemo {
    public static void main(String[] args) throws InterruptedException {
        CountDownLatch countDownLatch = new CountDownLatch(6);
        for (int i = 0; i < 5; i++) {
            new Thread(() -> {
                System.out.println(Thread.currentThread().getName() + "---->" + "GO");
                countDownLatch.countDown(); //计数器 - 1
            }, String.valueOf(i)).start();
        }
        countDownLatch.await();//等待countDownLatch归零，执行下面的操作
        System.out.println("Close door");
    }
}

```

原理：每次调用countDown()数量-1，假设计数器为零，countDownLatch.await()就会被唤醒，继续执行。

A CountDownLatch是一种通用的同步工具，可用于多种用途。 一个CountDownLatch为一个计数的CountDownLatch用作一个简单的开/关锁存器，或者门：所有线程调用await在门口等待，直到被调用countDown()的线程打开。 一个CountDownLatch初始化N可以用来做一个线程等待，直到N个线程完成某项操作，或某些动作已经完成N次。 

CountDownLatch一个有用的属性是，它不要求调用countDown线程等待计数到达零之前继续，它只是阻止任何线程通过await ，直到所有线程可以通过。

用法

- 第一个是启动信号，防止任何工作人员进入，直到驾驶员准备好继续前进; 
- 第二个是完成信号，允许司机等到所有的工作人员完成。 

### CyclicBarrier

>允许一组线程全部等待彼此达到共同屏障点的同步辅助。  循环阻塞在涉及固定大小的线程方的程序中很有用，这些线程必须偶尔等待彼此。 屏障被称为*循环*  ，因为它可以在等待的线程被释放之后重新使用。 

```java
/**
 * @author ：zsy
 * @date ：Created 2021/4/20 20:04
 * @description：循环等待屏障
 */
public class CyclicBarrierDemo {

    public static void main(String[] args) {
        CyclicBarrier cyclicBarrier = new CyclicBarrier(7, new Thread(() -> {
            System.out.println("召唤神龙");
        }));

        for (int i = 0; i < 7; i++) {
            final int tmp = i;
            new Thread(() -> {
                System.out.println(Thread.currentThread().getName() + "获得第" + tmp + "颗龙珠");
                try {
                    cyclicBarrier.await(); //当等待的类数量达到时，才会执行，否则一直等待
                } catch (InterruptedException e) {
                    e.printStackTrace();
                } catch (BrokenBarrierException e) {
                    e.printStackTrace();
                }
            }).start();
        }
    }
}
```

>作用：控制当前线程数量达到一定数量后，一起执行某项操作

CyclicBarrier对失败的同步尝试使用all-or-none断裂模型：如果线程由于中断，故障或超时而过早离开障碍点，那么在该障碍点等待的所有其他线程也将通过BrokenBarrierException （或InterruptedException）异常离开。 

### 在 Java 中 CyclicBarrier和 CountDownLatch有什么区别

|       区别       |                        CountDownLatch                        |                        CyclicBarrier                         |
| :--------------: | :----------------------------------------------------------: | :----------------------------------------------------------: |
|       作用       | 一般用于某个线程A等待若干个其他线程完成任务后，他才执行，强调多个线程完成某件事情 | 一般用于一组线程相互等待至某个状态，然后一组线程同时执行，强调多个线程互等，等大家都完成，再携手共进 |
| 对当前线程的影响 | 调用CountDownLatch的countDown方法会不导致当前线程阻塞，会继续执行下去 | 调用CyclicBarrier的await方法后，会阻塞当前线程（里面使用了ReentrantLock锁），直到CyclicBarrier指定的线程全部都到达了指定点的时候，才能继续往下执行 |
|       原理       |             利用继承AQS的共享锁来进行线程的通知              |         利用ReentrantLock的Condition来阻塞和通知线程         |
|     难易程度     |            CountDownLatch方法比较少，操作比较简单            | CyclicBarrier提供的方法更多，比如能够通过getNumberWaiting()，isBroken()这些方法获取当前多个线程的状态，并且CyclicBarrier的构造方法可以传入barrierAction，指定当所有线程都到达时执行的业务功能 |
|   是否可以复用   |                              否                              |                              是                              |

### Semaphore

>一个计数信号量。 在概念上，信号量维持一组许可证。 如果有必要，每个acquire()都会阻塞，直到许可证可用，然后才能使用它。  每个release()添加许可证，潜在地释放阻塞获取方。  但是，没有使用实际的许可证对象;  Semaphore只保留可用数量的计数，并相应地执行。 
>
>信号量通常用于限制线程数，而不是访问某些（物理或逻辑）资源。

```java
/**
 * @author ：zsy
 * @date ：Created 2021/4/20 20:32
 * @description：信号量
 */
public class SemaphoreDemo {

    public static void main(String[] args) {
        //指定只有3个信号量
        Semaphore semaphore = new Semaphore(3);
        for (int i = 0; i < 6; i++) {
            new Thread(() -> {
                try {
                    semaphore.acquire();//获取当前的信号量
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                System.out.println(Thread.currentThread().getName() + "获得");
                //doSomething()
                try {
                    TimeUnit.SECONDS.sleep(1);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                System.out.println(Thread.currentThread().getName() + "离开");
                semaphore.release();//释放当前信号量
            }, String.valueOf(i)).start();
        }
    }
}
```

> 作用

Semaphore 就是一个信号量，它的作用是限制某段代码块的并发数。Semaphore有一个构造函数，可以传入一个 int 型整数 n，表示某段代码最多只有 n 个线程可以访问，如果超出了 n，那么请等待，等到某个线程执行完毕这段代码块，下一个线程再进入。由此可以看出如果 Semaphore 构造函数中传入的 int 型整数 n=1，相当于变成了一个 synchronized 了。

Semaphore(信号量)-允许多个线程同时访问： synchronized 和 ReentrantLock 都是一次只允许一个线程访问某个资源，Semaphore(信号量)可以指定多个线程同时访问某个资源。

```java
public Semaphore(int permits, boolean fair) {
    sync = fair ? new FairSync(permits) : new NonfairSync(permits);
}
```

此类的构造函数可选择接受公平参数。 当设置为false时，此类不会保证线程获取许可的顺序。 特别是， 闯入是允许的，也就是说，一个线程调用acquire()可以提前已经等待线程分配的许可证-在等待线程队列的头部逻辑新的线程将自己。 当公平设置为真时，信号量保证调用acquire方法的线程被选择以按照它们调用这些方法的顺序获得许可（先进先出; FIFO）。 请注意，FIFO排序必须适用于这些方法中的特定内部执行点。 因此，一个线程可以在另一个线程之前调用acquire ，但是在另一个线程之后到达排序点，并且类似地从方法返回。 另请注意， 未定义的tryAcquire方法不符合公平性设置，但将采取任何可用的许可证。 

## 读写锁(ReadWriteLock)

>A ReadWriteLock维护一对关联的locks ，一个用于只读操作，一个用于写入。 read lock可以由多个阅读器线程同时进行，只要没有作者。 write lock是独家的。

```java
/**
 * @author ：zsy
 * @date ：Created 2021/4/20 21:33
 * @description：读写锁
 */
public class ReadWriteLockDemo {
    public static void main(String[] args) {

        MyCache myCache = new MyCache();

        for (int i = 0; i < 5; i++) {
            final String tmp = String.valueOf(i);
            new Thread(() -> {
                myCache.put(tmp, "a");
            }, String.valueOf(i)).start();
        }

        for (int i = 0; i < 5; i++) {
            final String tmp = String.valueOf(i);
            new Thread(() -> {
                myCache.get(tmp);
            }, String.valueOf(i)).start();
        }

    }
}

class MyCache {
    private Map<String, String > map = new HashMap<>();
    ReentrantReadWriteLock lock = new ReentrantReadWriteLock();

    public void put(String key, String value) {
        lock.writeLock().lock();
        try {
            System.out.println(Thread.currentThread().getName() + "开始写");
            map.put(key, value);
            System.out.println(Thread.currentThread().getName() + "写完成");
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            lock.writeLock().unlock();
        }

    }

    public String get(String key) {
        String val = null;
        lock.readLock().lock();
        try {
            System.out.println(Thread.currentThread().getName() + "开始读");
            val = map.get(key);
            System.out.println(Thread.currentThread().getName() + "读完成");
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            lock.readLock().unlock();
        }
        return val;
    }
}
1开始写
1写完成
0开始写
0写完成
2开始写
2写完成
3开始写
3写完成
4开始写
4写完成
0开始读
0读完成
2开始读
3开始读
3读完成
1开始读
4开始读
4读完成
1读完成
2读完成
```

### ReadWriteLock 是什么

首先明确一下，不是说 ReentrantLock 不好，只是 ReentrantLock 某些时候有局限。如果使用 ReentrantLock，可能本身是为了防止线程 A 在写数据、线程 B 在读数据造成的数据不一致，但这样，如果线程 C 在读数据、线程 D 也在读数据，读数据是不会改变数据的，没有必要加锁，但是还是加锁了，降低了程序的性能。因为这个，才诞生了读写锁 ReadWriteLock。

ReadWriteLock 是一个读写锁接口，读写锁是用来提升并发程序性能的锁分离技术，ReentrantReadWriteLock 是 ReadWriteLock 接口的一个具体实现，实现了读写的分离，读锁是共享的，写锁是独占的，读和读之间不会互斥，读和写、写和读、写和写之间才会互斥，提升了读写的性能。

而读写锁有以下三个重要的特性：

（1）公平选择性：支持非公平（默认）和公平的锁获取方式，吞吐量还是非公平优于公平。

（2）重进入：读锁和写锁都支持线程重进入。

（3）锁降级：遵循获取写锁、获取读锁再释放写锁的次序，写锁能够降级成为读锁。	

## 阻塞队列

![image-20210421152800616](https://gitee.com/zhang-songyao/blog-images/raw/master/20210421152802.png)

### 四组API

|      方式      | 抛出异常 | 不抛出异常，有返回值 | 阻塞一直等待 |            超时等待            |
| :------------: | :------: | :------------------: | :----------: | :----------------------------: |
|      添加      |   add    |        offer         |     put      | offer("c",2, TimeUnit.SECONDS) |
|      删除      |  remove  |         poll         |     take     |    poll(2,TimeUnit.SECONDS)    |
| 判断对列首元素 | element  |         peek         |      -       |               -                |

```java
/**
 * @author ：zsy
 * @date ：Created 2021/4/21 15:41
 * @description：阻塞队列
 */
public class bq {
    public static void main(String[] args) throws InterruptedException {
        test04();
    }

    /**
     * 抛出异常
     */
    public static void test01() {
        BlockingQueue blockingQueue = new ArrayBlockingQueue(2);
        blockingQueue.add("a");
        blockingQueue.add("b");
        //抛出异常IllegalStateException
        //blockingQueue.add("c");
        System.out.println(blockingQueue.element());
        blockingQueue.remove();
        blockingQueue.remove();
        //抛出异常NoSuchElementException
        //blockingQueue.remove();
    }

    /**
     * 不抛出异常，有返回值
     */
    public static void test02() {
        BlockingQueue blockingQueue = new ArrayBlockingQueue(2);
        System.out.println(blockingQueue.offer("a"));
        System.out.println(blockingQueue.offer("b"));
        System.out.println(blockingQueue.offer("c"));
        blockingQueue.peek();
        System.out.println(blockingQueue.poll());
        System.out.println(blockingQueue.poll());
        //blockingQueue.poll();
    }

    /**
     * 阻塞一直等待
     */
    public static void test03() throws InterruptedException {
        BlockingQueue blockingQueue = new ArrayBlockingQueue(2);
        blockingQueue.put("a");
        blockingQueue.put("b");
        blockingQueue.put("c");

        blockingQueue.take();
        blockingQueue.take();
        //blockingQueue.remove();
    }

    /**
     * 超时等待
     * @throws InterruptedException
     */
    public static void test04() throws InterruptedException {
        BlockingQueue blockingQueue = new ArrayBlockingQueue(2);
        System.out.println(blockingQueue.offer("a"));
        System.out.println(blockingQueue.offer("b"));
        blockingQueue.offer("c",2, TimeUnit.SECONDS);

        System.out.println(blockingQueue.poll());
        System.out.println(blockingQueue.poll());
        blockingQueue.poll(2,TimeUnit.SECONDS);
    }
}
```

### SynchronousQueue

>同步队列没有任何内部容量，甚至没有容量，每个插入操作必须等待另一个线程响应的删除操作

```java
/**
 * @author ：zsy
 * @date ：Created 2021/4/21 16:37
 * @description：同步队列
 */
public class SynchronousQueueDemo {

    public static void main(String[] args) throws InterruptedException {
        BlockingQueue<String> blockingQueue = new SynchronousQueue<>();
        new Thread(() -> {
            try {
                for (int i = 0; i < 3; i++) {
                    System.out.println(Thread.currentThread().getName() + "添加元素" + i);
                    blockingQueue.put("a" + i);
                    System.out.println(Thread.currentThread().getName() + "添加成功" + i);
                }
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }, "A").start();
        new Thread(() -> {

            for (int i = 0; i < 3; i++) {
                try {
                    TimeUnit.SECONDS.sleep(2);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                System.out.println(Thread.currentThread().getName() + "删除元素");
                System.out.println(blockingQueue.poll());
                System.out.println(Thread.currentThread().getName() + "删除成功");
            }
        }, "B").start();
    }
}
A添加元素0
B删除元素
a0
B删除成功
A添加成功0
A添加元素1
B删除元素
a1
B删除成功
A添加成功1
A添加元素2
B删除元素
a2
B删除成功
A添加成功2
```

## 线程池

>池化技术

程序运行的本质：占用系统的资源，优化资源的使用 => 池化技术

> 线程池的好处

1. 降低资源消耗
2. 提高响应速度
3. 方便管理
4. 线程复用，可以控制最大并发数，管理线程

### 创建线程池的四大方法

```java
/**
 * @author ：zsy
 * @date ：Created 2021/4/21 17:05
 * @description：线程池的三大方法
 */
public class Demo1 {
    public static void main(String[] args) {
        //ExecutorService threadPool = Executors.newSingleThreadExecutor();//创建一个线程
        //ExecutorService threadPool = Executors.newFixedThreadPool(5);//创建指定数量的线程
        ExecutorService threadPool = Executors.newCachedThreadPool();//创建足够多的线程

        try {
            for (int i = 0; i < 10; i++) {
                final int tmp = i;
                threadPool.execute(() -> {
                    System.out.println(Thread.currentThread().getName() + "----->" + tmp);
                });
            }
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            threadPool.shutdown();

        }
    }
}
```

>newScheduledThreadPool 创建一个支持定时及周期性的任务执行的线程池，多数情况下可用来替代Timer类。

#### 三大方法源码分析

> newSingleThreadExecutor

```java
public static ExecutorService newSingleThreadExecutor() {
    return new FinalizableDelegatedExecutorService
        (new ThreadPoolExecutor(1, 1,
                                0L, TimeUnit.MILLISECONDS,
                                new LinkedBlockingQueue<Runnable>()));
}
```

>newFixedThreadPool

```java
public static ExecutorService newFixedThreadPool(int nThreads) {
    return new ThreadPoolExecutor(nThreads, nThreads,
                                  0L, TimeUnit.MILLISECONDS,
                                  new LinkedBlockingQueue<Runnable>());
}
```

>newCachedThreadPool

```java
public static ExecutorService newCachedThreadPool() {
    return new ThreadPoolExecutor(0, Integer.MAX_VALUE,
                                  60L, TimeUnit.SECONDS,
                                  new SynchronousQueue<Runnable>());
}
```

### 参数分析

```java
public ThreadPoolExecutor(int corePoolSize, 					
                          int maximumPoolSize,					
                          long keepAliveTime,					
                          TimeUnit unit,						
                          BlockingQueue<Runnable> workQueue) {	
    this(corePoolSize, maximumPoolSize, keepAliveTime, unit, workQueue,
         Executors.defaultThreadFactory(), defaultHandler);
}

public ThreadPoolExecutor(int corePoolSize,//核心线程数
                          int maximumPoolSize,//最大线程数
                          long keepAliveTime,//存在时间。超过时间会释放线程
                          TimeUnit unit,//存在时间单位
                          BlockingQueue<Runnable> workQueue,//阻塞队列 存放请求
                          ThreadFactory threadFactory,//创建线程的工厂
                          RejectedExecutionHandler handler//拒绝策略
                         ) {
    if (corePoolSize < 0 ||
        maximumPoolSize <= 0 ||
        maximumPoolSize < corePoolSize ||
        keepAliveTime < 0)
        throw new IllegalArgumentException();
    if (workQueue == null || threadFactory == null || handler == null)
        throw new NullPointerException();
    this.corePoolSize = corePoolSize;
    this.maximumPoolSize = maximumPoolSize;
    this.workQueue = workQueue;
    this.keepAliveTime = unit.toNanos(keepAliveTime);
    this.threadFactory = threadFactory;
    this.handler = handler;
}
```

创建的线程池最大包含请求数量 = maximumPoolSize + workQueue的长度。超过最大请求数量，就会采取拒绝策略

![image-20210421174200019](https://gitee.com/zhang-songyao/blog-images/raw/master/20210421204024.png)

### 四个拒绝策略

- AbortPolicy：抛出异常RejectedExecutionException
- CallerRunsPolicy：执行调用者线程中的任务，除非执行线程已被关闭，否则任务被抛弃，返回上一级线程
- DiscardPolicy：丢弃拒绝的任务，不抛出异常
- DiscardOldestPolicy：被拒绝的任务的处理程序，丢弃最旧的未处理请求，然后重试 execute ，除非执行程序被关闭，在这种情况下，任务被丢弃。（与最先来的线程竞争）

![image-20210421171756151](https://gitee.com/zhang-songyao/blog-images/raw/master/20210421204025.png)

### 自定义线程池

```java
/**
 * @author ：zsy
 * @date ：Created 2021/4/21 17:50
 * @description：自定义线程池
 */
public class Demo2 {
    public static void main(String[] args) {
        ExecutorService threadPool = new ThreadPoolExecutor(2,
                5,
                3,
                TimeUnit.SECONDS,
                new LinkedBlockingDeque<>(3),
                new ThreadPoolExecutor.AbortPolicy());
        //线程池最大容量为5 + 3 = 8

        try {
            for (int i = 0; i < 8; i++) {
                final int tmp = i;
                threadPool.execute(new Thread(() -> {
                    System.out.println(Thread.currentThread().getName() + "---->" + tmp);
                }));
            }
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            threadPool.shutdown();
        }
    }
}

```

### 总结

> 最大线程池数量如何定义

- IO密集型：判断程序中十分消耗IO的线程数量，最大线程数量大于2倍的IO线程数量
- CPU密集型：判断CPU核数，最大线程池数量等于CPU核数，可以保持CPU的高效运行

### 线程池的工作流程

<img src="https://gitee.com/zhang-songyao/blog-images/raw/master/20210424211012.png" alt="image-20210424211008284" style="zoom:80%;" />

1）当提交一个新任务到线程池时，线程池判断corePoolSize线程池是否都在执行任务，如果有空闲线程，则从核心线程池中取一个线程来执行任务，直到当前线程数等于corePoolSize；

2）如果当前线程数为corePoolSize，继续提交的任务被保存到阻塞队列中，等待被执行；

3）如果阻塞队列满了，那就创建新的线程执行当前任务，直到线程池中的线程数达到maxPoolSize，这时再有任务来，由饱和策略来处理提交的任务

### 线程池都有哪些状态？

- RUNNING：这是最正常的状态，接受新的任务，处理等待队列中的任务。
- SHUTDOWN：不接受新的任务提交，但是会继续处理等待队列中的任务。
- STOP：不接受新的任务提交，不再处理等待队列中的任务，中断正在执行任务的线程。
- TIDYING：所有的任务都销毁了，workCount 为 0，线程池的状态在转换为 TIDYING 状态时，会执行钩子方法 terminated()。
- TERMINATED：terminated()方法结束后，线程池的状态就会变成这个。
  

## 四大函数式接口

> 只有一个方法的接口

<img src="https://gitee.com/zhang-songyao/blog-images/raw/master/20210421204026.png" alt="image-20210421192526470"/>

### 函数式接口

> 输入T类型返回R类型

```java
public interface Function<T, R> {
    /**
     * Applies this function to the given argument.
     *
     * @param t the function argument
     * @return the function result
     */
    R apply(T t);
```

实例

```java
/**
 * @author ：zsy
 * @date ：Created 2021/4/21 19:26
 * @description：函数式接口
 */
public class Demo1 {
    public static void main(String[] args) {
        Function<String, String> function = str -> {return str;};

        System.out.println(function.apply("zhangsna"));
    }
}
```

### 断定性接口

> 输入T类型返回boolean类型

```java
@FunctionalInterface
public interface Predicate<T> {

    /**
     * Evaluates this predicate on the given argument.
     *
     * @param t the input argument
     * @return {@code true} if the input argument matches the predicate,
     * otherwise {@code false}
     */
    boolean test(T t);
```

实例

```java
/**
 * @author ：zsy
 * @date ：Created 2021/4/21 19:29
 * @description：断定性接口
 */
public class Demo2 {
    public static void main(String[] args) {
        Predicate<String> predicate = str -> {return str.isEmpty();};

        predicate.test("zhangsan");

    }
}
```

### 消费型接口

> 只要输入类型，没有返回类型

```java
@FunctionalInterface
public interface Consumer<T> {

    /**
     * Performs this operation on the given argument.
     *
     * @param t the input argument
     */
    void accept(T t);
```

### 供给性接口

> 只有返回类型，没有输入类型

```java
@FunctionalInterface
public interface Supplier<T> {

    /**
     * Gets a result.
     *
     * @return a result
     */
    T get();
}
```

实例

```java
/**
 * @author ：zsy
 * @date ：Created 2021/4/21 19:37
 * @description：消费型接口和供给型接口
 */
public class Demo3 {

    public static void main(String[] args) {
        Consumer<String> consumer = o -> System.out.println(o);

        Supplier<Integer> supplier = () -> 1024;
        
    }
}
```

## 流式编程

大数据 ： 存储 + 计算

存储交给集合、数据库

计算交给流

```java
/**
 * @author ：zsy
 * @date ：Created 2021/4/21 20:05
 * @description：流操作
 */
public class Demo1 {
    public static void main(String[] args) {
        List<User> userList = new ArrayList<>();
        userList.add(new User("a", 10,30.0));
        userList.add(new User("b", 11,25.0));
        userList.add(new User("c", 12,40.0));
        userList.add(new User("d", 14,31.0));
        userList.add(new User("e", 16,35.0));
        //找出年龄大于11成绩大于33的用户倒序的第一位名字的大写
        userList.stream()
                .filter(u -> {return u.getAge() > 11;})
                .filter(u -> {return u.getPoint() > 33;})
                .sorted((u1, u2) -> {return u2.getUsername().compareTo(u1.getUsername());})
                .limit(1)
                .map(user -> {return user.getUsername().toUpperCase();})
                .forEach(System.out :: println);
    }
}
```

## Future

> 参考：[JAVA Future类详解](https://blog.csdn.net/qq_35190492/article/details/115609360?ops_request_misc=%257B%2522request%255Fid%2522%253A%2522161916261716780261915658%2522%252C%2522scm%2522%253A%252220140713.130102334.pc%255Fblog.%2522%257D&request_id=161916261716780261915658&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~blog~first_rank_v2~rank_v29-1-115609360.nonecase&utm_term=future)

### 什么是Future模式

Future模式是多线程开发中常见的设计模式，它的核心思想就是异步调用，他无法立即返回你需要的数据，但是他会返回一个契约，将来你可以凭借这个契约去获取你需要的信息。**例如**：点外卖，外卖订单相当于契约，你可以通过外面订单获取到你的外卖，而制作外卖是一个很长的过程，无法立刻获得外卖。

> 同步方法

客户端发出获取数据的请求，尽管请求需要很长一段时间才能返回，但是同步模式规定客户端一直等到到数据返回后才进行后续任务

### Java中的Future

![img](https://gitee.com/zhang-songyao/blog-images/raw/master/20210423162852.png)

首先，JDK内部有一个Future接口，这就是类似前面提到的订单，当然了，作为一个完整的商业化产品，这里的Future的功能更加丰富了，除了get()方法来获得真实数据以外，还提供一组辅助方法，比如：

- cancel()：如果等太久，你可以直接取消这个任务
- isCancelled()：任务是不是已经取消了
- isDone()：任务是不是已经完成了
- get()：有2个get()方法，不带参数的表示无穷等待，或者你可以只等待给定时间

### 什么是FutureTask

FutureTask表示一个异步运算任务，FutureTask里面可以传入一个Callable的具体实现类，可以对这个异步运算任务的结果进行等待获取、判断是否已经完成、取消任务等操作。只有当运算完成的时候结果才能取回，如果尚未完成get方法将阻塞，一个FutureTask对象可以对调用了Callable和Runnable的对象进行包装，由于由于 FutureTask 也是Runnable 接口的实现类，所以 FutureTask 也可以放入线程池中。

### ForkJoin

> 什么是ForkJoin 

并发执行任务，提高效率，大数据量

![image-20210422164719113](https://gitee.com/zhang-songyao/blog-images/raw/master/20210422164720.png)

```java
/**
 * @author ：zsy
 * @date ：Created 2021/4/22 17:10
 * @description：ForkJoin
 */
public class ForkJoinDemo extends RecursiveTask<Long> {

    private Long start;
    private Long end;
    private final Long tmp = 1000L;

    public ForkJoinDemo(Long start, Long end) {
        this.start = start;
        this.end = end;
    }

    @Override
    protected Long compute() {
        if (end - start > tmp) {
            Long sum = 0L;
            for (Long i = start; i <= end; i++) {
                sum += i;
            }
            return sum;
        } else {
            Long mid = start + end >> 1;
            ForkJoinDemo f1 = new ForkJoinDemo(start, mid);
            ForkJoinDemo f2 = new ForkJoinDemo(mid + 1, end);
            return  f1.join() + f2.join();
        }
    }

    public static void main(String[] args) {
        test1();
    }


    public static void test1() {
        long start = System.currentTimeMillis();
        Long sum = (0 + 10_0000_0000) * 10_0000_0000L / 2;
        long end = System.currentTimeMillis();
        System.out.println(end - start + "ms : " + sum);
    }

    public static void test2() {
        long start = System.currentTimeMillis();
        Long sum = 0L;
        ForkJoinPool forkJoinPool = new ForkJoinPool();
        ForkJoinDemo forkJoinDemo = new ForkJoinDemo(0L, 10_0000_0000L);
        ForkJoinTask submit = forkJoinPool.submit(forkJoinDemo);
        try {
            sum = (Long) submit.get();
        } catch (InterruptedException e) {
            e.printStackTrace();
        } catch (ExecutionException e) {
            e.printStackTrace();
        }
        long end = System.currentTimeMillis();
        System.out.println(end - start + "ms : " + sum);
    }

    public static void test3() {
        long start = System.currentTimeMillis();
        Long sum = 0L;
        sum = LongStream.rangeClosed(0,10_0000_0000L).parallel().reduce(0, Long :: sum);
        long end = System.currentTimeMillis();
        System.out.println(end - start + "ms : " + sum);
    }
}

```

### CompletableFuture

Future模式虽然好用，但也有一个问题，那就是将任务提交给线程后，调用线程并不知道这个任务什么时候执行完，如果执行调用get()方法或者isDone()方法判断，可能会进行不必要的等待，那么系统的吞吐量很难提高。

为了解决这个问题，JDK对Future模式又进行了加强，创建了一个CompletableFuture，它可以理解为Future模式的升级版本，它最大的作用是提供了一个回调机制，可以在任务完成后，自动回调一些后续的处理，这样，整个程序可以把“结果等待”完全给移除了。

开辟一个新的线程去执行异步任务，如果需要获取返回值，那么主线程将进入阻塞状态

> 异步调用线程

```java
/**
 * @author ：zsy
 * @date ：Created 2021/4/24 11:56
 * @description：
 */
public class CompletableDemo {

    public static void main(String[] args) {
        //创建异步任务
        CompletableFuture.supplyAsync(CompletableDemo::getPrice)
                //当getPrice()执行完后会回调当前函数
                .thenAccept(result -> {
                    System.out.println("price = " + result);
                })
                //出现异常后，会回调exceptionally
                .exceptionally(e -> {
                    System.out.println(e.getMessage());
                    return null;
                });
        System.out.println(1111);
        try {
            Thread.sleep(2000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println(2222);
    }

    public static Double getPrice() {
        try {
            Thread.sleep(1000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        if (Math.random() < 0.3) {
            throw new RuntimeException("Error");
        }
        return Math.random() * 20;
    }
}
```

在这个例子中，首先以getPrice()为基础创建一个异步调用，接着，使用thenAccept()方法，设置了一个后续的操作，也就是当getPrice()执行完成后的后续处理。

不难看到，CompletableFuture比一般的Future更具有实用性，因为它可以在Future执行成功后，自动回调进行下一步的操作，因此整个程序不会有任何阻塞的地方（也就是说你不用去到处等待Future的执行，而是让Future执行成功后，自动来告诉你）。

以上面的代码为例，CompletableFuture之所有会有那么神奇的功能，完全得益于AsyncSupply类（由上述代码中的supplyAsync()方法创建）。
AsyncSupply在执行时，如下所示：

```java
    public void run() {
        CompletableFuture<T> d; Supplier<T> f;
        if ((d = dep) != null && (f = fn) != null) {
            dep = null; fn = null;
            if (d.result == null) {
                try {
                    //这里就是你要执行的异步方法
                    //结果会被保存下来，放到d.result字段中
                    d.completeValue(f.get());
                } catch (Throwable ex) {
                    d.completeThrowable(ex);
                }
            }
            //执行成功了，进行后续处理，在这个后续处理中，就会调用thenAccept()中的消费者
            //这里就相当于Future完成后的通知
            d.postComplete();
        }
    }
```
继续看d.postComplete()，这里会调用后续一系列操作

```java
   final void postComplete() {
                //省略部分代码，重点在tryFire()里
                //在tryFire()里，真正触发了后续的调用，也就是thenAccept()中的部分
                f = (d = h.tryFire(NESTED)) == null ? this : d;
            }
        }
    }
```

## JMM

> JMM为了屏蔽各种硬件和操作系统的内存访问差异，以实现Java程序在各种平台下都能达到一直的访问

### 工作内存的八种操作

![image-20210423173533103](https://gitee.com/zhang-songyao/blog-images/raw/master/20210423173536.png)

关于主内存与工作内存之间的交互协议，即一个变量如何从主内存拷贝到工作内存。如何从工作内存同步到主内存中的实现细节。java内存模型定义了8种操作来完成。这8种操作每一种都是原子操作。8种操作如下：

- lock(锁定)：作用于主内存，它把一个变量标记为一条线程独占状态；
- read(读取)：作用于主内存，它把变量值从主内存传送到线程的工作内存中，以便随后的load动作使用；
- load(载入)：作用于工作内存，它把read操作的值放入工作内存中的变量副本中；
- use(使用)：作用于工作内存，它把工作内存中的值传递给执行引擎，每当虚拟机遇到一个需要使用这个变量的指令时候，将会执行这个动作；
- assign(赋值)：作用于工作内存，它把从执行引擎获取的值赋值给工作内存中的变量，每当虚拟机遇到一个给变量赋值的指令时候，执行该操作；
- store(存储)：作用于工作内存，它把工作内存中的一个变量传送给主内存中，以备随后的write操作使用；
- write(写入)：作用于主内存，它把store传送值放到主内存中的变量中。
- unlock(解锁)：作用于主内存，它将一个处于锁定状态的变量释放出来，释放后的变量才能够被其他线程锁定；

Java内存模型还规定了执行上述8种基本操作时必须满足如下规则:

（1）不允许read和load、store和write操作之一单独出现（即不允许一个变量从主存读取了但是工作内存不接受，或者从工作内存发起会写了但是主存不接受的情况），以上两个操作必须按顺序执行，但没有保证必须连续执行，也就是说，read与load之间、store与write之间是可插入其他指令的。

（2）不允许一个线程丢弃它的最近的assign操作，即变量在工作内存中改变了之后必须把该变化同步回主内存。

（3）不允许一个线程无原因地（没有发生过任何assign操作）把数据从线程的工作内存同步回主内存中。

（4）一个新的变量只能从主内存中“诞生”，不允许在工作内存中直接使用一个未被初始化（load或assign）的变量，换句话说就是对一个变量实施use和store操作之前，必须先执行过了assign和load操作。

（5）一个变量在同一个时刻只允许一条线程对其执行lock操作，但lock操作可以被同一个条线程重复执行多次，多次执行lock后，只有执行相同次数的unlock操作，变量才会被解锁。

（6）如果对一个变量执行lock操作，将会清空工作内存中此变量的值，在执行引擎使用这个变量前，需要重新执行load或assign操作初始化变量的值。

（7）如果一个变量实现没有被lock操作锁定，则不允许对它执行unlock操作，也不允许去unlock一个被其他线程锁定的变量。

（8）对一个变量执行unlock操作之前，必须先把此变量同步回主内存（执行store和write操作）。

### [JMM和底层实现原理](https://www.jianshu.com/p/8a58d8335270)

#### 现代计算机内存模型

其中一个重要的复杂性来源是绝大多数的运算任务都不可能只靠处理器“计算”就能完成，处理器至少要与内存交互，如读取运算数据、存储运算结果等，这个I/O操作是很难消除的（无法仅靠寄存器来完成所有运算任务）。早期计算机中cpu和内存的速度是差不多的，但在现代计算机中，cpu的指令速度远超内存的存取速度,由于计算机的存储设备与处理器的运算速度有几个数量级的差距，所以现代计算机系统都不得不加入一层读写速度尽可能接近处理器运算速度的高速缓存（Cache）来作为内存与处理器之间的缓冲：将运算需要使用到的数据复制到缓存中，让运算能快速进行，当运算结束后再从缓存同步回内存之中，这样处理器就无须等待缓慢的内存读写了。

![img](https://gitee.com/zhang-songyao/blog-images/raw/master/20210423180309.png)

> 缓存一致性问题

- 通过在总线加LOCK#锁的方式	
- 通过缓存一致性协议

在早期的CPU当中，是通过在总线上加LOCK#锁的形式来解决缓存不一致的问题。因为CPU和其他部件进行通信都是通过总线来进行的，如果对总线加LOCK#锁的话，也就是说阻塞了其他CPU对其他部件访问（如内存），从而使得只能有一个CPU能使用这个变量的内存。比如上面例子中 如果一个线程在执行 i = i +1，如果在执行这段代码的过程中，在总线上发出了LCOK#锁的信号，那么只有等待这段代码完全执行完毕之后，其他CPU才能从变量i所在的内存读取变量，然后进行相应的操作。这样就解决了缓存不一致的问题。

但是上面的方式会有一个问题，由于在锁住总线期间，其他CPU无法访问内存，导致效率低下。

所以就出现了缓存一致性协议。最出名的就是Intel 的MESI协议，MESI协议保证了每个缓存中使用的共享变量的副本是一致的。它核心的思想是：当CPU写数据时，如果发现操作的变量是共享变量，即在其他CPU中也存在该变量的副本，会发出信号通知其他CPU将该变量的缓存行置为无效状态，因此当其他CPU需要读取这个变量时，发现自己缓存中缓存该变量的缓存行是无效的，那么它就会从内存重新读取。

#### Java内存模型

JMM定义了Java 虚拟机(JVM)在计算机内存(RAM)中的工作方式。JMM定义了线程和主内存之间的抽象关系：线程之间的共享变量存储在主内存（Main Memory）中，每个线程都一个私有的本地内存（Local Memory），本地内存中存储了该线程以读/写共享变量的副本。本地内存是一个抽象概念，并不真实存在，它涵盖了缓存、写缓冲区、寄存器以及其他硬件和编译器优化

![img](https://gitee.com/zhang-songyao/blog-images/raw/master/20210423180906.png)

> JMM的核心是找到一个平衡性，在保证内存可见性的前提下，尽量放松对编译器和处理器的重排序限制

#### JVM对Java内存模型的实现

JVM中，Java内存模型将内存分为了两部分：线程栈区和堆区，JVM中运行的每个线程都有自己的线程栈，线程栈包含了当前线程执行的方法调用和相关信息，我们把它称作调用栈，调用栈不断变化。

![img](https://gitee.com/zhang-songyao/blog-images/raw/master/20210423181142.png)

- 所有的原始数据类型（8大原始数据类型）的局部变量，都是直接保存在线程栈中的，他们的值各个线程之间是相互独立的，无法共享，但是可以传递。
- 堆区包含了Java应用创建的所有对象信息不管对象是哪个线程创建的，其中的对象包括原始类型的封装类（如Byte、Integer、Long等等）。不管对象是属于一个成员变量还是方法中的局部变量，它都会被存储在堆区。
- 对于一个对象的成员方法，这些方法中包含局部变量，仍需要存储在栈区，即使它们所属的对象在堆区。 对于一个对象的成员变量，不管它是原始类型还是包装类型，都会被存储到堆区。Static类型的变量以及类本身相关信息都会随着类本身存储在堆区。

#### 重排序

> 重排序类型

![img](https://gitee.com/zhang-songyao/blog-images/raw/master/20210424145310.png)

- 编译器优化的重排序。编译器在不改变单线程程序语义的前提下，可以重新安排语句的执行顺序。
- 指令级并行的重排序。现代处理器采用了指令级并行技术（Instruction-LevelParallelism，ILP）来将多条指令重叠执行。如果不存在数据依赖性，处理器可以改变语句对应机器指令的执行顺序。
- 内存系统的重排序。由于处理器使用缓存和读/写缓冲区，这使得加载和存储操作看上去可能是在乱序执行。

#### as-if-serial

不管如何重排序，都必须保证代码在单线程下运行正确，连单线程下都无法正确运行，更不用考虑多线程并发情况，所以提出了as-if-serial概念

as-if-serial语义的意思是：不管怎么重排序（编译器和处理器为了提高并行度），单线程的执行结果不能被改变。所以编译器不会对存在数据依赖关系的操作做重排序，因为重排序会改变执行结果（这里所说的数据依赖仅仅针对单个处理器中执行的执行序列和单个线程中执行的操作，不同处理器之间和不同线程之间的**数据依赖性不能被编译器和处理器考虑**）但是，如果操作之间不存在数据依赖关系，这些操作依然可能被编译器和处理器重排序。

### [volatile关键字](https://www.cnblogs.com/dolphin0520/p/3920373.html)

#### volatile关键字的原理和实现机制

观察加入volatile关键字和没有加入volatile关键字时所生成的汇编代码发现，加入volatile关键字时，会多出一个lock前缀指令”，lock前缀指令实际上相当于一个内存屏障（也成内存栅栏），内存屏障会提供3个功能：

1. 它确保指令重排序时不会把其后面的指令排到内存屏障之前的位置，也不会把前面的指令排到内存屏障的后面；即在执行到内存屏障这句指令时，在它前面的操作已经全部完成；
2. 它会强制将对缓存的修改操作立即写入主存；
3. 如果是写操作，它会导致其他线程中（本地内存）对应的缓存行无效。

#### volatile关键字使用场景

synchronized关键字是防止多个线程同时执行一段代码，那么就会很影响程序执行效率，而volatile关键字在某些情况下性能要优于synchronized，但是要注意volatile关键字是无法替代synchronized关键字的，因为volatile**关键字**。通常来说，使用volatile必须具备以下2个条件：

1. 对变量的写操作不依赖于当前值
2. 该变量不会与其他状态变量一起纳入不变性条件中
3. 在访问变量时不需要加锁

　实际上，这些条件表明，可以被写入 volatile 变量的这些有效值独立于任何程序的状态，包括变量的当前状态。

　事实上，我的理解就是上面的2个条件需要保证操作是原子性操作，才能保证使用volatile关键字的程序在并发时能够正确执行。

#### Happen-Before

![img](https://gitee.com/zhang-songyao/blog-images/raw/master/20210423195912.png)

用happens-before的概念来阐述操作之间的内存可见性。如果一个操作的结果需要对另一个操作可见，那么两个操作之间必须要存在happens-before关系

两个操作之间具有happens-before关系，并不意味着前一个操作必须要在后一个操作之前执行！happens-before仅仅要求前一个操作（执行的结果）对后一个操作可见，且前一个操作按顺序排在第二个操作之前（the first is visible to and ordered before the second） 

- 程序次序规则：一个线程内，按照代码顺序，书写在前面的操作先行发生于书写在后面的操作
- 锁定规则：一个unLock操作先行发生于后面对同一个锁的lock操作
- volatile变量规则：对一个变量的写操作先行发生于后面对这个变量的读操作
- 传递规则：如果操作A先行发生于操作B，而操作B又先行发生于操作C，则可以得出操作A先行发生于操作C
- 线程启动规则：Thread对象的start()方法先行发生于此线程的每个一个动作
- 线程中断规则：对线程interrupt()方法的调用先行发生于被中断线程的代码检测到中断事件的发生
- 线程终结规则：线程中所有的操作都先行发生于线程的终止检测，我们可以通过Thread.join()方法结束、Thread.isAlive()的返回值手段检测到线程已经终止执行
- 对象终结规则：一个对象的初始化完成先行发生于他的finalize()方法的开始

### synchronized和volatile的区别

- volatile本质是在告诉JVM当前变量在寄存器（工作内存）中的值是不确定的，需要从主存中读取
- volatile仅能在变量级别使用，synchronized则可以使用在变量、方法、和类级别
- volatile仅能实现变量的修改可见性，不能保证原子性，而synchronized则可以保证变量的修改可见性和原子性
- volatile不会造成线程阻塞，synchronized可能会造成线程阻塞
- volatile标记的变量不会被编译器优化，synchronized标记的变量可以被编译器优化

## 单例模式

> 单例模式的特点

1. 单例类只能有一个实例
2. 单例类必须自己创建自己的唯一实例
3. 单例类必须给所有其他对象提供这一实例

单例模式确保某个类只有一个实例，而且自行实例化并向整个系统提供这个实例。在计算机系统中，线程池、缓存、日志对象、对话框、打印机、显卡的驱动程序对象常被设计成单例。这些应用都或多或少具有资源管理器的功能。每台计算机可以有若干个打印机，但只能有一个Printer Spooler，以避免两个打印作业同时输出到打印机中。每台计算机可以有若干通信端口，系统应当集中管理这些通信端口，以避免一个通信端口同时被两个请求同时调用。总之，选择单例模式就是为了避免不一致状态，避免政出多头。

### 饿汉式

>饿汉式在类创建的同时就已经创建好一个静态的对象供系统使用，以后不再改变，天生是线程安全的。

```java
/**
 * @author ：zsy
 * @date ：Created 2021/4/24 19:49
 * @description：饿汉式
 */
public class Hungry {
    //可能会浪费空间
    private Hungry() {}

    private final static Hungry HUNGRY = new Hungry();

    public static Hungry getInstance() {
        return HUNGRY;
    }
}
```

### 懒汉式

#### 普通懒汉式

```java
/**
 * @author ：zsy
 * @date ：Created 2021/4/24 19:51
 * @description：懒汉式
 */
public class LazyMan {

    private LazyMan() {}

    private static LazyMan LAZY_MAN = null;

    public static LazyMan getInstance() {
        if (LAZY_MAN == null) {
            LAZY_MAN = new LazyMan();
        }
        return LAZY_MAN;
    }
}
```

#### 同步方法的懒汉式

```java
/**
 * @author ：zsy
 * @date ：Created 2021/4/24 19:51
 * @description：懒汉式
 */
public class LazyMan {

    private LazyMan() {}

    private static LazyMan LAZY_MAN = null;

    public static synchronized LazyMan getInstance() {
        if (LAZY_MAN == null) {
            LAZY_MAN = new LazyMan();
        }
        return LAZY_MAN;
    }
}
```

这种写法是对getInstance()加了锁的处理，保证了同一时刻只能有一个线程访问并获得实例，但是缺点也很明显，因为synchronized是修饰整个方法，每个线程访问都要进行同步，而其实这个方法只执行一次实例化代码就够了，每次都同步方法显然效率低下，为了改进这种写法，就有了下面的双重检查懒汉式。

#### 双重检查懒汉式

> 存在问题多线程下调用getInstance()，会产生错误

```java
//双重检测锁模式的懒汉式单例 DCL懒汉式
public static LazyMan getInstance() {
    if (LAZY_MAN == null) {
        synchronized (LazyMan.class) {
            if (LAZY_MAN == null) {
                LAZY_MAN = new LazyMan();
            }
        }
    }
    return LAZY_MAN;
}
```

这种写法用了两个if判断，也就是Double-Check，并且同步的不是方法，而是代码块，效率较高，是对第三种写法的改进。为什么要做两次判断呢？这是为了线程安全考虑，还是那个场景，对象还没实例化，两个线程A和B同时访问静态方法并同时运行到第一个if判断语句，这时线程A先进入同步代码块中实例化对象，结束之后线程B也进入同步代码块，如果没有第二个if判断语句，那么线程B也同样会执行实例化对象的操作了。

> 分析为什么需要volatile关键字

通过双重检测锁模式的懒汉式单例模式多线程下仍然存在问题，因为 LAZY_MAN = new LazyMan();不是一个原子操作，而是分为三个步骤：

1. 分配内存空间
2. 执行构造方法创建对象
3. 把这个对象指向这个空间

下图由于指令重排，而发生了线程B拿到了LAZY_MAN对象，但是LAZY_MAN为null所以需要将LAZY_MAN设置为volatile

![image-20210424212047630](https://gitee.com/zhang-songyao/blog-images/raw/master/20210424212048.png)

```java
private volatile static LazyMan LAZY_MAN = new LazyMan();
```

> 存在问题：通过反射机制，打破封装仍然可以创建对象

```java
public static void main(String[] args) throws Exception {
    LazyMan instance1 = LazyMan.getInstance();
    Constructor<LazyMan> declaredConstructor = LazyMan.class.getDeclaredConstructor();
    declaredConstructor.setAccessible(true);
    LazyMan instance2 = declaredConstructor.newInstance();
    System.out.println(instance1);
    System.out.println(instance2);
}
```

> 改进措施

```java
private LazyMan() {
    if (LAZY_MAN != null) {
        throw new RuntimeException("使用反射破坏异常");
    }
}
```

> 存在问题：可以通过两次反射来创建对象

这次在构造方法加一个信号变量，只要创建了一次实例，就让信号变量置为true，以后再来调用直接抛出异常

```java
private static boolean isBuild = false;
private LazyMan() {
    if (!isBuild) {
        isBuild = true;
    } else {
        throw new RuntimeException("使用反射破坏异常");
    }	
}
```

当然这需要对信号变量进行加密，要不然仍然可以通过反射拿到信号变量。	

```java
public static void main(String[] args) throws Exception {
    //LazyMan instance1 = LazyMan.getInstance();
    Constructor<LazyMan> declaredConstructor = LazyMan.class.getDeclaredConstructor();
    declaredConstructor.setAccessible(true);
    //通过反射拿到信号变量
    Field isBuild = LazyMan.class.getDeclaredField("isBuild");
    isBuild.setAccessible(true);
    LazyMan instance1 = declaredConstructor.newInstance();
    //修改信号变量
    isBuild.set(instance1,false);
    LazyMan instance2 = declaredConstructor.newInstance();
    System.out.println(instance1);
    System.out.println(instance2);
}
```

> 完整代码

```java
public class LazyMan {

    private static boolean isBuild = false;

    private volatile static LazyMan INSTANCE;

    private LazyMan() {
        if (isBuild == false) {
            isBuild = true;
        } else {
            throw new RuntimeException("使用反射破坏异常");
        }

    }

    private volatile static LazyMan LAZY_MAN;

    //双重检测锁模式的懒汉式单例 DCL懒汉式
    public static LazyMan getInstance() {
        if (LAZY_MAN == null) {
            synchronized (LazyMan.class) {
                if (LAZY_MAN == null) {
                    LAZY_MAN = new LazyMan();
                    isBuild = true;
                }
            }
        }
        return LAZY_MAN;
    }
}


```

#### 静态内部类（推荐）

```java
/**
 * @author ：zsy
 * @date ：Created 2021/4/25 8:33
 * @description：静态内部类
 */
public class Singleton {

    private static class LazyHolder {
        private static final Singleton INSTANCE = new Singleton();
    }

    public static Singleton getInstance() {
        return LazyHolder.INSTANCE;
    }
}
```

这是很多开发者推荐的一种写法，这种静态内部类方式在Singleton类被装载时并不会立即实例化，而是在需要实例化时，调用getInstance方法，才会装载SingletonInstance类，从而完成对象的实例化。

同时，因为类的静态属性只会在第一次加载类的时候初始化，也就保证了SingletonInstance中的对象只会被实例化一次，并且这个过程也是线程安全的。

### Enum

不能使用反射破坏枚举

```java
/**
 * @author ：zsy
 * @date ：Created 2021/4/24 20:24
 * @description：枚举单例
 */
//enum本身也是一个Class
public enum EnumSingle {
    INSTANCE;

    public static EnumSingle getInstance() {
        return INSTANCE;
    }
}

class Test {
    public static void main(String[] args) throws Exception {
        EnumSingle instance1 = EnumSingle.getInstance();
        EnumSingle instance2 = EnumSingle.getInstance();

        Constructor<EnumSingle> declaredConstructor = EnumSingle.class.getDeclaredConstructor(String.class, int.class);
        declaredConstructor.setAccessible(true);
        EnumSingle enumSingle = declaredConstructor.newInstance();
        System.out.println(instance1);
        System.out.println(enumSingle);
    }
}
```

以上代码是通过反射创建枚举的实例抛出异常Exception in thread "main" java.lang.IllegalArgumentException: Cannot reflectively create enum objects

1. 线程安全问题。因为Java虚拟机在加载枚举类的时候会使用ClassLoader的方法，这个方法使用了同步代码块来保证线程安全。
2. 避免反序列化破坏对象，因为枚举的反序列化并不通过反射实现。

### 总结

#### 饿汉式和懒汉式区别

饿汉式：在类加载的时候就完成了初始化，所以类加载比较慢，但是获取对象的速度快

懒汉式：在类加载的时候不初始化，等到第一次使用时才初始化

#### 单例模式的优缺点

优点：

- 单例类只有一个实例，节省了内存资源。对于一些需要频繁创建销毁的对象，使用到单例模式可以提高系统性能；
- 单例模式可以在系统设置全局访问点，优化和共享数据，例如Web应用的页面计数器就可以用单例模式实现计数值的保存。

缺点：单例模式一般没有接口，扩展的话只能修改代码

## CAS

### 什么是CAS

CAS全称compareAndSwapObject，即比较替换，是实现并发应用到的一种技术。**compareAndSwapObject** 方法其实比较的就是两个 Java Object 的地址，如果相等则将新的地址（Java Object）赋给该字段。

CAS操作是一个原子操作，由一条CPU指令完成（CPU的并发原语），使用了三个基本操作数，内存地址V，旧的预期值A，要修改的新值B

>Java Unsafe包中的compareAndSwapObject()方法

比较并且交换Java Object 

```java
 /***
   * Compares the value of the object field at the specified offset
   * in the supplied object with the given expected value, and updates
   * it if they match.  The operation of this method should be atomic,
   * thus providing an uninterruptible way of updating an object field.
   *
   * @param obj the object containing the field to modify.
   * @param offset the offset of the object field within <code>obj</code>.
   * @param expect the expected value of the field.
   * @param update the new value of the field if it equals <code>expect</code>.
   * @return true if the field was changed.
   */
  public native boolean compareAndSwapObject(Object obj, long offset, Object expect, Object update);
```

CAS该方法的作用是原子操作比较并交换两个值，运用时底层硬件所提供的CAS支持，在Java API中该方法的四个参数

1. obj：包含要修改的字段对象
2. offset：字段在对象的偏移量
3. expect：字段的期望值
4. update：如果该字段的值等于字段的期望值，用于更新字段的新值

### 实例

```java
/**
 * @author ：zsy
 * @date ：Created 2021/4/18 9:16
 * @description：CAS测试
 */
public class UnsafeTest {

    private static Unsafe UNSAFE = null;
    private static long I_OFFSET;
    static {
        //Unsafe unsafe = Unsafe.getUnsafe();
        try {
            Field field = Unsafe.class.getDeclaredField("theUnsafe");
            field.setAccessible(true);
            UNSAFE = (Unsafe) field.get(null);
            I_OFFSET = UNSAFE.objectFieldOffset(Person.class.getDeclaredField("i"));
        } catch (NoSuchFieldException | IllegalAccessException e) {
            e.printStackTrace();
        }
    }

    public static void main(String[] args) {
        Person person = new Person();
        new Thread(() -> {
            while (true) {
                //person.i++;
                boolean b = UNSAFE.compareAndSwapInt(person, I_OFFSET, person.i, person.i + 1);
                if (b) {
                    System.out.println(Thread.currentThread().getName() + UNSAFE.getIntVolatile(person, I_OFFSET));
                    //System.out.println(Thread.currentThread().getName() + person.i);
                }

                try {
                    TimeUnit.MILLISECONDS.sleep(500);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        }, "A").start();
        new Thread(() -> {
            while (true) {
                //person.i++;
                boolean b = UNSAFE.compareAndSwapInt(person, I_OFFSET, person.i, person.i + 1);
                if (b) {
                    System.out.println(Thread.currentThread().getName() + UNSAFE.getIntVolatile(person, I_OFFSET));
                    //System.out.println(Thread.currentThread().getName() + person.i);
                }


                try {
                    TimeUnit.MILLISECONDS.sleep(500);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        }, "B").start();
    }
}
class Person {
    public int i;

}
```

### CAS存在的三大问题

- ABA问题
- 循环等待时间长开销大
- 只能保证一个共享变量的原子操作

#### ABA问题实现

<img src="https://gitee.com/zhang-songyao/blog-images/raw/master/20210421234833.jpg" alt="img" style="zoom: 67%;" />

> 乐观锁中详细介绍了ABA问题

```java
/**
 * @author ：zsy
 * @date ：Created 2021/4/24 21:45
 * @description：ABA问题
 */
public class ABADemo {

    private static AtomicInteger num = new AtomicInteger(1);

    public static void main(String[] args) {
        new Thread(() -> {
            System.out.println(Thread.currentThread().getName() + "---->" + num.get());
        }, "A").start();

        new Thread(() -> {
            num.compareAndSet(1, 21);
            System.out.println(Thread.currentThread().getName() + "修改了num：" + num.get());
        }, "B").start();

        new Thread(() -> {
            num.compareAndSet(21, 1);
            System.out.println(Thread.currentThread().getName() + "修改了num：" + num.get());
        }, "C").start();

        new Thread(() -> {
            try {
                Thread.sleep(100);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println(Thread.currentThread().getName() + "---->" + num.get());
        }, "A").start();
    }
}
A---->1
B修改了num：21
C修改了num：1
A---->1
```

#### ABA问题解决

> 引入原子运用

```java
/**
 * @author ：zsy
 * @date ：Created 2021/4/24 21:45
 * @description：ABA问题
 */
public class ABADemo {

    static AtomicStampedReference<Integer> num = new AtomicStampedReference<Integer>(1,1);

    public static void main(String[] args) {
        new Thread(() -> {
            System.out.println(Thread.currentThread().getName() + "版本号" + num.getStamp() + "---->" + num.getReference());
        }, "A").start();

        new Thread(() -> {
            try {
                Thread.sleep(11);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            num.compareAndSet(1, 21, num.getStamp(), num.getStamp() + 1);
            System.out.println(Thread.currentThread().getName() + "版本号" + num.getStamp() + "修改了num：" + num.getReference());
        }, "B").start();

        new Thread(() -> {
            try {
                Thread.sleep(31);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }

            num.compareAndSet(21, 1, num.getStamp(), num.getStamp() + 1);
            System.out.println(Thread.currentThread().getName() + "版本号" + num.getStamp() + "修改了num：" + num.getReference());
        }, "C").start();

        new Thread(() -> {
            try {
                Thread.sleep(100);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println(Thread.currentThread().getName()+ "版本号" + num.getStamp() + "---->" + num.getReference());
        }, "A").start();
    }
}

输出
A版本号1---->1
B版本号2修改了num：21
C版本号3修改了num：1
A版本号3---->1
```

# [并发编程面试题](https://thinkwon.blog.csdn.net/article/details/104863992)

# GC垃圾回收线程

finalize()方法什么时候被调用？析构函数(finalization)的目的是什么？
1）垃圾回收器（garbage colector）决定回收某对象时，就会运行该对象的finalize()方法；
finalize是Object类的一个方法，该方法在Object类中的声明protected void finalize() throws Throwable { }
在垃圾回收器执行时会调用被回收对象的finalize()方法，可以覆盖此方法来实现对其资源的回收。注意：一旦垃圾回收器准备释放对象占用的内存，将首先调用该对象的finalize()方法，并且下一次垃圾回收动作发生时，才真正回收对象占用的内存空间

2）GC本来就是内存回收了，应用还需要在finalization做什么呢？ 答案是大部分时候，什么都不用做(也就是不需要重载)。只有在某些很特殊的情况下，比如你调用了一些native的方法(一般是C写的)，可以要在finaliztion里去调用C的释放函数。g