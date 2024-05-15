[Kafka官网主页](https://www.oschina.net/action/GoToLink?url=http%3A%2F%2Fkafka.apache.org%2F)

[Kafka官方文档](https://kafka.apache.org/)

[笔记](https://my.oschina.net/jallenkwong/blog/4449224)

[视频](https://www.bilibili.com/video/BV19y4y1b7Uo?p=8)



https://mp.weixin.qq.com/s/SuALTpvI3IMPSja9pacJ7Q

# 入门

## 定义

Kafka是由Apache软件基金会开发的一个开源流处理平台，由Scala和Java编写。Kafka是一种高吞吐量的分布式==发布/订阅==消息系统（Message Queue），可以处理消费者在网站中的所有流数据，主要应用于大数据实时处理领域

Kafka是一款开源的、轻量级的、分布式、可区分和具有复制备份的、基于ZooKeeper协调管理的分布式流平台的功能强大的消息系统。与传统的消息系统相比，Kafka能够很好的处理活跃的流数据，使得数据在各个子系统中==高性能、低延迟==地不停的流转

## 设计目标

- 以时间复杂度为O(1)的方式提供消息持久化能力，即使对TB级以上的数据也能保证常数时间的访问性能
- 高吞吐率，即使在非常廉价的商用机器上也能做到单击支持每秒100K条消息传输
- 支持Kafka Server键的消息分区、即分布式消费，同时保证每个partition内的消息顺序传输
- 同时支持离线数据处理和实时数据处理
- 支持在线水平扩展

## 消息队列的两种模式

### 点对点模式

> 一对一，消费者主动拉取数据，消息收到后消息清除

消息生产者发送到Queue中，然后消息消费者从Queue中取出并且消费消息，消息被消费后，queue中不在存储，所以消费者不可能消费到已经被消费的消息，queue支持多个消费者，但是对于一个消息而言，只会有一个消费者可以消费

### 发布/订阅模式

> 一对多，消费者消费数据之后不会消除消息

消息生产者（发布）将消息发布到topic中，同时多个消息消费者（订阅）消费该消息。和点对点方式不同，发布到topic的消息会被所有订阅者消费（类似于公众号）

分类	

- 消费者主动拉取消息（Kafka）：消费者的消费速度由消费者自己决定；但是需要消费者轮询查看是否有消息
- 队列推送消息给消费者：推送速度相同，对于消费能力不足的消费者会使系统蹦掉，对于消费能力强的消费者会造成资源浪费

## Kafka基本概念

- 主题：Kafka将一组消息抽象归纳为一个主题（topic），一个主题就是对消息的一个分类。生产者将消息发送到特定的主题，消费者订阅主题或主题的某些分区进行消费。
- 消息：消息时Kafka的基本单位，由一个固定长度的消息头和一个可变长度的消息体构成
- 分区和副本：Kafka将一组消息归纳为一个主题，而每个主题又被分为一个或者多个分区（Partition）。每个分区由一系列有序、不可变的消息组成，是一个有序序列。每个分区又有一个至多个副本（Replica），分区的副本分布在集群的不同代理上，以提高可用性
- 分区偏移：每个分区消息具有称为“offset”的唯一序列标识

> 分区是的Kafka在并发处理上变得更加容易，理论上来说，分区越多吞吐量越高，需要根据集群实际环境及业务场景而定。同时，分区也是Kafka保证消息被顺序消费以及对消息负载均衡的基础

- Brokers（代理）
  - 代理是负责维护发布数据的简单系统，每个代理中的每个主题可以具有零个或者多个分区，假设，如果在一个主题和N个代理中有N个分区，每个代理将有一个分区
  - 假设一个主题中有N个分区并且多于N个代理（n + m），则第一个N代理将具有一个分区，并且下一个M代理将不具有用于该特定主题的任何分区
  - 假设在一个主题中有N个分区并且小于N个代理(n-m)，每个代理将在它们之间具有一个或多个分区共享。 由于代理之间的负载分布不相等，不推荐使用此方案
- Kafka Cluster（Kafka集群）：Kafka有多个代理被称为集群，可以扩展Kafka集群，无需停机。这些集群用于管理消息数据的持久性和复制
- Leader：负责给定分区的所有读取和写入数据的节点，每个分区都有一个服务器充当Leader
- Follower：跟随领导者指令的节点被称为Follower。 如果领导失败，一个追随者将自动成为新的领导者。 跟随者作为正常消费者，拉取消息并更新其自己的数据存储

## Kafka基本架构

### Topic和Partition

一个消息中间件，队列不单只有一个，往往会有多个队列，而生产者和消费者需要知道，把数据丢给那个队列，从哪个队列获取消息，需要给队列取名字，叫做topic（相当于数据库中表的概念）

这样多个生产者往同一个队列（topic）丢数据，多个消费者从同一个队列拿数据

为了提高一个队列（topic）的吞吐量，Kafka会把topic进行分区（Partition）

<img src="https://gitee.com/zhang-songyao/blog-images/raw/master/20210724160126.png" alt="image-20210724160122653" style="zoom:80%;" />

所以，生产者实际上是向一个topic中的分区丢数据，消费者实际上是从一个topic中的分区取数据

### Kafka集群

> 需要使用zookeeper进行消息注册

一台Kafka服务器叫做Broker，Kafka集群就是多台Kafka服务器，因此Kafka是天然分布式的

Kafka集群中一台broker（Kafka服务器）出现网络抖动或者挂了怎么办？

数据存在不同的Partition上，Kafka会为这些分区做备份，如图有两个分区分别存在两台Broker上，每个partition都会备份，备份落在不同的broker上，红色块代表主分区，==生产者和消费者只与主分区交互==

<img src="https://gitee.com/zhang-songyao/blog-images/raw/master/20210724161250.png" alt="image-20210724161248974" style="zoom:80%;" />

==备份分区仅仅由于备份，不做读写==，如果某个Broker挂了，那就会选举出其他的Broker的partition来做主分区，实现了==高可用==

Kafka数据写在partition上的，那partition是怎么将其持久化的呢？（不持久化，如果Broker中途挂了，那肯定会丢数据）

- Kafka是将partition的数据写在**磁盘**的(消息日志)，不过Kafka只允许**追加写入**(顺序访问)，避免缓慢的随机 I/O 操作
- Kafka会先缓存一部分数据，当数据量足够多或者等待一定的时间后再批量写入（flush）

#### 单消费者

一个消费者消费多个partition的消息

<img src="https://gitee.com/zhang-songyao/blog-images/raw/master/20210724162041.png" alt="image-20210724162038925" style="zoom:80%;" />

#### 多个消费者

原本是一个消费者消费两个分区，有了消费者组后，每个消费者去消费一个分区（提高了吞吐量），提高了消费能力

<img src="https://gitee.com/zhang-songyao/blog-images/raw/master/20220103163614.png" alt="image-20220103163609282" style="zoom:80%;" />

- 每一个分区只能被同一个消费者组中的一个消费者消费
- 如果消费者组中的某个消费者挂了，那么其中一个消费者可能就要消费两个分区
- 如果只有三个分区，而消费组中有四个消费者，那么一个消费者就会空闲
- 如果多加入一个消费者组，无论是新增的消费者组还是原本的消费者组，都能消费topic的全部数据。（消费者组之间从逻辑上它们是独立的）

> 防止消费者宕机后，从头开始消费，Kafka将每个消费者的消费偏移量保存起来，0.9版本之前offset存储在zk中，0.9版本及之后的offset存储在Kafka本地（一个系统的topic）

## Kafka的应用场景

我们通常吧Kafka应用在两类程序中

1. 建立实时数据管道，以可靠地在系统或者应用程序之间获取数据
2. 构建实时的流应用程序，以转换或者响应数据流

<img src="https://gitee.com/zhang-songyao/blog-images/raw/master/20220110163956.png" alt="image-20220110163951780" style="zoom:80%;" />

- Producers：可以看到有很多应用程序，将消息数据放入Kafka集群中
- Consumers：可以有很多应用程序，将消息数据从Kafka中拉取出来
- Connectors：Kafka的连接器可以将数据中的数据导入到Kafka中，也可以将Kafka的数据导出到数据库中
- Stream Processors：流处理器可以从Kafka中拉取数据，也可以将数据写入Kafka

## Kafaka的优势

| 特性              | ActiveMQ     | RabbitMQ               | Kafka                  | RocketMQ       |
| ----------------- | ------------ | ---------------------- | ---------------------- | -------------- |
| 所属社区/公司     | Apache       | Mozilla Public License | Apache                 | Apache/Ali     |
| 成熟度            | 成熟         | 成熟                   | 成熟                   | 比较成熟       |
| 生产者-消费者模式 | 支持         | 支持                   | 支持                   | 支持           |
| 发布-订阅         | 支持         | 支持                   | 支持                   | 支持           |
| REQUEST-REPLY     | 支持         | 支持                   | -                      | 支持           |
| API完备性         | 高           | 高                     | 高                     | 低（静态配置） |
| 多语言支持        | 支持JAVA优先 | 语言无关               | 支持，JAVA优先         | 支持           |
| 单机呑吐量        | 万级（最差） | 万级                   | ***十万级***           | 十万级（最高） |
| 消息延迟          | -            | 微秒级                 | ***毫秒级***           | -              |
| 可用性            | 高（主从）   | 高（主从）             | ***非常高（分布式）*** | 高             |
| 消息丢失          | -            | 低                     | ***理论上不会丢失***   | -              |
| 消息重复          | -            | 可控制                 | 理论上会有重复         | -              |
| 事务              | 支持         | 不支持                 | 支持                   | 支持           |
| 文档的完备性      | 高           | 高                     | 高                     | 中             |
| 提供快速入门      | 有           | 有                     | 有                     | 无             |
| 首次部署难度      | -            | 低                     | 中                     | 高             |



## 异步通信的原理

### 观察者模式

- 观察者模式（Observe），又叫发布-订阅模式（Publish/Subscribe）
- 定义对象间一种一对多的依赖关系，使得每当一个对象改变状态，则所有依赖于它的对象都会得到通知并自动更新
- 一个对象（目标对象）的状态发生改变，所有依赖对象（观察者对象）都将得到通知 

### 生产者消费者模式

#### 传统模式

- 生产者直接将消息传递给指定的消费者
- 耦合性特别高，当生产者或者消费者发生变法，都需要重写业务逻辑

#### 生产消费者模式

- 通过一个容器来解决生产者和消费者的强耦合问题。生产者和消费者彼此之间不直接通信，而通过阻塞队列来进行通讯
- 数据传递流程
  - 生产者消费者模式，即N个线程进行生产，同时N个线程进行消费，两种角色通过内存缓冲区进行通信
  - 生产者负责向缓冲区里面添加数据单元
  - 消费者负责从缓存区里面取出数据单元
    - 先进先出原则

<img src="https://gitee.com/zhang-songyao/blog-images/raw/master/20210724124800.png" alt="image-20210724124759205" style="zoom:80%;" />

### 缓冲区带来的好处

- 解耦
  - 假设消费者和生产者分别是两个类，如果让生产者调用消费者的某个方法，那么生产者对于消费者就会产生依赖
- 支持并发
- 均衡生产和消费速度差异（发的快收的慢情况）削峰

## 安装

> 下载安装包

[下载地址](https://kafka.apache.org/downloads)

```shell
# 解压
cd /data/kafka/
tar -xvzf kafka_2.12-2.4.1.tgz 
```

## 文件目录分析

| 目录名称  |                             说明                             |
| :-------: | :----------------------------------------------------------: |
|    bin    | Kafka的所有执行脚本都在这里。例如：启动Kafka服务器、创建Topic、生产者、消费者程序等等 |
|  config   |                     Kafka的所有配置文件                      |
|   libs    |                  运行Kafka所需要的所有JAR包                  |
|   logs    | Kafka的所有日志文件，如果Kafka出现一些问题，需要到该目录中去查看异常信息 |
| site-docs |                     Kafka的网站帮助文件                      |

## 搭建Kafka集群

> 创建目录

```shell
mkdir /home/kafka-c
```

> 复制目录到集群目录下

```shell
cd /data/kafka

cp -r kafka_2.12-2.4.1 /home/kafka-c/kafka9092
cp -r kafka_2.12-2.4.1 /home/kafka-c/kafka9093
cp -r kafka_2.12-2.4.1 /home/kafka-c/kafka9094
```

> 下面进行配置和启动

```shell
cd /home/kafka-c/kafka909?
# 指定broker的id
broker.id=0
# 指定端口号
prot=909?
# 指定Kafka数据的位置
log.dirs=/home/kafka-c/kafka909?data
```

==配置文件==

```properties
############################# Server Basics #############################

# The id of the broker. This must be set to a unique integer for each broker.
broker.id=0
port=9092

############################# Socket Server Settings #############################
num.network.threads=3

# The number of threads that the server uses for processing requests, which may include disk I/O
num.io.threads=8

# The send buffer (SO_SNDBUF) used by the socket server
socket.send.buffer.bytes=102400

# The receive buffer (SO_RCVBUF) used by the socket server
socket.receive.buffer.bytes=102400

# The maximum size of a request that the socket server will accept (protection against OOM)
socket.request.max.bytes=104857600


############################# Log Basics #############################

# A comma separated list of directories under which to store log files
log.dirs=/home/kafka-c/kafka9093/data

# The default number of log partitions per topic. More partitions allow greater
# parallelism for consumption, but this will also result in more files across
# the brokers.
num.partitions=1

# The number of threads per data directory to be used for log recovery at startup and flushing at shutdown.
# This value is recommended to be increased for installations with data dirs located in RAID array.
num.recovery.threads.per.data.dir=1

############################# Internal Topic Settings  #############################

offsets.topic.replication.factor=1
transaction.state.log.replication.factor=1
transaction.state.log.min.isr=1

############################# Log Flush Policy #############################
# The settings below allow one to configure the flush policy to flush data after a period of time or
# every N messages (or both). This can be done globally and overridden on a per-topic basis.

# The number of messages to accept before forcing a flush of data to disk
#log.flush.interval.messages=10000

# The maximum amount of time a message can sit in a log before we force a flush
#log.flush.interval.ms=1000

############################# Log Retention Policy #############################

# The following configurations control the disposal of log segments. The policy can
# be set to delete segments after a period of time, or after a given size has accumulated.
# A segment will be deleted whenever *either* of these criteria are met. Deletion always happens
# from the end of the log.

# The minimum age of a log file to be eligible for deletion due to age
log.retention.hours=168

# A size-based retention policy for logs. Segments are pruned from the log unless the remaining
# segments drop below log.retention.bytes. Functions independently of log.retention.hours.
#log.retention.bytes=1073741824

# The maximum size of a log segment file. When this size is reached a new log segment will be created.
log.segment.bytes=1073741824

# The interval at which log segments are checked to see if they can be deleted according
# to the retention policies
log.retention.check.interval.ms=300000

############################# Zookeeper #############################

# Zookeeper connection string (see zookeeper docs for details).
# This is a comma separated host:port pairs, each corresponding to a zk
# server. e.g. "127.0.0.1:3000,127.0.0.1:3001,127.0.0.1:3002".
# You can also append an optional chroot string to the urls to specify the
# root directory for all kafka znodes.
zookeeper.connect=localhost:2181

# Timeout in ms for connecting to zookeeper
zookeeper.connection.timeout.ms=6000


############################# Group Coordinator Settings #############################
group.initial.rebalance.delay.ms=0

```

> 当然也可以配置全局环境变量，单机情况下，当前用不到

```shell
vim /etc/profile
export KAFKA_HOME=/export/server/kafka_2.12-2.4.1
export PATH=:$PATH:${KAFKA_HOME}

# 加载环境变量
source /etc/profile
```

> 启动服务器

```shell
# 启动kafka服务器
cd /home/kafka-c/kafka909
# 后台启动
nohup bin/kafka-server-start.sh config/server.properties &

# 启动zookeeper
docker pull zookeeper
docker run -d -p 2181:2181 --name zk zookeeper
# 查看zk状态
docker exec -it zk bash
zkServer.sh status

# 测试Kafka集群是否启动成功
kafka-topics.sh --bootstrap-server localhost:9092 --list
```

### 基本操作

<img src="https://gitee.com/zhang-songyao/blog-images/raw/master/20220110235957.png" alt="image-20220110235954304" style="zoom:80%;" />

#### 创建topic

> 创建一个topic（主题）。Kafka中所有的消息都是保存在主题中，要生产消息到Kafka，首先必须要有一个确定的主题

```shell
# 创建名为test的主题
bin/kafka-topics.sh --create --bootstrap-server localhost:9092 --topic test
# 查看目前Kafka中的主题
bin/kafka-topics.sh --list --bootstrap-server localhost:9092
```

#### 生产消息

> 使用Kafka内置的测试程序，生产一些消息到Kafka的test主题中

```shell
# 指定当前Kafka服务器的地址
--bootstrap-server localhost:9092

# 生产消息
bin/kafka-console-producer.sh --broker-list localhost:9092 --topic test
```

#### 从机消费消息

> 使用下面的命令来消费 test 主题中的消息

```shell
bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic test --from-beginning
```

<img src="https://gitee.com/zhang-songyao/blog-images/raw/master/20220111000728.png" alt="image-20220111000723816" style="zoom:80%;" />

### 基准测试

基准[测试](http://www.blogjava.net/qileilove/archive/2012/07/05/382241.html)（benchmark testing）是一种测量和评估软件性能指标的活动。我们可以通过基准测试，了解到软件、硬件的性能水平。主要测试负载的执行时间、传输速度、吞吐量、资源占用率等

#### 基于1个分区1个副本的基准测试

- 启动Kafka集群
- 创建一个1个分区1个副本的topic: benchmark
- 同时运行生产者、消费者基准测试程序
- 观察结果

> 创建topic

```shell
bin/kafka-topics.sh --zookeeper localhost:2181 --create --topic benchmark --partitions 3 --replication-factor 3
```

> 生产消息基准测试

在生产环境中，推荐使用生产5000W消息，这样会性能数据会更准确些。为了方便测试，演示测试500W的消息作为基准测试

```shell
bin/kafka-producer-perf-test.sh --topic benchmark --num-records 5000000 --throughput -1 --record-size 1000 --producer-props bootstrap.servers=localhost:9092,localhost:9093,localhost:9094 acks=1

# 参数解释
bin/kafka-producer-perf-test.sh 
--topic topic的名字
--num-records	总共指定生产数据量（默认5000W）
--throughput	指定吞吐量——限流（-1不指定）
--record-size   record数据大小（字节）
--producer-props bootstrap.servers=192.168.1.20:9092,192.168.1.21:9092,192.168.1.22:9092 acks=1 指定Kafka集群地址，ACK模式
```

> 消费消息基准测试

```shell
bin/kafka-consumer-perf-test.sh --broker-list localhost:9092,localhost:9093,localhost:9094 --topic benchmark --fetch-size 1048576 --messages 5000000
```

## 安装Kafka-engle

> 下载

[下载地址](http://download.kafka-eagle.org/)

[文档](http://www.kafka-eagle.org/articles/docs/installation/windows.html)

> 解压

```shell
tar xf kafka-eagle-web-1.3.8-bin.tar.gz
```

> 设置环境变量

```shell
vim /etc/profile

export JAVA_HOME=
export KE_HOME=/data/kafka/kafka-eagle-bin-1.3.7/kafka-eagle-web-1.3.7
export PATH=$PATH:$KE_HOME/bin:$JAVA_HOME/bin
```

> 修改Kafka-engle的配置文件

```shell
#修改配置文件
vim system-config.properties


#kafkazookeeper节点配置属性多个可以添加一个，cluster1 
kafka.eagle.zk.cluster.alias=cluster1
cluster1.zk.list=192.168.163.131:2181
######################################
# zk 线程数量
######################################
kafka.zk.limit.size=25
 
######################################
# kafka eagle 的端口
######################################
kafka.eagle.webui.port=8048
 
######################################
# kafka offset storage
######################################
cluster1.kafka.eagle.offset.storage=kafka
 
######################################
# enable kafka 开启图表
# 及开始sql查询
######################################
kafka.eagle.metrics.charts=true
 
kafka.eagle.sql.fix.error=true
######################################
# 提醒的email
######################################
kafka.eagle.mail.enable=true
kafka.eagle.mail.sa=alert_sa
kafka.eagle.mail.username=alert_sa@163.com
kafka.eagle.mail.password=mqslimczkdqabbbh
kafka.eagle.mail.server.host=smtp.163.com
kafka.eagle.mail.server.port=25
 
######################################
# 删除kafka topic 的token
######################################
kafka.eagle.topic.token=keadmin
 
######################################
# kafka sasl authenticate
######################################
kafka.eagle.sasl.enable=false
kafka.eagle.sasl.protocol=SASL_PLAINTEXT
kafka.eagle.sasl.mechanism=PLAIN
 
######################################
# kafka jdbc 地址注意可以自己安装数据mysql也可以自带的
######################################
kafka.eagle.driver=org.sqlite.JDBC
kafka.eagle.url=jdbc:sqlite:/kafka/kafka-eagle-bin-1.2.4/kafka-eagle-web-1.2.4/db/ke.db
kafka.eagle.username=root
kafka.eagle.password=smartloli
```

> 启动

```shell
# 如果没有chomd命令 安装chomd
yum install coreutils

#进入bin目录后会看到 ke.sh 文件先修改文件的权限
cd bin
sudo chomd 777 ke.sh

#启动
./ke.sh start
#查看状态
./ke.sh status
#查看状态
./ke.sh stats
#关闭
./ke.sh stop
#重启
./ke.sh restart
```

## java操作Kakfa

### 配置依赖

> 配置阿里云镜像

```xml
<repositories><!-- 代码库 -->
    <repository>
        <id>central</id>
        <url>http://maven.aliyun.com/nexus/content/groups/public//</url>
        <releases>
            <enabled>true</enabled>
        </releases>
        <snapshots>
            <enabled>true</enabled>
            <updatePolicy>always</updatePolicy>
            <checksumPolicy>fail</checksumPolicy>
        </snapshots>
    </repository>
</repositories>
```

> 依赖

```xml
<!-- kafka客户端工具 -->
<dependency>
    <groupId>org.apache.kafka</groupId>
    <artifactId>kafka-clients</artifactId>
    <version>2.4.1</version>
</dependency>

<!-- 工具类 -->
<dependency>
    <groupId>org.apache.commons</groupId>
    <artifactId>commons-io</artifactId>
    <version>1.3.2</version>
</dependency>

<!-- SLF桥接LOG4J日志 -->
<dependency>
    <groupId>org.slf4j</groupId>
    <artifactId>slf4j-log4j12</artifactId>
    <version>1.7.6</version>
</dependency>

<!-- SLOG4J日志 -->
<dependency>
    <groupId>log4j</groupId>
    <artifactId>log4j</artifactId>
    <version>1.2.16</version>
</dependency>
```

### 生产者生产消息

```java
/**
 * @author ：zsy
 * @date ：Created 2022/1/11 15:39
 * @description：测试
 */
public class KafkaProducerTest {
    public static void main(String[] args) {
        Properties props = new Properties();
        // 当前Kafka服务器
        props.put("bootstrap.servers", "192.168.19.129:9092");
        // 写入策略，返回机制
        // 所有的分区副本都写入后，再返回
        // props.put("acks", "all");
        props.put("key.serializer", "org.apache.kafka.common.serialization.StringSerializer");
        props.put("value.serializer", "org.apache.kafka.common.serialization.StringSerializer");
        KafkaProducer<String, String> producer = new KafkaProducer<>(props);
        for (int i = 0; i < 10; i++) {
            try {
                Future<RecordMetadata> future = producer.send(
                        new ProducerRecord<String, String>("test", i + ""));
                future.get();
                System.out.println("第" + i + "条消息写入成功");
            } catch (InterruptedException e) {
                e.printStackTrace();
            } catch (ExecutionException e) {
                e.printStackTrace();
            } finally {
                producer.close();
            }
        }
        producer.close();
    }
}
```

### 消费者消费消息

```java
public class KafkaConsumerTest {
    public static void main(String[] args) throws InterruptedException {
        // 1.创建Kafka消费者配置
        Properties props = new Properties();
        props.setProperty("bootstrap.servers", "192.168.19.129:9092,192.168.19.129:9093");
        // 消费者组（可以使用消费者组将若干个消费者组织到一起），共同消费Kafka中topic的数据
        // 每一个消费者需要指定一个消费者组，如果消费者的组名是一样的，表示这几个消费者是一个组中的
        props.setProperty("group.id", "test");
        // 自动提交offset
        props.setProperty("enable.auto.commit", "true");
        // 自动提交offset的时间间隔
        props.setProperty("auto.commit.interval.ms", "1000");
        // 拉取的key、value数据的
        props.setProperty("key.deserializer", "org.apache.kafka.common.serialization.StringDeserializer");
        props.setProperty("value.deserializer", "org.apache.kafka.common.serialization.StringDeserializer");

        // 2.创建Kafka消费者
        KafkaConsumer<String, String> kafkaConsumer = new KafkaConsumer<>(props);

        // 3. 订阅要消费的主题
        // 指定消费者从哪个topic中拉取数据
        kafkaConsumer.subscribe(Arrays.asList("test"));

        KafkaConsumer<String, String> consumer = new KafkaConsumer<>(props);
        // 订阅要消费的主题
        // 指定消费者从哪个主题获取消息
        consumer.subscribe(Collections.singletonList("test"));

        while (true) {
            // 一次拉取5s内的一批消息
            ConsumerRecords<String, String> records = consumer.poll(Duration.ofSeconds(5));
            for (ConsumerRecord<String, String> record : records) {
                // 主题
                String topic = record.topic();
                // 偏移量
                long offset = record.offset();
                String key = record.key();
                String value = record.value();
                System.out.println("topic: " + topic + " offset:" + offset + " key:" + key + " value:" + value);
            }
            TimeUnit.SECONDS.sleep(1);
        }
    }
}
```

auto.create.topics.enable=true
delete.topic.enable=true





https://blog.csdn.net/weixin_42555971/article/details/125183026