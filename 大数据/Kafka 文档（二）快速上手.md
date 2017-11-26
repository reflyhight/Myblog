title: Kafka 文档（二）快速上手
tags: 
	- mq
	- zookeeper
	- kafka

categories:
	- 大数据
-------------------


>本文大致翻译kafka官网 [Quick Start](http://kafka.apache.org/quickstart)，部分细节加入个人理解。**Quick Start**基本能解决kafka集群安装以及简单使用。
 
###    第1步：下载解压
下载连接[Download](http://kafka.apache.org/downloads)，下载发布的tgz包，然后解压：
```
> tar -xzf kafka_2.11-0.11.0.1.tgz
> cd kafka_2.11-0.11.0.1
```
 
###    第2步：启动服务
kafka依赖于zookeeper服务，首先启动zookeeper服务，kafka中内置有zookeeper，简单配置既可以使用，启动zookeeper：
```
> bin/zookeeper-server-start.sh config/zookeeper.properties
[2013-04-22 15:01:37,495] INFO Reading configuration from: config/zookeeper.properties (org.apache.zookeeper.server.quorum.QuorumPeerConfig)
...
```
启动kafka server：
```
> bin/kafka-server-start.sh config/server.properties
[2013-04-22 15:01:47,028] INFO Verifying properties (kafka.utils.VerifiableProperties)
[2013-04-22 15:01:47,051] INFO Property socket.send.buffer.bytes is overridden to 1048576 (kafka.utils.VerifiableProperties)
...
```
 
###    第3步：创建topic
先创建一个名为'test'的topic，副本数设置为1
```
> bin/kafka-topics.sh --create --zookeeper localhost:2181 --replication-factor 1 --partitions 1 --topic test
```
通过--list选项查看已经创建的topic
```
> bin/kafka-topics.sh --list --zookeeper localhost:2181
test
```
 
###    第4步：发送消息
kafka自带了一个命令行工具，将通过标准输入发送消息，会将消息发送给kafka集群，默认一行内容会作为一条消息来发送
```
> bin/kafka-console-producer.sh --broker-list localhost:9092 --topic test
This is a message
This is another message
```
###    第5步：消费消息
kafka同样也自带了一个模拟消费者的命令行，会将消息显示到标准输入中
```
> bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic test --from-beginning
This is a message
This is another message
```
如果在不同终端运行上面的命令行，那么你就可以在生产者终端（发送消息工具）模拟消息发送，并在消费者终端中看到发送的消息了。
每个命令工具都有一些其他的参数，如果运行时不给任意参数，那么命令行会显示出该命令工具的使用说明
 
###    第6步：运行一个多broker的kafka集群
前面我们运行的是一个broker的服务（broker代表kafka server进程实例），现在让我们将节点扩展到3个broker，但3个broker都运行在同一台物理机器上
 
复制kafka配置文件：
```
> cp config/server.properties config/server-1.properties
> cp config/server.properties config/server-2.properties
```
分别修改两份配置文件为：
```
config/server-1.properties:
    broker.id=1
    listeners=PLAINTEXT://:9093
    log.dir=/tmp/kafka-logs-1
 
config/server-2.properties:
    broker.id=2
    listeners=PLAINTEXT://:9094
    log.dir=/tmp/kafka-logs-2
```
**注意**：***其中broker.id是broker的唯一标识，在集群中全局唯一，因为在一台物理机上运行多个broker事例，所以我们修改了listeners的端口号和log目录***
 
因为刚刚已经安装过zookeeper了，所以我们可以直接启动另外两个broker实例
```
> bin/kafka-server-start.sh config/server-1.properties &
...
> bin/kafka-server-start.sh config/server-2.properties &
...
```
现在我们使用新的kafka集群创建一个拥有三个副本因子(replication factor)的topic
```
> bin/kafka-topics.sh --create --zookeeper localhost:2181 --replication-factor 3 --partitions 1 --topic my-replicated-topic
```
现在我们已经在集群上创建了一个主题（topic），那么怎么知道当前状态的集群信息呢，使用**describe topics**命令来查询
```
> bin/kafka-topics.sh --describe --zookeeper localhost:2181 --topic my-replicated-topic
Topic:my-replicated-topic   PartitionCount:1    ReplicationFactor:3 Configs:
Topic: my-replicated-topic  Partition: 0    Leader: 1   Replicas: 1,2,0 Isr: 1,2,0
```

**关于分区和副本的理解：**    
- **分区(Partition)**是数据的逻辑结构，同样也是kafka底层存储时的物理机构（存在于不同的文件夹下面），暂时可以理解为my-replicated-topic的数据由几个**数据块**组成，就是分区的概念。
- **而副本因子（replication-factor）**表示每个分区在不同broker中的副本数，这里因为仅有一个分区，所以代表这个分区拥有三份相同的副本存放于不同的broker上。副本本来是防止数据丢失的机制，所以副本因子小于等于broker总数

通过查询topic信息得到的两行信息，其中第一行`Topic:my-replicated-topic   PartitionCount:1    ReplicationFactor:3 Configs:`是**my-replicated-topic**主题(topic)的梗概，对梗概信息做如下说明：
- **PartitionCount:1 **   ： 数据分区数目
- **ReplicationFactor:3**   ：数据的副本因子
- **Configs:**   ：其他分区配置，这里没有配置项

而除第一行之外，以`Topic: my-replicated-topic`开头的行是my-replicated-topic主题的分区信息，因为我们创建的my-replicated-topic主题只有一个分区，所以只有一行分区信息，下面对分区信息做一个大致说明
- **Partition: 0**   代表该分区的分区号
- **Leader: 1**   分区数据的副本（Replica）具有主次之分，`Leader: 1`代表主副本在broker.id号为1的broker上，而主副本负责数据的读写，其他的副本只做数据备份的作用
- **Replicas: 1,2,0**   代表分区副本分布在不同broker上的信息，而`1,2,0`表示broker.id
- **Isr: 1,2,0**    Isr是(in-sync)的意思，表示目前存活的broker中并且已经和主副本（Leader）数据完成同步的broker.id号
 
我们同样可以查看最开始我们创建的topic test的信息
```
> bin/kafka-topics.sh --describe --zookeeper localhost:2181 --topic test
Topic:test  PartitionCount:1    ReplicationFactor:1 Configs:
Topic: test Partition: 0    Leader: 0   Replicas: 0 Isr: 0
```
也很简单，topic test中副本因子是1，Leader在broker.id=0上，Replicas信息也仅有broker.id=0，Isr信息也仅有broker.id=0
 
现在让我们发送一些消息到my-replicated-topic中
```
> bin/kafka-console-producer.sh --broker-list localhost:9092 --topic my-replicated-topic
...
my test message 1
my test message 2
```
再使用一个consumer消费一次my-replicated-topic中的数据
```
> bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 --from-beginning --topic my-replicated-topic
...
my test message 1
my test message 2
```
现在让我们测试一下kafka的容错，broker.id=1的broker为目前是主副本节点，现在我们手动将该实例杀死
```
> ps aux | grep server-1.properties
7564 ttys002    0:15.91 /System/Library/Frameworks/JavaVM.framework/Versions/1.8/Home/bin/java...
> kill -9 7564
```
再次查看topic的信息，我们可以看到分区数据的leader角色已经转换为broker.id=2的broker上了
```
> bin/kafka-topics.sh --describe --zookeeper localhost:2181 --topic my-replicated-topic
Topic:my-replicated-topic   PartitionCount:1    ReplicationFactor:3 Configs:
Topic: my-replicated-topic  Partition: 0    Leader: 2   Replicas: 1,2,0 Isr: 2,0
```
但my-replicated-topic的数据现在任然出于可用状态
```
> bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 --from-beginning --topic my-replicated-topic
...
my test message 1
my test message 2
```
###    第7步：使用kafka connect导入、导出数据
虽然通过客户端写入读出数据是很方便的，但你不仅仅想要这样的功能。你可能有时候需要将kafka的数据引入到其他的系统中，你可以使用kafka connect将数据导入或者导出，而不需要编写整合客户端的代码。
kafka connect是数据导入、导出kafka的工具，kafka connect支持扩展性，可以根据需要实现业务逻辑，下面案例介绍如何将文本文件中的数据导入到kafka topic中，如何将kafka topic中的数据导出到一个文本文件中
 
首先创建一个文本文件，并写入两行(将foo和nbar做为两行写入test.txt
```
> echo -e "foo\nbar" > test.txt
```
然后我们创建两个连接器(connectors)使用单机模式运行(standalone mode)，单机模式表示运行一个本地，的独立进程。跟着后面输入三个配置文件参数，第一个配置文件配置了关于kafka数据源的配置，包含了要连接到的kafka集群信息，数据的序列化方式等信息，剩下的配置文件，每个配置文件会创建一个kafka connect，每个配置文件包含一个唯一的连接名，连接使用的class实例和其他配置项
```
> bin/connect-standalone.sh config/connect-standalone.properties config/connect-file-source.properties config/connect-file-sink.properties
```
这三个配置文件在当前版本的kafka安装包的config目录下都能找到，其中**connect-standalone.properties**主要配置项有：
```
bootstrap.servers=localhost:9092
key.converter=org.apache.kafka.connect.json.JsonConverter
value.converter=org.apache.kafka.connect.json.JsonConverter
key.converter.schemas.enable=true
value.converter.schemas.enable=true
internal.key.converter=org.apache.kafka.connect.json.JsonConverter
internal.value.converter=org.apache.kafka.connect.json.JsonConverter
internal.key.converter.schemas.enable=false
internal.value.converter.schemas.enable=false
offset.storage.file.filename=/tmp/connect.offsets
offset.flush.interval.ms=10000
```
**connect-file-source.properties**主要配置项有：
```
name=local-file-source
connector.class=FileStreamSource
tasks.max=1
file=test.txt
topic=connect-test
```
**connect-file-sink.properties**主要配置项有：
```
name=local-file-sink
connector.class=FileStreamSink
tasks.max=1
file=test.sink.txt
topics=connect-test
```
你现在创建了两个连接器(connectors)，一个名为**local-file-source**，负责监控文件`test.txt`中的内容，并发送到topic名为**connect-test**的topic中，一个名为**local-file-sink**，负责将topic名为**connect-test**的topic中的消息保存到文件`test.sink.txt`中
检验文件`test.sink.txt`中的内容：
```
> cat test.sink.txt
foo
bar
```
查看topic为connect-test中的消息
```
> bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic connect-test --from-beginning
{"schema":{"type":"string","optional":false},"payload":"foo"}
{"schema":{"type":"string","optional":false},"payload":"bar"}
```
此时再向test.txt中追加消息
```
echo "Another line" >> test.txt
```
再次查看topic **connect-test**中的消息
```
> bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic connect-test --from-beginning
{"schema":{"type":"string","optional":false},"payload":"foo"}
{"schema":{"type":"string","optional":false},"payload":"bar"}
{"schema":{"type":"string","optional":false},"payload":"Another line"}
```
再次查看文件`test.sink.txt`中的内容： 
```
> cat test.sink.txt
foo
bar
Another line
```
###    第8步：使用kafka streams处理数据
kafka streams是一个客户端库，支持java和scala。具体参考[streams quickstart](http://kafka.apache.org/0110/documentation/streams/quickstart)。(有空再翻译吧😭)


