​	https://blog.51cto.com/u_14932245/4340967

## 定义

Kafka 是一个分布式的基于发布/订阅模式的消息队列（Message Queue），主要应用于大数据实时处理领域。 

## 使用消息队列的好处

- 解耦：允许你独立的扩展或者修改两边的处理过程，只要确保他们遵守同样的接口约束。

- 可恢复性：系统的一部分组件失效时，不会影响到整个系统。消息队列降低了进程间的耦合度，所以即使一个处理消息的进程挂掉，加入队列中的消息仍然可以在系统恢复后被处理。

- 缓冲：有助于控制和优化数据流经过系统的速度，解决生产消息和消费消息的处理速度不一致的情况。

- 灵活性 & 峰值处理能力：在访问量剧增的情况下，应用仍然需要继续发挥作用，但是这样的突发流量并不常见。如果为以能处理这类峰值访问为标准来投入资源随时待命无疑是巨大的浪费。使用消息队列能够使关键组件顶住突发的访问压力，而不会因为突发的超负荷的请求而完全崩溃。

- 异步通信：很多时候，用户不想也不需要立即处理消息。消息队列提供了异步处理机制，允许用户把一个消息放入队列，但并不立即处理它。想向队列中放入多少消息就放多少，然后在需要的时候再去处理它们。


## 消息队列的两种模式

### 点对点模式

> 一对一，消费者主动拉取数据，消息收到后消息清除

消息生产者生产消息发送到Queue中，然后消息消费者从Queue中取出并且消费消息。消息被消费后，queue中不再有存储，所以消费者不可能消费到已经被消费的消息。Queue支持存在多个消费者，但是对于一个消息而言，只有一个消费者可以消费。

![image-20221008130826206](https://gitee.com/zhang-songyao/blog-images/raw/master/202210081308805.png)

### 发布/订阅模式

> 一对多，消费者消费数据之后不会清除消息

消息生产者（发布）将消息发布到topic中，同时有多个消费者（订阅）消费该消息。和点对点方式不同，发布到topic的消息会被所有订阅者消费。

![image-20221008131207071](https://gitee.com/zhang-songyao/blog-images/raw/master/202210081312776.png)

## kafka基础架构

![image-20221008131302682](https://gitee.com/zhang-songyao/blog-images/raw/master/202210081313887.png)

- Producer：消息生产者，就是向kafka broker发消息的客户端
- Consumer：消息消费者，向kafka broker取消息的客户端
- Consumer Group：消费者组，由多个consumer组成。消费者组每个消费者负责消费不同分区的数据，一个分区只能由一个组内消费者消费；消费者组之间互不影响。所有的消费者属于某个消费者组，即消费者组是逻辑上的一个订阅者
- Broker：一台Kafka服务器就是一个broker。一个集群由多个broker组成，一个broker可以容纳多个topic
- Topic：可以理解为一个队列，生产者和消费者面向的都是一个topic
- Partition：为了实现扩展性，一个非常大的topic可以分布到多个broker（即服务器）上，**一个** **topic** **可以分为多个** **partition**，每个 partition 是一个有序的队列
- Replication：副本，为保证集群中的某个节点发生故障时，该节点上的 partition 数据不丢失，且 kafka 仍然能够继续工作，kafka 提供了副本机制，一个 topic 的每个分区都有若干个副本，一个 leader 和若干个 follower
- leader：每个分区多个副本的“主”，生产者发送数据的对象，以及消费者消费数据的对象都是 leader。
- follower：每个分区多个副本中的“从”，实时从 leader 中同步数据，保持和 leader 数据的同步。leader 发生故障时，某个 follower 会成为新的 follower。 

## kafka架构

### kafka工作流程

![image-20221008141753730](https://gitee.com/zhang-songyao/blog-images/raw/master/202210081417821.png)

> kafka中消息是以topic进行分类的，生产者生产消息，消费者消费消息，都是面向topic的。

topic是逻辑上的概念，而partition都是物理上的概念，每个partition对应一个log文件，该log文件中存储的就是produer生产的数据。Producer生产的数据会被不断的追加到该log文件的末端，且每条数据都有自己的offset。消费者组中每个消费者，都会实时记录自己消费到那个offset，以便出错恢复时，从上次的位置继续消费。

### kafka文件存储机制

![image-20221008142532471](https://gitee.com/zhang-songyao/blog-images/raw/master/202210081425483.png)

由于生产者生产的消息会不断追加到log文件末尾，未防止log文件过大导致数据定位效率低下，kafka采用了**分片**和**索引**机制，将每个partition分为多个segment。每个segment对应两个文件——`.index`文件和`.log`文件。这些文件位于一个文件夹下，该文件夹的命名规则为：topic名称+分区序号。例如，first 这个 topic 有三个分区，则其对应的文件夹为 first-0,first-1,first-2。

```
00000000000000000000.index
00000000000000000000.log
00000000000000170410.index
00000000000000170410.log
00000000000000239430.index
00000000000000239430.log
```

index和log文件以当前segment的第一条消息的offset命名。

![image-20221008143237264](https://gitee.com/zhang-songyao/blog-images/raw/master/202210081432354.png)

> `.index`文件存储大量的索引信息，`.log`文件存储大量的数据，索引文件中的元数据执行对应数据文件中message的物理偏移地址。

### kafka生产者

#### 分区策略

##### 分区的原因

1. 方便在集群中扩展，每个partition可以通过调整以适应它所在的机器，而一个topic又可以有多个partition组成，因此整个集群就可以适应任何大小的数据了。
2. 可以提高并发，因为可以以partition为单位读写了。

##### 分区的原则

我们需要将 producer 发送的数据封装成一个 **ProducerRecord** 对象。 

![image-20221008143713787](https://gitee.com/zhang-songyao/blog-images/raw/master/202210081437410.png)

1. 指定partition的情况下，直接将指定的值作为partition值
2. 没有指定partition值但有key的情况下，将key的hash 值与 topic 的 partition 数进行取余得到 partition 值
3. 既没有 partition 值又没有 key 值的情况下，第一次调用时随机生成一个整数（后面每次调用在这个整数上自增），将这个值与 topic 可用的 partition 总数取余得到 partition 值，也就是常说的 round-robin 算法。

> Round Robin（中文翻译为轮询调度）是一种以轮询的方式依次将一个域名解析到多个IP地址的调度不同服务器的计算方法。

#### 数据可靠性保证

为保证producer发送的数据，即可靠的发送到指定的topic，topic的每个partition收到producer发送的数据后，都需要向producer发送ack（acknowledgement 确认收到），如果producer收到ack，就会进行下一轮发送，否则会重新发送数据。

![image-20221008144223597](https://gitee.com/zhang-songyao/blog-images/raw/master/202210081442686.png)

##### 副本数据同步策略

| 方案                        | 优点                                                 | 缺点                                                |
| --------------------------- | ---------------------------------------------------- | --------------------------------------------------- |
| 半数以上同步完成，就发送ack | 延迟低                                               | 选举新的leader时，容忍n台节点的故障，需要2n+1个副本 |
| 全部完成后，发送ack         | 选举新的leader时，能容忍n台节点的故障，需要n+1个副本 | 延迟高                                              |

kafka选择了第二种方案，原因：

1. 同样为了容忍 n 台节点的故障，第一种方案需要 2n+1 个副本，而第二种方案只需要 n+1个副本，而 Kafka 的每个分区都有大量的数据，第一种方案会造成大量数据的冗余。
2. 虽然第二种方案的网络延迟会比较高，但网络延迟对kafka的影响较小。

##### ISR

采用第二种方案之后，设想以下场景：leader收到数据，所有follower都开始同步数据，但有一个follower，因为某些故障，迟迟不能与leader进行同步，那么leader就要一直等下去，直到它完成同步，才能发送ack，这个问题怎么解决？

Leader 维护了一个动态的 in-sync replica set (ISR)，意为和 leader 保持同步的 follower 集合。当 ISR 中的 follower 完成数据的同步之后，leader 就会给 follower 发送 ack。如果 follower长时间未向leader 同步数据 ， 则该 follower将 被 踢 出 ISR，该时间阈值由**replica.lag.time.max.ms** 参数设定。Leader 发生故障之后，就会从 ISR 中选举新的 leader。

##### ack应答机制

对于某些不太重要的数据，对数据的可靠性要求不是很高，能够容忍数据的少量丢失，所以没有必要等ISR种的follower全部接收成功。

所以kafka为用户提供了三种可靠性级别，用户根据对可靠性和延迟的要求进行权衡，选择以下的配置。

ack参数配置：

- 0：producer不等待broker的ack，这一操作提供了一个最低的延迟，broker一接受到还没有写入磁盘就已经返回，当broker由故障的时候有可能丢失数据
- 1：producer等待broker的ack，partition的leader落盘成功后返回ack，如果follower同步成功之前leader故障，那么将丢失数据

丢失数据案例：

![image-20221008145713754](https://gitee.com/zhang-songyao/blog-images/raw/master/202210081457528.png)

- -1（all）：producer等待broker的ack，partition的leader和follower全部落盘成功后才返回ack。但是如果follower同步成功之后，broker发送ack之前，leader发生故障，那么会操成数据重复

数据重复案例

![image-20221008145744611](https://gitee.com/zhang-songyao/blog-images/raw/master/202210081457227.png)

##### 故障处理细节

![image-20221008150607507](https://gitee.com/zhang-songyao/blog-images/raw/master/202210081506832.png)

LEO：指的是每个副本最大的offset

HW：指的是消费者能见到的最大offset，ISR队列中最小的LEO

###### follower故障

follower发生故障后会被临时提出ISR，待该follower恢复后，follower会读取本地磁盘记录的上次的HW，并将log文件高于HW的部分截取掉，从HW开始向leader进行同步，等**该follower的LEO大于等于该Partition的HW**，即follower追上leader之后，就可以重新加入ISR了。

###### leader故障

leader发生故障后，会从ISR种选出一个新的leader，之后，为保证多个副本之间数据一致性，其余的follower会先将各自log文件高于HW的部分截掉，然后从新的leader同步数据。

> **注意：这只能保证副本之间的数据一致性，并不能保证数据不丢失或者不重复。**

#### Exactly Once语义

将服务器的ACK级别设置为-1，可以保证Producer到Server之间不会丢失数据，即At Least Once语义。相对的，将服务器ACK级别设置为0，可以保证每条消息只被发送一次，即At Most Once语义。

At least Once可以保证数据不丢失，但是不能保证数据不重复；相对的At Most Once可以保证数据不重复，但是不能保证数据不丢失。但是对弈一些非常重要的信息，比如说：交易数据，下游数据消费者要求数据既不重复，也不丢失，即Exactly Once语义，在0.11版本以前的kafka，对此是无能为力的，只能保证数据不丢失，再在下游消费者对数据全局去重。对于多个下游应用的情况，每个都需要单独做全局去重，这就对性能造成了很大的影响。

0.11版本的kafka，引入了一项重大特性：幂等性。所谓的幂等性就是指Producer无论向Server发送多少次重复的数据，Server端只会持久化一条。幂等性结合At Least Once语义，就构成了Kafka的Exactly Once语义。即：

> At Least Once + 幂等性 = Exactly Once

要启用幂等性，只需要将producer的参数中 `enable.idompotence` 设置为 true 即可。Kafka的幂等性实现其实就是将原来下游需要做的去重放在了数据上游。开启幂等性的 Producer 在初始化的时候会被分配一个 PID，发往同一 Partition 的消息会附带 Sequence Number。而Broker 端会对`<PID, Partition, SeqNumber>`做缓存，当具有相同主键的消息提交时，Broker 只会持久化一条。

>但是 PID 重启就会变化，同时不同的 Partition 也具有不同主键，所以幂等性无法保证跨分区跨会话的 Exactly Once。

### kafka消费者

#### 消费方式

consumer采用pull模式从broker读取数据。

push模式很难适应消费速率不同的消费者，因为消息发送的速率是有broker决定的。

它的目标是尽可能以最快速度传递消息，但是这样很容易造成 consumer 来不及处理消息，典型的表现就是拒绝服务以及网络拥塞。而 pull 模式则可以根据 consumer 的消费能力以适当的速率消费消息。

pull模式不足之处是：如果kafka没有数据，消费者可能陷入循环中，一直返回空数据。针对这一点kafka的消费者在消费数据时会传入一个时长参数timeout，如果当前没有数据可消费，consumer会等待一段时间之后再返回，这段时长即为timeout。

#### 分区分配策略

一个consumer group 中有多个consumer，一个topic有多个partition，所以必然会涉及到partition的分配问题，即确定那个partition由那个consumer来消费

##### Range（默认策略）

Range是对每个Topic而言的（即一个Topic一个Topic的分），首先对同一个Topic里面的分区按照序号进行排序，并对消费者按字母顺序进行排序，然后用partitions分区的个数除以消费者线程的总数来决定每个消费者线程消费几个分区，如果除不尽，那么前几个消费者线程将会多消费一个分区。

假如有10个分区，3个消费者线程，把分区按照序号排列0，1，2，3，4，5，6，7，8，9；消费者线程为C1-0，C2-0，C2-1，那么用partition数除以消费者线程的总数来决定每个消费者线程消费几个partition，如果除不尽，前面几个消费者将会多消费一个分区。在我们的例子里面，我们有10个分区，3个消费者线程，10/3 = 3，而且除除不尽，那么消费者线程C1-0将会多消费一个分区，所以最后分区分配的结果看起来是这样的：

```
C1-0：0，1，2，3
C2-0：4，5，6
C2-1：7，8，9
```

如果有11个分区将会是：

```
C1-0：0，1，2，3
C2-0：4，5，6，7
C2-1：8，9，10
```

假如我们有两个topic T1,T2，分别有10个分区，最后的分配结果将会是这样

```
C1-0：T1（0，1，2，3） T2（0，1，2，3）
C2-0：T1（4，5，6） T2（4，5，6）
C2-1：T1（7，8，9） T2（7，8，9）
```

可以看出， C1-0消费者线程比其他消费者线程多消费了2个分区

> 如上，只针对一个topic而言，C1-0消费者多消费1个分区影响不是很大。如果有 N 多个 topic，那么针对每个 topic，消费者 C1-0 都将多消费 1 个分区，topic越多，C1-0 消费的分区会比其他消费者明显多消费 N 个分区。这就是 Range 范围分区的一个很明显的弊端了

##### RoundRobin

RoundRobinAssignor策略的原理是将消费者组内所有消费者以及消费者所订阅的所有topic的partition按照字典序排序，然后通过轮询的方式逐个将分区以此分配给每个消费者。RoundRobinAssignor策略对应的partition.assignment.strategy参数值为：org.apache.kafka.clients.consumer.RoundRobinAssignor。

使用RoundRobinAssignor策略有两个前提条件必须满足：

1. 同一个消费者组里面的所有消费者的num.streams（消费者线程数）必须相等
2. 每个消费者订阅的主题必须相同

所以这里假设前面提到的2个消费者的num.streams = 2。RoundRobin策略的工作原理：将所有topic的分区组成 TopicAndPartition 列表，然后对 TopicAndPartition 列表按照 hashCode 进行排序

```scala
val allTopicPartitions = ctx.partitionsForTopic.flatMap { case(topic, partitions) =>
  info("Consumer %s rebalancing the following partitions for topic %s: %s"
       .format(ctx.consumerId, topic, partitions))
  partitions.map(partition => {
    TopicAndPartition(topic, partition)
  })
}.toSeq.sortWith((topicPartition1, topicPartition2) => {
  /*
   * Randomize the order by taking the hashcode to reduce the likelihood of all partitions of a given topic ending
   * up on one consumer (if it has a high enough stream count).
   */
  topicPartition1.toString.hashCode < topicPartition2.toString.hashCode
})
```

最后按照round-robin风格将分区分别分配给不同的消费者线程

例如：

按照hashcode排序完的topic-partitions组依次为T1-5, T1-3, T1-0, T1-8, T1-2, T1-1, T1-4, T1-7, T1-6, T1-9，我们的消费者线程排序为C1-0, C1-1, C2-0, C2-1，最后分区分配的结果为：

```
C1-0 将消费 T1-5, T1-2, T1-6 分区；
C1-1 将消费 T1-3, T1-1, T1-9 分区；
C2-0 将消费 T1-0, T1-4 分区；
C2-1 将消费 T1-8, T1-7 分区；
```

RoundRobin的两种情况

1. 如果同一个消费组内所有的消费者的订阅信息都是相同的，那么RoundRobinAssignor策略的分区分配会是均匀的。

举例，假设消费组中有2个消费者C0和C1，都订阅了主题t0和t1，并且每个主题都有3个分区，那么所订阅的所有分区可以标识为：t0p0、t0p1、t0p2、t1p0、t1p1、t1p2。最终的分配结果为：

```
消费者C0：t0p0、t0p2、t1p1 消费者C1：t0p1、t1p0、t1p2
```

1. 如果同一个消费组内的消费者所订阅的信息是不相同的，那么在执行分区分配的时候就不是完全的轮询分配，有可能会导致分区分配的不均匀。如果某个消费者**没有订阅**消费组内的某个topic，那么在分配分区的时候此消费者将分配不到这个topic的任何分区。

举例，假设消费组内有3个消费者C0、C1和C2，它们共订阅了3个主题：t0、t1、t2，这3个主题分别有1、2、3个分区，即整个消费组订阅了t0p0、t1p0、t1p1、t2p0、t2p1、t2p2这6个分区。具体而言，消费者C0订阅的是主题t0，消费者C1订阅的是主题t0和t1，消费者C2订阅的是主题t0、t1和t2，那么最终的分配结果为：

```
消费者C0：t0p0 消费者C1：t1p0 消费者C2：t1p1、t2p0、t2p1、t2p2
```

可以看到RoundRobinAssignor策略也不是十分完美，这样分配其实并不是最优解，因为完全可以将分区t1p1分配给消费者C1。

##### StickyAssignor

“sticky”这个单词可以翻译为“粘性的”，Kafka从0.11.x版本开始引入这种分配策略，它主要有两个目的：

1. 分区的分配要尽可能的均匀，分配给消费者的主题分区最多相差一个
2. 分区分配尽可能的与上次分配保持相同

当两者发生冲突时，第一个目标优先于第二个目标。鉴于这两个目标，stickyAssignor策略的具体实现比RangeAssignor和RoundRobinAssignor这两种分配策略要复杂很多。

> 这样的好处就是连接可以复用，要消费消息总是要和Broker建立连接，如果能够保持上一次分配的分区的话，那么就不用频繁的销毁创建连接了

举例：

假设消费组内有3个消费者：C0、C1和C2，它们都订阅了4个主题：t0、t1、t2、t3，并且每个主题有2个分区，也就是说整个消费组订阅了t0p0、t0p1、t1p0、t1p1、t2p0、t2p1、t3p0、t3p1这8个分区。最终的分配结果如下：

```
消费者C0：t0p0、t1p1、t3p0
消费者C1：t0p1、t2p0、t3p1
消费者C2：t1p0、t2p1
```

此时假设消费者C1脱离了消费组，那么消费组就会执行再平衡操作，进而消费分区会重新分配。如果采用RoundRobinAssignor策略，那么此时的分配结果如下：

```
消费者C0：t0p0、t1p0、t2p0、t3p0
消费者C2：t0p1、t1p1、t2p1、t3p1
```

如分配结果所示，RoundRobinAssignor策略会按照消费者C0和C2进行重新轮询分配。而如果此时使用的是StickyAssignor策略，那么分配结果为：

```text
消费者C0：t0p0、t1p1、t3p0、t2p0
消费者C2：t1p0、t2p1、t0p1、t3p1
```

可以看到分配结果中**保留了上一次分配中对于消费者C0和C2的所有分配结果**，并将原来消费者C1的“负担”分配给了剩余的两个消费者C0和C2，最终C0和C2的分配还保持了均衡。

> 如果发生分区重分配，那么对于同一个分区而言有可能之前的消费者和新指派的消费者不是同一个，对于之前消费者进行到一半的处理还要在新指派的消费者中再次复现一遍，这显然很浪费系统资源。StickyAssignor策略如同其名称中的“sticky”一样，让分配策略具备一定的“粘性”，尽可能地让前后两次分配相同，进而减少系统资源的损耗以及其它异常情况的发生。

到目前为止所分析的都是消费者的订阅信息都是相同的情况，我们来看一下**订阅信息不同**的情况下的处理。

举例，同样消费组内有3个消费者：C0、C1和C2，集群中有3个主题：t0、t1和t2，这3个主题分别有1、2、3个分区，也就是说集群中有t0p0、t1p0、t1p1、t2p0、t2p1、t2p2这6个分区。消费者C0订阅了主题t0，消费者C1订阅了主题t0和t1，消费者C2订阅了主题t0、t1和t2。

如果此时采用RoundRobinAssignor策略，那么最终的分配结果如下所示（和讲述RoundRobinAssignor策略时的一样，这样不妨赘述一下）：

```text
消费者C0：t0p0
消费者C1：t1p0
消费者C2：t1p1、t2p0、t2p1、t2p2
```

如果此时采用的是StickyAssignor策略，那么最终的分配结果为：

```text
消费者C0：t0p0
消费者C1：t1p0、t1p1
消费者C2：t2p0、t2p1、t2p2
```

可以看到这是一个最优解（消费者C0没有订阅主题t1和t2，所以不能分配主题t1和t2中的任何分区给它，对于消费者C1也可同理推断）。

假如此时消费者C0脱离了消费组，那么RoundRobinAssignor策略的分配结果为：

```text
消费者C1：t0p0、t1p1
消费者C2：t1p0、t2p0、t2p1、t2p2
```

可以看到RoundRobinAssignor策略保留了消费者C1和C2中原有的3个分区的分配：t2p0、t2p1和t2p2（针对结果集1）。而如果采用的是StickyAssignor策略，那么分配结果为：

```text
消费者C1：t1p0、t1p1、t0p0
消费者C2：t2p0、t2p1、t2p2
```

可以看到StickyAssignor策略保留了消费者C1和C2中原有的5个分区的分配：t1p0、t1p1、t2p0、t2p1、t2p2。

从结果上看StickyAssignor策略比另外两者分配策略而言显得更加的优异，这个策略的代码实现也是异常复杂。

#### offset的维护

由于consumer在消费过程中可能会出现断电宕机等故障，consumer恢复后，需要从故障前的位置继续消费，所以consumer需要实时记录自己消费到那个offset，以便故障恢复后继续消费

![image-20221008162929067](https://gitee.com/zhang-songyao/blog-images/raw/master/202210081629298.png)

Kafka 0.9 版本之前，consumer 默认将 offset 保存在 Zookeeper 中，从 0.9 版本开始，consumer 默认将 offset 保存在 Kafka 一个内置的 topic 中，该 topic 为**__consumer_offsets**。

### Kafka高效读写数据

#### 顺序写磁盘

kafka的producer生产数据，要写入log文件，写的过程是一直追加到文件末端，为顺序写。官网有数据表明，同样的磁盘，顺序写能到 600M/s，而随机写只有 100K/s。这与磁盘的机械机构有关，顺序写之所以快，是因为其省去了大量磁头寻址的时间。 

#### 零拷贝

传统IO将一个文件通过socket写出的步骤

```java
File f = new Flie("helloworld/data.txt");
RandomAccessFile flie = RandomAccessFile(f, "r");

byte[] buf = new byte[(int) f.length];
file.read(buf);

Socket socket = ...;
socket.getOutputStream().write(buf);
```

工作过程

<img src="https://gitee.com/zhang-songyao/blog-images/raw/master/20211123162954.png" alt="image-20211123161449044" style="zoom:80%;" />

1. Java本身不具备IO读写能力，因此read方法调用后，要从java程序的用户态切换至内核态，去调用操作系统（Kernel）的读能力，将数据读入内核缓冲区。这期间用户线程阻塞，操作系统使用DMA（Direct Memory Access）来实现文件读，其间也不会使用cpu

   >DMA也可以理解为硬件单元，用来解放cpu完成文件IO

2. 从内核态切换回用户态，将数据从内缓冲区读入用户缓冲区（即byte[] buf），这期间cpu会参与拷贝，无法利用DMA

3. 调用write方法，这时将数据从用户缓冲区（byte[] buf）写入socket缓冲区，cpu会参与拷贝

4. 接下来要向网卡写数据，这项能力java又不具备，因此又需要从用户态切换这内核态，调用操作系统的写能力，使用DMA将socket缓冲区的数据写入网卡，不会使用cpu

> 可以看到中间环节较多，java的IO实际不是物理设备级别的读写，而是缓存的复制，底层真正读写是操作系统来完成的

- 用户态与内核态切换发生了3次，这个操作比较耗费资源
- 数据拷贝了4次

零拷贝（linux2.4后）

内核缓冲区（页缓存）

<img src="https://gitee.com/zhang-songyao/blog-images/raw/master/20211123172409.png" alt="image-20211123172406807" style="zoom:80%;" />

1. java 调用 transferTo 方法后，要从 java 程序的**用户态**切换至**内核态**，使用 DMA将数据读入**内核缓冲区**，不会使用 cpu
2. 只会将一些 offset 和 length 信息拷入 **socket 缓冲区**，几乎无消耗
3. 使用 DMA 将 **内核缓冲区**的数据写入网卡，不会使用 cpu

整个过程仅只发生了一次用户态与内核态的切换，数据拷贝了 2 次。**所谓的【零拷贝】，并不是真正无拷贝，而是在不会拷贝重复数据到 jvm 内存中**，零拷贝的优点有

* 更少的用户态与内核态的切换
* 不利用 cpu 计算，减少 cpu 缓存伪共享
* 零拷贝适合小文件传输

kafka数据直接在内核完成输入和输出，不需要拷贝到用户空间再写出去，kafka数据在写入磁盘前，数据先写到进程的内存空间

#### Memory Mapped Files

> mmap将磁盘文件映射到内存，支持读和写，对内存的操作会反映在磁盘文件上

即便是顺序写入硬盘，硬盘的访问速度还是不可能追上内存。所以 Kafka 的数据并不是实时的写入硬盘 ，它充分利用了现代操作系统分页存储来利用内存提高 I/O 效率。

Memory Mapped Files(后面简称 mmap)也被翻译成 内存映射文件 ，在 64 位操作系统中一般可以表示 20G 的数据文件，它的工作原理是直接利用操作系统的 Page 来实现文件到物理内存的直接映射。

完成映射之后你对物理内存的操作会被同步到硬盘上（操作系统在适当的时候）。

通过 mmap，进程像读写硬盘一样读写内存（当然是虚拟机内存），也不必关心内存的大小有虚拟内存为我们兜底。

使用这种方式可以获取很大的 I/O 提升，省去了用户空间到内核空间复制的开销（调用文件的 read 会把数据先放到内核空间的内存中，然后再复制到用户空间的内存中。）

但也有一个很明显的缺陷——不可靠，写到 mmap 中的数据并没有被真正的写到硬盘，操作系统会在程序主动调用 flush 的时候才把数据真正的写到硬盘。

Kafka 提供了一个参数——producer.type 来控制是不是主动 flush，如果 Kafka 写入到 mmap 之后就立即 flush 然后再返回 Producer 叫 同步 (sync)；写入 mmap 之后立即返回 Producer 不调用 flush 叫异步 (async)。

## zookeeper在kafka中的作用

https://gitbook.cn/books/5ae1e77197c22f130e67ec4e/index.html

### Controller节点

kafka集群中有一个broker会被选举为Controller，负责管理集群broker的上下线，所有topic的分区副本分配和leader选举等工作

Controller的管理工作都是依赖于Zk的，以下为partition的leader选举过程

![image-20221008172212692](https://gitee.com/zhang-songyao/blog-images/raw/master/202210081722336.png)

### broker注册

- 为了记录broker的注册信息，在zk上，专门创建了属于kafka的一个几点，其路径为/brokers
- kafka的每个broker启动时，都会到zk中进行注册，告诉zk其broker.id，在整个集群中，broker.id应该是全局唯一的，并在zk上创建其属于自己的节点，其节点路径为`/brokers/ids/{broker.id}`
- 创建完节点后，kafka会将broker的broker.name及端口号记录到该节点
- 该broker节点属性为临时节点，当broker会话失效时，zk会删除该节点，这样，我们就可以方便的监控到broker节点的变化，及时调整负载均衡等

### topic注册

kafka中，所有的topic和broker的对应关系都是由zk进行维护，在zk中，建立专门的节点来记录这些信息，其节点路径为`/broker/topics/{topic.name}`。为了保障数据的可靠性，每个 Topic 的 Partitions 实际上是存在备份的，并且备份的数量由 Kafka 机制中的 replicas 来控制。那么问题来了：如下图所示，假设某个 TopicA 被分为 2 个 Partitions，并且存在两个备份，由于这 2 个 Partitions（1-2）被分布在不同的 broker 上，同一个 partiton 与其备份不能（也不应该）存储于同一个 broker 上。以 Partition1 为例，假设它被存储于 broker2，其对应的备份分别存储于 broker1 和 broker4，有了备份，可靠性得到保障，但数据一致性却是个问题。

![enter image description here](https://gitee.com/zhang-songyao/blog-images/raw/master/202210081733801.png)

为了保障数据的一致性，ZooKeeper 机制得以引入。基于 ZooKeeper，Kafka 为每一个 partition 找一个节点作为 leader，其余备份作为 follower；接续上图的例子，就 TopicA 的 partition1 而言，如果位于 broker2（Kafka 节点）上的 partition1 为 leader，那么位于 broker1 和 broker4 上面的 partition1 就充当 follower，则有下图：

![enter image description here](https://gitee.com/zhang-songyao/blog-images/raw/master/202210081733819.png)

基于上图的架构，当 producer push 的消息写入 partition（分区) 时，作为 leader 的 broker（Kafka 节点） 会将消息写入自己的分区，同时还会将此消息复制到各个 follower，实现同步。如果，某个follower 挂掉，leader 会再找一个替代并同步消息；如果 leader 挂了，follower 们会选举出一个新的 leader 替代，继续业务，这些都是由 ZooKeeper 完成的。

### consumer注册

#### 消费者组

当新的消费者组注册到zk上时，zk会创建专用的节点来保存相关信息，其节点路径为`ls/consumers/{group_id}`，其节点下有三个子节点，分别为 `[ids, owners, offsets]`。

- ids 节点：记录该消费组中当前正在消费的消费者；
- owners 节点：记录该消费组消费的 topic 信息；
- offsets 节点：记录每个 topic 的每个分区的 offset。

#### 消费者

当新的消费者注册到 Kafka 中时，会在 `/consumers/{group_id}/ids` 节点下创建临时子节点，并记录相关信息。

#### 监听消费者分组中消费者的变化

每个消费者都要关注其所属消费者组中消费者数目的变化，即监听 `/consumers/{group_id}/ids` 下子节点的变化。一单发现消费者新增或减少，就会触发消费者的负载均衡。

### producer负载均衡

对于同一个topic不同的partition，kafka会尽力将这些partition分布到不同的broker服务器上，这种均衡策略实际上是基于zk实现的，在一个broker启动时，会首先完成broker的注册过程，并注册一些诸如：“有哪些可以订阅的topic”之类的元数据信息。producer启动后也要到zk下注册，创建一个临时节点来监听broker服务器列表的变化。由于在zk下broker创建的也是临时节点，当brokers发生变化时，producer可以得到相关通知，从而改变自己的broker list。其它的诸如 topic 的变化以及broker 和 topic 的关系变化，也是通过 ZooKeeper 的这种 Watcher 监听实现的。

生产环境中，必须制定topic，对于partition，有两种制定方式：

- 明确制定partition（0-N），则数据被发送到制定的partition
- 设置为`RD_KAFKA_PARTITION_UA`，则kafka会回调partitioner进行负载均衡，partitioner方法需要自己实现。可以轮询或者传入key进行hash。未实现采用默认的随机方`rd_kafka_msg_partitioner_random`，随机选择

### consumer负载均衡

Kafka 保证同一 consumer group 中只有一个 consumer 可消费某条消息，实际上，Kafka 保证的是稳定状态下每一个 consumer 实例只会消费某一个或多个特定的数据，而某个 partition 的数据只会被某一个特定的 consumer 实例所消费。这样设计的劣势是无法让同一个 consumer group 里的 consumer 均匀消费数据，优势是每个 consumer 不用都跟大量的 broker 通信，减少通信开销，同时也降低了分配难度，实现也更简单。另外，因为同一个 partition 里的数据是有序的，这种设计可以保证每个 partition 里的数据也是有序被消费。

**consumer 数量不等于 partition 数量**

如果某 consumer group 中 consumer 数量少于 partition 数量，则至少有一个 consumer 会消费多个 partition 的数据；如果 consumer 的数量与 partition 数量相同，则正好一个 consumer 消费一个 partition 的数据，而如果 consumer 的数量多于 partition 的数量时，会有部分 consumer 无法消费该 topic 下任何一条消息。

**借助 ZooKeeper 实现负载均衡**

关于负载均衡，对于某些低级别的 API，consumer 消费时必须指定 topic 和 partition，这显然不是一种友好的均衡策略。基于高级别的 API，consumer 消费时只需制定 topic，借助 ZooKeeper 可以根据 partition 的数量和 consumer 的数量做到均衡的动态配置。

consumers 在启动时会到 ZooKeeper 下以自己的 conusmer-id 创建临时节点 `/consumer/[group-id]/ids/[conusmer-id]`，并对 `/consumer/[group-id]/ids` 注册监听事件，当消费者发生变化时，同一 group 的其余消费者会得到通知。当然，消费者还要监听 broker 列表的变化。librdkafka 通常会将 partition 进行排序后，根据消费者列表，进行轮流的分配。

### 记录消费进度offset

在 consumer 对指定消息 partition 的消息进行消费的过程中，需要定时地将 partition 消息的消费进度 Offset 记录到 ZooKeeper上，以便在该 consumer 进行重启或者其它 consumer 重新接管该消息分区的消息消费权后，能够从之前的进度开始继续进行消息消费。Offset 在 ZooKeeper 中由一个专门节点进行记录，其节点路径为：

```
#节点内容就是Offset的值。
/consumers/[group_id]/offsets/[topic]/[broker_id-partition_id]
```

PS：Kafka 已推荐将 consumer 的 Offset 信息保存在 Kafka 内部的 topic 中，即：

```
__consumer_offsets(/brokers/topics/__consumer_offsets)
```

并且默认提供了 `kafka_consumer_groups.sh` 脚本供用户查看consumer 信息（命令：`sh kafka-consumer-groups.sh –bootstrap-server * –describe –group *`）。在当前版本中，offset 存储方式要么存储在本地文件中，要么存储在 broker 端，具体的存储方式取决 `offset.store.method` 的配置，默认是存储在 broker 端。

### 记录 Partition 与 Consumer 的关系

consumer group 下有多个 consumer（消费者），对于每个消费者组（consumer group），Kafka都会为其分配一个全局唯一的 group ID，group 内部的所有消费者共享该 ID。订阅的 topic 下的每个分区只能分配给某个 group 下的一个consumer（当然该分区还可以被分配给其它 group）。同时，Kafka 为每个消费者分配一个 consumer ID，通常采用 `hostname:UUID` 形式表示。

在Kafka中，规定了每个 partition 只能被同组的一个消费者进行消费，因此，需要在 ZooKeeper 上记录下 partition 与 consumer 之间的关系，每个 consumer 一旦确定了对一个 partition 的消费权力，需要将其 consumer ID 写入到 ZooKeeper 对应消息分区的临时节点上，例如：

```
/consumers/[group_id]/owners/[topic]/[broker_id-partition_id]
```

其中，[`broker_id-partition_id`] 就是一个消息分区的标识，节点内容就是该消息分区 消费者的 consumer ID。

## kafka事务

kafka从0.11版本后开始引入了对事务的支持，事务可以保证kafka在Exactly Once语义的基础上，生产和消费可以跨分区和会话，要么全部成功，要么全部失败

### producer事务

为了实现跨分区夸会话事务，需要引入一个全局唯一的Transaction ID，并将Producer获得的PID和Transaction ID绑定。这样在重启后就可以通过正在进行的Transaction ID获得原来的PID。

为了管理Transaction，kafka引入了一个新的组件Transaction Coordinator。Producer就是通过和 Transaction Coordinator 交互获得 Transaction ID 对应的任务状态。Transaction Coordinator 还负责将事务所有写入 Kafka 的一个内部 Topic，这样即使整个服务重启，由于事务状态得到保存，进行中的事务状态可以得到恢复，从而继续进行。

### consumer事务

上述事务机制主要是从 Producer 方面考虑，对于 Consumer 而言，事务的保证就会相对较弱，尤其时无法保证 Commit 的信息被精确消费。这是由于 Consumer 可以通过 offset 访问任意信息，而且不同的 Segment File 生命周期不同，同一事务的消息可能会出现重启后被删除的情况。

## Q&A

https://mp.weixin.qq.com/s/SuALTpvI3IMPSja9pacJ7Q

https://www.cnblogs.com/luozhiyun/p/11811835.html

> 谈谈你对Kafka的理解

kafka是一个流式数据处理平台，他具有消息系统的能力，也有实时流式数据处理分析能力，只是我们更偏向于把它当做消息队列来使用。

大致可分为三层

第一层zookeeper，相当于注册中心，他负责kafka集群元数据的管理，以及集群的协调工作，在每个kafka服务器启动的时候去连接到zookeeper，把自己注册到zookeeper中

第二层是kafka的核心层，这里包含很多kafka的基本概念

- record：代表消息
- topic：主题，消息都会由一个主题方式来组织，可以理解为消息的一个分类
- producer：生产者，负责发送消息
- consumer：消费者，负责消费消息
- broker：kafka服务器
- partition：分区，主题会由多个分区组成，通常每个分区的消息都是按照顺序来读取的，不同的分区无法保证顺序性，分区就是我们常说的数据分片Sharding机制（类似ES），主要目的是为了提高系统的伸缩能力，通过分区，消息的读写可以负载均衡到多个不同的节点上
- leader和follower：分区的副本，为了保证高可用，分区都会有一些副本，每个分区都会有一个leader主副本负责读写数据，follower从副本只负责和leader副本保持数据同步，不对外提供服务
- offset：偏移量，分区中每一条消息都会根据时间先后顺序有一个递增的序号，这个序号就是offset偏移量
- consumer group：消费者组，由多个消费者组成，一个组内只会由一个消费者消费一个分区的消息
- Coordinator：协调者，主要是为消费者组分配分区以及平衡Rebalance操作
- controller：控制器，其实就是一个broker，用于协调和管理整个kafka集群，他会负责分区leader的选举、主题管理等工作，在zookeeper第一个创建临时节点/controller的就会成为控制器

第三层是存储层用来保存kafka的核心数据，他们都会以日志的形式最终写入磁盘中。

> 消息队列模型？kafka是怎么做到支持这两种模型的？

传统的消息队列支持两种模型：

1. 点对点：也就是消息只能被一个消费者消费，消费完后消息删除
2. 发布订阅：相当于广播模式，消息可以被所有消费者消费

**kafka通过Consumer Group同时支持了这两个模型**

如果说所有的消费者都属于一个Group，消息只能被同一个Group内的一个消费者消费，那就是点对点模式

如果每个消费者都是一个单独的Group，那么就是发布订阅模式

> 发送消息的时候怎么选择分区

如果指定分区，则选择指定分区

如果没有指定，有两种方式：

1. 轮询，按照顺序消息依次发送到不同的分区
2. 随机，随机发送到某个分区

如果消息指定key，那么会根据消息的key进行hash，然后对partition分区数量取模，决定落在那个分区，所以对于相同的key来说，总是会发送到同一个分区上，也是我们常说的消息分区有序性。

很常见的场景就是希望下单、支付消息有顺序，这样以订单ID为key发送消息就达到了分区有序性的目的。

如果没有指定key，会执行默认的轮询负载策略，比如第一条消息落在了P0，第二条消息落在P1，然后第三条又落在P1。

对于一些特定的业务场景和需求，还可以通过实现partitioner接口，重写configure和partition方法来达到自定义分区的效果。

>为什么需要分区？有什么好处？

如果不分区的话，我们发消息写数据都只能保存到一个节点上，这样的话就算这个服务器节点性能再好最终也撑不住。

实际上，**分布式系统都面临这个问题，要么收到消息后进行数据切分，要么提前切分**，kafka正式选择的前者，通过分区可以把数据均匀的分配到不同节点。

分区带了了负载均衡和横向扩展的能力。

发送消息时可以根据分区的数量落在不同的kafka服务器节点上，提示了并发写消息的性能，消费消息的时候又和消费者绑定，可以从不同节点的不同分区消费消息，挺高了读消息的能力。

引入分区的同时，又引入了副本，冗余的副本保证了kafka的高可用和高持久性。

> kafka通信过程原理？

1. 首先kakfa broker启动时，会向zookeeper注册自己的ID（创建临时节点），这个ID可以配置也可以自动生成，同时会去订阅zookeeper的brokers/ids路径，当有新的broker加入或者退出时，可以得到当前所有broker信息
2. 生产者启动的时候，会指定bootstrap.servers，通过指定的broker地址，kafka就会和这些broker建立连接（通常我们不用配置所有的broker服务器地址，否则kafka会和配置的所有broker都建立TCP连接）
3. 随便连接到任何一台broker之后，然后再发送请求元数据信息（包含有哪些主题、主题有哪些分区、分区有哪些副本，分区的leader副本等信息）
4. 接着就会创建和所有broker的TCP连接
5. 发送消息
6. 消费者和生产者一样，也会制定bootstrap.servers属性，然后选择一台broker创建TCP连接，发送请求找到协调者所在的broker
7. 然后再和协调者broker创建TCP连接
8. 根据分区Leader节点所在的broker节点，和这些broker分别创建连接
9. 开始消费消息

![image-20221009184409066](https://gitee.com/zhang-songyao/blog-images/raw/master/202210091844705.png)

> 消费者组和消费者重平衡

kafka中的消费者组订阅topic主题的消息，一般来说消费者的数量最好和所有主题分区的数量保持一致最好。

当消费者数量小于分区数量的时候，那么必然会有一个消费者消费多个分区的消息。

当消费者数量超过分区数量的时候，那么必然有消费者没有分区可以消费。

所以，消费者组的好处，一方面，可以支持多种消息模型，另一方面，可以根据消费者和分区的消费关系，支撑横向扩容伸缩。

明确消费者怎么消费分区后，那么就会出现一个问题：消费者消费的分区是怎么分配的，有新加入的消费者的时候怎么处理？

旧版本的重平衡过程主要通过zk监听器的方式来触发，每个消费者客户端自己去执行分区分配算法；新版本则是通过协调者完成，每一次新的消费者加入都会发送请求给协调者去获取分区的分配，这个分区分配的算法的逻辑是有协调者来完成的。

而重平衡Rebalance就是指的有新消费者加入的情况，比如刚开始我们只有消费者A在消费消息，过了一段时间消费者B和C加入了，这时候分区就需要重新分配，这就是重平衡，也可以叫做再平衡，但是重平衡的过程和我们的GC时候STW很像，会导致整个消费群组停止工作，重平衡期间都无法消息消息。

另外，发生重平衡并不是只有这一种情况，因为消费者和分区总数是存在绑定关系的，上面也说了，消费者数量最好和所有主题的分区总数一样。

那只要**消费者数量**、**主题数量**（比如用的正则订阅的主题）、**分区数量**任何一个发生改变，都会触发重平衡。

重平衡的过程：

重平衡机制依赖消费者和协调者之间的心跳来维持的，消费者会有一个独立的线程去定时发送心跳给协调者，这个可以通过参数heartbeat.interval.ms来控制发送心跳的间隔时间。

1. 每个消费者第一次加入组的时候都会想协调者发送JionGroup请求，第一个发送这个请求的消费者，会成为”群主“，协调者会返回组成员列表给群主。
2. 群主执行分区分配策略，然后吧分配结果通过SyncGroup请求发送给协调者，协调者收到分区分配结果。
3. 其他组内成员也想协调者发送SyncGroup请求，协调者把每个消费者的分区分配分别相应给他们。

![image-20221010150412026](https://gitee.com/zhang-songyao/blog-images/raw/master/202210101504728.png)

> 分区分配策略

上面已经讲过了

> 如何保证消息可靠性

消息可靠性的保证基本上要从三个方面来阐述

`生产者发送消息丢失`

kafka支持三种方式发送消息：

1. 直接发送，直接调用sentd方法，不管结果，虽然可以开启自动重试，但是肯定会有消息丢失的可能
2. 同步发送，同步发送返回Future对象，我们可以知道发送结果，然后进行处理
3. 异步发送，发送消息，同时指定一个回调函数，根据结果进行相应的处理

为了保险起见，一般都会使用异步发送带有回调的方式进行发送消息，再设置参数为发送消息失败不停重试

acks=all，这个参数有可以配置0|1|all。

- 0表示生产者写入消息不管服务器的响应，可能消息还在网络缓冲区，服务器根本没有收到消息，当然会丢失消息。
- 1表示至少有一个副本收到消息才认为成功，一个副本那肯定就是集群的Leader副本了，但是如果刚好Leader副本所在的节点挂了，Follower没有同步这条消息，消息仍然丢失了。
- 配置all的话表示所有ISR都写入成功才算成功，那除非所有ISR里的副本全挂了，消息才会丢失。

retries=N，设置一个非常大的值，可以让生产者发送消息失败后不停重试

`kakfa自身消息丢失`

kafka因为写消息写入是通过PageCache异步写入磁盘的，因此仍然存在丢失消息的可能

针对kafka自身消息丢失可能设置的参数：

- replication.factor=N，设置一个比较大的值，保证至少有2个或者以上的副本。
- min.insync.replicas=N，代表消息如何才能被认为是写入成功，设置大于1的数，保证至少写入1个或者以上的副本才算写入消息成功。
- unclean.leader.election.enable=false，这个设置意味着没有完全同步的分区副本不能成为Leader副本，如果是true的话，那些没有完全同步Leader的副本成为Leader之后，就会有消息丢失的风险。

`消费者消息丢失`

关闭自动提交位移即可，改为业务处理成功手动提交

因为重平衡发生的时候，消费者会读取上一次提交的偏移量，自动提交默认每5s执行一次，这会导致重复消费或者丢失消息。

enable.auto.commit=false，设置手动提交

auto.offset.reset=earliest，这个参数代表没有偏移量可以提交或者broker上不存在偏移量的时候，消费者如何处理。earliest代表从分区的开始位置读取，可能会重复读取消息，但是不会丢失，消费方一般我们肯定要自己保证幂等，另外一种latest表示从分区末尾读取，那就会有概率丢失消息。

综合以上参数设置，可以保证消息不会丢失，保证了可靠性

> 副本和它的同步原理

kakfa的副本，分为leader和follower副本，也就是主副本和从副本，于mysql不同的是，kafka中只有leader副本会对外提供服务，follower副本只是单纯的和leader副本保持数据同步，作为数据冗余容灾的作用。

kafka中所有的副本的集合统称为AR（Assigned Replicas），和leader副本保持同步的副本集合称为ISR（InSyncReplicas）

ISR是一个动态集合，维持这个集合会通过replica.lag.time.max.ms参数来控制，这个代表落后leader副本的最长时间，默认值为10秒，所以只要follower副本没有落后leader副本超过10秒以上，就可以认为是和leader副本同步的（简单理解为同步时间差），超过该阈值会把follower剔除出ISR，加入OSR（OutOfSynReplicas）列表，新加入的follower也会先存放在OSR

ISR + OSR = AR

另外还有两个关键的概念用于副本之间的同步：

![image-20221010155119686](https://gitee.com/zhang-songyao/blog-images/raw/master/202210101551775.png)

**HW（High Watermark）：**高水位，也叫做复制点，表示副本间同步的位置。如下图所示，0~4绿色表示已经提交的消息，这些消息已经在副本之间进行同步，消费者可以看见这些消息并且进行消费，4~6黄色的则是表示未提交的消息，可能还没有在副本间同步，这些消息对于消费者是不可见的。

**LEO（Log End Offset）：**下一条待写入消息的位移

副本间同步的过程依赖的就是HW和LEO的更新，以他们的值变化来演示副本同步消息的过程，绿色表示Leader副本，黄色表示Follower副本。



首先，生产者不停地向Leader写入数据，这时候Leader的LEO可能已经达到了10，但是HW依然是0，两个Follower向Leader请求同步数据，他们的值都是0。



![图片](https://gitee.com/zhang-songyao/blog-images/raw/master/202210101552477.jpeg)



然后，消息还在继续写入，Leader的LEO值又发生了变化，两个Follower也各自拉取到了自己的消息，于是更新自己的LEO值，但是这时候Leader的HW依然没有改变。



![图片](https://gitee.com/zhang-songyao/blog-images/raw/master/202210101552411.jpeg)



此时，Follower再次向Leader拉取数据，这时候Leader会更新自己的HW值，取Follower中的最小的LEO值来更新。



![图片](https://gitee.com/zhang-songyao/blog-images/raw/master/202210101552502.jpeg)



之后，Leader响应自己的HW给Follower，Follower更新自己的HW值，因为又拉取到了消息，所以再次更新LEO，流程以此类推。



![图片](https://gitee.com/zhang-songyao/blog-images/raw/master/202210101552152.jpeg)

> 新版本的kafka为什么抛弃了zookeeper

首先，从运维的复杂度来看，Kafka本身是一个分布式系统，他的运维就已经很复杂了，那除此之外，还需要重度依赖另外一个ZK，这对成本和复杂度来说都是一个很大的工作量。

其次，应该是考虑到性能方面的问题，比如之前的提交位移的操作都是保存在ZK里面的，但是ZK实际上不适合这种高频的读写更新操作，这样的话会严重影响ZK集群的性能，这一方面后来新版本中Kafka也把提交和保存位移用消息的方式来处理了。

另外Kafka严重依赖ZK来实现元数据的管理和集群的协调工作，如果集群规模庞大，主题和分区数量很多，会导致ZK集群的元数据过多，集群压力过大，直接影响到很多Watch的延时或者丢失。

> kafka为什么快

顺序IO

kafka写消息到分区采用追加的方式，也就是顺序写入磁盘，不是随机写入，这个速度比普通的随机IO要快非常多，集合可以和网络IO的速度相匹配

Page Cache和零拷贝

kafka在写入消息数据的时候通过mmap内存映射的方式，不是真正的立刻写入磁盘，而是利用操作系统的文件缓存PageCache异步写入，提高了写入消息的性能，另外再消费小的时候又通过sendfile实现了零拷贝

批量处理和压缩

kafka在发送消息的时候不是一条条发送的，而是会把多条消息合并成一个批次进行处理发送，消费消息也是一个道理，一次拉取一批次的消息进行消费。

并且producer、broker、consumer都是用优化后的压缩算法，发送和消息消息使用压缩节省了网络传输的开销，Broker存储使用压缩则降低了磁盘存储的空间。

> kafka怎么体现消息顺序性

可以通过分区策略体现消息顺序性

分区策略包括：轮询策略、随机策略、按消息键保序策略

按消息键保序策略：一旦消息被定义了key，那么你可以保证同一个key的所有消息都进入到相同的分区中，由于每个分区下的消息处理都是有顺序的，故这个策略被称为按消息键保序策略

kafka只能保证分区内消息顺序有序，无法保证全局有序

1. 生产者：通过分区leader副本负责数据顺序写入，来保证消息顺序性
2. 消费者：同一个分区内的消息只能被一个group里面的一个消费者消费，保证分区内消费有序

做不到全局有序，因为消息会发送到不同的分区，分区之间发送消息的顺序是无法保证的

> Kafka中的分区器、序列化器、拦截器是否了解？它们之间的处理顺序是什么？

序列化器：生产者需要用序列化器（Serializer）把对象转换成字节数组才能通过网络发送给kafka，而在对侧，消费者需要用反序列化器吧从kafka中收到的字节数组转成相应对象。

分区器：分区器的作用就是为消息分区分配。如果消息ProducerRecord中没有指定partition字段，那么就需要依赖分区器，根据key，来计算partition的值。

kafka一共有两种拦截器：生产者拦截器和消费者拦截器

- 生产者拦截器既可以用来在消息发送前做一些准备工作，比如按照某个规则过滤不符合要求的消息、修改消息的内容等，也可以用来在发送回调逻辑前做一些定制化的需求，比如统计类工作。
- 消费者拦截器主要在消费到消息或在提交消费位移时进行一些定制化的操

处理顺序 ：拦截器->序列化器->分区器

KafkaProducer 在将消息序列化和计算分区之前会调用生产者拦截器的 onSend() 方法来对消息进行相应的定制化操作。然后生产者需要用序列化器（Serializer）把对象转换成字节数组才能通过网络发送给 Kafka。
最后可能会被发往分区器为消息分配分区。

> kafka生产者客户端整体结构

![img](https://gitee.com/zhang-songyao/blog-images/raw/master/202210101814956.png)

真个生产者客户端由两个线程协调运行，这两个线程分别为主线程和Sender线程（发送线程）

在主线程中由 KafkaProducer 创建消息，然后通过可能的拦截器、序列化器和分区器的作用之后缓存到消息累加器（RecordAccumulator，也称为消息收集器）中。
Sender 线程负责从 RecordAccumulator 中获取消息并将其发送到 Kafka 中。
RecordAccumulator 主要用来缓存消息以便 Sender 线程可以批量发送，进而减少网络传输的资源消耗以提升性能。

> 消费者提交消费位移时提交的是当前消费到的最新消息的offset还是offset+1?

在旧消费者客户端中，消费位移是存储在 ZooKeeper 中的。而在新消费者客户端中，消费位移存储在 Kafka 内部的主题__consumer_offsets 中。
当前消费者需要提交的消费位移是offset+1

> topic分区数可以增加或者减少吗？

可以增加使用 kafka-topics 脚本，结合 --alter 参数来增加某个主题的分区数，命令如下：

```shell
Copybin/kafka-topics.sh --bootstrap-server broker_host:port --alter --topic <topic_name> --partitions <新分区数>
```

当分区数增加时，就会触发该订阅该主题所有Group开启Rebalance。

首先，Rebalance 过程对 Consumer Group 消费过程有极大的影响。在 Rebalance 过程中，所有 Consumer 实例都会停止消费，等待 Rebalance 完成。这是 Rebalance 为人诟病的一个方面。

其次，目前 Rebalance 的设计是所有 Consumer 实例共同参与，全部重新分配所有分区。其实更高效的做法是尽量减少分配方案的变动。

最后，Rebalance 实在是太慢了。

不支持减少，因为删除的分区中的消息不好处理。如果直接存储到现有分区的尾部，消息的时间戳就不会递增，如此对于 Spark、Flink 这类需要消息时间戳（事件时间）的组件将会受到影响；如果分散插入现有的分区，那么在消息量很大的时候，内部的数据复制会占用很大的资源，而且在复制期间，此主题的可用性又如何得到保障？与此同时，顺序性问题、事务性问题，以及分区和副本的状态机切换问题都是不得不面对的。

>创建topic时如何选择合适的分区数

在 Kafka 中，性能与分区数有着必然的关系，在设定分区数时一般也需要考虑性能的因素。对不同的硬件而言，其对应的性能也会不太一样。
可以使用Kafka 本身提供的用于生产者性能测试的 kafka-producer- perf-test.sh 和用于消费者性能测试的 kafka-consumer-perf-test.sh来进行测试。
增加合适的分区数可以在一定程度上提升整体吞吐量，但超过对应的阈值之后吞吐量不升反降。如果应用对吞吐量有一定程度上的要求，则建议在投入生产环境之前对同款硬件资源做一个完备的吞吐量相关的测试，以找到合适的分区数阈值区间。
分区数的多少还会影响系统的可用性。如果分区数非常多，如果集群中的某个 broker 节点宕机，那么就会有大量的分区需要同时进行 leader 角色切换，这个切换的过程会耗费一笔可观的时间，并且在这个时间窗口内这些分区也会变得不可用。
分区数越多也会让 Kafka 的正常启动和关闭的耗时变得越长，与此同时，主题的分区数越多不仅会增加日志清理的耗时，而且在被删除时也会耗费更多的时间。