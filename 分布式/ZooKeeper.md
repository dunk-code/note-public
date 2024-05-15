# 入门

## 基本概念

大数据生态系统里很多组件的命名都是某种动物，例如Hadoop是🐘，hive是🐝，Zookeeper就是动物园管理者，是管理大数据生态系统各组件的管理员

Zookeeper是经典的分布式数据一致性解决方案，致力于为分布式应用提供一个高性能，高可用，且具有严格顺序访问控制能力的分布式协调存储服务

主要功能包括：

- 配置管理
- 分布式锁
- 集群管理

## 设计目标

- 高性能
  将数据存储在内存中，直接服务于客户端的所有非事务请求，尤其适合读为主的应用场景
- 高可用
  一般以集群方式对外提供服务，每台机器都会在内存中维护当前的服务状态，每台机器之间都保持通信。只要集群中超过一般机器都能正常工作，那么整个集群就能够正常对外服务。
- 严格顺序访问
  对于客户端的每个更新请求，Zookeeper都会生成全局唯一的递增编号，这个编号反应了所有事务操作的先后顺序。


## 下载Zookeeper

> 下载安装包

[官网](https://zookeeper.apache.org/)

> 解压

```shell
tar -zxvf apache-zookeeper-3.5.9-bin.tar.gz
```

> 修改配置

```shell
cd /data/zk/apache-zookeeper-3.5.9-bin/conf
cp zoo_sample.cfg zoo.cfg

# 修改数据目录
dataDir=/data/zk/apache-zookeeper-3.5.9-bin/data
```

## Zookeeper命令操作

### Zookeeper数据模型

> Zookeeper是一个树形目录服务，其数据模型和Unix的文件系统目录树类似，拥有一个层次化结构

Unix文件系统目录树：

<img src="https://gitee.com/zhang-songyao/blog-images/raw/master/20220112164536.png" alt="image-20220112164531743" style="zoom: 50%;" />

Zookeeper数据模型：

<img src="https://gitee.com/zhang-songyao/blog-images/raw/master/20220112180609.png" alt="image-20220112180601583" style="zoom:67%;" />

Zookeeper的数据节点可以视为树状结构（或者目录），树中各节点称为znode，一个znode可以有多个子节点Zookeeper结点在结构上表现为树状，使用路径来定位某个znode

znode兼具文件和目录两种特点（保存了自己的信息{少量1MB}和节点信息），既像文件一样维护着数据、元信息、ACL、时间戳等数据结构，又像目录一样可以作为路径标识的一部分

znode大体上分为三部分（使用get命令查看）

- 节点的数据
- 节点的子结点
- 节点的状态

节点类型（在创建时被确定且不能更改）

- 临时节点：生命周期依赖于创建它的会话，会话结束节点将被自动删除。临时节点不允许拥有子节点  `-e`
- 持久化节点：生命周期不依赖于会话，只有在客户端显式执行删除操作时才被删除
- 持久化顺序节点   `-s`
- 临时顺序节点  `-es`

### Zookeeper服务端常用命令

```shell
# 启动服务
./zkServer.sh start
# 关闭服务
./zkServer.sh stop
# 查看服务状态
./zkServer.sh status
# 重启服务
./zkServer.sh restart
```

### Zookeeper客户端常用命令

```shell
# 连接
zkCli.sh -server localhost:2181
# 关闭
quit
# 查看节点
ls	
ls2 <==> ls -s
# 创建节点
create node_name (node_val)
# 创建临时节点
create -e node_name
# 穿件顺序节点
create -s node_name
create -ed node_name
# 获取节点数据
get node_name 
# 设置节点数据
set node_name node_val
# 删除节点
delete node_name 
# 删除目录
deleteall 
```

## ZooKeeper Java API操作

### Curator介绍

[官网](https://curator.apache.org/)

Curator是 Zookeeper 的Java客户端

创建的Zookeeper Java 客户端有：

- 原生的Java API
- ZkClient
- Curator

### 依赖

```xml
<!-- 封装了一些高级特性，如：Cache事件监听、选举、分布式锁、分布式Barrier -->
<!-- https://mvnrepository.com/artifact/org.apache.curator/curator-recipes -->
<dependency>
    <groupId>org.apache.curator</groupId>
    <artifactId>curator-recipes</artifactId>
    <version>5.0.0</version>
</dependency>
<!-- 对zookeeper的底层api的一些封装 -->
<!-- https://mvnrepository.com/artifact/org.apache.curator/curator-framework -->
<dependency>
    <groupId>org.apache.curator</groupId>
    <artifactId>curator-framework</artifactId>
    <version>5.0.0</version>
</dependency>
```

### 测试连接

```java
@Test
public void testConnect() {
    /*
     * @param baseSleepTimeMs 第一次重试等待时间
     * @param maxRetries max number of times to retry
     * @param maxSleepMs 每次重试之间的睡眠时间
     */
    RetryPolicy retryPolicy = new ExponentialBackoffRetry(2000, 10, 1000);
    /*
     * Create a new client
     * @param connectString       list of servers to connect to
     * @param sessionTimeoutMs    session timeout
     * @param connectionTimeoutMs connection timeout
     * @param retryPolicy         retry policy to use
     * @return client
     */
    CuratorFramework client = CuratorFrameworkFactory.newClient(
            "192.168.19.129:2181",
            60 * 1000,
            15 * 1000,
            retryPolicy);
    // 开启连接
    client.start();
}
```

> 链式写法

```java
@Test
public void testConnect0() {
    RetryPolicy retryPolicy = new ExponentialBackoffRetry(2000, 10, 1000);
    CuratorFramework client;
    client = CuratorFrameworkFactory
            .builder()
            .sessionTimeoutMs(60 * 1000)
            .connectString("192.168.19.129:2181")
            .connectionTimeoutMs(15 * 1000)
            .retryPolicy(retryPolicy)
	        // 名称空间 每次操作zk /dunkcode
           	.namespace("dunkcode")
            .build();
    client.start();
}
```

### 操作节点

> CentOS7关闭防火墙

```shell
# 查看当前防火墙状态
systemctl status firewalld.service
# 关闭防火墙命令
systemctl stop firewalld.service
# 关闭开机自启动，禁止防火墙服务器
systemctl disable firewalld.service
# 开启开机启动
systemctl enable firewalld.service
```

> 增

```java
@Test
public void create() throws Exception {
    // 基本创建
    String path = client.create().forPath("/app1");
    // 创建节点带有数据
    client.create().forPath("/app2", "Hello Zookeeper".getBytes());
    // 设置节点类型
    client.create().withMode(CreateMode.EPHEMERAL_SEQUENTIAL).forPath("/app3");
    // 创建多级节点
    client.create().creatingParentsIfNeeded().forPath("/app4/p1");
    System.out.println(path);
    TimeUnit.SECONDS.sleep(10);
}
```

> 删

```java
@Test
public void delete() throws Exception {
    // 普通删除
    client.delete().forPath("/app1");
    // 删除带子节点的节点
    client.delete().deletingChildrenIfNeeded().forPath("/app4");
    // 保证一定删除节点
    client.delete().guaranteed().forPath("/app2");
    // 回调方式删除
    client.delete().inBackground((client, event) -> System.out.println(event)).forPath("/app3");
}
```

> 改

```java
@Test
public void set() throws Exception {
    // 基本修改
    client.setData().forPath("/app1", "hehe".getBytes());
    System.out.println("修改成功");
    TimeUnit.SECONDS.sleep(10);
    // 根据版本修改
    Stat stat = new Stat();
    client.getData().storingStatIn(stat).forPath("/app1");
    int version = stat.getVersion();
    client.setData().withVersion(version).forPath("/app1", "haha".getBytes());
    System.out.println("根据版本修改成功,当前版本--->" + stat.getVersion());
}
```

> 查

```java
@Test
public void get() throws Exception {
    // 获取节点数据
    byte[] bytes = client.getData().forPath("/app1");
    System.out.println(new String(bytes));
    // ls
    List<String> list = client.getChildren().forPath("/");
    System.out.println(list);
    // ls2 ls -s 
    Stat stat = new Stat();
    byte[] bytes1 = client.getData().storingStatIn(stat).forPath("/app1");
    System.out.println(new String(bytes1));
    System.out.println(stat);
}
```

### Watch事件监听

Zookeeper运用用户在指定节点上注册一些Watcher，并且在一些特定事件触发的时候，Zookeeper服务端会将事件通知到指感情去的客户端上去，该机制是Zookeeper实现分布式协调服务的重要特性

Zookeeper中引入Watcher机制来实现发布/订阅功能，能够让多个订阅者同时监听某一个对象，当一个对象自身状态发生变更的时候，会通知所有的订阅者

ZooKeeper 原生支持通过注册Watcher来进行事件监听，但是其使用并不是特别方便需要开发人员自己反复注册Watcher，比较繁琐

Curator引入了Cache 来实现对 ZooKeeper 服务端事件的监听

ZooKeeper提供了三种Watches:

- NodeCache:只是监听某一个特定的节点（根节点）
- PathChildrenCache: 监控一个ZNode的子节点（子节点）
- TreeCache：可以监控整个树上的所有节点，类似于PathChildrenCache和NodeCache的组合（根节点和子节点）

> 高版本的Curator将三个Watches合并

```java
@Test
public void watch0() throws InterruptedException {
    // 监听某一个节点
    CuratorCache cache = CuratorCache.build(client, "/app1");
    // 注册监听
    cache.listenable().addListener(new CuratorCacheListener() {
        @Override
        public void event(Type type, ChildData oldData, ChildData data) {
            System.out.println(type.name());
            switch (type) {
                case NODE_CREATED:
                    System.out.println(String.format("创建了新的节点->%s", data.getPath()));
                    break;
                case NODE_CHANGED:
                    if (oldData.getData() == null) {
                        System.out.println(String.format("节点赋值->%s", new String(data.getData())));
                    }
                    System.out.println(
                            String.format("原来的数据->%s,修改后的数据->%s",
                                    new String(oldData.getData()),
                                    new String(data.getData())));
                    break;
                case NODE_DELETED:
                    System.out.println(String.format("节点已删除->%s", oldData.getPath()));
                    break;
                default:
                    break;
            }
        }
    });
    // 开启监听
    cache.start();
    System.out.println("开启监听...");
    TimeUnit.SECONDS.sleep(1000);
}
```

### 分布式锁

进行单击应用开发，涉及并发同步的时候，往往采用synchronized和Lock的方式来解决多线程间代码同步问题，这时多线程的运行都是在同一个JVM下，没有问题

但是当应用是分布式集群工作的情况下，属于多JVM的工作环境，跨JVM之间已经无法通过多线程锁来解决同步问题

那么需要一种更加高级的锁机制，来处理==跨机器的进程之间的数据同步问题==——分布式锁

分布式锁的实现方式：

- 基于缓存实现分布式锁
  - Redis
  - Memcache
- Zookeeper实现分布式锁
- 数据库层面的分布式锁

#### Zookeeper实现分布式锁的原理

> 核心思想：当客户端获取锁的时候，创建一个节点，使用完锁后，删除节点

<img src="https://gitee.com/zhang-songyao/blog-images/raw/master/20220113133436.png" alt="image-20220113133410557" style="zoom: 67%;" />

- 客户端获取锁时，在lock节点下创建==临时顺序==节点
  - 临时：如果客户端宕机，Zookeeper服务器会删除当前节点
- 然后获取lock节点的所有子节点，客户端获取到所有子节点后，如果发现自己创建的节点序号最小，那么认为该客户端获取到了锁，使用完锁后将该节点删除
- 如果发现自己创建的节点并非lock下所有节点中最小的，说明还没有获取到锁，此时客户端需要找到比自己小的那个节点，同时对其注册事件监听器，监听删除事件
- 如果发现比自己小的那个节点被删除，则客户端的Watcher会收到相应通知，此时再次判断自己创建的节点是否是lock子节点中序号最小的，如果是则获取到了锁，如果不是则重复以上步骤继续获取到比自己小的一个节点并注册监听

### 模拟售票案例

> Curator实现分布式锁API

五种锁方案

- InterProcessSemaphoreMutex：分布式排它锁（非可重入锁）
- InterProcessMutex：分布式可重入排它锁
- InterProcessReadWriteLock：分布式读写锁
- InterProcessMultiLock：将多个锁作为单个实体管理的容器
- InterProcessSemaphoreV2：共享信号量

> 创建售票窗口类

```java
/**
 * @author ：zsy
 * @date ：Created 2022/1/13 14:01
 * @description： 售票窗口
 */
public class TicketWindow {

    // 总票数
    private int tickets = 50;
    private InterProcessMutex lock;

    public TicketWindow() {
        RetryPolicy retryPolicy = new ExponentialBackoffRetry(3000, 10);
        CuratorFramework client = CuratorFrameworkFactory
                .builder()
                .namespace("dunkcode")
                .connectionTimeoutMs(15 * 1000)
                .sessionTimeoutMs(60 * 1000)
                .retryPolicy(retryPolicy)
                .connectString("localhost:2181")
                .build();
        client.start();
        System.out.println("连接zk成功...");
        lock = new InterProcessMutex(client, "/lock");
        System.out.println("锁初始化成功...");
    }


    public void saleTicket() {
        if (tickets > 0) {
            try {
                lock.acquire();
                // 模拟买票延迟
                try {
                    TimeUnit.MILLISECONDS.sleep(50);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                System.out.println(
                        Thread.currentThread().getName()
                                + "买到了->第"
                                + (50 - tickets + 1)
                                + "张票");
                tickets--;
            } catch (Exception e) {
                e.printStackTrace();
            } finally {
                try {
                    lock.release();
                } catch (Exception e) {
                    e.printStackTrace();
                }
            }
        }
    }
}
```

> 客户类

```java
/**
 * @author ：zsy
 * @date ：Created 2022/1/13 13:46
 * @description：分布式锁测试
 */
public class LockTest {

    public static void main(String[] args) {
        TicketWindow window = new TicketWindow();
        new Thread(() -> {
            for (int j = 0; j < 50; j++) {
                window.saleTicket();
            }
        }, "携程用户").start();
        new Thread(() -> {
            for (int j = 0; j < 50; j++) {
                window.saleTicket();
            }
        }, "去哪儿用户").start();
    }
}
```

> 查看Zookeeper状态

<img src="https://gitee.com/zhang-songyao/blog-images/raw/master/20220113143557.png" alt="image-20220113143551095" style="zoom: 80%;" />

## Zookeeper 集群搭建

### 集群介绍

Leader选举：

- ServerID：服务器ID
  - 比如有三台服务器，编号为1,2,3。编号越大在选择算法中的权重越大
- ZxID：数据ID
  - 服务器中存放的最大数据ID.值越大说明数据越新，在选举算法中数据越新权重越大

> 在Leader选举的过程中，如果某台ZooKeeper获得了超过半数的选票,则此Zookeeper就可以成为Leader了

<img src="https://gitee.com/zhang-songyao/blog-images/raw/master/20220504172750.png" alt="image-20220113144028865" style="zoom:80%;" />

### 集群搭建

> 准备集群

```shell
# 创建目录
mkdir /home/zk-c
# 复制zk
cp -r /data/zk/apache-zookeeper-3.5.9-bin /home/zk-c/zookeeper2182 
cp -r /data/zk/apache-zookeeper-3.5.9-bin /home/zk-c/zookeeper2183
cp -r /data/zk/apache-zookeeper-3.5.9-bin /home/zk-c/zookeeper2184
# 修改配置文件 配置数据地址
vim /home/zk-c/zookeeper2182/conf/zoo.cfg
clientPort=2182
dataDir=/data/zk/zookeeper2182/data
```

> 配置集群

在每个Zookeeper的data目录下创建一个myid文件，内容分别为1,2,3。这个文件就是记录每个服务器的ID

```shell
echo 1 > /home/zk-c/zookeeper2182/data/myid
echo 2 > /home/zk-c/zookeeper2183/data/myid
echo 3 > /home/zk-c/zookeeper2184/data/myid
```

在每个Zookeeper的zoo.cfg配置客户端的集群服务器IP列表

```shell
vim /home/zk-c/zookeeper2182/conf/zoo.cfg
# 服务器IP列表
server.1=192.168.19.129:2882:3182
server.2=192.168.19.129:2883:3183
server.3=192.168.19.129:2884:3184


# 授权/home/zk-c/zookeeper2182/data/version2下的文件
chmod 777 acceptedEpoch 
chmod 777 currentEpoch 
chmod 777 snapshot.0 
```

server.服务器ID=服务器ip地址:服务器通信端口:服务器之间投票选举端口

> 启动集群

<img src="https://gitee.com/zhang-songyao/blog-images/raw/master/20220113163008.png" alt="image-20220113163000032" style="zoom:80%;" />

> 测试集群

当前2号节点为leader

测试

- 当停掉1号follower节点，集群正常工作
- 当停掉1号3号follower节点，2号节点也停止，不能正常工作
- 当停掉1号3号follower节点，恢复1号节点后，集群恢复正常，2号节点依旧是leader
- 当停掉2号节点，集群正常工作，3号节点选举为leader
- 当2号停掉，3号选举为leader后，恢复2号节点，3号节点依旧是leader

### 集群角色

在ZooKeeper集群服中务中有三个角色:

- Leader 领导者：
  - 处理事务请求（增删改）
  - 集群内部各服务器的调度者
- Follower 跟随者：
  - 处理客户端非事务请求（查询），转发事务请求给Leader服务器
  - 参与Leader选举投票

- Observer 观察者:
  - 处理客户端非事务请求，转发事务请求给Leader服务器
  - ==不参与Leader的选举==

> Follower节点接收到事务请求时会转发给Leader节点

<img src="https://gitee.com/zhang-songyao/blog-images/raw/master/20220113163855.png" alt="image-20220113163852115" style="zoom:80%;" />

## ZooKeeper核心理论







[视频地址](https://www.bilibili.com/video/BV1M741137qY?p=21&spm_id_from=333.1007.top_right_bar_window_history.content.click)

[ZooKeeper面试题（2020最新版）](https://blog.csdn.net/ThinkWon/article/details/104397719?ops_request_misc=%257B%2522request%255Fid%2522%253A%2522164206364816780271970399%2522%252C%2522scm%2522%253A%252220140713.130102334.pc%255Fblog.%2522%257D&request_id=164206364816780271970399&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~blog~first_rank_ecpm_v1~rank_v31_ecpm-1-104397719.nonecase&utm_term=zookeeper&spm=1018.2226.3001.4450)	