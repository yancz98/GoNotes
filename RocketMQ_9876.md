## 一、安装 & 部署

### 1、安装 JDK8+

> Java 官网：https://www.oracle.com/java/technologies/downloads/

````shell
# Install & Deploy
> curl -O https://download.oracle.com/java/17/latest/jdk-17_linux-x64_bin.tar.gz
> tar -zxvf jdk-17_linux-x64_bin.tar.gz
> mv jdk-17.0.4.1/ /usr/local/jdk17/

# 配置环境变量
> vi /etc/profile

`
# jdk 安装目录
export JAVA_HOME=/usr/local/jdk17
export JRE_HOME=${JAVA_HOME}/jre
export CLASSPATH=.:${JAVA_HOME}/lib:${JRE_HOME}/lib:$CLASSPATH
export JAVA_PATH=${JAVA_HOME}/bin:${JRE_HOME}/bin
export PATH=$PATH:${JAVA_PATH}
`

> soure /etc/profile

# 安装完成
> java -version
````

### 2、安装 RocketMQ

[官网下载](https://rocketmq.apache.org/zh/download/)

（1）下载安装

```shell
# 下载二进制包
> curl -O https://dist.apache.org/repos/dist/release/rocketmq/5.1.3/rocketmq-all-5.1.3-bin-release.zip

# 解压
> unzip rocketmq-all-5.1.3-bin-release.zip

> mv rocketmq-all-5.1.3-bin-release/ /usr/local/rocketmq5

# 添加环境变量
> vim /etc/profile

`
export PATH=$PATH:/usr/local/rocketmq5
`
```

（2）启动 NameServer

```shell
# 启动 NameServer
> nohup mqnamesrv > /dev/null 2> /dev/null &

# 验证 namesrv 是否启动成功
> tail -f ~/logs/rocketmqlogs/namesrv.log
The Name Server boot success...
```

> 异常处理

```shell
# 内存不足

Java HotSpot(TM) 64-Bit Server VM warning: INFO: os::commit_memory(0x0000000700000000, 4294967296, 0) failed; error='Not enough space' (errno=12)
#
# There is insufficient memory for the Java Runtime Environment to continue.
# Native memory allocation (mmap) failed to map 4294967296 bytes for committing reserved memory.
# An error report file with more information is saved as:
# /usr/local/rocketmq5/bin/hs_err_pid3427.log
```

> 修改内存配置

```shell
> vim runserver.sh

`
choose_gc_options()
{
    # Example of JAVA_MAJOR_VERSION value : '1', '9', '10', '11', ...
    # '1' means releases befor Java 9
    JAVA_MAJOR_VERSION=$("$JAVA" -version 2>&1 | awk -F '"' '/version/ {print $2}' | awk -F '.' '{print $1}')
    if [ -z "$JAVA_MAJOR_VERSION" ] || [ "$JAVA_MAJOR_VERSION" -lt "9" ] ; then
      # 改这里 -server -Xms4g -Xmx4g -Xmn2g
      JAVA_OPT="${JAVA_OPT} -server -Xms1g -Xmx1g -Xmn1g -XX:MetaspaceSize=128m -XX:MaxMetaspaceSize=320m"
      JAVA_OPT="${JAVA_OPT} -XX:+UseConcMarkSweepGC -XX:+UseCMSCompactAtFullCollection -XX:CMSInitiatingOccupancyFraction=70 -XX:+CMSParallelRemarkEnabled -XX:SoftRefLRUPolicyMSPerMB=0 -XX:+CMSClassUnloadingEnabled -XX:SurvivorRatio=8 -XX:-UseParNewGC"
      JAVA_OPT="${JAVA_OPT} -verbose:gc -Xloggc:${GC_LOG_DIR}/rmq_srv_gc_%p_%t.log -XX:+PrintGCDetails -XX:+PrintGCDateStamps"
      JAVA_OPT="${JAVA_OPT} -XX:+UseGCLogFileRotation -XX:NumberOfGCLogFiles=5 -XX:GCLogFileSize=30m"
    else
      # 改这里 -server -Xms4g -Xmx4g
      JAVA_OPT="${JAVA_OPT} -server -Xms1g -Xmx1g -XX:MetaspaceSize=128m -XX:MaxMetaspaceSize=320m"
      JAVA_OPT="${JAVA_OPT} -XX:+UseG1GC -XX:G1HeapRegionSize=16m -XX:G1ReservePercent=25 -XX:InitiatingHeapOccupancyPercent=30 -XX:SoftRefLRUPolicyMSPerMB=0"
      JAVA_OPT="${JAVA_OPT} -Xlog:gc*:file=${GC_LOG_DIR}/rmq_srv_gc_%p_%t.log:time,tags:filecount=5,filesize=30M"
    fi
}
`
```

（3）启动 Broker + Proxy

```shell
### 先启动broker
> nohup mqbroker -n localhost:9876 --enable-proxy > /dev/null 2> /dev/null &

### 验证broker是否启动成功, 比如, broker的ip是192.168.1.2 然后名字是broker-a
> tail -f ~/logs/rocketmqlogs/proxy.log 
The broker[broker-a,192.169.1.2:10911] boot success...
```

（4）关闭服务器

```shell
> mqshutdown broker
The mqbroker(36695) is running...
Send shutdown request to mqbroker with proxy enable OK(36695)

> mqshutdown namesrv
The mqnamesrv(36664) is running...
Send shutdown request to mqnamesrv(36664) OK
```

3、创建主题

```shell
# 创建主题
> mqadmin updateTopic -n localhost:9876 -t Yan_Test -b localhost:10911

# 查看主题列表
> mqadmin topicList -n localhost:9876
```



## 二、领域模型

### 1、RocketMQ 领域模型

RocketMQ 是一款典型的分布式架构下的中间件产品，使用异步通信方式和发布订阅的消息传输模型。

RocketMQ 产品具备异步通信的优势，系统拓扑简单、上下游耦合较弱，主要应用于异步解耦，流量削峰填谷等场景。

> RocketMQ 中消息的生命周期主要分为消息生产、消息存储、消息消费这三部分。

![领域模型](https://rocketmq.apache.org/zh/assets/images/mainarchi-9b036e7ff5133d050950f25838367a17.png)

#### （1）消息生产

##### 生产者（Producer）

生产者是 RocketMQ 系统中用来构建并传输消息到服务端的运行实体。生产者通常被集成在业务调用链的上游，将业务消息按照要求封装成消息并发送至服务端。生产者是轻量级匿名无身份的。

> 内部属性

- 客户端ID
- 通信参数
- 预绑定主题列表
- 事务检查器
- 发送重试策略



#### （2）消息存储

##### 主题（Topic）

RocketMQ 中消息传输和存储的顶层容器，用于标识同一类业务逻辑的消息。主题内部由多个队列组成，消息的存储和水平扩展实际是通过主题内的队列实现的。

##### 消息队列（MessageQueue）

RocketMQ 中消息存储和传输的实际容器，也是消息的最小存储单元，类比于其他消息队列中的分区。RocketMQ 的所有主题都是由多个队列组成，以此实现队列数量的水平拆分和队列内部的流式存储。队列通过 QueueId 来做唯一标识和区分。RocketMQ 通过流式特性的无限队列结构来存储消息，消息在队列内具备顺序性存储特征。

> 内部属性

- 读写权限：定义当前队列是否可以读写数据：

  - 6：读写状态，当前队列允许读取消息和写入消息。

  - 4：只读状态，当前队列只允许读取消息，不允许写入消息。

  - 2：只写状态，当前队列只允许写入消息，不允许读取消息。

  - 0：不可读写状态，当前队列不允许读取消息和写入消息。



##### 消息类型（MessageType）

RocketMQ 中按照消息传输特性的不同而定义的分类，用于类型管理和安全校验。RocketMQ 支持的消息类型有普通消息（Normal）、顺序消息（FIFO）、事务消息（Transaction）、定时/延时消息（Delay）。

注：从 5.0 版本开始，支持强制校验消息类型，即每个主题只允许发送一种类型的消息。

- 事务消息：事务消息是 RocketMQ 提供的一种高级消息类型，支持在分布式场景下保障消息生产和本地事务的最终一致性。
- 顺序消息：顺序消息是 RocketMQ 提供的一种高级消息类型，支持消费者按照发送消息的先后顺序获取消息，从而实现业务场景中的顺序处理。
- 定时/延时消息：定时/延时消息是 RocketMQ 提供的一种高级消息类型，消息被发送至服务端后，在指定时间后才能被消费者消费。通过设置一定的定时时间可以实现分布式场景的延时调度触发效果。

##### 消息（Message）

RocketMQ 中的最小数据传输单元。生产者将业务数据的负载和拓展属性包装成消息发送到服务端，服务端按照相关语义将消息投递到消费端进行消费。

> RocketMQ 的消息模型具备如下特点：

- **消息不可变性**：消息本质上是已经产生并确定的事件，一旦产生后，消息的内容不会发生改变。即使经过传输链路的控制也不会发生变化，消费端获取的消息都是只读消息视图。

- **消息持久化**：RocketMQ 会默认对消息进行持久化，即将接收到的消息存储到 RocketMQ 服务端的存储文件中，保证消息的可回溯性和系统故障场景下的可恢复性。

> 消息内部属性

- 主题名称
- 消息类型
- 消息队列
- 消息位点
- 消息ID
- 索引 Key 列表（可选）
- 过滤标签 Tag（可选）
- 定时时间（可选）
- 消息发送时间
- 消息保存时间戳
- 消费重试次数
- 业务自定义属性
- 消息负载



#### （3）消息消费

##### 消费者分组（ConsumerGroup）

消费者分组是 RocketMQ 系统中承载多个消费行为一致的消费者的负载均衡分组。和消费者不同，消费者分组并不是运行实体，而是一个逻辑资源。在 RocketMQ 中，通过消费者分组内初始化多个消费者实现消费性能的水平扩展以及高可用容灾。

在消费者分组中，统一定义以下消费行为，同一分组下的多个消费者将按照分组内统一的消费行为和负载均衡策略消费消息。

- 订阅关系：Apache RocketMQ 以消费者分组的粒度管理订阅关系，实现订阅关系的管理和追溯。
- 投递顺序性：Apache RocketMQ 的服务端将消息投递给消费者消费时，支持顺序投递和并发投递，投递方式在消费者分组中统一配置。
- 消费重试策略： 消费者消费消息失败时的重试策略，包括重试次数、死信队列设置等。

##### 消费者（Consumer）

消费者是 RocketMQ 中用来接收并处理消息的运行实体。消费者通常被集成在业务调用链的下游，从服务端获取消息，并将消息转化成业务可以理解的信息，供业务逻辑处理。消费者必须关联一个指定的消费者分组，以获取分组内统一定义的行为配置和消费状态。

> 消费者类型

PushConsumer类型、SimpleConsumer类型、PullConsumer类型（仅推荐流处理场景使用）。

> 内部属性

- 消费者分组名称
- 客户端ID
- 通信参数
- 预绑定订阅关系列表
- 消费监听器

##### 消费结果（ConsumerResult）

RocketMQ 中 PushConsumer 消费监听器处理消息完成后返回的处理结果，用来标识本次消息是否正确处理。消费结果包含消费成功和消费失败。

##### 订阅关系（Subscription）

订阅关系是 RocketMQ 系统中消费者获取消息、处理消息的规则和状态配置。订阅关系由消费者分组动态注册到服务端系统，并在后续的消息传输中按照订阅关系定义的过滤规则进行消息匹配和消费进度维护。

RocketMQ 发布订阅模型中消息过滤、重试、消费进度的规则配置。订阅关系以消费组粒度进行管理，消费组通过定义订阅关系控制指定消费组下的消费者如何实现消息过滤、消费重试及消费进度恢复等。

RocketMQ 的订阅关系除过滤表达式之外都是持久化的，即服务端重启或请求断开，订阅关系依然保留。

> 订阅关系判断原则

RocketMQ 的订阅关系按照消费者分组和主题粒度设计，因此，一个订阅关系指的是指定某个消费者分组对于某个主题的订阅，判断原则如下：

- 不同消费者分组对于同一个主题的订阅相互独立。
- 同一个消费者分组对于不同主题的订阅也相互独立。

> 内部属性

- 过滤类型

- 过滤表达式



#### （4）基本概念

- - 

- **消息视图**（MessageView）：消息视图是 RocketMQ 面向开发视角提供的一种消息只读接口。通过消息视图可以读取消息内部的多个属性和负载信息，但是不能对消息本身做任何修改。
- **消息标签**（MessageTag）：消息标签是 RocketMQ 提供的细粒度消息分类属性，可以在主题层级之下做消息类型的细分。消费者通过订阅特定的标签来实现细粒度过滤。
- **消息位点**（MessageQueueOffset）：消息是按到达 RocketMQ 服务端的先后顺序存储在指定主题的多个队列中，每条消息在队列中都有一个唯一的 Long 类型坐标，这个坐标被定义为消息位点。
- **消费位点**（ConsumerOffset）：一条消息被某个消费者消费完成后，不会立即从队列中删除，RocketMQ 会基于每个消费者分组记录消费过的最新一条消息的位点，即消费位点。
- **消息索引**（MessageKey）：消息索引是 RocketMQ 提供的面向消息的索引属性。通过设置的消息索引可以快速查找到对应的消息内容。
- **事务检查器**（TransactionChecker）：RocketMQ 中生产者用来执行本地事务检查和异常事务恢复的监听器。事务检查器应该通过业务侧数据的状态来检查和判断事务消息的状态。
- **事务状态**（TransactionResolution）：RocketMQ 中事务消息发送过程中，事务提交的状态标识，服务端通过事务状态控制事务消息是否应该提交和投递。事务状态包括事务提交、事务回滚和事务未决。
- **消息过滤**：消费者可以通过订阅指定消息标签（Tag）对消息进行过滤，确保最终只接受被过滤后的消息集合。过滤规则的计算和匹配在 RocketMQ 的服务端完成。
- **重置消费位点**：以时间轴为坐标，在消息持久化存储的时间范围内，重新设置消费者分组对已订阅主题的消费进度，设置完成后消费者将接收设定时间点之后，由生产者发送到 RocketMQ 服务端的消息。
- **消息轨迹**：在一条消息从生成者发出到消费者接收并处理过程中，由各个相关节点的时间、地点等数据汇聚而成的完整链路信息。通过消息轨迹，您能清晰定位消息从生产者发出，经由 RocketMQ 服务端，投递给消费者的完整链路，方便定位排查问题。
- **消息堆积**：生产者已经将消息发送到 RocketMQ 的服务端，但由于消费者的消费能力有限，未能在短时间内将所有消息正确消费掉，此时在服务端保存着未被消费的消息，该状态即消息堆积。

### 2、通信方式

#### （1）同步通信

![同步调用](https://rocketmq.apache.org/zh/assets/images/syncarchi-ebbd41e1afd6adf432792ee2d7a91748.png)

同步通信调用模型下，不同系统之间直接进行调用通信，每个请求直接从调用方发送到被调用方，然后要求被调用方立即返回响应结果给调用方，以确定本次调用结果是否成功。

#### （2）异步通信

![异步调用](https://rocketmq.apache.org/zh/assets/images/asyncarchi-e7ee18dd77aca472fb80bb2238d9528b.png)

异步消息通信模式下，各子系统之间无需强耦合直接连接，调用方只需要将请求转化成异步事件（消息）发送给中间代理，发送成功即可认为该异步链路调用完成，剩下的工作中间代理会负责将事件可靠通知到下游的调用系统，确保任务执行完成。该中间代理一般就是消息中间件。

异步通信的优势如下：

- 系统拓扑简单。由于调用方和被调用方统一和中间代理通信，系统是星型结构，易于维护和管理。

- 上下游耦合性弱。上下游系统之间弱耦合，结构更灵活，由中间代理负责缓冲和异步恢复。 上下游系统间可以独立升级和变更，不会互相影响。

- 容量削峰填谷。基于消息的中间代理往往具备很强的流量缓冲和整形能力，业务流量高峰到来时不会击垮下游。

### 3、消息传输模型

#### （1）点对点模型

点对点模型也叫队列模型，具有如下特点：

- 消费匿名：消息上下游沟通的唯一的身份就是队列，下游消费者从队列获取消息无法申明独立身份。
- 一对一通信：基于消费匿名特点，下游消费者即使有多个，但都没有自己独立的身份，因此共享队列中的消息，每一条消息都只会被唯一一个消费者处理。因此点对点模型只能实现一对一通信。

#### （2）发布订阅模型

发布订阅模型具有如下特点：

- 消费独立：相比队列模型的匿名消费方式，发布订阅模型中消费方都会具备的身份，一般叫做订阅组（订阅关系），不同订阅组之间相互独立不会相互影响。
- 一对多通信：基于独立身份的设计，同一个主题内的消息可以被多个订阅组处理，每个订阅组都可以拿到全量消息。因此发布订阅模型可以实现一对多通信。





## 四、功能特性

### 1、普通消息

普通消息是 RocketMQ 基本消息功能，支持生产者和消费者的异步解耦通信。

普通消息生命周期：

初始化 => 待消费 => 消费中 => 消费提交 => 消息删除

普通消息仅支持使用MessageType为Normal主题，即普通消息只能发送至类型为普通消息的主题中，发送的消息的类型必须和主题的类型一致。

普通消息支持设置消息索引键、消息过滤标签等信息，用于消息过滤和搜索查找。

使用建议：

发送消息时，建议设置业务上唯一的信息作为索引，方便后续快速定位消息。例如，订单ID，用户ID等。

### 2、定时/延时消息

在分布式定时调度触发、任务超时处理等场景，需要实现精准、可靠的定时事件触发。使用 RocketMQ 的定时消息可以简化定时调度任务的开发逻辑，实现高性能、可扩展、高可靠的定时触发能力。

定时消息生命周期：

初始化 => 定时中 => 待消费 => 消费中 => 消费提交 => 消息删除

RocketMQ 定时消息的定时时长参数精确到毫秒级，定时消息的状态支持持久化存储，系统由于故障重启后，仍支持按照原来设置的定时时间触发消息投递。若存储系统异常重启，可能会导致定时消息投递出现一定延迟。

> 创建延迟主题

```
> mqadmin updateTopic -c DefaultCluster -t DelayTopic -n 127.0.0.1:9876 -a +message.type=DELAY
```

使用建议：

避免将大量定时消息的定时时间设置为同一时刻，到达该时刻后会有大量消息同时需要被处理，造成系统压力过大，导致消息分发延迟，影响定时精度。

### 3、顺序消息

在有序事件处理、撮合交易、数据实时增量同步等场景下，异构系统间需要维持强一致的状态同步，上游的事件变更需要按照顺序传递到下游进行处理。在这类场景下使用 RocketMQ 的顺序消息可以有效保证数据传输的顺序性。

> 保证消息的顺序性
>
> RocketMQ 的消息的顺序性分为两部分，生产顺序性和消费顺序性。

- 生产顺序性：
  - 单一生产者。
  - 串行发送。
- 消费顺序性：
  - 投递顺序。
  - 有限重试。

> 创建 FIFO 主题

```
> mqadmin updateTopic -c DefaultCluster -t FIFOTopic -o true -n 127.0.0.1:9876 -a +message.type=FIFO

  -o  创建顺序消息
```



### 4、事务消息

事务消息是 Apache RocketMQ 提供的一种高级消息类型，支持在分布式场景下保障消息生产和本地事务的最终一致性。

> 事务消息处理流程

![事务消息](https://rocketmq.apache.org/zh/assets/images/transflow-0b07236d124ddb814aeaf5f6b5f3f72c.png)

1. 生产者将消息发送至Apache RocketMQ服务端。
2. Apache RocketMQ服务端将消息持久化成功之后，向生产者返回Ack确认消息已经发送成功，此时消息被标记为"暂不能投递"，这种状态下的消息即为半事务消息。
3. 生产者开始执行本地事务逻辑。
4. 生产者根据本地事务执行结果向服务端提交二次确认结果（Commit或是Rollback），服务端收到确认结果后处理逻辑如下：
   - 二次确认结果为Commit：服务端将半事务消息标记为可投递，并投递给消费者。
   - 二次确认结果为Rollback：服务端将回滚事务，不会将半事务消息投递给消费者。
5. 在断网或者是生产者应用重启的特殊情况下，若服务端未收到发送者提交的二次确认结果，或服务端收到的二次确认结果为Unknown未知状态，经过固定时间后，服务端将对消息生产者即生产者集群中任一生产者实例发起消息回查。 **说明** 服务端回查的间隔时间和最大回查次数，请参见[参数限制](https://rocketmq.apache.org/zh/docs/introduction/03limits)。
6. 生产者收到消息回查后，需要检查对应消息的本地事务执行的最终结果。
7. 生产者根据检查到的本地事务的最终状态再次提交二次确认，服务端仍按照步骤4对半事务消息进行处理。

> 创建事务主题

```
> mqadmin updatetopic -n localhost:9876 -t TestTopic -c DefaultCluster -a +message.type=TRANSACTION
```



### 5、消息发送重试和流控机制



### 6、消费者分类



### 7、消息过滤



### 8、消费者负载均衡



### 9、消费进度管理



### 10、消费重试



### 11、消息存储和清理机制



## 五、部署 & 运维

### 1、部署



### 2、Admin Tool

```
# 显示帮助信息
> mqadmin
```



### 3、RocketMQ Dashboard

[GitHub 地址：](https://github.com/apache/rocketmq-dashboard)

> 前提：启动 RocketMQ

### 1、Docker 镜像安装

```
docker run -d \
  --name rocketmq-dashboard \
  -e "JAVA_OPTS=-Drocketmq.namesrv.addr=127.0.0.1:9876" \
  -p 8080:8080 \
  -t apacherocketmq/rocketmq-dashboard:latest
```

