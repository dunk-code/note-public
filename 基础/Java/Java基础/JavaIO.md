

[IO与NIO](https://www.jianshu.com/p/5bb812ca5f8e)

# JavaIO流

## 阻塞（Block）与非阻塞（Non-Block）

> 阻塞和非阻塞是进程在访问数据的时候，数据是否准备就绪的一种处理方式。

当数据没有准备的时候

**阻塞**：往往需要等待缓冲区中的数据准备好之后才处理其他的事情，否则一直等待。

**非阻塞**：当我们的进程访问我们的数据缓冲区的时候，如果数据没有准备好，则直接返回，不会等待，如果数据已经准备好，也直接返回

### 阻塞IO

![img](https://gitee.com/zhang-songyao/blog-images/raw/master/20210507174328.jpeg)

### 非阻塞IO

![img](https://gitee.com/zhang-songyao/blog-images/raw/master/20210507174351.jpeg)

## 同步（Synchronous）与异步（Asynchronous）

同步和异步都是基于应用程序和操作系统处理IO时间所采用的方式

**同步**：是应用程序要直接参与IO读写操作

**异步**：所有的IO读写都交给操作系统去处理，应用程序只需要等待通知。

同步方式在处理 IO 事件的时候，必须阻塞在某个方法上面等待我们的 IO 事件完成（阻塞 IO 事件或者通过轮询 IO事件的方式），对于异步来说，所有的 IO 读写都交给了操作系统。这个时候，我们可以去做其他的事情，并不需要去完成真正的 IO 操作，当操作完成 IO 后，会给我们的应用程序一个通知。

> 举例

同步阻塞：普通水壶烧水，站在旁边，主动看水开了没有

同步非阻塞：去干点别的事，每过一段时间去看看水开了没有，水没开就走人

异步阻塞：站在旁边，不会每过一段时间主动看水开了没有。如果水开了，水壶自动通知他。 

异步非阻塞：去干点别的事，如果水开了，水壶自动通知他。异步非阻塞

> 所以异步相比于同步带来的好处就是在我们处理IO数据的时候，异步的方式我们可以把这部分等待所消耗的资源用于处理其他事务，提升我们服务自身的性能

### 同步IO

![img](https://gitee.com/zhang-songyao/blog-images/raw/master/20210507175052.jpeg)

### 异步IO

![img](https://gitee.com/zhang-songyao/blog-images/raw/master/20210507175103.jpeg)

## 总结

同步和异步是通信机制，阻塞和非阻塞是调用状态

- 同步IO是用户线程发起IO请求后需要等待或轮询内核IO操作完成后才能继续执行

- 异步IO是用户线程发起IO请求后可以继续执行，当内核IO操作完成后会通知用户线程，或调用用户线程注册的回调函数
- 阻塞IO是IO操作需要彻底完成后才能返回用户空间
- 非阻塞IO是IO操作调用后立即返回一个状态值，无需等待IO操作彻底完成

## NIO模型

### NIO（Non-blocking / New I/O）

NIO是一种同步非阻塞的IO模型，于 Java 1.4 中引入，对应 `java.nio` 包，提供了Channel、Selector、Buffer等抽象。NIO中的N可以理解为Non-blocking，不单纯是 New。它支持面向缓冲的，基于通道的 I/O 操作方法。NIO提供了与传统BIO模型中的`Socket`和`ServerSocket`相对应的`SocketChannel`和`ServerSocketChannel`两种不同的套接字通道实现,两种通道都支持阻塞和非阻塞两种模式。对于高负载、高并发的（网络）应用，应使用 NIO 的非阻塞模式来开发



NIO的非阻塞模式，是一个线程从某通道发送请求读取数据，但是它仅能得到目前可用的数据，如果目前没有数据可用，就可以什么都不获取，而不是保持线程阻塞，所以直到数据变得可以读取之前，该线程可以继续做其他的事情，非阻塞写也是如此，一个线程请求写入一些数据到某通道，但不需要等待它完全写入，这个线程同时可以去做别的事情。 **线程通常将非阻塞IO的空闲时间用于在其它通道上执行IO操作，所以一个单独的线程现在可以管理多个输入和输出通道**

### NIO的特点

- 一个线程可以处理多个通道，减少线程数创建的数量
- 读写非阻塞，节约资源，没有可写\可读数据时，不会发生阻塞导致线程资源的浪费

### 核心组件

#### Channel（通道）

Channel是一个双向通道，与传统IO操作只允许单向的读写不同的是，NIO的Channel允许在一个通道上进行读和写的操作，替换了传统BIO中的Stream，不能直接访问数据，要通过Buffer来读取数据，也可和其他Channel交互

> 主要实现

`FileChannel`、`SocketChannel`、`ServerSocketChannel`、`DatagramChannel`

#### Buffer（缓冲区）

Buffer顾名思义，他是一个缓冲区，实际上是一个容器，一个连续数组，Channel提供从文件、网络读取数据的渠道，但是读写的数据都必须经过Buffer

![img](https://gitee.com/zhang-songyao/blog-images/raw/master/20210507200113.webp)

Buffer缓冲区本质是一块可以写入数据，然后可以从中读取数据的内存这块内存被包装成NIO Buffer对象，并提供了一组方法，用来方便的访问该模块内存。为了理解Buffer的工作原理，需要熟悉它的三个属性：

- position：下次读写数据的位置
- limit：本次读写的极限位置
- capacity：最大容量

使用步骤：向 Buffer 写数据，调用 flip 方法转为读模式，从 Buffer 中读数据，调用 clear 或 compact 方法清空缓冲区

- `flip` 将写转为读，底层实现原理把 position 置 0，并把 limit 设为当前的 position 值
- `clear` 将读转为写模式（用于读完全部数据的情况，把 position 置 0，limit 设为 capacity）
- `compact` 将读转为写模式（用于存在未读数据的情况，让 position 指向未读数据的下一个）

通道方向和 Buffer 方向相反，读数据相当于向 Buffer 写，写数据相当于从 Buffer 读

#### Selector（多路复用器）

轮询检查多个Channel的状态，判断注册事件是否发生，即判断Channel是否处于可读或可写状态，适用前需要将Channel注册到Selector中，注册后会得到一个`SelectionKey`，通过`SelectionKey`获取Channel和Selector的相关信息

## BIO模型

### BIO（传统IO）

- BIO是一个同步并阻塞的IO模式，传统的 `java.io` 包，它基于流模型实现，提供了我们最熟知的一些 IO 功能，比如**File抽象、输入输出流**等。**交互方式是同步、阻塞的方式**
- 在读取输入流或者写入输出流时，在读、写动作完成之前，线程会一直阻塞在那里，它们之间的调用是可靠的线性顺序
- 服务器实现模式为一个连接请求对应一个线程，服务器需要为每一个客户端请求创建一个线程，如果这个连接不做任何事会造成不必要的线程开销。可以通过线程池改善，这种IO称为伪异步IO，适用连接数目少且服务器资源多的场景

## Java BIO与NIO比较

| IO模型 |  BIO   |   NIO    |
| :----: | :----: | :------: |
|  通信  | 面向流 | 面向缓冲 |
|  处理  | 阻塞IO | 非阻塞IO |
|  触发  |   无   |  选择器  |

## AIO概念

AIO是JDK7引入的异步非阻塞IO。服务器实现模式为一个有效的请求对应一个线程，客户端的IO请求都是由操作系统先完成IO操作后，再通知服务器应用来直接使用准备好的数据，适用连接数目多且连接时间长的场景

实现方式包括通过 Future 的 `get` 方法进行阻塞式调用以及实现 CompletionHandler 接口，重写请求成功的回调方法 `completed` 和请求失败回调方法 `failed`

## JavaIO流分为哪几种

- 按照流的流向分，可以分为输入流和输出流
- 按照操作单元划分，可以分为字节流和字符流
- 按照流的角色划分为节点流和处理流

> 字符流一般用于文本文件，字节流一般用于图像或者其他文件

字符流包括了字符输入流Reader和字符输出流Writer，字节流包括字节输入流`inputStream`和字节输出流`OutputStream`。

字符流和字节流都有对应的缓冲流，字节流也可以分为字符流，缓冲流带有一个8KB的缓冲数组，可以提高流的读写效率。

除了缓冲流外还有过滤流`FilterReader`、字符数组流`CharArrayReader`、字节数组流`ByteArrayInputStream`、文件流`FileInputStream`

<img src="https://gitee.com/zhang-songyao/blog-images/raw/master/20210612210945.jpeg" alt="IO-操作对象分类" style="zoom:80%;" />