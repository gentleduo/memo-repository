# 消息队列

## 优点

1. 解耦

   允许你独立的扩展或修改两边的处理过程，只要确保它们遵守同样的接口约束。

2. 可恢复性

   系统的一部分组件失效时，不会影响到整个系统。消息队列降低了进程间的耦合度，所

   以即使一个处理消息的进程挂掉，加入队列中的消息仍然可以在系统恢复后被处理。

3. 缓冲

   有助于控制和优化数据流经过系统的速度，解决生产消息和消费消息的处理速度不一致

   的情况。

4. 灵活性 & 峰值处理能力

   在访问量剧增的情况下，应用仍然需要继续发挥作用，但是这样的突发流量并不常见。

   如果为以能处理这类峰值访问为标准来投入资源随时待命无疑是巨大的浪费。使用消息队列

   能够使关键组件顶住突发的访问压力，而不会因为突发的超负荷的请求而完全崩溃。

5. 异步通信

   很多时候，用户不想也不需要立即处理消息。消息队列提供了异步处理机制，允许用户

   把一个消息放入队列，但并不立即处理它。想向队列中放入多少消息就放多少，然后在需要

   的时候再去处理它们。

## 消息队列的两种模式

### 点对点模式（一对一，消费者主动拉取数据，消息收到后消息清除）

消息生产者生产消息发送到Queue中，然后消息消费者从Queue中取出并且消费消息。消息被消费以后，queue中不再有存储，所以消息消费者不可能消费到已经被消费的消息。Queue支持存在多个消费者，但是对一个消息而言，只会有一个消费者可以消费。

### 发布/订阅模式（一对多，消费者消费数据之后不会清除消息）

消息生产者（发布）将消息发布到topic中，同时有多个消息消费者（订阅）消费该消息。和点对点方式不同，发布到topic的消息会被所有订阅者消费。

## Kafka基础架构

1. Producer：消息生产者，就是向kafka broker发消息的客户端；
2. Consumer：消息消费者，向kafka broker取消息的客户端；
3. Consumer Group（CG）：消费者组，由多个consumer组成。消费者组内每个消费者负责消费不同分区的数据，一个分区只能由一个组内消费者消费；消费者组之间互不影响。所有的消费者都属于某个消费者组，即消费者组是逻辑上的一个订阅者。
4. Broker：一台kafka服务器就是一个broker。一个集群由多个broker组成。一个broker可以容纳多个topic
5. Topic：可以理解为一个队列，生产者和消费者面向的都是一个topic；
6. Partition：为了实现扩展性，一个非常大的topic可以分布到多个broker（即服务器）上，一个topic可以分为多个partition，每个partition是一个有序的队列；
7. Replica：副本，为保证集群中的某个节点发生故障时，该节点上的partition数据不丢失，且kafka仍然能够继续工作，kafka提供了副本机制，一个topic的每个分区都有若干个副本，一个leader和若干个follower。
8. leader：每个分区多个副本的“主”，生产者发送数据的对象，以及消费者消费数据的对象都是leader。
9. follower：每个分区多个副本中的“从”，实时从leader中同步数据，保持和leader数据的同步。leader发生故障时，某个follower会成为新的follower。

## Kafka命令行操作

```bash
#启动集群
kafka-server-start.sh -daemon config/server.properties
#关闭集群
kafka-server-stop.sh stop
#查看当前服务器中的所有topic
kafka-topics.sh --zookeeper server01:2181 --list
#查看某个Topic的详情
kafka-topics.sh --zookeeper server01:2181 --describe --topic gentleduo-topic-1
#创建topic
kafka-topics.sh --zookeeper server01:2181 --create --replication-factor 2 --partitions 3 --topic gentleduo-topic-1
选项说明：
--topic 定义topic名
--replication-factor 定义副本数
--partitions 定义分区数
#删除topic（需要server.properties中设置delete.topic.enable=true否则只是标记删除。）
kafka-topics.sh --zookeeper server01:2181 --delete --topic gentleduo-topic-1
#修改分区数
kafka-topics.sh --zookeeper server01:2181 --alter --topic first --partitions 6
#发送消息
kafka-console-producer.sh --brokerlist server01:9092 --topic gentleduo-topic-1
#消费消息（--from-beginning：会把主题中以往所有的数据都读取出来。）
kafka-console-consumer.sh --zookeeper server01:2181 --topic gentleduo-topic-1
kafka-console-consumer.sh --bootstrap-server server01:9092 --topic gentleduo-topic-1
kafka-console-consumer.sh --bootstrap-server server01:9092 --from-beginning --topic gentleduo-topic-1
#查询消费者组
kafka-consumer-groups.sh --bootstrap-server server01:9092 --list
#查询消费者组详情
#LogEndOffset 下一条将要被加入到日志的消息的位移
#CurrentOffset 当前消费的位移
#LAG 消息堆积量：消息中间件服务端中所留存的消息与消费掉的消息之间的差值即为消息堆积量也称之为消费滞后量
kafka-consumer-groups.sh --bootstrap-server server01:9092 --describe --group groupname
```

# Kafka架构深入

## Kafka工作流程及文件存储机制

Kafka中消息是以topic进行分类的，生产者生产消息，消费者消费消息，都是面向topic的。topic是逻辑上的概念，而partition是物理上的概念，每个partition对应于一个log文件，该log文件中存储的就是producer生产的数据。Producer生产的数据会被不断追加到该log文件末端，且每条数据都有自己的offset。消费者组中的每个消费者，都会实时记录自己消费到了哪个offset，以便出错恢复时，从上次的位置继续消费。

由于生产者生产的消息会不断追加到log文件末尾，为防止log文件过大导致数据定位效率低下，Kafka采取了分片和索引机制，将每个partition分为多个segment。每个segment对应两个文件——“.index”文件和“.log”文件。这些文件位于一个文件夹下，该文件夹的命名规则为：topic名称+分区序号。例如，test这个first有三个分区，则其对应的文件夹为first-0,first-1,first-2。

```
00000000000000000000.index
00000000000000000000.log
00000000000000170410.index
00000000000000170410.log
00000000000000239430.index
00000000000000239430.log
```

index和log文件以当前segment的第一条消息的offset命名。".index"文件存储大量的索引信息，".log"文件存储大量的数据，索引文件中的元数据指向对应数据文件中message的物理偏移地址。

## Kafka生产者

### 分区策略

1. 分区的原因
   1. 方便在集群中扩展，每个Partition可以通过调整以适应它所在的机器，而一个topic又可以有多个 Partition组成，因此整个集群就可以适应任意大小的数据了；
   2. 可以提高并发，因为可以以Partition为单位读写了。
2. 分区的原则
   1. 指明partition的情况下，直接将指明的值直接作为partiton值；
   2. 没有指明partition值但有key的情况下，将key的hash值与topic的partition数进行取余得到partition值；
   3. 既没有partition值又没有key值的情况下，第一次调用时随机生成一个整数（后面每次调用在这个整数上自增），将这个值与topic可用的partition总数取余得到partition值，也就是常说的round-robin算法。

### 数据可靠性保证

#### 副本数据同步策略

为保证producer发送的数据，能可靠的发送到指定的topic，topic的每个partition收到producer发送的数据后，都需要向producer发送ack（acknowledgement确认收到），如果producer收到ack，就会进行下一轮的发送，否则重新发送数据。

1. 何时发送ack？确保有follower与leader同步完成，leader再发送ack，这样才能保证leader挂掉之后，能在follower中选举出新的leader
2. 多少个follower同步完成之后发送ack？有如下两个方案：

| 方案                      | 优点                                               | 缺点                                                |
| ------------------------- | -------------------------------------------------- | --------------------------------------------------- |
| 半数以上完成同步就发送ack | 延迟低                                             | 选举新的leader时，容忍n台节点的故障，需要2n+1个副本 |
| 全部完成同步，才发送ack   | 选举新的leader时，容忍n台节点的故障，需要n+1个副本 | 延迟高                                              |

Kafka选择了第二种方案，原因如下：

1. 同样为了容忍n台节点的故障，第一种方案需要2n+1个副本，而第二种方案只需要n+1个副本，而Kafka的每个分区都有大量的数据，第一种方案会造成大量数据的冗余。
2. 2.虽然第二种方案的网络延迟会比较高，但网络延迟对Kafka的影响较小。

#### ISR

采用第二种方案之后，设想以下情景：leader收到数据，所有follower都开始同步数据，但有一个follower，因为某种故障，迟迟不能与leader进行同步，那leader就要一直等下去，直到它完成同步，才能发送ack。这个问题怎么解决呢？Leader维护了一个动态的in-sync-replicaset(ISR)，意为和leader保持同步的follower集合。当ISR中的follower完成数据的同步之后，leader就会给follower发送ack。如果follower长时间未向leader同步数据，则该follower将被踢出ISR，该时间阈值由replica.lag.time.max.ms参数设定。Leader发生故障之后，就会从ISR中选举新的leader。

#### ack应答机制

对于某些不太重要的数据，对数据的可靠性要求不是很高，能够容忍数据的少量丢失，所以没必要等ISR中的follower全部接收成功。所以Kafka为用户提供了三种可靠性级别，用户根据对可靠性和延迟的要求进行权衡，选择以下的配置。acks参数配置：

acks：

- 0：producer不等待broker的ack，这一操作提供了一个最低的延迟，broke 一接收到还没有写入磁盘就已经返回，当broker故障时有可能丢失数据；
- 1：producer等待broker的ack，partition的leader落盘成功后返回ack，如果在follower同步成功之前leader 故障，那么将会丢失数据；
- -1（all）：producer等待broker的ack，partition的leader和follower全部落盘成功后才返回ack。但是如果在follower同步完成后，broker发送ack之前，leader发生故障，那么会造成数据重复。（acks=all再某种极限场合也会丢数据，比如：ISR里只剩下leader这一个broker了，当leader完成同步后由于ISR里面没有其他的follower了，此时leader会发送ack给producer；发送完后leader宕机了，由于其他不在ISR里面的follower还没有跟leader同步完数据，于是就发生了数据丢失）

#### 故障处理细节

LEO(Log End Offset)：每个副本的最后一个offset，即每个副本最大的offset；
HW(High Watermark) ：所有副本中最小的LEO，即消费者能见到的最大的offset，ISR队列中最小的LEO

设置HW的原因：主要是保证副本之前数据一致性。比如ISR里面有三个broker，leader中最大的offset是19，其他两个follower中最大的offset分别是12和15，如果没有设置HW那么原来leader中所有的数据对于消费者都是可见的，当消费者消费到offset=17时leader宕机了，然后新选举出来的leader中只有offset<=15的数据，消费者从新的leader中拉取offset=17之后的数据就会报错。

1. follower故障：follower发生故障后会被临时踢出ISR，待该follower恢复后，follower会读取本地磁盘记录的上次的HW，并将log文件高于HW的部分截取掉，从HW开始向leader进行同步。
2. leader发生故障之后，会从ISR中选出一个新的leader之后，为保证多个副本之间的数据一致性，其余的follower会先将各自的log文件高于HW的部分截掉，然后从新的leader同步数据。

### Exactly Once语义

将服务器的ACK级别设置为-1，可以保证Producer到Server之间不会丢失数据，即AtLeast Once语义。相对的，将服务器ACK级别设置为0，可以保证生产者每条消息只会被发送一次，即AtMost Once语义。

At Least Once可以保证数据不丢失，但是不能保证数据不重复；相对的，At Least Once可以保证数据不重复，但是不能保证数据不丢失。但是，对于一些非常重要的信息，比如说交易数据，下游数据消费者要求数据既不重复也不丢失，即Exactly Once语义。在0.11版本以前的Kafka，对此是无能为力的，只能保证数据不丢失，再在下游消费者对数据做全局去重。对于多个下游应用的情况，每个都需要单独做全局去重，这就对性能造成了很大影响。

0.11版本的Kafka，引入了一项重大特性：幂等性。所谓的幂等性就是指Producer不论向Server发送多少次重复的数据，Server端都只会持久化一条。幂等性再结合At Least Once语义，就构成了Kafka的Exactly Once语义。即：At Least Once + 幂等性 = Exactly Once

要启用幂等性，只需要将Producer的参数中enable.idompotence设置为true即可。Kafka的幂等性实现其实就是将原来下游需要做的去重放在了数据上游。开启幂等性的Producer在初始化的时候会被分配一个PID，发往同一Partition的消息会附带SequenceNumber。而Broker端会对<PID,Partition,SeqNumber>做缓存，当具有相同主键的消息提交时，Broker只会持久化一条。

但是PID重启就会变化，同时不同的Partition也具有不同主键，所以幂等性无法保证跨分区跨会话的Exactly Once。

## Kafka消费者

### 消费方式

consumer采用pull（拉）模式从broker中读取数据。consumer采用pull（拉）模式从broker中读取数据。push（推）模式很难适应消费速率不同的消费者，因为消息发送速率是由broker决定的。它的目标是尽可能以最快速度传递消息，但是这样很容易造成consumer来不及处理消息，典型的表现就是拒绝服务以及网络拥塞。而pull模式则可以根据consumer的消费能力以适当的速率消费消息。pull模式不足之处是，如果kafka没有数据，消费者可能会陷入循环中，一直返回空数据。针对这一点，Kafka的消费者在消费数据时会传入一个时长参数timeout，如果当前没有数据可供消费，consumer会等待一段时间之后再返回，这段时长即为timeout。

### 分区分配策略

一个consumergroup中有多个consumer，一个topic有多个partition，所以必然会涉及到partition的分配问题，即确定那个partition由哪个consumer来消费。Kafka有两种分配策略，一是RoundRobin，一是Range。

1. RoundRobin

   按组来划分（前提：必须保证消费者组里面的消费者订阅的主题完全一致）如果不一致会出现下面的问题

   consumer group里面有consumer-0，consumer-1

   topic-0里面有partition-0，partition-1，partition-2
   topic-1里面有partition-0，partition-1，partition-2

   然后consumer-0订阅topic-0；consumer-1订阅topic-1

   如果采用RoundRobin进行分配的话，那么它会将consumer-0和consumer-1订阅的主题topic-0和topic-1里面的所有partition全部当作一个整理来轮询分配：

   那么consumer-0分到的就是：topic-0的partition-0，partition-2和topic-1的partition-1

   那么consumer-1分到的就是：topic-0的partition-1和topic-1的partition-0，partition-2

2. Range

   以主题划分（按照主题划分的时候，它是先要找到谁订阅了这个主题，然后再考虑组，他不会将分区划分给组里面没有订阅他的消费者）

   实例1：

   consumer group里面有consumer-0，consumer-1

   topic-0里面有partition-0，partition-1，partition-2
   topic-1里面有partition-0，partition-1，partition-2

   consumer-0和consumer-1都订阅了topic-0和topic-1

   如果采用Range进行分配的话，由于Range以topic划分，所以先会看topic-0被哪几个consumer订阅了，上述的例子中topic-0被2个consumer订阅于是将topic-0的partition数除以consumer的数量（然后将partition-0，partition-1分配给consumer-0；partition-2分给了consumer-1），对于topic-1也是执行同样的处理；最后的分配结果如下：

   consumer-0：partition-0(opic-0)，partition-1(opic-0)，partition-0(opic-1)，partition-1(opic-1)
   consumer-1：partition-2(opic-0)，partition-2(opic-1):

   实例2：

   topic-0里面有partition-0，partition-1，partition-2
   topic-1里面有partition-0，partition-1，partition-2
   consumer-group-0里面有consumer-a和consumer-b；
   consumer-group-1里面有consumer-c；
   consumer-a、consumer-b、consumer-c都订阅了topic-0；consumer-b还订阅了topic-1；
   如果按照RoundRobin策略划分的话，那会先把consumer-group-0组所订阅的所有topic：topic-0和topic-1里面的所有的partition当作一个整体，然后轮询分配到组里面的两个消费者：consumer-a和consumer-b，那就可能出现将topic-1里面的partition分配给consumer-a的情况；
   如果按照Range划分的话，首先它先找到有哪些消费者订阅了topic-0，它发现有consumer-group-0的consumer-a和consumer-b和consumer-group-1的consumer-c订阅了topic-0，接着他会根据range算法把topic-0的partition-0，partition-1分配给consumer-a，topic-0的partition-2分配给consumer-b，然后把topic-0里面有partition-0，partition-1，partition-2都分配给consumer-group-1的consumer-c；然后找到有哪些消费者订阅了topic-1，它发现只有consumer-group-0的consumer-b订阅了，于是他会将topic-1的所有分区全部划分给consumer-c

### offset的维护

由于consumer在消费过程中可能会出现断电宕机等故障，consumer恢复后，需要从故障前的位置的继续消费，所以consumer需要实时记录自己消费到了哪个offset，以便故障恢复后继续消费。Kafka0.9版本之前，consumer默认将offset保存在Zookeeper中，从0.9版本开始，consumer默认将offset保存在Kafka一个内置的topic中，该topic为__consumer_offsets。(消费者组+主题+分区确定一个offset)

读取__consumer_offsets中的offset

1. 修改配置文件 consumer.properties

   exclude.internal.topics=false

2. 读取 offset

   0.11.0.0 之前版本:

   bin/kafka-console-consumer.sh --topic __consumer_offsets --zookeeper server01:2181 --formatter "kafka.coordinator.GroupMetadataManager\$OffsetsMessageFormatter" --consumer.config config/consumer.properties --from-beginning

   0.11.0.0 之后版本(含):

   bin/kafka-console-consumer.sh --topic __consumer_offsets --zookeeper server01:2181 --formatter "kafka.coordinator.group.GroupMetadataManager\$OffsetsMessageFormatter" --consumer.config config/consumer.properties --from-beginning

