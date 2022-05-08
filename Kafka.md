---
title: Kafka
categories:
  - Kafka
tags:
  - Kafka
top_img: 'https://cdn.jsdelivr.net/gh/839777408/tupian/img/wp7.jpg'
cover: 'https://cdn.jsdelivr.net/gh/839777408/tupian/img/wp7.jpg'
abbrlink: 13749
date: 2022-02-25 18:09:36
updated: 2022-3-19 23:56:42
---

> 参考视频：【尚硅谷】2022版Kafka3.x教程 https://www.bilibili.com/video/BV1vr4y1677k

# 概述

## 定义

Kafka传统定义：Kafka是一个**分布式**的基于**发布/订阅模式**的**消息队列**（Message Queue），主要应用于大数据实时处理领域。

发布/订阅：消息的发布者不会将消息直接发送给特定的订阅者，而是**将发布的消息分为不同的类别**，订阅者**只接收感兴趣的消息**。

Kafka最新定义： Kafka是一个开源的**分布式事件流平台**（Event Streaming Platform），被数千家公司用于高性能**数据管道、流分析、数据集成和关键任务应用**。

## 应用场景

**缓冲/消峰**：有助于控制和优化数据流经过系统的速度，解决生产消息和消费消息的处理速度不一致的情况

**解耦：**允许你独立的扩展或修改两边的处理过程，只要确保它们遵守同样的接口约束。

**异步通信：**允许用户把一个消息放入队列，但并不立即处理它，然后在需要的时候再去处理它们。

## 消息队列的两种模式

- 点对点模式（一个生产者，一个消费者）
  - 消费者主动拉取数据，消息收到后清除消息

![](https://cdn.jsdelivr.net/gh/839777408/tupian/blog2/20220225185206.png)

- 发布订阅模式（一个生产者，多个消费者）
  - 可以有多个topic主题（浏览、点赞、收藏、评论等）
  - 消费者消费数据之后，不删除数据
  - 每个消费者相互独立，都可以消费到数据

![](https://cdn.jsdelivr.net/gh/839777408/tupian/blog2/20220225185325.png)

## 基础架构

![](https://cdn.jsdelivr.net/gh/839777408/tupian/blog2/20220226105508.png)

- Broker：代理；一台Kafka服务器就是一个broker。一个集群由多个broker组成。一个broker可以容纳多个 topic。 
- Topic：主题；可以理解为一个队列，**生产者和消费者面向的都是一个topic**。
- Producer：消息生产者；就是向 Kafka broker 发消息的客户端。 

- Consumer：消息消费者；向 Kafka broker 取消息的客户端。 
- Partition：分区；为了实现扩展性和提高吞吐量，一个非常大的 topic 可以分布到多个 broker（即服务器）上，一个 topic 可以分为多个 partition，每个 partition 是一个**有序**的队列。 

- Consumer Group（CG）：消费者组；由多个 consumer 组成。消费者组内每个消费者负责消费不同分区的数据，**一个分区只能由组内一个消费者消费，组内每个消费者并行消费**；**消费者组之间互不影响**。所有的消费者都属于某个消费者组，即**消费者组是逻辑上的一个订阅者**。 

- Replica：副本。一个 topic 的每个分区都有若干个副本，一个 Leader 和若干个Follower。 

- Leader：每个分区多个副本的“主”，生产者发送数据的对象，以及消费者消费数据的对象都是 Leader。 

- Follower：每个分区多个副本中的“从”，实时从 Leader 中同步数据，保持和Leader 数据的同步。Leader 发生故障时，某个 Follower 会成为新的 Leader，保证服务的可用性和数据一致性。
- Offset：消息在对应 Topic 中的偏移量

>注：
>
>- Replica（Leader\Follower）对应的是分区 Partition，Partition(Partition-0\Partition-1)对应的是Topic
>
>- Zookeeper 中会记录整个集群中那些 broker 可用/上线【/brokers/ids[0,1,2]】，也会记录每一个 partition 中的 leader 信息【/brokers/topics/主题名称/partition/0/state】
>
>- 2.8.0 之后也可以不配置 Zookeeper 作为注册中心，未来的趋势也是去 Zookeeper  化

# 入门

## 安装部署

集群规划：

| 服务器名称 | hadoop102 | hadoop103 | hadoop104 |
| ---------- | --------- | --------- | --------- |
| 依赖的服务 | ZooKeeper | ZooKeeper | ZooKeeper |
| 服务       | Kafka     | Kafka     | Kafka     |

官网下载链接，选择Binary downloads： https://kafka.apache.org/downloads.html

Kafka是由Scala语言编写，[kafka_2.12-3.1.0.tgz](https://www.apache.org/dyn/closer.cgi?path=/kafka/3.1.0/kafka_2.12-3.1.0.tgz)代表是使用scala 2.12 版本编写的3.1.0版本的Kafka

```shell
#上传至服务器并解压
[root@nanzx opt]# tar -zxvf kafka_2.12-3.1.0.tgz -C /opt
#修改解压后的文件名称
[root@nanzx opt]# mv kafka_2.12-3.1.0/ kafka
```

进入config目录修改配置文件 server.properties（主要是broker.id/log.dirs/zookeeper.connect）：

```properties
#broker的全局唯一编号，不能重复，只能是数字。
broker.id=0
#处理网络请求的线程数量
num.network.threads=3
#用来处理磁盘 IO 的线程数量
num.io.threads=8
#发送套接字的缓冲区大小
socket.send.buffer.bytes=102400
#接收套接字的缓冲区大小
socket.receive.buffer.bytes=102400
#请求套接字的缓冲区大小
socket.request.max.bytes=104857600
#kafka运行日志(数据)存放的路径，路径不需要提前创建，kafka自动帮你创建，默认是/temp路径下，会被自动清理
#可以配置多个磁盘路径，路径与路径之间可以用"，"分隔，
log.dirs=/opt/kafka/datas
#topic 在当前 broker 上的分区个数
num.partitions=1
#用来恢复和清理 data 下数据的线程数量
num.recovery.threads.per.data.dir=1
# 每个 topic 创建时的副本数，默认是 1 个副本
offsets.topic.replication.factor=1
#segment文件保留的最长时间，超时将被删除
log.retention.hours=168
#每个 segment 文件的大小，默认最大 1G
log.segment.bytes=1073741824
#检查过期数据的时间，默认 5 分钟检查一次是否数据过期
log.retention.check.interval.ms=300000
#配置连接 Zookeeper 集群地址（在 zk 根目录下创建/kafka，方便管理）
zookeeper.connect=hadoop102:2181,hadoop103:2181,hadoop104:2181/kafka
```

配置环境变量`vi /etc/profile`：

```properties
#KAFKA_HOME
export KAFKA_HOME=/opt/kafka
export PATH=$PATH:$KAFKA_HOME/bin
```

使环境变量生效：`source /etc/profile`

分发安装包以及环境变量文件，通过【[xsync同步脚本](https://blog.csdn.net/nalw2012/article/details/98322637)】将文件分发到其他服务器上，注意修改broker.id，一个集群环境中不能重复。

## 启动和停止

- 先启动 Zookeeper 集群，然后启动 Kafka
  - ` zk.sh start `

+ 单节点启动Kafka，需依次在 hadoop102、hadoop103、hadoop104 节点上执行：

  + -daemon 表示守护线程启动，再指定一个配置文件的路径
  + `bin/kafka-server-start.sh -daemon config/server.properties`

+ 单节点关闭Kafka，需依次在 hadoop102、hadoop103、hadoop104 节点上执行：

  -  `bin/kafka-server-stop.sh`

+ 可以写个shell脚本一键启动/关闭集群服务，如果使用 ZK，那么在启动 kafka 之前一定要先启动 ZK：

  ```shell
  #! /bin/bash
  case $1 in
  "start")
  	for i in hadoop102 hadoop103 hadoop104
    	do
    		echo "启动 $i kafka"
          ssh $i "/opt/kafka/bin/kafka-server-start.sh -daemon /opt/kafka/config/server.properties"
    	done
  ;;
  "stop")
  	for i in hadoop102 hadoop103 hadoop104
    	do
          echo "停止 $i kafka"
          ssh $i "/opt/kafka/bin/kafka-server-stop.sh"
    	done
  ;;
  esac
  ```

- 查看kafka是否启动成功：`jps`，一个显示当前所有java进程pid的命令

>**注意：**停止 Kafka 集群时，一定要等 Kafka 所有节点进程全部停止后再停止 Zookeeper集群。因为 Zookeeper 集群当中记录着 Kafka 集群相关信息，Zookeeper 集群一旦先停止，Kafka 集群就没有办法再获取停止进程的信息，只能手动杀死 Kafka 进程了。



## 命令行操作

 **主题命令行操作**

查看操作主题命令参数：`[root@nanzx kafka]# bin/kafka-topics.sh`

| 参数                                              | 描述                                   |
| ------------------------------------------------- | -------------------------------------- |
| --bootstrap-server <String: server toconnect to>  | 连接的 Kafka Broker 主机名称和端口号。 |
| --topic <String: topic>                           | 操作的 topic 名称。                    |
| --create                                          | 创建主题。                             |
| --delete                                          | 删除主题。                             |
| --alter                                           | 修改主题。                             |
| --list                                            | 查看所有主题。                         |
| --describe                                        | 查看主题详细描述。                     |
| --partitions <Integer: # of partitions>           | 设置分区数。                           |
| --replication-factor<Integer: replication factor> | 设置分区副本。                         |
| --config <String: name=value>                     | 更新系统默认的配置。                   |

```shell
#创建主题 
#--partitions 定义分区数
#--replication-factor 定义副本数
#--topic 定义 topic 名
[root@nanzx kafka]# bin/kafka-topics.sh --bootstrap-server localhost:9092 --create --partitions 1 --replication-factor 1 --topic firstTopic
Created topic firstTopic.

#查看所有主题
[root@nanzx kafka]# bin/kafka-topics.sh --bootstrap-server localhost:9092 --list

#查看主题的详情
[root@nanzx kafka]# bin/kafka-topics.sh --bootstrap-server localhost:9092 --describe --topic firstTopic
Topic: firstTopic	TopicId: tuPooPIlRq6nLJw4Yi-XkQ	PartitionCount: 1	ReplicationFactor: 1	Configs: segment.bytes=1073741824
	Topic: firstTopic	Partition: 0	Leader: 0	Replicas: 0	Isr: 0

#修改分区数，分区数只能增加，不能减少
[root@nanzx kafka]# bin/kafka-topics.sh --bootstrap-server localhost:9092 --alter --topic firstTopic  --partitions 3

#再次查看主题的详情
[root@nanzx kafka]# bin/kafka-topics.sh --bootstrap-server localhost:9092 --describe --topic firstTopic
Topic: firstTopic	TopicId: tuPooPIlRq6nLJw4Yi-XkQ	PartitionCount: 3	ReplicationFactor: 1	Configs: segment.bytes=1073741824
	Topic: firstTopic	Partition: 0	Leader: 0	Replicas: 0	Isr: 0
	Topic: firstTopic	Partition: 1	Leader: 0	Replicas: 0	Isr: 0
	Topic: firstTopic	Partition: 2	Leader: 0	Replicas: 0	Isr: 0

#删除主题	
[root@nanzx kafka]# bin/kafka-topics.sh --bootstrap-server localhost:9092 --delete --topic firstTopic
```

---

**生产者命令行操作**

查看操作生产者命令参数：`bin/kafka-console-producer.sh`

| 参数                                             | 描述                                   |
| ------------------------------------------------ | -------------------------------------- |
| --bootstrap-server <String: server toconnect to> | 连接的 Kafka Broker 主机名称和端口号。 |
| --topic <String: topic>                          | 操作的 topic 名称。                    |

```shell
#发送消息
[root@nanzx kafka]# bin/kafka-console-producer.sh --bootstrap-server localhost:9092 --topic firstTopic
>hello nan
>nanzx.top
```

---

**消费者命令行操作**

查看操作消费者命令参数：`bin/kafka-console-consumer.sh`

| 参数                                             | 描述                                   |
| ------------------------------------------------ | -------------------------------------- |
| --bootstrap-server <String: server toconnect to> | 连接的 Kafka Broker 主机名称和端口号。 |
| --topic <String: topic>                          | 操作的 topic 名称。                    |
| --from-beginning                                 | 从头开始消费。                         |
| --group <String: consumer group id>              | 指定消费者组名称。                     |

```shell
#消费主题中的数据, --from-beginning可以把主题中所有的数据都读取出来（包括历史数据）
#[root@nanzx kafka]# bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 --from-beginning --topic firstTopic
hello nan
nanzx.top
```

# Kafka 生产者

## 消息发送流程

在消息发送的过程中，涉及到了两个线程：**main 线程和 sender 线程**。在 main 线程中创建了一个**双端队列 RecordAccumulator**（默认 32 m，由内存池分配空间）。main 线程将消息发送给 RecordAccumulator，sender 线程不断从 RecordAccumulator 中拉取消息发送到 Kafka Broker。

![](https://cdn.jsdelivr.net/gh/839777408/tupian/blog2/20220226192238.png)

**main线程：**

1. 在 main 线程中创建 Producer 对象，Producer 调用 send 方法，经过拦截器（可选项，建议不用），拦截器可以对数据进行加工包装（例：Flume 拦截器）

2. 随后经过序列化器，对数据进行传输前的[序列化](https://blog.csdn.net/tree_ifconfig/article/details/82766587?ops_request_misc=%257B%2522request%255Fid%2522%253A%2522164587532316780271917661%2522%252C%2522scm%2522%253A%252220140713.130102334.pc%255Fall.%2522%257D&request_id=164587532316780271917661&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~all~first_rank_ecpm_v1~rank_v31_ecpm-4-82766587.pc_search_result_cache&utm_term=%E5%BA%8F%E5%88%97%E5%8C%96&spm=1018.2226.3001.4187)操作，转换为可存储和传输的字节序列。不使用Java序列化器是因为会携带大量辅助和保证安全传输的数据，在大数据场景下不适用。

3. 接着是分区器，分区器会判断要将数据发送到哪一个分区，并将数据（每16K一批）放到不同分区对应的DQuene（一个分区创建一个DQuene，方便数据的管理）中

**sender 线程：**

1. 当RecordAccumulator中的数据积累到`batch.size`（默认16k）或者数据迟迟未达到 `batch.size`，而Sender 已经等待 `linger.time` （默认是0ms，表示没有延迟）之后 Sender 线程就会从缓存双端队列中拉取并发送数据。
2. Sender 线程将拉取的数据封装成请求并放到 Broker 对应的发送队列中，发送到 Kafka 集群的某个节点，默认每个 broker 节点最多缓存5个请求。
3. 由 Selector 进行发送操作，kafka 集群收到数据之后就会同步到对应的**副本**当中，同时也会提供 ACK 应答机制：
   1. 0表示生产者发送过来的数据，不需要等数据落盘应答。
   2. 1表示生产者发送过来的数据，Leader收到数据后应答。
   3. -1或all表示生产者发送过来的数据，Leader和ISR队列里面的所有节点（可以理解成所有副本）收齐数据后应答。
4. Selector 反馈是成功时，清理 Broker 对应的发送队列中的请求，同时清理双端队列对应的分区数据，反馈是失败时，则会重新发送请求，默认次数是Int类型的最大值（无限，可以修改）

---

## 生产者重要参数列表

| 参数名称                              | 描述                                                         |
| ------------------------------------- | ------------------------------------------------------------ |
| bootstrap.servers                     | 生产者连接集群所需的broker地址清 单 。例如hadoop102:9092,hadoop103:9092,hadoop104:9092，可以中间用逗号隔开。注意这里并非需要所有的 broker 地址，因为生产者可以从给定的 broker里查找到其他 broker 信息。 |
| key.serializer 和 value.serializer    | 指定发送消息的 key 和 value 的序列化类型。一定要写全类名。   |
| buffer.memory                         | RecordAccumulator 缓冲区总大小，**默认32m**。                |
| batch.size                            | 缓冲区一批数据最大值，**默认16k**。适当增加该值，可以提高吞吐量，但是如果该值设置太大，会导致数据传输延迟增加。 |
| linger.ms                             | 如果数据迟迟未达到 batch.size，sender 等待 linger.time之后就会发送数据。单位ms，**默认值是 0ms**，表示没有延迟。生产环境建议该值大小为 5-100ms 之间。 |
| acks                                  | 0：生产者发送过来的数据，不需要等数据落盘应答。<br/>1：生产者发送过来的数据，Leader 收到数据后应答。<br/>-1（all）：生产者发送过来的数据，Leader+和 isr 队列里面的所有节点收齐数据后应答。<br/>**默认值是-1**，-1 和all 是等价的。 |
| max.in.flight.requests.per.connection | 允许最多没有返回 ack 的次数，**默认为 5**，开启幂等性要保证该值是 1-5 的数字。 |
| retries                               | 当消息发送出现错误的时候，系统会重发消息。retries表示重试次数。**默认是 int 最大值，**2147483647。<br/>如果设置了重试，还想保证消息的有序性，需要设置MAX_IN_FLIGHT_REQUESTS_PER_CONNECTION=1<br/>否则在重试此失败消息的时候，其他的消息可能发送成功了。 |
| retry.backoff.ms                      | 两次重试之间的时间间隔，默认是 100ms。                       |
| enable.idempotence                    | 是否开启幂等性，**默认 true，开启幂等性**。                  |
| compression.type                      | 生产者发送的所有数据的压缩方式。**默认是 none，也就是不压缩**。   支持压缩类型：none、gzip、snappy、lz4 和 zstd。 |

## 异步发送API

**普通异步发送：**

异步发送指的是外部数据发送到双端队列 RecordAccumulator，不需要等待前一批数据成功发送到kafka后才发送。

![](https://cdn.jsdelivr.net/gh/839777408/tupian/blog2/20220226192238.png)

引入kafka相关依赖：

```xml
        <dependency>
            <groupId>org.apache.kafka</groupId>
            <artifactId>kafka-clients</artifactId>
            <version>3.0.0</version>
        </dependency>
```

编写不带回调函数的 API 代码：

```java
    @Test
    void CustomProducer() {
        // 1. 创建 kafka 生产者的配置对象
        Properties properties = new Properties();

        // 2. 给 kafka 配置对象添加配置信息：bootstrap.servers
        properties.setProperty(ProducerConfig.BOOTSTRAP_SERVERS_CONFIG, "192.168.2.110:9092");
        // key,value 序列化（必须）
        properties.setProperty(ProducerConfig.KEY_SERIALIZER_CLASS_CONFIG, StringSerializer.class.getName());
        properties.setProperty(ProducerConfig.VALUE_SERIALIZER_CLASS_CONFIG, StringSerializer.class.getName());

        // 3. 创建 kafka 生产者对象
        KafkaProducer<String, String> producer = new KafkaProducer<String, String>(properties);

        // 4. 调用 send 方法,发送消息
        for (int i = 0; i < 5; i++) {
            producer.send(new ProducerRecord<>("firstTopic", "Test" + i));
        }

        // 5. 关闭资源
        producer.close();
    }
```

测试：在 IDEA 中执行代码，观察虚拟机控制台中是否接收到消息。

---

**带回调函数的异步发送：**

回调函数会在 producer 收到 ack （双端队列发送的）时调用，为异步调用，该方法有两个参数，分别是元数据信息（RecordMetadata）和异常信息（Exception），如果 Exception 为 null，说明消息发送成功，如果 Exception 不为 null，说明消息发送失败。

```java
 @Test
    void CustomProducerCallback() throws InterruptedException {
        // 1. 创建 kafka 生产者的配置对象
        Properties properties = new Properties();

        // 2. 给 kafka 配置对象添加配置信息：bootstrap.servers
        properties.setProperty(ProducerConfig.BOOTSTRAP_SERVERS_CONFIG, "192.168.2.110:9092");
        // key,value 序列化（必须）
        properties.setProperty(ProducerConfig.KEY_SERIALIZER_CLASS_CONFIG, StringSerializer.class.getName());
        properties.setProperty(ProducerConfig.VALUE_SERIALIZER_CLASS_CONFIG, StringSerializer.class.getName());

        // 3. 创建 kafka 生产者对象
        KafkaProducer<String, String> producer = new KafkaProducer<String, String>(properties);

        // 4. 调用 send 方法,发送消息
        for (int i = 0; i < 5; i++) {
            producer.send(new ProducerRecord<>("firstTopic", "Test" + i), new Callback() {
                @Override
                public void onCompletion(RecordMetadata recordMetadata, Exception exception) {
                    if (exception == null) {
                        // 没有异常,输出信息到控制台
                        System.out.println(" 主题： " + recordMetadata.topic() + "->" + "分区：" + recordMetadata.partition());
                    } else {
                        // 出现异常打印
                        exception.printStackTrace();
                    }
                }
            });

            // 延迟一会会看到数据发往不同分区
            Thread.sleep(2);
        }

        // 5. 关闭资源
        producer.close();
    }
```

## 同步发送API

同步发送指的是外部数据发送到双端队列 RecordAccumulator，需要等待前一批数据成功发送到kafka后才发送。

只需在异步发送的基础上，再调用一下 get()方法即可。

```java
    @Test
    void CustomProducerSync() throws ExecutionException, InterruptedException {
        // 1. 创建 kafka 生产者的配置对象
        Properties properties = new Properties();

        // 2. 给 kafka 配置对象添加配置信息：bootstrap.servers
        properties.setProperty(ProducerConfig.BOOTSTRAP_SERVERS_CONFIG, "192.168.2.110:9092");
        // key,value 序列化（必须）
        properties.setProperty(ProducerConfig.KEY_SERIALIZER_CLASS_CONFIG, StringSerializer.class.getName());
        properties.setProperty(ProducerConfig.VALUE_SERIALIZER_CLASS_CONFIG, StringSerializer.class.getName());

        // 3. 创建 kafka 生产者对象
        KafkaProducer<String, String> producer = new KafkaProducer<String, String>(properties);

        // 4. 调用 send 方法,发送消息
        for (int i = 0; i < 5; i++) {
            producer.send(new ProducerRecord<>("firstTopic", "Test" + i)).get();
        }

        // 5. 关闭资源
        producer.close();
    }
```

## 分区策略

Kafka分区的好处：

- **便于合理使用存储资源**，每个Partition在一个Broker上存储，可以把海量的数据按照分区切割成一块一块数据存储在多台Broker上。合理控制分区的任务，可以实现负载均衡的效果。 

- **提高并行度**，生产者可以以分区为单位发送数据；消费者可以以分区为单位进行消费数据，不同分区之间互不干扰。

---

KafkaProducer的默认的分区器是 **DefaultPartitioner**：

```java
/**
 * The default partitioning strategy:
 * <ul>
 * <li>If a partition is specified in the record, use it
 * <li>If no partition is specified but a key is present choose a partition based on a hash of the key
 * <li>If no partition or key is present choose the sticky partition that changes when the batch is full.
 * 
 * See KIP-480 for details about sticky partitioning.
 */
public class DefaultPartitioner implements Partitioner {

```

默认的分区策略是：

- 指明partition的情况下，**直接将指明的值作为partition值**；例如partition=0，所有数据写入分区0
- 没有指明partition值但有key的情况下，将key的hash值与topic的partition数进行**取余**得到partition值；
  - 例如：key1的hash值=5， key2的hash值=6 ，topic的partition数=2，那 么key1 对应的value1写入1号分区，key2对应的value2写入0号分区。
- 既没有partition值又没有key值的情况下，Kafka采用**Sticky Partition**（黏性分区器），会**随机**选择一个分区，并尽可能一直使用该分区，待该分区的batch已满或者已完成，Kafka再随机一个分区进行使用（和上一次的分区**不同**）。
  - 例如：第一次随机选择0号分区，等0号分区当前批次满了（默认16k）或者linger.ms设置的时间到， Kafka再随机一个分区进行使用（如果还是0会继续随机）。

查看send方法的参数**ProducerRecord的构造方法**：

```java
//指明partition的情况下，直接将指明的值作为partition值；例如partition=0，所有数据写入分区0
public ProducerRecord(String topic, Integer partition, Long timestamp, K key, V value, Iterable<Header> headers)
    
public ProducerRecord(String topic, Integer partition, Long timestamp, K key, V value)
    
public ProducerRecord(String topic, Integer partition, K key, V value, Iterable<Header> headers)
    
public ProducerRecord(String topic, Integer partition, K key, V value)

//没有指明partition值但有key的情况下，将key的hash值与topic的partition数进行取余得到partition值
public ProducerRecord(String topic, K key, V value)
 
//采用Sticky Partition（黏性分区器），会随机选择一个分区
public ProducerRecord(String topic, V value)
```

---

**自定义分区器：**

1. 定义类实现 Partitioner 接口。 

2. 重写 partition()方法。

3. 使用分区器的方法，在生产者的配置中添加分区器参数。`properties.setProperty(ProducerConfig.PARTITIONER_CLASS_CONFIG,"top.nanzx.kafka.producer.MyPartitioner");`

## 生产经验

### 提高吞吐量

```java
//根据需求选择合适参数：        
        // batch.size：批次大小，默认16K
        properties.put(ProducerConfig.BATCH_SIZE_CONFIG, 16384);
        // linger.ms：等待时间，默认0ms
        properties.put(ProducerConfig.LINGER_MS_CONFIG, 1);
        // buffer.memory：RecordAccumulator缓冲区大小，默认 32M
        properties.put(ProducerConfig.BUFFER_MEMORY_CONFIG,33554432);
        // compression.type：压缩，默认 none，可配置值 gzip、snappy、lz4 和 zstd
        properties.put(ProducerConfig.COMPRESSION_TYPE_CONFIG,"snappy");
```

### 数据可靠性（ACK应答机制）

> ACK应答级别：
>
> - 0：生产者发送过来的数据，不需要等数据落盘应答。**可靠性差，效率高；**
>
> - 1：生产者发送过来的数据，Leader收到数据后应答。**可靠性中等，效率中等；**
> - -1（all）：生产者发送过来的数据，Leader和ISR队列里面的所有节点收齐数据后应答。**可靠性高，效率低；**

**acks=0**，生产者发送过来数据就不管了：

![](https://cdn.jsdelivr.net/gh/839777408/tupian/blog2/20220301223217.png)

---

**acks=1**，生产者发送过来数据Leader应答：

![](https://cdn.jsdelivr.net/gh/839777408/tupian/blog2/20220301223231.png)

---

**acks=-1（all）**，生产者发送过来数据Leader和ISR队列里面所有Follwer应答：

![](https://cdn.jsdelivr.net/gh/839777408/tupian/blog2/20220301223931.png)

**思考**：Leader收到数据，所有Follower都开始同步数据，但有一个Follower，因为某种故障，迟迟不能与Leader进行同步，那这个问题怎么解决呢？

Leader维护了一个动态的**in-sync replica set（ISR）**：和Leader**保持同步**的Follower+Leader集合(leader:0,isr:0,1,2)。

如果Follower长时间未向Leader发送通信请求或同步数据，则该Follower将被踢出ISR。该时间阈值由`replica.lag.time.max.ms`参数设定，默认30s。例如2超时，(leader:0, isr:0,1)。

这样就不用等长期联系不上或者已经故障的节点。

**总结：**在生产环境中，acks=0 很少使用；acks=1 一般用于传输普通日志，允许丢个别数据；acks=-1，一般用于传输和钱相关的数据，对可靠性要求比较高的场景。

**数据可靠性分析：**

如果分区副本设置为1个，或者ISR里应答的最小副本数量（ min.insync.replicas 默认为1）设置为1，和ack=1的效果是一样的，仍然有丢数的风险（leader：0，isr：0）。

> **数据完全可靠条件 = ACK级别设置为-1 + 分区副本大于等于2 + ISR里应答的最小副本数量大于等于2**

---

### 数据去重（幂等性和事务）

**数据重复分析：**

acks： -1（all）：生产者发送过来的数据，Leader和ISR队列里面的所有节点收齐数据后应答。

Leader和ISR队列里面的所有节点收齐数据后Leader准备应答时挂了，Producer没有收到ack应答，于是重新给新Leader节点发送数据造成数据重复。



![](https://cdn.jsdelivr.net/gh/839777408/tupian/blog2/20220302201453.png)

**数据传递语义：**

• 至少一次（At Least Once）= ACK级别设置为-1 + 分区副本大于等于2 + ISR里应答的最小副本数量大于等于2 （数据完全可靠条件），可以保证数据不丢失，但是不能保证数据不重复

• 最多一次（At Most Once）= ACK级别设置为0 ，可以保证数据不重复，但是不能保证数据不丢失。

• 精确一次（Exactly Once）：对于一些非常重要的信息，比如和钱相关的数据，要求数据既不能重复也不丢失。

> Kafka 0.11版本以后，引入了一项重大特性：**幂等性**和**事务**。

---

**幂等性**

幂等性是指Producer不论向Broker发送多少次重复数据，Broker端都只会持久化一条，保证了不重复。

精确一次（Exactly Once） = 幂等性 + 至少一次（ ack=-1 + 分区副本数>=2 + ISR最小副本数量>=2） 。 

**重复数据的判断标准**：具有`<PID, Partition, SeqNumber>`相同主键的消息提交时，Broker只会持久化一条。其中PID是Kafka每次重启都会分配一个新的；Partition 表示分区号；Sequence Number是单调自增的。

所以幂等性只能保证的是在**单分区单会话**内不重复。

![](https://cdn.jsdelivr.net/gh/839777408/tupian/blog2/20220302204338.png)

**如何使用幂等性：**开启参数 **enable.idempotence** 默认为 true，false 关闭。

---

**事务**

幂等性只能保证单分区单会话，远远不够，跨分区跨会话仍会导致数据重复问题，因此引入事务。

**说明：开启事务，必须开启幂等性。**

![](https://cdn.jsdelivr.net/gh/839777408/tupian/blog2/20220302220620.png)

```java
    @Test
    void CustomProducerTransaction() {
        // 1. 创建 kafka 生产者的配置对象
        Properties properties = new Properties();

        // 2. 给 kafka 配置对象添加配置信息：bootstrap.servers
        properties.setProperty(ProducerConfig.BOOTSTRAP_SERVERS_CONFIG, "192.168.2.110:9092");
        // key,value 序列化（必须）
        properties.setProperty(ProducerConfig.KEY_SERIALIZER_CLASS_CONFIG, StringSerializer.class.getName());
        properties.setProperty(ProducerConfig.VALUE_SERIALIZER_CLASS_CONFIG, StringSerializer.class.getName());

        // 设置事务 id（必须），事务 id 任意起名
        properties.put(ProducerConfig.TRANSACTIONAL_ID_CONFIG, "transaction_id_0");

        // 3. 创建 kafka 生产者对象
        KafkaProducer<String, String> producer = new KafkaProducer<String, String>(properties);

        // 初始化事务
        producer.initTransactions();
        // 开启事务
        producer.beginTransaction();
        try {
            // 4. 调用 send 方法,发送消息
            for (int i = 0; i < 5; i++) {
                producer.send(new ProducerRecord<>("firstTopic", "Test" + i));
            }
            // 提交事务
            producer.commitTransaction();
        } catch (Exception e) {
            // 终止事务
            producer.abortTransaction();
        } finally {
            // 5. 关闭资源
            producer.close();
        }
    }
```

---

### 数据有序

**单分区内有序**，但多分区时，分区与分区间无序是无序的。

为了保证多分区时数据有序，有两种解决方案：

- 生产者端统一采用**一个分区**发送请求
- 消费者端对请求重排序，保证数据的有序处理，但是效率低下，需要等待请求全部送达

kafka在1.x版本之前保证数据单分区有序，条件如下：

- `max.in.flight.requests.per.connection`=1（不需要考虑是否开启幂等性，因为只有一个请求）。 

kafka在1.x及以后版本保证数据单分区有序，条件如下：

- 开启幂等性：`max.in.flight.requests.per.connection`需要设置小于等于5。 

- 未开启幂等性：`max.in.flight.requests.per.connection`需要设置为1。

![](https://cdn.jsdelivr.net/gh/839777408/tupian/blog2/20220302221914.png)

原因说明：在kafka1.x以后，启用幂等后，kafka服务端会缓存producer发来的最近5个request的元数据并进行排序，故无论如何，都可以保证最近5个request的数据都是有序的。

---

# Kafka Broker

## Broker 工作流程

启动 Zookeeper 客户端，使用 ls 命令在kafka目录下（配置文件中连接ZK时的设置），可以查看 Zookeeper 存储的 Kafka 相关信息：

![](https://cdn.jsdelivr.net/gh/839777408/tupian/blog2/20220302233820.png)

- Zookeeper 中会记录整个集群中那些 broker 可用/上线【/brokers/ids[0,1,2]】，0 1 2表示broker的id，在kafka配置文件指定
- 也会记录每一个 partition 中的 leader 信息以及动态维护的**in-sync replica set（ISR）**（和Leader**保持同步**的Follower+Leader集合）【/brokers/topics/主题名称/partition/第几个分区/state】

---

 **Broker总体工作流程：**

![](https://cdn.jsdelivr.net/gh/839777408/tupian/blog2/20220302234808.png)

1. 每台 broker 启动后在 ZK 节点中注册
2. 每个broker都有各自的Controller模块，抢占 ZK 节点的controller ，谁先注册谁就是Controller Leader
3. 选举出来的 Controller 监听 brokers 节点变化
4. Controller 决定副本 Leader 的选举，以**在 ISR 中存活为前提**，按照 AR （Assigned Replica，Kafka分区中所有副本的统称）中**排在最前面**的优先。
5. 将Leader 副本信息和 ISR 信息上传到 ZK 中存储
6. 其他 Controller节点从 ZK 中同步消息，防止Controller Leader挂了，随时准备成为Leader
7. Producer向Leader发送信息，Follower主动向Leader同步信息。同步信息在底层以1G的Segmen的log文件配合index文件的方式进行存储。
8. 如果 Leader 所在的 broker 挂了，Controller Leader监听到 ZK 注册的节点发生了变化，会获取 ISR 再重新选举Leader 副本并更新 ZK 中的Leader信息以及 ISR（ISR为前提，**AR为先后顺序**）

## Broker 重要参数列表

| 参数名称                                | 描述                                                         |
| --------------------------------------- | ------------------------------------------------------------ |
| replica.lag.time.max.ms                 | ISR 中，如果 Follower 长时间未向 Leader 发送通信请求或同步数据，则该 Follower 将被踢出 ISR。该时间阈值默认是 **30s**。 |
| auto.leader.rebalance.enable            | 默认是 true。 自动 Leader Partition 平衡。                   |
| leader.imbalance.per.broker.percentage  | **默认是 10%**。每个 broker 允许的不平衡的 leader的比率。如果每个 broker 超过了这个值，控制器会触发 leader 的平衡。 |
| leader.imbalance.check.interval.seconds | **默认值 300 秒**。检查 leader 负载是否平衡的间隔时间。      |
| log.segment.bytes                       | Kafka 中 log 日志是分成一块块存储的，此配置是指 log 日志划分成块的大小，**默认值 1G**。 |
| log.index.interval.bytes                | **默认 4kb**，kafka 里面每当写入了 4kb 大小的日志（.log），然后就往 index 文件里面记录一个索引。 |
| log.retention.hours                     | Kafka 中数据保存的时间，**默认 7 天**。                      |
| log.retention.minutes                   | Kafka 中数据保存的时间，**分钟级别**，默认关闭。             |
| log.retention.ms                        | Kafka 中数据保存的时间，**毫秒级别**，默认关闭。             |
| log.retention.check.interval.ms         | 检查数据是否保存超时的间隔，**默认是 5 分钟**。              |
| log.retention.bytes                     | **默认等于-1，表示无穷大**。超过设置的所有日志总大小，删除最早的 segment。 |
| log.cleanup.policy                      | **默认是 delete**，表示所有数据启用删除策略；<br/>如果设置值为 compact，表示所有数据启用压缩策略。 |
| num.io.threads                          | 负责写磁盘的线程数，**默认是 8**。这个参数值占总核数的 50%   |
| num.replica.fetchers                    | 副本拉取线程数，这个参数占总核数的 50%的 1/3                 |
| num.network.threads                     | 数据传输线程数，**默认是 3**。这个参数占总核数的50%的 2/3 。 |
| log.flush.interval.messages             | 强制页缓存刷写到磁盘的条数，默认是 long 的最大值，9223372036854775807。一般不建议修改，交给系统自己管理。 |
| log.flush.interval.ms                   | 每隔多久刷数据到磁盘，默认是 null。一般不建议修改，交给系统自己管理。 |

## Kafka 副本

- Kafka 副本作用：提高数据可靠性。 

- Kafka 默认副本 1 个，生产环境一般配置为 2 个，保证数据可靠性；太多副本会增加磁盘存储空间，增加网络上数据传输，降低效率。

- Kafka 中副本分为：Leader 和 Follower。Kafka 生产者只会把数据发往 Leader，然后 Follower 找 Leader 进行同步数据。

- Kafka 分区中的所有副本统称为 AR（Assigned Repllicas）。

  - AR = ISR + OSR

  - **ISR**，表示和 Leader 保持同步的 Follower 集合。如果 Follower 长时间未向 Leader 发送通信请求或同步数据，则该 Follower 将被踢出 ISR。该时间阈值由 **replica.lag.time.max.ms**参数设定，默认 30s。Leader 发生故障之后，就会从 ISR 中选举新的 Leader。
  - **OSR**，表示 Follower 与 Leader 副本同步时，延迟过多的副本。

---

### Leader 和 Follower 故障处理细节

- Offset：消息在对应 Topic 中的偏移量

- LEO（Log End Offset）：每个副本的最后一个offset，LEO其实就是最新的offset + 1。

- HW（High Watermark）：所有副本中最小的LEO 。 

**Follower故障：**

（1） Follower发生故障后会被临时踢出ISR

（2） 这个期间Leader和Follower继续接收数据

（3）待该Follower恢复后，Follower会读取本地磁盘记录的上次的HW，并将log文件高于HW的部分截取掉，从HW开始向Leader进行同步。

（4）等该Follower的LEO大于等于该Partition的HW，即Follower追上Leader之后，就可以重新加入ISR了。

![](https://cdn.jsdelivr.net/gh/839777408/tupian/blog2/20220304233728.png)

**Leader故障：**

（1） Leader发生故障之后，会从ISR中选出一个新的Leader

（2）为保证多个副本之间的数据一致性，其余的Follower会先将各自的log文件高于HW的部分截掉，然后从新的Leader同步数据。

![](https://cdn.jsdelivr.net/gh/839777408/tupian/blog2/20220304233744.png)

**注意：**这只能保证副本之间的数据一致性，并不能保证数据不丢失或者不重复。

---

### 分区副本分配

如果 kafka 服务器只有 4 个节点，那么设置 kafka 的分区数大于服务器台数，在 kafka底层如何分配存储副本呢？

创建一个新的 topic，16 分区，3 个副本：

```shell
[nanzx kafka]$ bin/kafka-topics.sh --bootstrap-server hadoop102:9092 --create --partitions 16 --replication-factor 3 --topic second

[nanzx kafka]$ bin/kafka-topics.sh --bootstrap-server hadoop102:9092 --describe --topic second
Topic: second4 Partition: 0 Leader: 0 Replicas: 0,1,2 Isr: 0,1,2
Topic: second4 Partition: 1 Leader: 1 Replicas: 1,2,3 Isr: 1,2,3
Topic: second4 Partition: 2 Leader: 2 Replicas: 2,3,0 Isr: 2,3,0
Topic: second4 Partition: 3 Leader: 3 Replicas: 3,0,1 Isr: 3,0,1
Topic: second4 Partition: 4 Leader: 0 Replicas: 0,2,3 Isr: 0,2,3
Topic: second4 Partition: 5 Leader: 1 Replicas: 1,3,0 Isr: 1,3,0
Topic: second4 Partition: 6 Leader: 2 Replicas: 2,0,1 Isr: 2,0,1
Topic: second4 Partition: 7 Leader: 3 Replicas: 3,1,2 Isr: 3,1,2
Topic: second4 Partition: 8 Leader: 0 Replicas: 0,3,1 Isr: 0,3,1
Topic: second4 Partition: 9 Leader: 1 Replicas: 1,0,2 Isr: 1,0,2
Topic: second4 Partition: 10 Leader: 2 Replicas: 2,1,3 Isr: 2,1,3
Topic: second4 Partition: 11 Leader: 3 Replicas: 3,2,0 Isr: 3,2,0
Topic: second4 Partition: 12 Leader: 0 Replicas: 0,1,2 Isr: 0,1,2
Topic: second4 Partition: 13 Leader: 1 Replicas: 1,2,3 Isr: 1,2,3
Topic: second4 Partition: 14 Leader: 2 Replicas: 2,3,0 Isr: 2,3,0
Topic: second4 Partition: 15 Leader: 3 Replicas: 3,0,1 Isr: 3,0,1
```

![](https://cdn.jsdelivr.net/gh/839777408/tupian/blog2/20220305161930.png)

---

### 手动调整分区副本存储

- 当有节点服役和退役时，副本应如何重新分配（使用生成的副本存储计划）；

- 在生产环境中，每台服务器的配置和性能不一致，但是Kafka只会根据自己的代码规则创建对应的分区副本，就会导致个别服务器存储压力较大。所以需要手动调整分区副本的存储（自己指定副本存储计划）。
- 在生产环境当中，由于某个主题的重要等级需要提升，我们考虑增加副本。副本数的增加需要先制定计划，然后根据计划执行（自己指定副本存储计划）。

创建一个主题需要均衡的json文件：

```shell
[nanzx kafka]$ vim topics-to-move.json
{
 "topics": [
 	{"topic": "first"}
 ],
 "version": 1
}
```

生成一个负载均衡的计划（--broker-list是选择将副本分配到哪些broker上）：

```shell
[nanzx kafka]$ bin/kafka-reassign-partitions.sh --bootstrap-server 127.0.0.1:9092 --topics-to-move-json-file topics-to-move.json --broker-list "0,1,2,3" --generate
Current partition replica assignment
{"version":1,"partitions":[{"topic":"first","partition":0,"replicas":[0,2,1],"log_dirs":["any","any","any"]},{"topic":"first","partition":1,"replicas":[2,1,0],"log_dirs":["any","any","any"]},{"topic":"first","partition":2,"replicas":[1,0,2],"log_dirs":["any","any","any"]}]}
Proposed partition reassignment configuration
{"version":1,"partitions":[{"topic":"first","partition":0,"replicas":[2,3,0],"log_dirs":["any","any","any"]},{"topic":"first","partition":1,"replicas":[3,0,1],"log_dirs":["any","any","any"]},{"topic":"first","partition":2,"replicas":[0,1,2],"log_dirs":["any","any","any"]}]}
```

创建副本存储计划（使用生成的计划将所有副本存储在 broker0、broker1、broker2、broker3 中）

```shell
[nanzx kafka]$ vim increase-replication-factor.json
{"version":1,"partitions":[{"topic":"first","partition":0,"replicas":[2,3,0],"log_dirs":["any","any","any"]},{"topic":"first","partition":1,"replicas":[3,0,1],"log_dirs":["any","any","any"]},{"topic":"first","partition":2,"replicas":[0,1,2],"log_dirs":["any","any","any"]}]}
```

执行副本存储计划

```shell
[nanzx kafka]$ bin/kafka-reassign-partitions.sh --bootstrap-server 127.0.0.1:9092 --reassignment-json-file increase-replication-factor.json --execute
```

验证副本存储计划

```shell
[nanzx kafka]$ bin/kafka-reassign-partitions.sh --bootstrap-server 127.0.0.1:9092 --reassignment-json-file increase-replication-factor.json --verify
Status of partition reassignment:
Reassignment of partition first-0 is complete.
Reassignment of partition first-1 is complete.
Reassignment of partition first-2 is complete.
Clearing broker-level throttles on brokers 0,1,2,3
Clearing topic-level throttles on topic first
```

---

### Leader Partition自动平衡

正常情况下，Kafka本身会自动把Leader Partition均匀分散在各个机器上，来保证每台机器的读写吞吐量都是均匀的。但是如果某些broker宕机，会导致Leader Partition过于集中在其他少部分几台broker上，这会导致少数几台broker的读写请求压力过高，其他宕机的broker重启之后都是follower partition，读写请求很低，造成集群负载不均衡。

> • auto.leader.rebalance.enable，默认是true。自动 Leader Partition 平衡。
>
> • leader.imbalance.per.broker.percentage，默认是10%。每个broker允许的不平衡的leader的比率。如果每个broker超过了这个值，控制器会触发leader的平衡。
>
> • leader.imbalance.check.interval.seconds，默认值300秒。检查leader负载是否平衡的间隔时间。

下面拿一个主题举例说明，假设集群只有一个主题如下图所示：

![](https://cdn.jsdelivr.net/gh/839777408/tupian/blog2/20220305205620.png)

分区2的**AR优先副本是0节点**（第三行，replicas可以理解成AR），但是0节点却不是Leader节点，所以broker0节点的不平衡数加1，AR副本总数是4，所以broker0节点不平衡率为1/4>10%，需要再平衡。

broker2和broker3节点和broker0不平衡率一样，需要再平衡。Broker1的不平衡数为0，不需要再平衡。

## 文件存储机制

Topic是逻辑上的概念，而partition是物理上的概念，每个partition对应于一个log文件，该log文件中存储的就是Producer生产的数据。Producer生产的数据会被不断**追加**到该log文件末端，为防止log文件过大导致数据定位效率低，Kafka采取了**分片**和**索引**机制， 将**每个partition分为多个segment**。每个segment包括：“.index”文件、“.log”文件和.timeindex等文件。这些文件位于一个文件夹下，文件夹的命名规则为：topic名称+分区序号，例如：first-0。

![](https://cdn.jsdelivr.net/gh/839777408/tupian/blog2/20220306001210.png)

说明：index文件和log文件都是以当前segment的第一条消息的offset命名。

```shell
#查看的/opt/kafka/datas/first-1 （first-0、first-2）路径上的文件。
[nanzx first-1]$ ls
00000000000000000092.index
00000000000000000092.log
00000000000000000092.snapshot
00000000000000000092.timeindex
leader-epoch-checkpoint
partition.metadata

#cat查看会中文乱码
[nanzx first-1]$ cat 00000000000000000092.log 
\CYnF|©|©ÿÿÿÿÿÿÿÿÿÿÿÿÿÿ"hello world

#使用kafka的工具查看index文件
[nanzx first-1]$ kafka-run-class.sh kafka.tools.DumpLogSegments --files./00000000000000000000.index
Dumping ./00000000000000000000.index
offset: 3 position: 152

#使用kafka的工具查看log文件
[nanzx first-1]$ kafka-run-class.sh kafka.tools.DumpLogSegments 
--files ./00000000000000000000.log
Dumping datas/first-0/00000000000000000000.log
Starting offset: 0
baseOffset: 0 lastOffset: 1 count: 2 baseSequence: -1 lastSequence: -1 producerId: -1 
producerEpoch: -1 partitionLeaderEpoch: 0 isTransactional: false isControl: false position: 
0 CreateTime: 1636338440962 size: 75 magic: 2 compresscodec: none crc: 2745337109 isvalid: 
true
baseOffset: 2 lastOffset: 2 count: 1 baseSequence: -1 lastSequence: -1 producerId: -1 
producerEpoch: -1 partitionLeaderEpoch: 0 isTransactional: false isControl: false position: 
75 CreateTime: 1636351749089 size: 77 magic: 2 compresscodec: none crc: 273943004 isvalid: 
true
baseOffset: 3 lastOffset: 3 count: 1 baseSequence: -1 lastSequence: -1 producerId: -1 
producerEpoch: -1 partitionLeaderEpoch: 0 isTransactional: false isControl: false position: 
152 CreateTime: 1636351749119 size: 77 magic: 2 compresscodec: none crc: 106207379 isvalid: 
true
baseOffset: 4 lastOffset: 8 count: 5 baseSequence: -1 lastSequence: -1 producerId: -1 
producerEpoch: -1 partitionLeaderEpoch: 0 isTransactional: false isControl: false position: 
229 CreateTime: 1636353061435 size: 141 magic: 2 compresscodec: none crc: 157376877 isvalid: 
true
baseOffset: 9 lastOffset: 13 count: 5 baseSequence: -1 lastSequence: -1 producerId: -1 
producerEpoch: -1 partitionLeaderEpoch: 0 isTransactional: false isControl: false position: 
370 CreateTime: 1636353204051 size: 146 magic: 2 compresscodec: none crc: 4058582827 isvalid: 
true
```

![](https://cdn.jsdelivr.net/gh/839777408/tupian/blog2/20220306234956.png)

说明：日志存储参数配置

| 参数                     | 描述                                                         |
| ------------------------ | ------------------------------------------------------------ |
| log.segment.bytes        | Kafka 中 log 日志是分成一块块存储的，此配置是指 log 日志划分成块的大小，**默认值 1G**。 |
| log.index.interval.bytes | **默认 4kb**，kafka 里面每当写入了 4kb 大小的日志（.log），然后就往 index 文件里面记录一个索引。 稀疏索引。 |

## 文件清理策略

Kafka 中**默认的日志保存时间为 7 天**，可以通过调整如下参数修改保存时间。

- log.retention.hours，最低优先级小时，默认 7 天。 

- log.retention.minutes，分钟。 
- log.retention.ms，最高优先级毫秒。 
- log.retention.check.interval.ms，负责设置检查周期，默认 5 分钟。

那么日志一旦超过了设置的时间，怎么处理呢？Kafka 中提供的日志清理策略有 delete 和 compact 两种。 

**delete 日志删除**：将过期数据删除

log.cleanup.policy = delete，所有数据启用删除策略

- 基于时间：默认打开。以 segment 中所有记录中的**最大时间戳**作为该文件时间戳。如果一个 segment 中有一部分数据过期，一部分没有过期，则保留。

- 基于大小：默认关闭。超过设置的所有日志总大小，删除最早的 segment。log.retention.bytes，默认等于-1，表示无穷大。

**compact日志压缩**：对于相同key的不同value值，只保留最后一个版本。

 log.cleanup.policy = compact 所有数据启用压缩策略

![](https://cdn.jsdelivr.net/gh/839777408/tupian/blog2/20220307235402.png)



压缩后的offset可能是不连续的，比如上图中没有6，当从这些offset消费消息时，将会拿到比这个offset大的offset对应的消息，实际上会拿到offset为7的消息，并从这个位置开始消费。

这种策略只适合特殊场景，比如消息的key是用户ID，value是用户的资料，通过这种压缩策略，整个消息集里就保存了所有用户最新的资料。 

---

## 高效读写数据

- Kafka 本身是**分布式集群**，可以采用**分区**技术，并行度高

- 读数据采用**稀疏索引**，可以快速定位要消费的数据

- 顺序写磁盘
  - Kafka 的 producer 生产数据，要写入到 log 文件中，写的过程是一直**追加**到文件末端，为顺序写。官网有数据表明，同样的磁盘，顺序写能到 600M/s，而随机写只有 100K/s。这与磁盘的机械机构有关，顺序写之所以快，是因为其省去了大量磁头寻址的时间。

![](https://cdn.jsdelivr.net/gh/839777408/tupian/blog2/20220306235743.png)

- 页缓存 + 零拷贝技术

  - **零拷贝：**Kafka的数据加工处理操作交由Kafka生产者和Kafka消费者处理。Kafka Broker应用层不关心存储的数据，所以就**不用走应用层**，传输效率高。

  - **PageCache页缓存**：Kafka重度依赖底层操作系统提供的PageCache功 能。当上层有写操作时，操作系统只是将数据写入PageCache。当读操作发生时，先从PageCache中查找，如果找不到，再去磁盘中读取。实际上PageCache是把尽可能多的空闲内存都当做了磁盘缓存来使用。

![](https://cdn.jsdelivr.net/gh/839777408/tupian/blog2/20220307000053.png)

| 参数                        | 描述                                                         |
| --------------------------- | ------------------------------------------------------------ |
| log.flush.interval.messages | 强制页缓存刷写到磁盘的条数，默认是 long 的最大值，9223372036854775807。一般不建议修改，交给系统自己管理。 |
| log.flush.interval.ms       | 每隔多久，刷数据到磁盘，默认是 null。一般不建议修改，交给系统自己管理。 |

# Kafka 消费者

## Kafka 消费方式

- Push 模式：**队列推送消息给消费者**

1. - 缺陷：由broker决定消息发送速率，很难适应所有消费者的消费速率，导致 Consumer 来不及处理消息

- Pull 模式：Kafka采用这种方式。**消费者主动到队列中拉取**

1. - 缺陷：如果队列中没有数据，消费者可能会陷入循环中，一直返回空数据

2. - 改进：设定一个 timeout 参数并在消费者消费数据时由队列传给消费者，表示消费者在没有消息处理时等待一段时间后再来拉取
   - Pull 模式提高消费吞吐量：
     - 增加 Topic 分区数，并同时增加消费者数量
     - 提高每批次拉取的数据量，避免消费者资源浪费

## 消费者组

Consumer Group：消费者组，由多个consumer组成。形成一个消费者组的条件是所有消费者的**groupid相同**。 

- 消费者组内每个消费者负责消费不同分区的数据，**一个分区只能由组内一个消费者消费**。 

- **消费者组之间互不影响**。所有的消费者都属于某个消费者组，即消费者组是逻辑上的一个订阅者。

- 如果向消费组中添加更多的消费者，超过主题分区数量，则有一部分消费者就会**闲置**，不会接收任何消息。

---

### 消费者组初始化流程 

**coordinator**：辅助实现消费者组的初始化和分区的分配。每个消费者的offset由消费者提交到coordinator中保存。

coordinator节点选择 = groupid的hashcode值 % 50（ __consumer_offsets的分区数量）

例如： groupid的hashcode值 = 1，1% 50 = 1，那么__consumer_offsets 主题的1号分区在哪个broker上，就选择这个节点的coordinator作为这个消费者组的老大。消费者组下的所有的消费者提交offset的时候就往这个分区去提交offset。 

![](https://cdn.jsdelivr.net/gh/839777408/tupian/blog2/20220307235316.png)

### 消费者组详细消费流程

![](https://cdn.jsdelivr.net/gh/839777408/tupian/blog2/20220308000040.png)

## 消费者重要参数列表

| 参数名称                               | 描述                                                         |
| -------------------------------------- | ------------------------------------------------------------ |
| bootstrap.servers                      | 向 Kafka 集群建立初始连接用到的 host/port 列表。             |
| key.deserializer 和 value.deserializer | 指定接收消息的 key 和 value 的反序列化类型。一定要写全类名。 |
| group.id                               | 标记消费者所属的消费者组。                                   |
| enable.auto.commit                     | **默认值为 true**，消费者会自动周期性地向服务器提交偏移量。  |
| auto.commit.interval.ms                | 如果设置了 enable.auto.commit 的值为 true， 则该值定义了消费者偏移量向 Kafka 提交的频率，**默认 5s**。 |
| auto.offset.reset                      | 当 Kafka 中没有初始偏移量或当前偏移量在服务器中不存在<br/>（如，数据被删除了），该如何处理？ <br/>earliest：自动重置偏移量到最早的偏移量。 <br/> **latest：默认**，自动重置偏移量为最新的偏移量。 <br/> none：如果消费组原来的（previous）偏移量不存在，则向消费者抛异常。anything：向消费者抛异常。 |
| offsets.topic.num.partitions           | __consumer_offsets 的分区数，**默认是 50 个分区**。          |
| heartbeat.interval.ms                  | Kafka 消费者和 coordinator 之间的心跳时间，**默认 3s**。<br/>该条目的值必须小于 session.timeout.ms ，也不应该高于session.timeout.ms 的 1/3。 |
| session.timeout.ms                     | Kafka 消费者和 coordinator 之间连接超时时间，**默认 45s**。<br/>超过该值，该消费者被移除，消费者组执行再平衡。 |
| max.poll.interval.ms                   | 消费者处理消息的最大时长，**默认是 5 分钟**。<br/>超过该值，该消费者被移除，消费者组执行再平衡。 |
| fetch.min.bytes                        | **默认 1 个字节**。消费者获取服务器端一批消息最小的字节数。  |
| fetch.max.wait.ms                      | **默认 500ms**。如果没有从服务器端获取到一批数据的最小字节数。该时间到，仍然会返回数据。 |
| fetch.max.bytes                        | 默认 Default: 52428800（**50 m**）。消费者获取服务器端一批消息最大的字节数。如果服务器端一批次的数据大于该值（50m）仍然可以拉取回来这批数据，因此，这不是一个绝对最大值。<br/>一批次的大小受 message.max.bytes （broker config）or max.message.bytes （topic config）影响。 |
| max.poll.records                       | 一次 poll 拉取数据返回消息的最大条数，**默认是 500 条**。    |

## 消费者API

**注意：**在消费者 API 代码中必须配置消费者组 id。命令行启动消费者不填写消费者组id会被自动填写随机的消费者组 id。 

**订阅主题：**

```java
   @Test
    void CustomCustomer() {
        // 1.创建消费者的配置对象
        Properties properties = new Properties();
        // 2.给消费者配置对象添加参数
        properties.put(ConsumerConfig.BOOTSTRAP_SERVERS_CONFIG, "192.168.2.110:9092");
        // 配置序列化 必须
        properties.put(ConsumerConfig.KEY_DESERIALIZER_CLASS_CONFIG, StringDeserializer.class.getName());
        properties.put(ConsumerConfig.VALUE_DESERIALIZER_CLASS_CONFIG, StringDeserializer.class.getName());
        // 配置消费者组（组名任意起名） 必须
        properties.put(ConsumerConfig.GROUP_ID_CONFIG, "test");

        // 创建消费者对象
        KafkaConsumer<String, String> kafkaConsumer = new KafkaConsumer<>(properties);
        // 注册要消费的主题（可以消费多个主题）
        ArrayList<String> topics = new ArrayList<>();
        topics.add("first");
        kafkaConsumer.subscribe(topics);
        // 拉取数据打印
        while (true) {
            // 设置 1s 中消费一批数据
            ConsumerRecords<String, String> consumerRecords = kafkaConsumer.poll(Duration.ofSeconds(1));
            // 打印消费到的数据
            for (ConsumerRecord<String, String> consumerRecord : consumerRecords) {
                System.out.println(consumerRecord);
            }
        }
    }
```

**订阅分区：**

```java
    @Test
    void CustomConsumerPartition() {
        Properties properties = new Properties();

        properties.put(ConsumerConfig.BOOTSTRAP_SERVERS_CONFIG, "192.168.2.110:9092");
        // 配置序列化 必须
        properties.put(ConsumerConfig.KEY_DESERIALIZER_CLASS_CONFIG, StringDeserializer.class.getName());
        properties.put(ConsumerConfig.VALUE_DESERIALIZER_CLASS_CONFIG, StringDeserializer.class.getName());
        // 配置消费者组（必须），名字可以任意起
        properties.put(ConsumerConfig.GROUP_ID_CONFIG, "test");
        
        KafkaConsumer<String, String> kafkaConsumer = new KafkaConsumer<>(properties);
        // 消费某个主题的某个分区数据
        ArrayList<TopicPartition> topicPartitions = new ArrayList<>();
        topicPartitions.add(new TopicPartition("first", 0));
        kafkaConsumer.assign(topicPartitions);
        while (true) {
            ConsumerRecords<String, String> consumerRecords =
                    kafkaConsumer.poll(Duration.ofSeconds(1));
            for (ConsumerRecord<String, String> consumerRecord :
                    consumerRecords) {
                System.out.println(consumerRecord);
            }
        }
    }
```

## 分区的分配以及再平衡

一个消费者组中有多个consumer组成，一个 topic有多个partition组成，现在的问题是，到底由哪个consumer来消费哪个partition的数据。 

Kafka有四种主流的分区分配策略： Range、RoundRobin、Sticky、CooperativeSticky。可以通过配置参数`partition.assignment.strategy`，修改分区的分配策略。默认策略是**Range + CooperativeSticky**。Kafka可以同时使用多个分区分配策略。

当以下事件发生时，Kafka 将会进行一次**分区分配（也称为 Rebalance 重平衡）**：

- 同一个 Consumer Group 内新增消费者（组内消费者数量变化）
- 消费者离开当前所属的 Consumer Group，包括 shuts down 或 crashes
- 订阅的主题新增分区

```java
// 修改分区分配策略
properties.put(ConsumerConfig.PARTITION_ASSIGNMENT_STRATEGY_CONFIG, "org.apache.kafka.clients.consumer.RoundRobinAssignor");
```

###  Range 以及再平衡

**Range 分区策略原理**：

- Range 是对**每个 topic** 而言的。

- 首先对同一个 topic 里面的**分区按照序号进行排序**，并对**消费者按照字母顺序进行排序**。
  - 假如现在有 7 个分区，3 个消费者，排序后的分区将会是0,1,2,3,4,5,6；消费者排序完之后将会是C0,C1,C2。

- 通过 partitions数/consumer数来决定每个消费者应该消费几个分区。如果除不尽，那么前面几个消费者将会多消费 1 个分区。
  - 例如，7/3 = 2 余 1 ，除不尽，那么 消费者 C0 便会多消费 1 个分区。 8/3=2余2，除不尽，那么C0和C1分别多消费一个。

注意：如果只是针对 1 个 topic 而言，C0消费者多消费1个分区影响不是很大。但是如果有 N 多个 topic，那么针对每个 topic，消费者 C0都将多消费 1 个分区，topic越多，C0消 费的分区会比其他消费者明显多消费 N 个分区。 容易产生**数据倾斜**！

![](https://cdn.jsdelivr.net/gh/839777408/tupian/blog2/20220313235556.png)

**Range 分区分配再平衡案例：**

（1）停止掉 0 号消费者，快速重新发送消息观看结果（45s 以内，越快越好）。 

- 1 号消费者：消费到 3、4 号分区数据。 

- 2 号消费者：消费到 5、6 号分区数据。 

- 0 号消费者的任务会整体被分配到 1 号消费者或者 2 号消费者。

说明：0 号消费者挂掉后，消费者组需要按照超时时间 45s 来判断它是否退出，所以需要等待时间到了 45s 后，判断它真的退出就会把任务分配给其他 broker 执行。

（2）再次重新发送消息观看结果（45s 以后）。 

- 1 号消费者：消费到 0、1、2、3 号分区数据。 

- 2 号消费者：消费到 4、5、6 号分区数据。

说明：消费者 0 已经被踢出消费者组，所以重新按照 range 方式分配。

---

### RoundRobin 以及再平衡

**RoundRobin 分区策略原理：**

RoundRobin 针对集群中**所有Topic**而言。

RoundRobin 轮询分区策略，是把所有的 partition 和所有的consumer 都列出来，然后**按照 hashcode 进行排序**，最后通过**轮询算法**来分配 partition 给到各个消费者。

![](https://cdn.jsdelivr.net/gh/839777408/tupian/blog2/20220313235342.png)

**RoundRobin** **分区分配再平衡案例**

（1）停止掉 0 号消费者，快速重新发送消息观看结果（45s 以内，越快越好）。 

- 1 号消费者：消费到 1、4 号分区数据

- 2 号消费者：消费到 2、5 号分区数据

- 0 号消费者的任务会按照 RoundRobin 的方式，把数据轮询分成 0 、3 和 6 号分区数据，分别由 1 号消费者或者 2 号消费者消费。

说明：0 号消费者挂掉后，消费者组需要按照超时时间 45s 来判断它是否退出，所以需要等待，时间到了 45s 后，判断它真的退出就会把任务分配给其他 broker 执行。

（2）再次重新发送消息观看结果（45s 以后）。 

- 1 号消费者：消费到 0、2、4、6 号分区数据

- 2 号消费者：消费到 1、3、5 号分区数据

说明：消费者 0 已经被踢出消费者组，所以重新按照 RoundRobin 方式分配。

---

###  Sticky 以及再平衡

**粘性分区定义：**可以理解为分配的结果带有“粘性的”。即在执行一次新的分配之前，考虑上一次分配的结果，尽量少的调整分配的变动，可以节省大量的开销。

粘性分区是 Kafka 从 0.11.x 版本开始引入这种分配策略，首先会尽量均衡的放置分区到消费者上面，在同一消费者组内消费者出现问题的时候，会尽量保持原有分配的分区不变化。

**Sticky** **分区分配再平衡案例**

（1）停止掉 0 号消费者，快速重新发送消息观看结果（45s 以内，越快越好）。 

- 1 号消费者：消费到 2、5、3 号分区数据。 

- 2 号消费者：消费到 4、6 号分区数据。 

- 0 号消费者的任务会按照粘性规则，尽可能均衡的随机分成 0 和 1 号分区数据，分别由 1 号消费者或者 2 号消费者消费。

说明：0 号消费者挂掉后，消费者组需要按照超时时间 45s 来判断它是否退出，所以需要等待，时间到了 45s 后，判断它真的退出就会把任务分配给其他 broker 执行。

（2）再次重新发送消息观看结果（45s 以后）。 

- 1 号消费者：消费到 2、3、5 号分区数据。 

- 2 号消费者：消费到 0、1、4、6 号分区数据。

说明：消费者 0 已经被踢出消费者组，所以重新按照粘性方式分配。

---

## offset位移

![](https://cdn.jsdelivr.net/gh/839777408/tupian/blog2/20220314235740.png)

__consumer_offsets 主题里面采用 key 和 value 的方式存储数据。**key 是 group.id+topic+分区号**，value 就是当前 offset 的值。每隔一段时间，kafka 内部会对这个 topic 进行compact，也就是每个group.id+topic+分区号就保留最新数据。

**消费 offset 案例：**

在配置文件 config/consumer.properties 中添加配置 **exclude.internal.topics=false**，默认是 true，表示不能消费系统主题。为了查看该系统主题数据，所以该参数修改为 false。

采用命令行方式，创建一个新的 topic：

```shell
[nanzx kafka]$ bin/kafka-topics.sh --bootstrap-server 192.168.2.110:9092 --create --topic mytopic --partitions 2 --replication-factor 2
```

启动生产者往 mytopic 生产数据：

```shell
[nanzx kafka]$ bin/kafka-console-producer.sh --topic mytopic --bootstrap-server 192.168.2.110:9092
```

启动消费者消费 mytopic 数据：

> 注意：指定消费者组名称，更好观察数据存储位置（key 是 group.id+topic+分区号）。 

```shell
[nanzx kafka]$ bin/kafka-console-consumer.sh --bootstrap-server 192.168.2.110:9092 --topic atguigu --group test
```

查看消费者消费主题__consumer_offsets：

```shell
[nanzx kafka]$ bin/kafka-console-consumer.sh --topic __consumer_offsets --bootstrap-server 192.168.2.110:9092 --consumer.config config/consumer.properties --formatter 
"kafka.coordinator.group.GroupMetadataManager\$OffsetsMessageFormatter" --from-beginning
...
[test,mytopic,1]::OffsetAndMetadata(offset=7,leaderEpoch=Optional[0], metadata=,commitTimestamp=1622442520203, expireTimestamp=None)
[test,mytopic,0]::OffsetAndMetadata(offset=8, leaderEpoch=Optional[0], metadata=,commitTimestamp=1622442520203, expireTimestamp=None)
```

### 自动提交 offset

为了使我们能够专注于自己的业务逻辑，Kafka提供了自动提交offset的功能。

自动提交offset的相关参数：

- enable.auto.commit：是否开启自动提交offset功能，**默认是true**，消费者会周期性自动地向服务器提交偏移量。

- auto.commit.interval.ms：自动提交offset的时间间隔，默认是5s

![](https://cdn.jsdelivr.net/gh/839777408/tupian/blog2/20220315230324.png)

```java
// 是否自动提交 offset
properties.put(ConsumerConfig.ENABLE_AUTO_COMMIT_CONFIG, true);
// 提交 offset 的时间周期 1000ms，默认是5s
properties.put(ConsumerConfig.AUTO_COMMIT_INTERVAL_MS_CONFIG, 1000);
```

### 手动提交 offset

虽然自动提交offset十分简单便利，但由于其是基于时间提交的，开发人员难以把握offset提交的时机。因此Kafka还提供了手动提交offset的API。

手动提交offset的方法有两种：

- commitSync（同步提交）：必须等待offset提交完毕，再去消费下一批数据。

- commitAsync（异步提交） ：发送完提交offset请求后，就开始消费下一批数据了。

两者的相同点是，都会将本次提交的一批数据最高的偏移量提交；不同点是，同步提交阻塞当前线程，一直到提交成功，并且会自动失败重试（由不可控因素导致，也会出现提交失败）；而异步提交则没有失败重试机制，故有可能提交失败。 

**同步提交 offset：**由于同步提交 offset 有失败重试机制，故更加可靠，但是由于一直等待提交结果，提交的效率比较低。以下为同步提交 offset 的示例：

```java
    @Test
    void CustomConsumerByHandSync() {
        Properties properties = new Properties();

        properties.put(ConsumerConfig.BOOTSTRAP_SERVERS_CONFIG, "hadoop102:9092");
        // 配置序列化 必须
        properties.put(ConsumerConfig.KEY_DESERIALIZER_CLASS_CONFIG, StringDeserializer.class.getName());
        properties.put(ConsumerConfig.VALUE_DESERIALIZER_CLASS_CONFIG, StringDeserializer.class.getName());
        // 配置消费者组
        properties.put(ConsumerConfig.GROUP_ID_CONFIG, "test");
        // 是否自动提交 offset
        properties.put(ConsumerConfig.ENABLE_AUTO_COMMIT_CONFIG, false);
        // 创建 kafka 消费者
        KafkaConsumer<String, String> kafkaConsumer = new KafkaConsumer<>(properties);
        // 设置消费主题 形参是列表
        kafkaConsumer.subscribe(Collections.singletonList("first"));

        while (true) {
            // 读取消息
            ConsumerRecords<String, String> consumerRecords = kafkaConsumer.poll(Duration.ofSeconds(1));
            for (ConsumerRecord<String, String> consumerRecord : consumerRecords) {
                System.out.println(consumerRecord);
            }
            // 同步提交 offset
            kafkaConsumer.commitSync();
        }
    }
```

**异步提交 offset：**虽然同步提交 offset 更可靠一些，但是由于其会阻塞当前线程，直到提交成功。因此吞吐量会受到很大的影响。因此**更多的情况下，会选用异步提交** offset 的方式。以下为异步提交 offset 的示例：

```java
    @Test
    void CustomConsumerByHandAsync() {
        Properties properties = new Properties();

        properties.put(ConsumerConfig.BOOTSTRAP_SERVERS_CONFIG, "hadoop102:9092");
        // 配置序列化 必须
        properties.put(ConsumerConfig.KEY_DESERIALIZER_CLASS_CONFIG, StringDeserializer.class.getName());
        properties.put(ConsumerConfig.VALUE_DESERIALIZER_CLASS_CONFIG, StringDeserializer.class.getName());
        // 配置消费者组
        properties.put(ConsumerConfig.GROUP_ID_CONFIG, "test");
        // 是否自动提交 offset
        properties.put(ConsumerConfig.ENABLE_AUTO_COMMIT_CONFIG, false);
        // 创建 kafka 消费者
        KafkaConsumer<String, String> kafkaConsumer = new KafkaConsumer<>(properties);
        // 设置消费主题 形参是列表
        kafkaConsumer.subscribe(Collections.singletonList("first"));

        while (true) {
            // 读取消息
            ConsumerRecords<String, String> consumerRecords = kafkaConsumer.poll(Duration.ofSeconds(1));
            for (ConsumerRecord<String, String> consumerRecord : consumerRecords) {
                System.out.println(consumerRecord);
            }
            // 异步提交 offset
            kafkaConsumer.commitAsync();
        }
    }
```

### 指定 Offset 消费

auto.offset.reset = earliest | latest | none 默认是 latest。 

当 Kafka 中没有初始偏移量（消费者组第一次消费）或服务器上不再存在当前偏移量时（例如该数据已被删除），该怎么办？ 

（1）earliest：自动将偏移量重置为最早的偏移量，**--from-beginning**。 

（2）latest（**默认值**）：自动将偏移量重置为最新偏移量。

（3）none：如果未找到消费者组的先前偏移量，则向消费者抛出异常。

![](https://cdn.jsdelivr.net/gh/839777408/tupian/blog2/20220315231605.png)

任意指定 offset 位移开始消费（注意：每次执行完，需要修改消费者组名）：

```java
    @Test
    void CustomConsumerSeek() {
        Properties properties = new Properties();

        properties.put(ConsumerConfig.BOOTSTRAP_SERVERS_CONFIG, "hadoop102:9092");
        // 配置序列化 必须
        properties.put(ConsumerConfig.KEY_DESERIALIZER_CLASS_CONFIG, StringDeserializer.class.getName());
        properties.put(ConsumerConfig.VALUE_DESERIALIZER_CLASS_CONFIG, StringDeserializer.class.getName());
        // 配置消费者组
        properties.put(ConsumerConfig.GROUP_ID_CONFIG, "test");
        // 创建 kafka 消费者
        KafkaConsumer<String, String> kafkaConsumer = new KafkaConsumer<>(properties);
        // 设置消费主题 形参是列表
        kafkaConsumer.subscribe(Collections.singletonList("first"));

        Set<TopicPartition> assignment = new HashSet<>();
        while (assignment.size() == 0) {
            kafkaConsumer.poll(Duration.ofSeconds(1));
            // 获取消费者分区分配信息（有了分区分配信息才能开始消费）
            assignment = kafkaConsumer.assignment();
        }
        // 遍历所有分区，并指定 offset 从 1700 的位置开始消费
        for (TopicPartition tp : assignment) {
            kafkaConsumer.seek(tp, 1700);
        }
        while (true) {
            // 读取消息
            ConsumerRecords<String, String> consumerRecords = kafkaConsumer.poll(Duration.ofSeconds(1));
            for (ConsumerRecord<String, String> consumerRecord : consumerRecords) {
                System.out.println(consumerRecord);
            }
        }
    }
```

### 指定时间消费

需求：在生产环境中，会遇到最近消费的几个小时数据异常，想重新按照时间消费。

例如要求按照时间消费前一天的数据，怎么处理？

```java
    @Test
    void CustomConsumerForTime() {
        Properties properties = new Properties();

        properties.put(ConsumerConfig.BOOTSTRAP_SERVERS_CONFIG, "hadoop102:9092");
        // 配置序列化 必须
        properties.put(ConsumerConfig.KEY_DESERIALIZER_CLASS_CONFIG, StringDeserializer.class.getName());
        properties.put(ConsumerConfig.VALUE_DESERIALIZER_CLASS_CONFIG, StringDeserializer.class.getName());
        // 配置消费者组
        properties.put(ConsumerConfig.GROUP_ID_CONFIG, "test");
        // 是否自动提交 offset
        properties.put(ConsumerConfig.ENABLE_AUTO_COMMIT_CONFIG, false);
        // 创建 kafka 消费者
        KafkaConsumer<String, String> kafkaConsumer = new KafkaConsumer<>(properties);
        // 设置消费主题 形参是列表
        kafkaConsumer.subscribe(Collections.singletonList("first"));

        Set<TopicPartition> assignment = new HashSet<>();
        while (assignment.size() == 0) {
            kafkaConsumer.poll(Duration.ofSeconds(1));
            // 获取消费者分区分配信息（有了分区分配信息才能开始消费）
            assignment = kafkaConsumer.assignment();
        }
        HashMap<TopicPartition, Long> timestampToSearch = new HashMap<>();
        // 封装集合存储，每个分区对应一天前的数据
        for (TopicPartition topicPartition : assignment) {
            timestampToSearch.put(topicPartition, System.currentTimeMillis() - 1 * 24 * 3600 * 1000);
        }
        // 获取从 1 天前开始消费的每个分区的 offset
        Map<TopicPartition, OffsetAndTimestamp> offsets = kafkaConsumer.offsetsForTimes(timestampToSearch);
        // 遍历每个分区，对每个分区设置消费时间。
        for (TopicPartition topicPartition : assignment) {
            OffsetAndTimestamp offsetAndTimestamp = offsets.get(topicPartition);
            // 根据时间指定开始消费的位置
            if (offsetAndTimestamp != null) {
                kafkaConsumer.seek(topicPartition, offsetAndTimestamp.offset());
            }
        }
        while (true) {
            // 读取消息
            ConsumerRecords<String, String> consumerRecords = kafkaConsumer.poll(Duration.ofSeconds(1));
            for (ConsumerRecord<String, String> consumerRecord : consumerRecords) {
                System.out.println(consumerRecord);
            }
        }
    }
```

## 漏消费和重复消费

**重复消费：**已经消费了数据，但是 offset 没提交。

![](https://cdn.jsdelivr.net/gh/839777408/tupian/blog2/20220316232534.png)

**漏消费：**设置offset为手动提交，当offset被提交时，数据还在内存中未落盘，此时刚好消费者线程被kill掉，那么offset已经提交，但是数据未处理，导致这部分内存中的数据丢失。

![](https://cdn.jsdelivr.net/gh/839777408/tupian/blog2/20220316232709.png)

如果想完成Consumer端的精准一次性消费，那么需要Kafka消费端将消费过程和提交offset过程做**原子绑定**。此时我们需要将Kafka的offset保存到支持事务的自定义介质（比如MySQL）。这部分知识会在后续项目部分涉及。

![](https://cdn.jsdelivr.net/gh/839777408/tupian/blog2/20220316232839.png)

## 数据积压

- 如果是Kafka消费能力不足，则可以考虑增加Topic的分区数，并且同时提升消费组的消费者数量，消费者数 = 分区数。（两者缺一不可） 
- 如果是下游的数据处理不及时：提高每批次拉取的数量。批次拉取数据过少（拉取数据/处理时间 < 生产速度），使处理的数据小于生产的数据，也会造成数据积压。

![](https://cdn.jsdelivr.net/gh/839777408/tupian/blog2/20220316233043.png)

# Kafka监控

Kafka-Eagle 框架可以监控 Kafka 集群的整体运行情况，在生产环境中经常使用。

## 其他环境准备

 **MySQL 环境准备：**

```shell
docker run -p 3306:3306 --name mysql 
-v /root/mysql/conf:/etc/mysql/conf.d 
-v /root/mysql/logs:/logs 
-v /root/mysql/data:/var/lib/mysql 
-e MYSQL_ROOT_PASSWORD=123456 
-d mysql
```

**Zookeeper环境准备：**

```shell
docker run -d
-p 2181:2181
-v /root/zookeeper/data/:/data/
--name=zookeeper
--privileged zookeeper
```

**Kafka 环境准备：**

修改/opt/kafka/bin/kafka-server-start.sh 命令中的配置（可选，如果kafka启动不了，根目录有hs_err_pid的log，大概率是内存，可将启动脚本中堆内存参数改小，或者增加服务器内存，EKAF启动同理）：

```shell
if [ "x$KAFKA_HEAP_OPTS" = "x" ]; then
 export KAFKA_HEAP_OPTS="-server -Xms2G -Xmx2G -XX:PermSize=128m -XX:+UseG1GC -XX:MaxGCPauseMillis=200 -XX:ParallelGCThreads=8 -XX:ConcGCThreads=5 -XX:InitiatingHeapOccupancyPercent=70"
 export JMX_PORT="9999"
 #export KAFKA_HEAP_OPTS="-Xmx1G -Xms1G"
 #export KAFKA_HEAP_OPTS="-Xmx256M -Xms128M" #启动不了可以这样设置，efak也可以改
fi
```

## Kafka-Eagle 安装

- 官网：https://www.kafka-eagle.org/

- 上传压缩包 kafka-eagle-bin-2.1.0.tar.gz 到集群/root/software 目录

- 解压到opt目录：

`[root@nanzx kafka-eagle-bin-2.1.0]# tar -zxvf kafka-eagle-bin-2.1.0.tar.gz -C /opt/`

- 进入刚才解压后的目录继续解压：

`[root@nanzx kafka-eagle-bin-2.1.0]# tar -zxvf efak-web-2.1.0-bin.tar.gz -C /opt/`

- 修改名称

`[root@nanzx kafka-eagle-bin-2.1.0]# mv efak-web-2.1.0 efak`

- 修改配置文件 /opt/module/efak/conf/system-config.properties

```properties
######################################
# multi zookeeper & kafka cluster list
# Settings prefixed with 'kafka.eagle.' will be deprecated, use 'efak.' instead
######################################
efak.zk.cluster.alias=cluster1
cluster1.zk.list=192.168.2.110:2181,192.168.2.111:2181,192.168.2.112:2181

######################################
# zookeeper enable acl
######################################
cluster1.zk.acl.enable=false
cluster1.zk.acl.schema=digest
cluster1.zk.acl.username=test
cluster1.zk.acl.password=test123

######################################
# broker size online list
######################################
cluster1.efak.broker.size=20

######################################
# zk client thread limit
######################################
kafka.zk.limit.size=16

######################################
# EFAK webui port
######################################
efak.webui.port=8048

######################################
# EFAK enable distributed
######################################
efak.distributed.enable=false
efak.cluster.mode.status=master
efak.worknode.master.host=localhost
efak.worknode.port=8085

######################################
# kafka jmx acl and ssl authenticate
######################################
cluster1.efak.jmx.acl=false
cluster1.efak.jmx.user=keadmin
cluster1.efak.jmx.password=keadmin123
cluster1.efak.jmx.ssl=false
cluster1.efak.jmx.truststore.location=/data/ssl/certificates/kafka.truststore
cluster1.efak.jmx.truststore.password=ke123456

######################################
# kafka offset storage
######################################
cluster1.efak.offset.storage=kafka

######################################
# kafka jmx uri
######################################
cluster1.efak.jmx.uri=service:jmx:rmi:///jndi/rmi://%s/jmxrmi

######################################
# kafka metrics, 15 days by default
######################################
efak.metrics.charts=true
efak.metrics.retain=15

######################################
# kafka sql topic records max
######################################
efak.sql.topic.records.max=5000
efak.sql.topic.preview.records.max=10

######################################
# delete kafka topic token
######################################
efak.topic.token=keadmin

######################################
# kafka sasl authenticate
######################################
cluster1.efak.sasl.enable=false
cluster1.efak.sasl.protocol=SASL_PLAINTEXT
cluster1.efak.sasl.mechanism=SCRAM-SHA-256
cluster1.efak.sasl.jaas.config=org.apache.kafka.common.security.scram.ScramLoginModule required username="kafka" password="kafka-eagle";
cluster1.efak.sasl.client.id=
cluster1.efak.blacklist.topics=
cluster1.efak.sasl.cgroup.enable=false
cluster1.efak.sasl.cgroup.topics=
cluster2.efak.sasl.enable=false
cluster2.efak.sasl.protocol=SASL_PLAINTEXT
cluster2.efak.sasl.mechanism=PLAIN
cluster2.efak.sasl.jaas.config=org.apache.kafka.common.security.plain.PlainLoginModule required username="kafka" password="kafka-eagle";
cluster2.efak.sasl.client.id=
cluster2.efak.blacklist.topics=
cluster2.efak.sasl.cgroup.enable=false
cluster2.efak.sasl.cgroup.topics=

######################################
# kafka ssl authenticate
######################################
cluster3.efak.ssl.enable=false
cluster3.efak.ssl.protocol=SSL
cluster3.efak.ssl.truststore.location=
cluster3.efak.ssl.truststore.password=
cluster3.efak.ssl.keystore.location=
cluster3.efak.ssl.keystore.password=
cluster3.efak.ssl.key.password=
cluster3.efak.ssl.endpoint.identification.algorithm=https
cluster3.efak.blacklist.topics=
cluster3.efak.ssl.cgroup.enable=false
cluster3.efak.ssl.cgroup.topics=

######################################
# kafka sqlite jdbc driver address
######################################
#efak.driver=org.sqlite.JDBC
#efak.url=jdbc:sqlite:/hadoop/kafka-eagle/db/ke.db
#efak.username=root
#efak.password=www.kafka-eagle.org

######################################
# kafka mysql jdbc driver address
######################################
efak.driver=com.mysql.cj.jdbc.Driver
efak.url=jdbc:mysql://127.0.0.1:3306/ke?useUnicode=true&characterEncoding=UTF-8&zeroDateTimeBehavior=convertToNull
efak.username=root
efak.password=123456
```

- 配置环境变量`vi /etc/profile`：

  ```properties
  # kafkaEFAK
  export KE_HOME=/opt/efak
  export PATH=$PATH:$KE_HOME/bin
  ```

  使环境变量生效：`source /etc/profile`

- 启动：`[root@nanzx efak]# bin/ke.sh start`
  - 注意：启动之前需要先启动 ZK 以及 KAFKA

- 登录页面查看监控数据：http://192.168.2.110:8048/
  - 启动成功后控制台会打印出用户名密码和HTTP访问地址相关信息。要注意的是，**启动失败**也会打印出这些信息，所以得通过链接访问成功了才能判断启动没问题。

# Kafka-Kraft 模式

## 架构

![](https://cdn.jsdelivr.net/gh/839777408/tupian/blog2/20220319193800.png)

左图为 Kafka 现有架构，元数据在 zookeeper 中，运行时动态选举 controller，由controller 进行 Kafka 集群管理。右图为 kraft 模式架构（实验性），不再依赖 zookeeper 集群，而是用三台 controller 节点代替 zookeeper，元数据保存在 controller 中，由 controller 直接进行 Kafka 集群管理。

这样做的好处有以下几个： 

- Kafka 不再依赖外部框架，而是能够独立运行； 

- controller 管理集群时，不再需要从 zookeeper 中先读取数据，集群性能上升； 

- 由于不依赖 zookeeper，集群扩展时不再受到 zookeeper 读写能力限制； 

- controller **不再动态选举**，而是由配置文件规定。这样我们可以有针对性的加强controller 节点的配置，而不是像以前一样对随机 controller 节点的高负载束手无策。

## 安装部署

安装Kafka，以前是进入config目录修改配置文件 server.properties，现在是修改/config/**kraft**/server.properties 配置文件：

```properties
#kafka的角色（controller相当于主机、broker节点相当于从机，主机类似 zk 功 能）
process.roles=broker, controller
#节点ID，不能重复
node.id=2
#controller 服务协议别名
controller.listener.names=CONTROLLER
#全 Controller 列表，前面需要加node.id
controller.quorum.voters=2@hadoop102:9093,3@hadoop103:9093,4@hadoop104:9093
#不同服务器绑定的端口
listeners=PLAINTEXT://:9092,CONTROLLER://:9093
#broker 服务协议别名
inter.broker.listener.name=PLAINTEXT
#broker 对外暴露的地址
advertised.Listeners=PLAINTEXT://hadoop102:9092
#协议别名到安全协议的映射
listener.security.protocol.map=CONTROLLER:PLAINTEXT,PLAINTEXT:PLA
INTEXT,SSL:SSL,SASL_PLAINTEXT:SASL_PLAINTEXT,SASL_SSL:SASL_SSL
#kafka 数据存储目录
log.dirs=/opt/kafka2/data
```

初始化集群数据目录：

```shell
# 首先生成存储目录唯一 ID。
[nanzx kafka2]$ bin/kafka-storage.sh random-uuid
J7s9e8PPTKOO47PxzI39VA

# 用该 ID 格式化 kafka 存储目录（三台节点）。
[nanzx kafka2]$ bin/kafka-storage.sh format -t J7s9e8PPTKOO47PxzI39VA -c 
/opt/kafka2/config/kraft/server.properties

# 启动 kafka 集群
[nanzx kafka2]$ bin/kafka-server-start.sh -daemon config/kraft/server.properties

# 停止 kafka 集群
[nanzx kafka2]$ bin/kafka-server-stop.sh
```

# Kafka整合SpringBoot Demo

引入依赖：

```xml
<dependency>
    <groupId>org.springframework.kafka</groupId>
    <artifactId>spring-kafka</artifactId>
    <version>2.8.3</version>
</dependency>
<dependency>
    <groupId>org.springframework.kafka</groupId>
    <artifactId>spring-kafka-test</artifactId>
    <version>2.8.3</version>
    <scope>test</scope>
</dependency>
```
修改配置文件：
```properties
# 应用名称
spring.application.name=Learn_Kafka

# 生产者配置
spring.kafka.bootstrap-servers=192.168.2.110:9092
spring.kafka.producer.key-serializer=org.apache.kafka.common.serialization.StringSerializer
spring.kafka.producer.value-serializer=org.apache.kafka.common.serialization.StringSerializer

#消费者配置
spring.kafka.consumer.key-deserializer=org.apache.kafka.common.serialization.StringDeserializer
spring.kafka.consumer.value-deserializer=org.apache.kafka.common.serialization.StringDeserializer
spring.kafka.consumer.group-id=nanzx
```

生产者：

```java
@RestController
public class MyProducer {

    @Autowired
    private KafkaTemplate<String,String> kafkaTemplate;


    @GetMapping("/producer/{message}")
    public String sendMessage1(@PathVariable("message") String normalMessage) {
        kafkaTemplate.send("firstTopic", normalMessage);
        return "OK";
    }
}
```

消费者：

```java
@Component
public class MyConsumer {

    @KafkaListener(topics = "firstTopic")
    void consume(ConsumerRecord<?, ?> record) {
        System.out.println("简单消费：" + record.topic() + "-" + record.partition() + "-" + record.value());
    }
}
```

>**SpringBoot集成kafka全面实战：**https://blog.csdn.net/yuanlong122716/article/details/105160545/

