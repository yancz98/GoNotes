## 一、Kafka 概述

### 1、定义

Kafka 是一个分布式、支持分区的、多副本的，基于 zookeeper 协调的分布式消息系统。

- 旧：Kafka 是一个分布式的基于发布/订阅模式的消息队列，主要应用于大数据实时处理领域。

- 新：Kafka 是一个开源的分布式事件流平台，被数千家公司用于高性能数据管道、流分析、数据集成、关键任务应用。

### 2、应用场景

- 缓冲、削峰：秒杀活动。
- 解耦：各端的处理过程独立。
- 异步通信：发送短信。

### 3、两种模式

- 点对点模式

一个生产者对应一个消费者，消费者主动拉取数据，消息收到后清除消息。

- 发布/订阅模式

可以有多个 topic 主题（浏览、点赞、收藏、评论等）；

消费者消费数据后，不删除数据；

每个消费者相互独立，都可以消费收到数据；

### 4、Kafka 的优势

- 构建实时的数据流管道，可靠的获取系统和应用程序间的数据。
- 构建实时的流应用程序，对数据流进行转换或反应。

### 5、重要概念

- kafka 作为一个集群运行在一个或多个服务器上。
- kafka 集群存储的消息是以 topic 为类别的。
- 每条消息是由一个 key 、一个 value 和时间戳构成。

> 基本术语

- Topic：Kafka 消息的类别，每一类消息称为一个 Topic。
- Producer：生产者，往某个 Topic 中发送消息，并负责选择往 Topic 的哪个分区上发送。
- Consumer：消费者，从主题中消费消息。
- Broker：代理，Kafka 集群的节点。
- Log：Kafka 集群每个 Topic 维护的一个分区 Log。
- Partition：分区，将消息存储到多个区域。优势：①可以存储更多的数据，不受单台服务器的限制（集群）；②分区可以作为并行处理单元，提高处理效率，但一个分区只能由同一消费者组的一个消费者处理。
- Offset：偏移量，每个分区中的每条消息都有一个唯一的偏移量，消费者可以任意控制偏移量。
- Replication-factor：复制因子，可以有0或多个；每个分区有一个 leader，零或多个 follower（类似主从复制）。leader 宕机时，其它 follower 会被推举为新的 leader。
- Zookeeper：记录 Kafka 集群上下线信息，leader 信息（即保存 Broker 的元数据）。

### 6、消息队列的流派

- 有 brocker 的 MQ：
  - 重 topic：Kafka、RocketMQ、ActiveMQ
  - 轻 topic：RabbitMQ
- 无 brocker 的 MQ：
  - zeroMQ（直接使用 socket 通信）



## 二、下载 / 安装

### 1、安装 Kafka

> 官网下载：https://kafka.apache.org/downloads
>
> - Binary downloads:
>   - Scala 2.13  - [kafka_2.13-3.2.3.tgz](https://downloads.apache.org/kafka/3.2.3/kafka_2.13-3.2.3.tgz)

````
# 下载
curl -O https://downloads.apache.org/kafka/3.2.3/kafka_2.13-3.2.3.tgz

# 解压
tar -zxvf kafka_2.13-3.2.3.tgz

# 移动到 /usr/local/kafka
mv kafka_2.13-3.2.3/ /usr/local/kafka/

# 配置环境变量
vi /etc/profile
```
# 配置 kafka
export PATH=$PATH:/usr/local/kafka/bin
```
source /etc/profile
````

- 目录结构

```
/use/local/kafka
    bin     # 可执行脚本（命令）
        kafka-server-start.sh      # Kafka 启动脚本
        kafka-server-stop.sh       # Kafka 停止脚本
        kafka-topics.sh            # Kafka 主题
        kafka-console-producer.sh  # Kafka 生产者
        kafka-console-consumer.sh  # Kafka 消费者
        kafka-consumer-groups.sh   # Kafka 消费者组
        zookeeper-server-start.sh  # Zookeeper 启动脚本
        zookeeper-server-stop.sh   # Zookeeper 停止脚本
        zookeeper-shell.sh         # 查看 kafka 在 zookeeper 中的配置
        ...
    config  # 配置文件
        server.properties		# Kafka 服务配置
        zookeeper.properties	# Zookeeper 服务配置
        producer.properties		# 生产者配置
        consumer.properties		# 消费者配置
        ...
    libs	# 第三方包
        ...
```

### 2、安装 JDK8+

> Java 官网：https://www.oracle.com/java/technologies/downloads/

````
# Install & Deploy
curl -O https://download.oracle.com/java/17/latest/jdk-17_linux-x64_bin.tar.gz
tar -zxvf jdk-17_linux-x64_bin.tar.gz
mv jdk-17.0.4.1/ /usr/local/jdk17/

# 配置环境变量
vi /etc/profile

```
# jdk 安装目录
export JAVA_HOME=/usr/local/jdk17
export JRE_HOME=${JAVA_HOME}/jre
export CLASSPATH=.:${JAVA_HOME}/lib:${JRE_HOME}/lib:$CLASSPATH
export JAVA_PATH=${JAVA_HOME}/bin:${JRE_HOME}/bin
export PATH=$PATH:${JAVA_PATH}
```

soure /etc/profile

# 安装完成
java -version
````

### 3、在 Docker 中安装

> docker-compose.yml

```yaml
version: '3'
services:
  zookeeper:
    image: wurstmeister/zookeeper
    container_name: zookeeper
    ports:
      - "2181:2181"
  kafka:
    image: wurstmeister/kafka:latest
    container_name: kafka
    ports:
      - "9092:9092"
    environment:
      KAFKA_ADVERTISED_HOST_NAME: 192.168.56.101
      KAFKA_ZOOKEEPER_CONNECT: zookeeper:2181
    volumes:
      - /data/kafka/:/var/run/docker.sock
```

### 4、启动 Kafka

```
# 运行 kafka 需要使用 Zookeeper，所以先要启动 Zookeeper
cd /usr/local/kafka
bin/zookeeper-server-start.sh -daemon config/zookeeper.properties

# 启动 kafka 服务
cd /usr/local/kafka
bin/kafka-server-start.sh -daemon config/server.properties

注：
    -daemon	守护进程方式启动

# 用 JDK 工具 jps 查看 Java 进程
> jsp
3322 Jps
3292 Kafka
2526 QuorumPeerMain    # Zookeeper 进程
```



## 三、操作

### 1、主题（topic）

```
bin/kafka-topics.sh [Options]

Options：
    --bootstrap-server    必须：要连接的 kafka 服务器
    --topic               主题名
    --create              创建主题
    --delete              删除主题
    --alter               修改主题（如：增加分区数）
    --list                查看所有主题
    --describe            查看主题详情，不指定主题时，查看所有主题详情
    --partitions          分区数
    --replication-factor  副本数（副本数应小于等于 broker 数量）
    --config <name=value> 指定配置
```

#### （1）创建主题

```shell
# 创建主题
bin/kafka-topics.sh --bootstrap-server localhost:9092 \
    --create \
    --topic msg \
    --replication-factor 2 \
    --partitions 2 
```

> 主题创建成功后，会生成 `log.dirs=/tmp/kafka-logs/msg-0` 、`log.dirs=/tmp/kafka-logs/msg-1` 目录（命名规则：主题-分区），用于存放主题数据。

```
# ll /tmp/kafka-logs/msg-0/

00000000000000000000.index
00000000000000000000.log			// 存放消息
00000000000000000000.timeindex
leader-epoch-checkpoint
partition.metadata
```

#### （2）查看主题详情

```shell
# 查看主题详情
bin/kafka-topics.sh --bootstrap-server localhost:9092 \
    --describe \
    --topic msg 
	 
Topic: msg    TopicId: *** PartitionCount: 2    ReplicationFactor: 1    Configs: ***
    Topic: msg    Partition: 0    Leader: 0    Replicas: 0    Isr: 0
    Topic: msg    Partition: 1    Leader: 0    Replicas: 0    Isr: 0	 
```

> 主题详情

| Topic      | TopicId      | PartitionCount                           | ReplicationFactor         | Configs                                  |
| ---------- | ------------ | ---------------------------------------- | ------------------------- | ---------------------------------------- |
| 主题名     | 主题ID       | 分区数                                   | 副本数                    | 配置                                     |
| Topic: *** | Partition: 0 | Leader: 1                                | Replicas: 1,0             | Isr: 1,0                                 |
| 主题名     | 分区号       | 从所有副本中选出一个 Leader（broker.id） | 副本在 brokers 的分布情况 | in-sync-replics 与 Leader 保持同步的集合 |
| ......     |              |                                          |                           |                                          |

> ISR：

Kafka 为 partition 动态维护一个 ISR 集合，该集合中的所有 replica 都与 Leader 保持同步。只有该集合中的 replica 才可能被选举为 Leader，只有该集合中所有的 replica 都收到了同一条消息，Kafka 才会将该消息置于“已提交”状态，即发送成功。

Kafka 的消息交付承诺： 保证在 ISR 存活的情况下“已提交”的消息不会丢失。

#### （3）增加分区数

```
# 增加分区数
kafka-topics.sh --bootstrap-server 192.168.56.101:9092 \
    --alter \
    --topic msg \
    --partitions 2 
```



### 2、生产者

```
bin/kafka-console-producer.sh [Options]

Options：
    --bootstrap-server           REQUIRED：要连接的服务器。
    --topic                      REQUIRED：向主题生产消息。
    --sync                       同步发送消息，消息会一条一条的发。（默认：异步）
    --request-required-acks      生成者请求的 ACK 确认方式（默认：-1），
    --message-send-max-retries   最大发送重试次数（默认：3）		
    --retry-backoff-ms           重试时间间隔（默认：100 ms）
    --producer.config            指定生成者配置文件 config/producer.properties，
                                 --producer-property 优先于配置文件。 			
    --max-memory-bytes           生产者缓冲区大小（默认：33554432 B = 32 M）
    --batch-size                 异步发送时，单批发送的消息数（默认：16384），
                                 max-partition-memory-bytes 会替换该选项。
    --max-partition-memory-bytes 为分区分配的缓冲区大小，生产者会尝试拼凑到此大小（默认：16384）
```

#### （1）简单生产

```
# 生产无 key 的消息
bin/kafka-console-producer.sh \
    --bootstrap-server localhost:9092 \
    --topic msg
	
# 生成有 key 的消息
bin/kafka-console-producer.sh \
    --bootstrap-server localhost:9092 \
    --topic msg
    --property parse.key=true
	
注：
    key 和 value 之间用 Tab 键分隔。
    相同的 key 会被分配到同一分区，可以保证消息的顺序性。
```

#### （2）同步发送与异步发送

```
# 同步发送（缓冲区机制无效）
# 效率低，可靠性高
# 不加 --sync 选项就是异步发送
bin/kafka-console-producer.sh \
	--bootstrap-server localhost:9092 \
	--topic msg \
	--sync \
	--request-required-acks -1
```

#### （3）消息的可靠性保证

```
# 确认机制 + 重试机制
bin/kafka-console-producer.sh \
    --bootstrap-server localhost:9092 \
    --topic msg \
    --request-required-acks -1 \
    --message-send-max-retries 3 \
    --retry-backoff-ms 100

ACK 参数详解：
    acks = -1/all：需要得到 ISR 集合中全部副本的确认；
    acks = 0：不需要等待服务器的确认，立即写入缓冲区，并返回偏移量 -1；
    acks = 1：Leader 会将记录写入log，但不需要 follower 的确认；	
```

#### （4）消息发送缓冲区

> 减少磁盘 IO 次数。

```
# 缓冲区机制
bin/kafka-console-producer.sh \
    --bootstrap-server localhost:9092 \
    --topic msg \
    --max-memory-bytes 33554432 \
    --batch-size 16384
```



### 3、消费者

```
kafka-console-consumer.sh [Options]

Options：
    --bootstrap-server      REQUIRED：要连接的服务器。
    --consumer-property     消费者属性，key=value 格式定义。
    --property              用于初始化消息格式化程序的属性
        print.timestamp=true|false  # 打印 时间戳        
        print.key=true|false        # 打印 key
        print.offset=true|false	    # 打印 offset
        print.partition=true|false  # 打印 partition
        print.headers=true|false    # 打印 header
        print.value=true|false      # 打印 value（默认：true）
        key.separator=<key.separator>         
        line.separator=<line.separator>       
        headers.separator=<headers.separator>    
        null.literal=<null.literal>           
        key.deserializer=<key.deserializer>   
        value.deserializer=<value.deserializer>                        
        header.deserializer=<header.deserializer>                
    --consumer.config       指定消费者配置文件 config/consumer.properties，
                            consumer-property 优先于配置文件。
    --group                 消费者的消费者组ID。
    --include               用正则指定要是用的主题。
    --offset                偏移量（默认：latest），非负数；
                            earliest：从开始消费，latest 从末尾消费。
    --from-beginning        从头开始消费，否则只消费连接后生产的消息
    --partition	            分区，要消费的分区。
    --topic                 要使用的主题。
    --skip-message-on-error 出错时跳过。
    --partition	            要消费的分区，可以指定 --offset ，否则从分区的末尾开始消费。
    --max-messages          消费的最大数据量，若不指定，则持续消费下去。
```

#### （1）低级消费

```
kafka-console-consumer.sh --bootstrap-server localhost:9092 \
    --topic msg \
    --from-beginning	# 从头开始消费，否则只消费上线后生产的消息
```

#### （2）消费者组消费

```
kafka-console-consumer.sh --bootstrap-server localhost:9092 \
    --topic msg \
    --group msgGroup1
```

#### （3）消费 Offset 

```
# __consumer_offset-0 ~ __consumer_offsets-49
kafka 内部默认会创建 __consumer_offsets 主题包含50个分区（通过 offset.topic.num.partitions 配置），这个主题用来存放消费者消费某个主题的偏移量。

消费者定期将自己消费分区的 offset 提交给 kafka 内部的 __consumer_offsets 主题，key 是 consumerGroupID+topic+分区号，value 是当前 offset 的值，kafka 会定期清理 topic 里的消息，最后就保留最新的那条数据。
通过公式：【hash(consumerGroupID) % __consumer_offsets 分区数】计算出 consumer 消费的 offset 要提交到 __consumer_offsets 的哪个分区。
```

> 注：自动提交在 poll 到数据后立马提交 offset，若此时 consumer 来不及消费就挂掉了，则消息丢失。



### 4、消费者组

```
# 所有消费者组
kafka-consumer-groups.sh --bootstrap-server localhost:9092 --list

# 消费者组的消费信息
kafka-consumer-groups.sh --bootstrap-server localhost:9092 \
    --describe 
    --group msgGroup1

Options：
    --bootstrap-server  REQUIRED：要连接的服务器。
    --all-groups        应用到整个消费者组（如：与 --describe 显示所有成员信息）。
    --all-topics        为组中所有主题 --reset-offsets。				
    --list              列出所有消费者组。
    --describe          描述消费者组并列出与给定组相关的偏移滞后（尚未处理的消息数）。
    --members           查看消费者组成员信息，只能与 --describe 选项一起使用。
    --verbose           展示分配给每个成员的分区
    --delete            删除一个或多个消费者组
    --topic             1、应从组中删除的主题；
                        2、应重置偏移量的主题：用 `topic1:0,1,2` 格式指定重置的分区。
```

### 5、kafka 集群

> 注意：kafka 日志 /tmp/kafka-log/ 和 zookeeper 日志 /tmp/zookeeper/ 目录对启动集群的影响。

#### （1）配置文件

- 服务器：192.168.56.101

```
# server.properties

broker.id = 101
listeners=PLAINTEXT://192.168.56.101:9092
log.dirs=/tmp/kafka-logs
zookeeper.connect=192.168.56.101:2181
```

- 服务器：192.168.56.102

```
# server.properties

broker.id = 102
listeners=PLAINTEXT://192.168.56.102:9092
log.dirs=/tmp/kafka-logs
zookeeper.connect=192.168.56.101:2181
```

#### （2）启动服务

- 服务器：192.168.56.101

```
# 启动 zookeeper
zookeeper-server-start.sh -daemon config/zookeeper.properties

# 启动 Kafka
kafka-server-start.sh -daemon config/server.properties 

# 查看 zookeeper 中的节点
zookeeper-shell.sh 192.168.56.101:2181

> ls /brokers/ids
[101]
```

- 服务器：192.168.56.102

```
# 启动 Kafka
kafka-server-start.sh -daemon config/server.properties 

# 查看 zookeeper 中的节点
zookeeper-shell.sh 192.168.56.101:2181

> ls /brokers/ids
[101, 102]	=================> 集群搭建成功
```

#### （3）生产 & 消费

```
# 创建主题
kafka-topics.sh --bootstrap-server 192.168.56.101:9092 \
    --create \
    --topic ct1 \
    --partitions 5 \
    --replication-factor 2

# 往集群中生产消息（也可以只写一个 broker 节点）
kafka-console-producer.sh \
    --bootstrap-server 192.168.56.101:9092, 192.168.56.102:9092 \
    --topic ct1
	
# 从集群中消费消息（也可以只写一个 broker 节点）
kafka-console-consumer.sh \
    --bootstrap-server 192.168.56.101:9092, 192.168.56.102:9092 \
    --topic ct1 \
    --from-beginning
    
# 集群消费者组
kafka-console-consumer.sh \
    --bootstrap-server 192.168.56.101:9092 \
    --topic ct1 \
    --group ctGroup1
	
kafka-console-consumer.sh 
    --bootstrap-server 192.168.56.102:9092 \
    --topic ct1 \
    --group ctGroup1	
	
# 查看消费者组的消费信息
kafka-consumer-groups.sh \
    --bootstrap-server 192.168.56.102:9092 \
    --describe \
    --group ctGroup1

# 查看消费者组的成员信息
kafka-consumer-groups.sh \
    --bootstrap-server 192.168.56.101:9092 \
    --describe \
    --group ctGroup1 \
    --members
```

> 消费者组的消费信息

| GROUP    | TOPIC | PARTITION | CURRENT-OFFSET | LOG-END-OFFSET                   | LAG      | CONSUMER-ID      | HOST            | CLIENT-ID        |
| -------- | ----- | --------- | -------------- | -------------------------------- | -------- | ---------------- | --------------- | ---------------- |
| 组名     | 主题  | 分区号    | 当前消费偏移量 | 消息总量（最后一条消息的偏移量） | 滞后数量 | 组成员ID         | 主机            |                  |
| ctGroup1 | ct1   | 0         | 1              | 1                                | 0        | ***-1e54051f1605 | /192.168.56.101 | console-consumer |
| ctGroup1 | ct1   | 1         | 1              | 1                                | 0        | ***-1e54051f1605 | /192.168.56.101 | console-consumer |
| ctGroup1 | ct1   | 2         | 1              | 1                                | 0        | ***-1e54051f1605 | /192.168.56.101 | console-consumer |
| ctGroup1 | ct1   | 3         | 1              | 1                                | 0        | ***-6e2c350ed3b0 | /192.168.56.102 | console-consumer |
| ctGroup1 | ct1   | 4         | 0              | 0                                | 0        | ***-6e2c350ed3b0 | /192.168.56.102 | console-consumer |

> 消费者组的成员信息

| GROUP    | CONSUMER-ID                | HOST            | CLIENT-ID        | #PARTITIONS |
| -------- | -------------------------- | --------------- | ---------------- | ----------- |
| 组名     | 组成员ID                   | 主机            |                  | 消费分区数  |
| ctGroup1 | ***-6e2c350ed3b0           | /192.168.56.102 | console-consumer | 2           |
| ctGroup1 | ***-4004-aa1b-1e54051f1605 | /192.168.56.101 | console-consumer | 3           |

> 注：
>
> - 一个分区只能被消费组中的一个消费者消费，从而保证消费顺序。
> - 一个消费者组中的一个消费者可以消费多个分区。
>
> - Kafka 只能保证单个分区内消息的顺序，无法在同一个 topic 的多个分区中保证所有消息的顺序。
>
> - 当消费者组中的消费者数量大于 topic 分区数时，多出的消费者消费不到消息。
>

#### （4）Kafka 为什么使用消费者组？

- 消费效率高（横向扩展）
- 消费模式灵活
  - 单播（点对点）模式：将所有消费者放到同一个组；
  - 广播（发布订阅）模式：将所有消费者放到不同的组；

- 故障容灾
  - 消费者组的 rebalance；



## 四、设计

### 1、Kafka 集群中 Controller 的作用

Kafka 集群中的 broker 上线时，会在 zk 中创建临时序号节点，序号最小的节点（最先创建的节点）将作为集群的 controller，负责管理整个集群中的所有分区和副本状态：

- 当某个分区的 Leader 副本出现故障时，由 controller 负责从 ISR 的最左边获得新的 Leader；
- 当检测到某个分区的 ISR 集合发生变化时，由 controller 负责通知所有 broker 更新其元数据；
- 当使用 kafka-topic.sh 脚本为某个 topic 增加分区数量时，还是由 controller 负责让新分区被其它节点认知。

### 2、Rebalance 机制

Rebalance 触发条件：当消费者没有指定消费分区，且消费者组的消费者与分区的关系发生变化时，会触发 Rebalance 机制。该机制会重新调整消费者与分区的消费关系。

消费者消费分区的分配策略：

- range：通过公式计算消费者消费哪个分区（第一个：分区数/消费者数+1，后面：分区数/消费者数）；
- 轮询：按分区依次分配给消费者。
- sticky：触发 Rebalance 机制后，在消费者消费的原分区不变的基础上进行调整。

注意：Rebalance 的过程中，所有实例都会停止消费，等待 Rebalance 完成。而且 Rebalance 过程很慢，所以要尽量避免 Rebalance 的发生。

### 3、HW 和 LEO 机制

LEO（log-end-offset）：副本中消息结尾的偏移量。

HW：高水位，ISR 集合中已完成同步的偏移量。

Consumer 最多只能消费到 HW 所在的位置，每个 Replica 都有自己的 HW，并负责更新各自的 HW 状态。对于 Leader 新写入的消息，Consumer 不能立即消费， Leader 会等待消息被 ISR 集合中的 Replica 同步后更新 HW，HW 更新后才能被消费。防止消息丢失。



## 五、高频考点

### 1、如何防止消息丢失？

- 生产者：acks = -1/all；

- 消费者：手动提交 offset；

### 2、如何防止消息的重复消费？

在防止消息丢失的方案中，如果生产者发送完成后，由于网络抖动没有收到 ack，但实际 broker 已收到消息，此时生产者会重试，于是 broker 就会收到多条相同的消息，而造成消费者的重复消费。

解决：

- 生产者关闭重试：会丢消息（不建议）
- 消费者解决非幂等性消费问题：
  - 联合主键
  - 分布式锁

### 3、如何实现顺序消费？

- 主题只设置一个分区；
- 指定分区发送；

注：Kafka 顺序消费会牺牲性能，RocketMQ 中有专门的功能实现。

### 4、如何解决消息积压？

消费者速度赶不上生产者的速度，导致 kafka 中有大量数据没有被消费，会造成消息积压。消费者寻址的性能会越来越差，最后导致整个 Kafka 对外提供服务的性能很差。

- 消费者使用多线程，充分利用机器性能；
- 创建多个消费组，多个消费者并行消费；

### 5、如何实现延时队列？

场景：在订单创建成功，如果超过30分钟没有付款，则取消该订单。

有点扯......



## 六、Kafka 监控平台 Eagle







## ==、配置文件

### 1、`server.properties` 

```properties
############################# Server Basics #############################

# 集群 broker 的唯一标识（整数）
broker.id = 0

############################# Socket Server Settings #############################

# socket 服务器监听地址（kafka 提供服务的 host:port）
#  EXAMPLE:
#    listeners = PLAINTEXT://your.host.name:9092
listeners=PLAINTEXT://192.168.56.101:9092

# socket 服务器的配置
num.network.threads=3
num.io.threads=8
socket.send.buffer.bytes=102400
socket.receive.buffer.bytes=102400
socket.request.max.bytes=104857600

############################# Log Basics #############################

# 存储日志文件的目录（多个用逗号分隔）
# /tmp/kafka-logs/meta.properties 保存了 kafka 的元数据，
# 修改配置后二次启动可能会影响集群参数（报错）
log.dirs=/tmp/kafka-logs

# 创建主题时默认的日志分区数
num.partitions=1

# 每个数据目录在启动和关闭时用于日志恢复的线程数
num.recovery.threads.per.data.dir=1

############################# Internal Topic Settings  #############################

# 组元数据内部主题 "__consumer_offsets" 和 "__transaction_state" 的复制因子
# 正式环境环境，建议使用大于 1 的值来确保可用性
offsets.topic.replication.factor=1
transaction.state.log.replication.factor=1
transaction.state.log.min.isr=1

############################# Log Flush Policy #############################
##### 日志刷新策略 #####

# 强制将数据刷新到磁盘之前要接受的消息数
# log.flush.interval.messages=10000

# 在强制刷新之前，消息可以在日志中停留的最长时间
# log.flush.interval.ms=1000

############################# Log Retention Policy #############################
##### 日志保留策略 #####

# 只要满足任一条件，就会删除一个日志段，总是从日志末尾开始删除。

# 基于时间的保留策略（默认保留：7天）
log.retention.hours=168

# 基于大小的保留策略（功能独立于 log.retention.hours）
# log.retention.bytes=1073741824

# 单个日志文件的大小限制，达到限制后，将创建新的日志段。
log.segment.bytes=1073741824

# 保留策略检查时间间隔
log.retention.check.interval.ms=300000

############################# Zookeeper #############################

# zookeeper 连接字符串（host1:port1, host2:port2/kafka, ...）
zookeeper.connect=192.168.56.101:2181

# 连接到 zookeeper 的超时时间
zookeeper.connection.timeout.ms = 18000

############################# Group Coordinator Settings ############################
group.initial.rebalance.delay.ms=0
```



### 2、Topic 配置

```properties
# 自动创建主题
auto.create.topics.enable=true
# 默认主题的分区数
num.partitions=8
# 默认分区副本（不得超过 kafka 节点数）
default.replication.factor=3
```



### 3、`producer.properties`

```

```



### 4、Consumer 配置



### 5、Kafka 连接配置



### 6、Kafka 流配置



### 7、AdminClient 配置