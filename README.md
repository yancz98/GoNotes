# Golang 技术栈 & 笔记



## Language

- [x] [Go 基础](GoBasic.md)
- [x] [Go 高级](GoAdvanced.md)

- [x] [Python](Python.md)




## Storages

- [x] [MySQL](MySQL_3306.md)

- [ ] [PostgreSQL](PostgreSQL_5432.md)

- [ ] [SQLite](SQLite.md)

- [x] [MongoDB](MongoDB_27017.md)

- [x] [Redis](Redis_6379.md)

- [x] [ElasticSearch](ElasticSearch_9200.md)

- [x] [MinIO](MinIO_9000.md)

- [x] [ETCD](ETCD_2379.md)

- [ ] [ZooKeeper](zookeeper_2181.md)

- [ ] [Consul](Consul.md)

> 常见存储系统排行
>
> https://db-engines.com/en/ranking

| OLTP       | OLAP       | NoSQL         | 大数据    |
| ---------- | ---------- | ------------- | --------- |
| MySQL      | ClickHouse | Redis         | HDFS      |
| PostgreSQL | Hive       | MongoDB       | HBase     |
| Oracle     | Kylin      | Elasticsearch | Cassandra |
|            |            | InfluxDB      | Ceph      |

> Key-Value Stores Comparison Chart

|      | ETCD | Zookeeper | Consul | NewSQL |
| ---- | ---- | --------- | ------ | ------ |
|      |      |           |        |        |



## Messages

- [x] [Kafka](Kafka_9092.md)
- [x] [RocketMQ](RocketMQ_9876.md)
- [ ] [ActiveMQ]()
- [ ] [RabbitMQ](RabbitMQ_4369.md)

> 各消息中间件对比

|                  | Kafka                                                        | RocketMQ                                   | ActiveMQ                                                     | RabbitMQ |
| ---------------- | ------------------------------------------------------------ | ------------------------------------------ | ------------------------------------------------------------ | -------- |
| 协议和规范       | Pull 模式，支持 TCP                                          | Pull 模式，支持 TCP、JMS、OpenMessaging    | Push 模式，支持 OpenWrite、STOMP、AMQP、MQTT、JMS            |          |
| 有序消息         | 确保消息在分区内的顺序                                       | 确保消息的严格排序，并且可以优雅地横向扩展 | Exclusive Consumer 或 Exclusive Queues 可以确保有序          |          |
| 调度消息         | ×                                                            | √                                          | √                                                            |          |
| 批量消息         | 支持，使用异步生产者                                         | 支持，具有同步模式，避免消息丢失           | ×                                                            |          |
| 广播消息         | ×                                                            | √                                          | √                                                            |          |
| 消息过滤         | 支持，可以使用 Kafka Streams 过滤消息                        | 支持，基于 SQL92 的属性过滤器表达式        | √                                                            |          |
| 服务端触发重试   | ×                                                            | √                                          | ×                                                            |          |
| 消息存储         | 高性能文件存储                                               | 高性能和低延迟文件存储                     | 支持使用 JDBC 和高性能日志（如 levelDB、kahaDB）的非常快速的持久化 |          |
| 消息追溯         | 支持的偏移指示                                               | 支持的时间戳和偏移量两个表示               | √                                                            |          |
| 消息优先级       | ×                                                            | ×                                          | √                                                            |          |
| 高可用和故障转移 | 支持，需要 ZooKeeper 服务器                                  | 支持，主从式，无需其他套件                 | 支持，取决于存储，如果使用 levelDB 则需要 ZooKeeper 服务器   |          |
| 消息跟踪         | ×                                                            | √                                          | ×                                                            |          |
| 配置             | Kafka 使用键值对格式进行配置。这些值可以从文件或以编程方式提供。 | 开箱即用，用户只需注意几个配置             | 默认配置为低级别，用户需要优化配置参数                       |          |
| 管理运营工具     | 支持，使用终端命令暴露核心指标                               | 支持丰富的 Web 和终端命令以公开核心指标    | √                                                            |          |



## Cloud Native

- [x] [Docker](Docker.md)

- [ ] [Kubernetes](https://kubernetes.io/)

- [x] [DevOps](DevOps.md)

  - [x] [CICD (GitLab)](GitLab.md)

- [ ] 容器编排（Docker / K8S）

- [ ] 服务治理

- [ ] 服务注册发现（Consul、ETCD）

- [ ] 配置中心（Apollo、ETCD）

- [ ] 负载均衡（Nginx）

- [ ] 服务网关（Envoy）

- [x] [可观测性](Observability.md)

  - 日志中心（ELK）

  - 监控&报警（Prometheus + Grafana）

  - 链路追踪（OpenTelemetry、Zipkin）

- [x] [微服务](MicroService.md)



## Internal Skill

- [x] [数据结构与算法](Algorithm.md)
- [x] [设计模式](DesignPattern.md)
- [x] [分布式](Distributed.md)



## Tools

- [x] [Linux](Linux.md)

- [x] [Git](Git.md)

- [x] [Nginx](Nginx_80.md)

  

