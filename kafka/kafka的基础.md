# [补充](url)
# 1.kafka概述
## 1. 消息队列
### 1. 传统消息队列的应用场景
### 2. 消息队列的2种模式
- 点对点模式：
```1对1，消费者主动拉取消息，消息收到后，消息清除。```
消息生产者生产消息，发送到Queue中，虽然可以对应一个或者多个消费者，但是只要有一个消费者消费后，就会将该消息删除，其他消费者是不能进行消费的(```一个消息唯一对应一个消费者，自我理解```)
![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/3bde9b22284241b380bb6068cba6671c~tplv-k3u1fbpfcp-watermark.image)
- 发布/订阅模式
```1对多,消费者消费消息后不会清除数据。```消息生产者将消息发送到topic中，会有多个消费者（订阅）消费该消息。(```消息发布到了topic中，订阅该topic的所有组都能收到或拉取该消息进行消费，自我理解```)
![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/7ebf1a9cef5247bd93d08fae70a229b5~tplv-k3u1fbpfcp-watermark.image)
## 2. kafka的定义
kafka是一种基于发布/订阅模式的消息队列，主要应用于大数据实时处理领域。
## 3. kafka的基础架构
- 为了方便扩展，提高吞吐量，一个topic分为多个partition
- 配合分区设计，提出消费组概念，组内每个消费者并行消费(```单个消息只能被组内的一个消费者消费```)
- 为了提高可用性，为每个partition增加若干个副本
![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/413188520e16465484c78da87e3624a2~tplv-k3u1fbpfcp-watermark.image)
1. producer：消息生产者，向kafka broker发消息的客户端
2. consumer：消息消费者，向kafka broker取消息的客户端
3. consumer group：消费者组，由多个consumer组成，```消费组内的消费者负责消费不同分区的消息，一个分区只能由一个消费者消费```，消费者组互不影响，所有的消费者都属于某个消费者组，即消费者组是逻辑上的一个订阅者。
4. broker：一台kafka服务器就是一个broker，一个集群由多个broker组成，一个broker可以容纳很多的topic
5. topic：可以理解为一个队列，```生产者和消费者面向的都是一个topic```
6. partition：为了实现扩展性，一个大的topic可以分布在不同的broker上(```分布在不同broker上的partition就是组成这个topic```)，每个partition都是一个有序的队列
7. replica：副本，为了保证集群发生故障时，```该节点的partition数据不会丢失且kafka正常工作，kafka提供了副本机制```，一个topic的每个partition都有若干的副本，一个leader和若干follower。
8. leader:每个分区多个副本的主，生产者发送数据对象，消费者消费数据的对象都是leader
9. follower：每个分区多个副本的从，实时从kafka中同步数据，保持和leader数据的同步，某个leader发生故障时，某个follower会成为leader。
# 2. kafka安装相关
1. [kafka的安装](https://juejin.cn/post/6933748378534674445/)
2. [kafka监控安装](https://juejin.cn/post/6932820870201245709)
# 3. kafka命令行操作
## 1）查看当前服务器中的所有topic
```shell
[mayi@mayi101 kafka]$ bin/kafka-topics.sh --zookeeper mayi101:2181/kafka --list
```
## 2）创建topic
```shell
[mayi@mayi101 kafka]$ bin/kafka-topics.sh --zookeeper mayi:2181/kafka \
--create --replication-factor 3 --partitions 1 --topic first
选项说明：
--topic 定义topic名
--replication-factor  定义副本数
--partitions  定义分区数
```
## 3）删除topic
```shell
[mayi@mayi kafka]$ bin/kafka-topics.sh --zookeeper mayi101:2181/kafka \
--delete --topic first
需要server.properties中设置delete.topic.enable=true否则只是标记删除。
```
## 4）发送消息
```shell
[mayi@mayi101 kafka]$ bin/kafka-console-producer.sh \
--broker-list mayi101:9092 --topic first
>hello world
```
## 5）消费消息
```shell
[mayi@mayi101 kafka]$ bin/kafka-console-consumer.sh \
--bootstrap-server mayi101:9092 --from-beginning --topic first

[mayi@mayi101 kafka]$ bin/kafka-console-consumer.sh \
--bootstrap-server mayi101:9092 --from-beginning --topic first
--from-beginning：会把主题中以往所有的数据都读取出来。
```
## 6）查看某个Topic的详情
```shell
[mayi@mayi101 kafka]$ bin/kafka-topics.sh --zookeeper mayi101:2181/kafka \
--describe --topic first
```
## 7）修改分区数
```shell
[mayi@mayi101 kafka]$bin/kafka-topics.sh --zookeeper mayi101:2181/kafka --alter --topic first --partitions 6
```
# 4. kafka深入
 [kafka深入理解，可以参考](https://blog.csdn.net/lingbo229/article/details/80761778)
## 1. kafka工作流程
![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/24590c7b30cc4ae98d7aeedc38718371~tplv-k3u1fbpfcp-watermark.image)
- Kafka中消息是以topic进行分类的，生产者生产消息，消费者消费消息，都是面向topic的。
- topic是逻辑上的概念，而partition是物理上的概念，每个partition对应于一个log文件，该log文件中存储的就是producer生产的数据。Producer生产的数据会被不断追加到该log文件末端，且每条数据都有自己的offset。```消费者组中的每个消费者，都会实时记录自己消费到了哪个offset，以便出错恢复时，从上次的位置继续消费```。
## 2. kafka文件存储机制
[kafka存储参考](https://cloud.tencent.com/developer/article/1586720)
![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b650f598596542b990655f5a857a0fa7~tplv-k3u1fbpfcp-watermark.image)
- 由于生产者生产的消息会不断追加到log文件末尾，为防止log文件过大导致数据定位效率低下，Kafka采取了```分片```和```索引```机制，将每个partition分为多个segment。每个segment对应两个文件——“.index”文件和“.log”文件。这些文件位于一个文件夹下，该文件夹的命名规则为：topic名称+分区序号。例如，first这个topic有三个分区，则其对应的文件夹为first-0,first-1,first-2。
```
-rw-rw-r--. 1 mayi mayi 10485760 2月  26 11:03 00000000000000000000.index
-rw-rw-r--. 1 mayi mayi        0 2月  23 14:38 00000000000000000000.log
-rw-rw-r--. 1 mayi mayi 10485756 2月  26 11:03 00000000000000000000.timeindex
```
### index和log文件详解
![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/340cabd6147c4d39a723e16f2a29cca4~tplv-k3u1fbpfcp-watermark.image)
- Index:index文件是消息的物理地址的索引文件。
- Log：是真正的消息内容。
- timeindex:它是映射时间戳和相对offset
- snapshot:记录了producer的事务信息
- leader-epoch-checkpoint:保存了每一任leader开始写入消息时的offset, 会定时更新.

# 5. kafka生产者
## 1. 分区策略
### 1）分区的原因
- （1）```方便在集群中扩展```，每个Partition可以通过调整以适应它所在的机器，而一个topic又可以有多个Partition组成，因此整个集群就可以适应任意大小的数据了；
- （2）```可以提高并发```，因为可以以Partition为单位读写了。
### 2）分区的原则
我们需要将producer发送的数据封装成一个ProducerRecord对象。
![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/545155f9fd794729a44c7e1a8823d946~tplv-k3u1fbpfcp-watermark.image)
- （1）指明 partition 的情况下，直接将指明的值直接作为 partiton 值；
- （2）没有指明 partition 值但有 key 的情况下，将 key 的 hash 值与 topic 的 partition 数进行取余得到 partition 值；
- （3）既没有 partition 值又没有 key 值的情况下，第一次调用时随机生成一个整数（后面每次调用在这个整数上自增），将这个值与 topic 可用的 partition 总数取余得到 partition 值，也就是常说的 round-robin 算法。
## 2. ```数据可靠性保证```
为保证producer发送的数据，能可靠的发送到指定的topic，```topic的每个partition收到producer发送的数据后，都需要向producer发送ack（acknowledgement确认收到）```，如果producer收到ack，就会进行下一轮的发送，否则重新发送数据。
![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/65b7ec0bfc9341119d77a415061887ba~tplv-k3u1fbpfcp-watermark.image)
### 副本同步策略，2种方案
![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/2235626442cf44b4871bea167044ab99~tplv-k3u1fbpfcp-watermark.image)
### ISR(副本同步集)
- 采用第二种方案之后，设想以下情景：leader收到数据，所有follower都开始同步数据，但有一个follower，因为某种故障，迟迟不能与leader进行同步，那leader就要一直等下去，直到它完成同步，才能发送ack。这个问题怎么解决呢？
- Leader维护了一个动态的in-sync replica set (ISR)，意为和```leader保持同步的follower集合```。当ISR中的follower完成数据的同步之后，leader就会给producer发送ack。如果follower长时间未向leader同步数据，则该follower将被踢出ISR，该时间阈值由**replica.lag.time.max.ms**参数设定。Leader发生故障之后，就会从ISR中选举新的leader。
### ack应答机制
acks参数配置

acks:
- 0:producer不等待broker的ack，这一操作提供了一个最低的延迟，broker一接收到还没有写入磁盘就已经返回，当broker故障时有可能```丢失数据```
- 1:producer等待broker的ack，partition的leader落盘成功后返回ack，如果在follower同步成功之前leader故障，那么将会```丢失数据```；
acks:1 丢失数据案例
![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/41ef3c6453b7425484075f5718322d6e~tplv-k3u1fbpfcp-watermark.image)
- -1（all）：producer等待broker的ack，partition的leader和follower全部落盘成功后才返回ack。但是如果在follower同步完成后，broker发送ack之前，leader发生故障，那么会造成```数据重复。```
acks:-1 数据重复案例
![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/71890063e9f4423681c2c7c267a434af~tplv-k3u1fbpfcp-watermark.image)

**总结**：
```只有leader故障才会发生重复或数据丢失问题```
- 数据丢失：```响应了ack + 数据未同步到follower => 数据丢失```
1. ```acks:0``` 	leader成功响应ACK后，leader故障了，```未同步到follower```
2. ```acks:1```	leader本地落盘后响应了ack，```未同步就故障了```
- 数据重复：```数据同步了 + 没有响应ack(producer再次发送) => 数据重复```
3. ```acks:-1```	leader同步数据后，```来不及响应ack，leader故障了```，producer再次发数据，自然就重复了
### 发生故障kafka处理细节
log文件中的HW和LEO
![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/12a8e42ff5af4f8397e36a12bb7dce81~tplv-k3u1fbpfcp-watermark.image)
- follower故障
follower发生故障后会被临时踢出ISR，待该follower恢复后，follower会读取本地磁盘记录的上次的HW，并将log文件高于HW的部分截取掉，从HW开始向leader进行同步。等该follower的LEO大于等于该Partition的HW，即follower追上leader之后，就可以重新加入ISR了。
- leader故障
leader发生故障之后，会从ISR中选出一个新的leader，之后，为保证多个副本之间的数据一致性，其余的follower会先将各自的log文件高于HW的部分截掉，然后从新的leader同步数据。

注意：这只能保证副本之间的数据一致性，并不能保证数据不丢失或者不重复。
###  Exactly Once语义
- 将服务器的ACK级别设置为-1，可以保证Producer到Server之间不会丢失数据，即At Least Once语义。相对的，将服务器ACK级别设置为0，可以保证生产者每条消息只会被发送一次，即At Most Once语义。
  
- At Least Once可以保证数据不丢失，但是不能保证数据不重复；相对的，At Least Once可以保证数据不重复，但是不能保证数据不丢失。但是，对于一些非常重要的信息，比如说交易数据，下游数据消费者要求数据既不重复也不丢失，即Exactly Once语义。在0.11版本以前的Kafka，对此是无能为力的，只能保证数据不丢失，再在下游消费者对数据做全局去重。对于多个下游应用的情况，每个都需要单独做全局去重，这就对性能造成了很大影响。
- 0.11版本的Kafka，引入了一项重大特性：幂等性。所谓的幂等性就是指Producer不论向Server发送多少次重复数据，Server端都只会持久化一条。幂等性结合At Least Once语义，就构成了Kafka的Exactly Once语义。即：
- At Least Once + 幂等性 = Exactly Once
- 要启用幂等性，只需要将Producer的参数中```enable.idompotence设置为true即可```。Kafka的幂等性实现其实就是将原来下游需要做的去重放在了数据上游。开启幂等性的Producer在初始化的时候会被分配一个PID，发往同一Partition的消息会附带Sequence Number。而Broker端会对<PID, Partition, SeqNumber>做缓存，当具有相同主键的消息提交时，Broker只会持久化一条。
- 但是PID重启就会变化，同时不同的Partition也具有不同主键，所以幂等性无法保证跨分区跨会话的Exactly Once。
## 6. kafka消费者
### 消费方式
- consumer采用pull（拉）模式从broker中读取数据。
- ```push（推）模式很难适应消费速率不同的消费者，因为消息发送速率是由broker决定的。```它的目标是尽可能以最快速度传递消息，但是这样很容易造成consumer来不及处理消息，典型的表现就是拒绝服务以及网络拥塞。而pull模式则可以根据consumer的消费能力以适当的速率消费消息。
- ```pull模式不足之处是，如果kafka没有数据，消费者可能会陷入循环中，一直返回空数据。```针对这一点，Kafka的消费者在消费数据时会传入一个时长参数timeout，如果当前没有数据可供消费，consumer会等待一段时间之后再返回，这段时长即为timeout。
### 分区分配策略
一个consumer group中有多个consumer，一个 topic有多个partition，所以必然会涉及到partition的分配问题，即确定那个partition由哪个consumer来消费。
Kafka有两种分配策略，一是```round-robin```轮询，一是```range```分区。
### offset维护
- 由于consumer在消费过程中可能会出现断电宕机等故障，consumer恢复后，需要从故障前的位置的继续消费，所以consumer需要实时记录自己消费到了哪个offset，以便故障恢复后继续消费。
- Kafka 0.9版本之前，consumer默认将offset保存在Zookeeper中，从0.9版本开始，consumer默认将offset保存在Kafka一个内置的topic中，该topic为__consumer_offsets。
## kafka高效读写数据
1. 顺序读写磁盘
```Kafka的producer生产数据，要写入到log文件中，写的过程是一直追加到文件末端，为顺序写。```官网有数据表明，同样的磁盘，顺序写能到到600M/s，而随机写只有100k/s。这与磁盘的机械机构有关，顺序写之所以快，是因为其省去了大量磁头寻址的时间。

2. 应用pagecache
Kafka数据持久化是直接持久化到Pagecache中，这样会产生以下几个好处： 

- I/O Scheduler 会将连续的小块写组装成大块的物理写从而提高性能
- I/O Scheduler 会尝试将一些写操作重新按顺序排好，从而减少磁盘头的移动时间
- 充分利用所有空闲内存（非 JVM 内存）。如果使用应用层 Cache（即 JVM 堆内存），会增加 GC 负担
- 读操作可直接在 Page Cache 内进行。如果消费和生产速度相当，甚至不需要通过物理磁盘（直接通过 Page Cache）交换数据
- 如果进程重启，JVM 内的 Cache 会失效，但 Page Cache 仍然可用

 尽管持久化到Pagecache上可能会造成宕机丢失数据的情况，但这可以被Kafka的Replication机制解决。如果为了保证这种情况下数据不丢失而强制将 Page Cache 中的数据 Flush 到磁盘，反而会降低性能。

3. 零拷贝技术
![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/22fdd8729f3a4dc9a025c840167fbb8a~tplv-k3u1fbpfcp-watermark.image)

## 7.zookeeper在kafka中的作用
Kafka集群中有一个broker会被选举为```Controller```，```负责管理集群broker的上下线，所有topic的分区副本分配和leader选举等工作。```

Controller的管理工作都是依赖于Zookeeper的。

## 8.kafka事务
Kafka从0.11版本开始引入了事务支持。事务可以保证Kafka在Exactly Once语义的基础上，生产和消费可以跨分区和会话，要么全部成功，要么全部失败。
### 1. producer事务
- 为了实现跨分区跨会话的事务，需要引入一个全局唯一的Transaction ID，并将Producer获得的PID和Transaction ID绑定。这样当Producer重启后就可以通过正在进行的Transaction ID获得原来的PID。
- 为了管理Transaction，Kafka引入了一个新的组件Transaction Coordinator。Producer就是通过和Transaction Coordinator交互获得Transaction ID对应的任务状态。Transaction Coordinator还负责将事务所有写入Kafka的一个内部Topic，这样即使整个服务重启，由于事务状态得到保存，进行中的事务状态可以得到恢复，从而继续进行。
### 2. consumer事务
- 上述事务机制主要是从Producer方面考虑，对于Consumer而言，事务的保证就会相对较弱，尤其时无法保证Commit的信息被精确消费。这是由于Consumer可以通过offset访问任意信息，而且不同的Segment File生命周期不同，同一事务的消息可能会出现重启后被删除的情况。
- 如果想完成Consumer端的精准一次性消费，那么需要kafka消费端将消费过程和提交offset过程做原子绑定。此时我们需要将kafka的offset保存到支持事务的自定义介质中（比如mysql）
# 6. kafka API
## 1. producer API
### 消息发送流程
Kafka的Producer发送消息采用的是```异步发送```的方式。在消息发送的过程中，涉及到了两个线程——```main线程```和```Sender线程```，以及一个线程共享变量——```RecordAccumulator```。main线程将消息发送给RecordAccumulator，Sender线程不断从RecordAccumulator中拉取消息发送到Kafka broker。

kafka消息发送流程
![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c39e4a09c9af423e8062fbe9a4565f97~tplv-k3u1fbpfcp-watermark.image)
相关参数：
- batch.size：只有数据积累到batch.size之后，sender才会发送数据。
- linger.ms：如果数据迟迟未达到batch.size，sender等待linger.time之后就会发送数据。
### 异步发送
导入依赖
```xml
<dependency>
  <groupId>org.apache.kafka</groupId>
  <artifactId>kafka-clients</artifactId>
  <version>2.4.1</version>
</dependency>
```
编写代码：
- KafkaProducer：需要创建一个生产者对象，用来发送数据
- ProducerConfig：获取所需的一系列配置参数
- ProducerRecord：每条数据都要封装成一个ProducerRecord对象

#### （1）不带回调函数的API
```java
import org.apache.kafka.clients.producer.*;

import java.util.Properties;
import java.util.concurrent.ExecutionException;

public class CustomProducer {

    public static void main(String[] args) throws ExecutionException, InterruptedException {
        Properties props = new Properties();
        props.put("bootstrap.servers", "hadoop102:9092");//kafka集群，broker-list
        props.put("acks", "all");
        props.put("retries", 1);//重试次数
        props.put("batch.size", 16384);//批次大小
        props.put("linger.ms", 1);//等待时间
        props.put("buffer.memory", 33554432);//RecordAccumulator缓冲区大小
        props.put("key.serializer", "org.apache.kafka.common.serialization.StringSerializer");
        props.put("value.serializer", "org.apache.kafka.common.serialization.StringSerializer");

        Producer<String, String> producer = new KafkaProducer<>(props);
        for (int i = 0; i < 100; i++) {
            producer.send(new ProducerRecord<String, String>("first", Integer.toString(i), Integer.toString(i)));
        }
        producer.close();
    }
}
```
#### （2）带回调函数的API
回调函数会在producer收到ack时调用，为异步调用，该方法有两个参数，分别是RecordMetadata和Exception，如果Exception为null，说明消息发送成功，如果Exception不为null，说明消息发送失败。

注意：消息发送失败会自动重试，不需要我们在回调函数中手动重试。
```java
import org.apache.kafka.clients.producer.*;

import java.util.Properties;
import java.util.concurrent.ExecutionException;

public class CustomProducer {

    public static void main(String[] args) throws ExecutionException, InterruptedException {
        Properties props = new Properties();
        props.put("bootstrap.servers", "hadoop102:9092");//kafka集群，broker-list
        props.put("acks", "all");
        props.put("retries", 1);//重试次数
        props.put("batch.size", 16384);//批次大小
        props.put("linger.ms", 1);//等待时间
        props.put("buffer.memory", 33554432);//RecordAccumulator缓冲区大小
        props.put("key.serializer", "org.apache.kafka.common.serialization.StringSerializer");
        props.put("value.serializer", "org.apache.kafka.common.serialization.StringSerializer");

        Producer<String, String> producer = new KafkaProducer<>(props);
        for (int i = 0; i < 100; i++) {
            producer.send(new ProducerRecord<String, String>("first", Integer.toString(i), Integer.toString(i)), new Callback() {

                //回调函数，该方法会在Producer收到ack时调用，为异步调用
                @Override
                public void onCompletion(RecordMetadata metadata, Exception exception) {
                    if (exception == null) {
                        System.out.println("success->" + metadata.offset());
                    } else {
                        exception.printStackTrace();
                    }
                }
            });
        }
        producer.close();
    }
}
```

### 同步发送API
```同步发送的意思就是，一条消息发送之后，会阻塞当前线程，直至返回ack。```
由于send方法返回的是一个Future对象，根据Futrue对象的特点，我们也可以实现同步发送的效果，只需在调用Future对象的```get方法```即可。
```java
import org.apache.kafka.clients.producer.KafkaProducer;
import org.apache.kafka.clients.producer.Producer;
import org.apache.kafka.clients.producer.ProducerRecord;

import java.util.Properties;
import java.util.concurrent.ExecutionException;

public class CustomProducer {

    public static void main(String[] args) throws ExecutionException, InterruptedException {
        Properties props = new Properties();
        props.put("bootstrap.servers", "hadoop102:9092");//kafka集群，broker-list
        props.put("acks", "all");
        props.put("retries", 1);//重试次数
        props.put("batch.size", 16384);//批次大小
        props.put("linger.ms", 1);//等待时间
        props.put("buffer.memory", 33554432);//RecordAccumulator缓冲区大小
        props.put("key.serializer", "org.apache.kafka.common.serialization.StringSerializer");
        props.put("value.serializer", "org.apache.kafka.common.serialization.StringSerializer");

        Producer<String, String> producer = new KafkaProducer<>(props);
        for (int i = 0; i < 100; i++) {
            producer.send(new ProducerRecord<String, String>("first", Integer.toString(i), Integer.toString(i))).get();
        }
        producer.close();
    }
}
```

## 2. consumer API
- Consumer消费数据时的可靠性是很容易保证的，因为数据在Kafka中是持久化的，故不用担心数据丢失问题。
- 由于consumer在消费过程中可能会出现断电宕机等故障，consumer恢复后，需要从故障前的位置的继续消费，所以consumer需要实时记录自己消费到了哪个offset，以便故障恢复后继续消费。所以```offset的维护是Consumer消费数据是必须考虑的问题。```
### 自动提交offset
导入依赖
```xml
<dependency>
  <groupId>org.apache.kafka</groupId>
  <artifactId>kafka-clients</artifactId>
  <version>2.4.1</version>
</dependency>
```
编写代码
- KafkaConsumer：需要创建一个消费者对象，用来消费数据
- ConsumerConfig：获取所需的一系列配置参数
- ConsuemrRecord：每条数据都要封装成一个ConsumerRecord对象
为了使我们能够专注于自己的业务逻辑，Kafka提供了自动提交offset的功能。 
自动提交offset的相关参数：
```properties
enable.auto.commit：是否开启自动提交offset功能
auto.commit.interval.ms：自动提交offset的时间间隔
```
自动提交offset的代码
```java
import org.apache.kafka.clients.consumer.ConsumerRecord;
import org.apache.kafka.clients.consumer.ConsumerRecords;
import org.apache.kafka.clients.consumer.KafkaConsumer;

import java.util.Arrays;
import java.util.Properties;

public class CustomConsumer {

    public static void main(String[] args) {
        Properties props = new Properties();
        props.put("bootstrap.servers", "hadoop102:9092");
        props.put("group.id", "test");
        props.put("enable.auto.commit", "true");
        props.put("auto.commit.interval.ms", "1000");
        props.put("key.deserializer", "org.apache.kafka.common.serialization.StringDeserializer");
        props.put("value.deserializer", "org.apache.kafka.common.serialization.StringDeserializer");
        KafkaConsumer<String, String> consumer = new KafkaConsumer<>(props);
        consumer.subscribe(Arrays.asList("first"));
        while (true) {
            ConsumerRecords<String, String> records = consumer.poll(100);
            for (ConsumerRecord<String, String> record : records)
                System.out.printf("offset = %d, key = %s, value = %s%n", record.offset(), record.key(), record.value());
        }
    }
}
```
### 手动提交offset
- 虽然自动提交offset十分简介便利，但由于其是基于时间提交的，开发人员难以把握offset提交的时机。因此Kafka还提供了手动提交offset的API。
- 手动提交offset的方法有两种：分别是```commitSync（同步提交）```和```commitAsync（异步提交）```。两者的相同点是，都会将本次poll的一批数据最高的偏移量提交；不同点是，commitSync阻塞当前线程，一直到提交成功，并且会自动失败重试（由不可控因素导致，也会出现提交失败）；而commitAsync则没有失败重试机制，故有可能提交失败。

同步提交代码示例
```java
import org.apache.kafka.clients.consumer.ConsumerRecord;
import org.apache.kafka.clients.consumer.ConsumerRecords;
import org.apache.kafka.clients.consumer.KafkaConsumer;

import java.util.Arrays;
import java.util.Properties;

/**
 * @author liubo
 */
public class CustomComsumer {

    public static void main(String[] args) {

        Properties props = new Properties();
        props.put("bootstrap.servers", "hadoop102:9092");//Kafka集群
        props.put("group.id", "test");//消费者组，只要group.id相同，就属于同一个消费者组
        props.put("enable.auto.commit", "false");//关闭自动提交offset
        props.put("key.deserializer", "org.apache.kafka.common.serialization.StringDeserializer");
        props.put("value.deserializer", "org.apache.kafka.common.serialization.StringDeserializer");

        KafkaConsumer<String, String> consumer = new KafkaConsumer<>(props);
        consumer.subscribe(Arrays.asList("first"));//消费者订阅主题

        while (true) {
            ConsumerRecords<String, String> records = consumer.poll(100);//消费者拉取数据
            for (ConsumerRecord<String, String> record : records) {
                System.out.printf("offset = %d, key = %s, value = %s%n", record.offset(), record.key(), record.value());
            }
            consumer.commitSync();//同步提交，当前线程会阻塞知道offset提交成功
        }
    }
}
```
### 异步提交offset
虽然同步提交offset更可靠一些，但是由于其会阻塞当前线程，直到提交成功。因此吞吐量会收到很大的影响。因此更多的情况下，会选用异步提交offset的方式
```java
import org.apache.kafka.clients.consumer.*;
import org.apache.kafka.common.TopicPartition;

import java.util.Arrays;
import java.util.Map;
import java.util.Properties;

/**
 * @author liubo
 */
public class CustomConsumer {

    public static void main(String[] args) {

        Properties props = new Properties();
        props.put("bootstrap.servers", "hadoop102:9092");//Kafka集群
        props.put("group.id", "test");//消费者组，只要group.id相同，就属于同一个消费者组
        props.put("enable.auto.commit", "false");//关闭自动提交offset
        props.put("key.deserializer", "org.apache.kafka.common.serialization.StringDeserializer");
        props.put("value.deserializer", "org.apache.kafka.common.serialization.StringDeserializer");

        KafkaConsumer<String, String> consumer = new KafkaConsumer<>(props);
        consumer.subscribe(Arrays.asList("first"));//消费者订阅主题

        while (true) {
            ConsumerRecords<String, String> records = consumer.poll(100);//消费者拉取数据
            for (ConsumerRecord<String, String> record : records) {
                System.out.printf("offset = %d, key = %s, value = %s%n", record.offset(), record.key(), record.value());
            }
            consumer.commitAsync(new OffsetCommitCallback() {
                @Override
                public void onComplete(Map<TopicPartition, OffsetAndMetadata> offsets, Exception exception) {
                    if (exception != null) {
                        System.err.println("Commit failed for" + offsets);
                    }
                }
            });//异步提交
        }
    }
}
```

### 数据漏消费和重复消费分析

无论是同步提交还是异步提交offset，都有可能会造成数据的漏消费或者重复消费。先提交offset后消费，有可能造成数据的漏消费；而先消费后提交offset，有可能会造成数据的重复消费。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/39c7dc06c6074c96945c7332c2e7d487~tplv-k3u1fbpfcp-watermark.image)