### 1、介绍

GitHub：https://github.com/alibaba/canal

Canal 组件是一个基于 MySQL 数据库增量日志（binlog）解析，提供增量数据订阅和消费，支持将增量数据投递到下游消费者（如：Kafka、RocketMQ 等）或者存储（ES、HBase）的组件。

> 应用场景
>

![应用场景](https://img-blog.csdnimg.cn/20191104101735947.png)

### 2、准备

#### （1）Docker 安装 MySQL

```sh
# 需要先创建 my.cnf 文件（内容见下）
$ docker run -d \
  --name mysql \
  -p 3306:3306 \
  -v /data/docker-run/mysql/my.cnf:/etc/my.cnf \
  -v /data/docker-run/mysql/data:/var/lib/mysql \
  -e MYSQL_ROOT_PASSWORD=root \
  mysql:5.7
```

#### （2）开启 MySQL 的 binlog 日志

> 修改 my.cnf 开启 binlog

```ini
[mysqld]

# 配置 MySQL replaction 需要定义，不要和 canal 的 slaveId 重复
server-id=1

# binlog 日志文件名
log-bin=mysql-bin
# binlog 日志格式：row|mixed|statement
binlog_format=row
# 指定开启 binlog 日志的数据库，不指定时所有数据库都开启
binlog-do-db=canal_sync
```

> 验证

```sh
$ docker exec -it mysql /bin/bash
bash-4.2# mysql -uroot -p

mysql> SHOW VARIABLES LIKE "%log_bin%";
+---------------------------------+--------------------------------+
| Variable_name                   | Value                          |
+---------------------------------+--------------------------------+
| log_bin                         | ON                             |
| log_bin_basename                | /var/lib/mysql/mysql-bin       |
| log_bin_index                   | /var/lib/mysql/mysql-bin.index |
| log_bin_trust_function_creators | OFF                            |
| log_bin_use_v1_row_events       | OFF                            |
| sql_log_bin                     | ON                             |
+---------------------------------+--------------------------------+
6 rows in set (0.00 sec)

mysql> SHOW VARIABLES LIKE 'binlog_format%';
+---------------+-------+
| Variable_name | Value |
+---------------+-------+
| binlog_format | ROW   |
+---------------+-------+
1 row in set (0.00 sec)
```

#### （3）授权 canal 账号权限

```sh
# 创建 canal 用户，密码 canal 
mysql> CREATE USER canal IDENTIFIED BY 'canal';
Query OK, 0 rows affected (0.00 sec)

# 授权 canal 账号具有作为 MySQL slave 的权限
mysql> GRANT SELECT, REPLICATION SLAVE, REPLICATION CLIENT ON *.* TO 'canal'@'%';
Query OK, 0 rows affected (0.01 sec)

# 授权全部权限
# GRANT ALL PRIVILEGES ON *.* TO 'canal'@'%' ;

# 刷新权限
mysql> FLUSH PRIVILEGES;
Query OK, 0 rows affected (0.01 sec)
```

#### （4）建库/表（测试用）

```sql
# 建库
CREATE DATABASE `canal_sync` CHARACTER SET 'utf8mb4' COLLATE 'utf8mb4_general_ci';

# 用户表
CREATE TABLE `user`  (
  `id` int(10) UNSIGNED NOT NULL AUTO_INCREMENT,
  `user` varchar(255) CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci NOT NULL DEFAULT '',
  `passwd` varchar(255) CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci NOT NULL DEFAULT '',
  `role` varchar(255) CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci NOT NULL,
  `created_at` timestamp(0) NOT NULL DEFAULT CURRENT_TIMESTAMP(0),
  PRIMARY KEY (`id`) USING BTREE
) ENGINE = InnoDB CHARACTER SET = utf8mb4 COLLATE = utf8mb4_general_ci ROW_FORMAT = Dynamic;

# 角色表
CREATE TABLE `role`  (
  `id` int(10) UNSIGNED NOT NULL AUTO_INCREMENT,
  `role` varchar(255) CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci NOT NULL DEFAULT '',
  `created_at` timestamp(0) NOT NULL DEFAULT CURRENT_TIMESTAMP(0),
  PRIMARY KEY (`id`) USING BTREE
) ENGINE = InnoDB CHARACTER SET = utf8mb4 COLLATE = utf8mb4_general_ci ROW_FORMAT = Dynamic;
```



### 3、Canal Admin

 安装  Canal Admin 为了方便管理 canal-server 和 instance。

#### （1）下载

```sh
$ wget https://github.com/alibaba/canal/releases/download/canal-1.1.7/canal.admin-1.1.7.tar.gz
$ mkdir /data/canal-admin
$ tar zxvf canal.admin-1.1.7.tar.gz -C /data/canal-admin/
$ cd /data/canal-admin
$ tree
.
├── bin
│   ├── restart.sh
│   ├── startup.bat
│   ├── startup.sh
│   └── stop.sh
├── conf
│   ├── application.yml             # canal_manager 的主配置文件
│   ├── canal_manager.sql           # canal_manager 的数据库初始数据
│   ├── canal-template.properties
│   ├── instance-template.properties
│   ├── logback.xml
│   └── public                      # Web 页面相关
│       ├── avatar.gif
│       ├── index.html
│       ├── logo.png
│       └── static
│           ├── css
│           │   └── *.css
│           ├── fonts
│           │   ├── element-icons.2fad952a.woff
│           │   └── element-icons.6f0a7632.ttf
│           ├── img
│           │   ├── 404.a57b6f31.png
│           │   └── 404_cloud.0f4bc32b.png
│           └── js
│               └── *.js
├── lib
│   └── *.jar
└── logs
```

#### （2）配置

> conf/application.yml

```yaml
server:
  port: 8089
spring:
  jackson:
    date-format: yyyy-MM-dd HH:mm:ss
    time-zone: GMT+8

spring.datasource:
  # 配置这几个数据库相关参数就行（基本不用动）
  address: 127.0.0.1:3306
  database: canal_manager
  username: root   # 这个账号需要有 Insert 权限，否则 Canal-Admin 无法管理
  password: root
  driver-class-name: com.mysql.jdbc.Driver
  url: jdbc:mysql://${spring.datasource.address}/${spring.datasource.database}?useUnicode=true&characterEncoding=UTF-8&useSSL=false
  hikari:
    maximum-pool-size: 30
    minimum-idle: 1

canal:
  adminUser: admin
  adminPasswd: admin
```

#### （3）导入初始数据

```sh
$ mysql -h127.0.0.1 -uroot -p

# 导入初始化SQL
mysql> source conf/canal_manager.sql
```

#### （4）启动

```sh
$ ./bin/startup.sh

$ jps
368329 Jps
368265 CanalAdminApplication

# 查看日志
$ cat logs/admin.log
2024-08-22 16:22:17.734 [main] INFO  com.alibaba.otter.canal.admin.CanalAdminApplication - Starting CanalAdminApplication using Java 1.8.0_422 on 172.16.56.53 with PID 368265 (/data/canal-admin/lib/canal.admin-web-1.1.7.jar started by dev in /data/canal-admin/bin)
2024-08-22 16:22:17.747 [main] INFO  com.alibaba.otter.canal.admin.CanalAdminApplication - No active profile set, falling back to default profiles: default
2024-08-22 16:22:18.310 [main] INFO  o.s.boot.web.embedded.tomcat.TomcatWebServer - Tomcat initialized with port(s): 8089 (http)
2024-08-22 16:22:18.316 [main] INFO  org.apache.coyote.http11.Http11NioProtocol - Initializing ProtocolHandler ["http-nio-8089"]
2024-08-22 16:22:18.317 [main] INFO  org.apache.catalina.core.StandardService - Starting service [Tomcat]
2024-08-22 16:22:18.317 [main] INFO  org.apache.catalina.core.StandardEngine - Starting Servlet engine: [Apache Tomcat/9.0.52]
2024-08-22 16:22:18.353 [main] INFO  o.a.catalina.core.ContainerBase.[Tomcat].[localhost].[/] - Initializing Spring embedded WebApplicationContext
2024-08-22 16:22:18.353 [main] INFO  o.s.b.w.s.context.ServletWebServerApplicationContext - Root WebApplicationContext: initialization completed in 581 ms
2024-08-22 16:22:18.460 [main] INFO  io.ebean.EbeanVersion - ebean version: 11.41.1
2024-08-22 16:22:18.466 [main] INFO  io.ebean.config.properties.LoadContext - loaded properties from [application.yml]
2024-08-22 16:22:18.481 [main] INFO  com.zaxxer.hikari.HikariDataSource - HikariPool-1 - Starting...
2024-08-22 16:22:18.483 [main] WARN  com.zaxxer.hikari.util.DriverDataSource - Registered driver with driverClassName=com.mysql.jdbc.Driver was not found, trying direct instantiation.
2024-08-22 16:22:18.611 [main] INFO  com.zaxxer.hikari.HikariDataSource - HikariPool-1 - Start completed.
2024-08-22 16:22:18.612 [main] WARN  io.ebean.internal.DefaultContainer - DataSource [ebeanServer] has autoCommit defaulting to true!
2024-08-22 16:22:18.622 [main] INFO  io.ebean.internal.DefaultContainer - DatabasePlatform name:ebeanServer platform:mysql
2024-08-22 16:22:18.883 [main] INFO  o.s.b.a.web.servlet.WelcomePageHandlerMapping - Adding welcome page: class path resource [public/index.html]
2024-08-22 16:22:18.985 [main] INFO  org.apache.coyote.http11.Http11NioProtocol - Starting ProtocolHandler ["http-nio-8089"]
2024-08-22 16:22:18.996 [main] INFO  o.s.boot.web.embedded.tomcat.TomcatWebServer - Tomcat started on port(s): 8089 (http) with context path ''
2024-08-22 16:22:19.005 [main] INFO  com.alibaba.otter.canal.admin.CanalAdminApplication - Started CanalAdminApplication in 11.474 seconds (JVM running for 11.74)

# 访问：http://127.0.0.1:8089
# 默认密码：admin/123456
```

#### （5）Docker（推荐）

```sh
# 一步到位
$ docker run -d \
  --restart=always \
  --name canal-admin \
  -p 8089:8089 \
  canal/canal-admin
  
# 访问：http://127.0.0.1:8089
# 默认密码：admin/123456
```



### 4、Canal Server

#### （1）下载

```sh
# 下载 deployer
$ wget https://github.com/alibaba/canal/releases/download/canal-1.1.7/canal.deployer-1.1.7.tar.gz
$ mkdir /data/canal-server
$ tar zxvf canal.deployer-1.1.7.tar.gz -C /data/canal-server
$ cd /data/canal-server
$ tree
.
├── bin                  # 可执行文件目录
│   ├── restart.sh       # 重启 canal
│   ├── startup.bat
│   ├── startup.sh       # 启动 canal
│   └── stop.sh          # 停止 canal
├── conf                         # 配置目录 
│   ├── canal_local.properties
│   ├── canal.properties         # canal 的主要配置文件
│   ├── example                  # 监听的目标数据库（可定义多个目录监听多个库，如 database_*）
│   │   └── instance.properties  # 数据库实例的配置
│   ├── logback.xml
│   ├── metrics
│   │   └── Canal_instances_tmpl.json
│   └── spring
│       ├── base-instance.xml
│       ├── default-instance.xml
│       ├── file-instance.xml
│       ├── group-instance.xml
│       ├── memory-instance.xml
│       └── tsdb
│           ├── h2-tsdb.xml
│           ├── mysql-tsdb.xml
│           ├── sql
│           │   └── create_table.sql
│           └── sql-map
│               ├── sqlmap-config.xml
│               ├── sqlmap_history.xml
│               └── sqlmap_snapshot.xml
├── lib
│   └── *.jar
├── logs
└── plugin
    ├── connector.kafka-1.1.7-jar-with-dependencies.jar
    ├── connector.pulsarmq-1.1.7-jar-with-dependencies.jar
    ├── connector.rabbitmq-1.1.7-jar-with-dependencies.jar
    └── connector.rocketmq-1.1.7-jar-with-dependencies.jar
```

#### （2）配置 Canal

> conf/canal.properties

```properties
#################################################
#########       common argument     #############
#################################################
# tcp bind ip
canal.ip =
# register ip to zookeeper
canal.register.ip =
canal.port = 11111                # Canal 服务时监听的端口
canal.metrics.pull.port = 11112   # Prometheus 指标 /metrics 监听的端口
# canal instance user/passwd
# canal.user = canal
# canal.passwd = E3619321C1A937C46A0D8BD1DAC39F93B27D4458

# canal admin config （用 conf/canal_local.properties 配置覆盖）
#canal.admin.manager = 127.0.0.1:8089
canal.admin.port = 11110
canal.admin.user = admin
canal.admin.passwd = 4ACFE3202A5FF5CF467898FC58AAB1D615029441
# admin auto register
#canal.admin.register.auto = true
#canal.admin.register.cluster =
#canal.admin.register.name =

canal.zkServers =
# flush data to zk
canal.zookeeper.flush.period = 1000
canal.withoutNetty = false
# tcp, kafka, rocketMQ, rabbitMQ, pulsarMQ
#
# 服务模式：
#  - tcp：启动一个 TCP 服务，通过 client 去消费
#  - MQ ：直接发送到 MQ
canal.serverMode = tcp
# flush meta cursor/parse position to file
canal.file.data.dir = ${canal.conf.dir}
canal.file.flush.period = 1000
## memory store RingBuffer size, should be Math.pow(2,n)
canal.instance.memory.buffer.size = 16384
## memory store RingBuffer used memory unit size , default 1kb
canal.instance.memory.buffer.memunit = 1024
## meory store gets mode used MEMSIZE or ITEMSIZE
canal.instance.memory.batch.mode = MEMSIZE
canal.instance.memory.rawEntry = true

## detecing config
canal.instance.detecting.enable = false
#canal.instance.detecting.sql = insert into retl.xdual values(1,now()) on duplicate key update x=now()
canal.instance.detecting.sql = select 1
canal.instance.detecting.interval.time = 3
canal.instance.detecting.retry.threshold = 3
canal.instance.detecting.heartbeatHaEnable = false

# support maximum transaction size, more than the size of the transaction will be cut into multiple transactions delivery
canal.instance.transaction.size =  1024
# mysql fallback connected to new master should fallback times
canal.instance.fallbackIntervalInSeconds = 60

# network config
canal.instance.network.receiveBufferSize = 16384
canal.instance.network.sendBufferSize = 16384
canal.instance.network.soTimeout = 30

# binlog filter config
canal.instance.filter.druid.ddl = true
canal.instance.filter.query.dcl = false
canal.instance.filter.query.dml = false
canal.instance.filter.query.ddl = false
canal.instance.filter.table.error = false
canal.instance.filter.rows = false
canal.instance.filter.transaction.entry = false
canal.instance.filter.dml.insert = false
canal.instance.filter.dml.update = false
canal.instance.filter.dml.delete = false

# binlog format/image check
canal.instance.binlog.format = ROW,STATEMENT,MIXED
canal.instance.binlog.image = FULL,MINIMAL,NOBLOB

# binlog ddl isolation
canal.instance.get.ddl.isolation = false

# parallel parser config
canal.instance.parser.parallel = true
## concurrent thread number, default 60% available processors, suggest not to exceed Runtime.getRuntime().availableProcessors()
#canal.instance.parser.parallelThreadSize = 16
## disruptor ringbuffer size, must be power of 2
canal.instance.parser.parallelBufferSize = 256

# table meta tsdb info
canal.instance.tsdb.enable = true
canal.instance.tsdb.dir = ${canal.file.data.dir:../conf}/${canal.instance.destination:}
canal.instance.tsdb.url = jdbc:h2:${canal.instance.tsdb.dir}/h2;CACHE_SIZE=1000;MODE=MYSQL;
canal.instance.tsdb.dbUsername = canal
canal.instance.tsdb.dbPassword = canal
# dump snapshot interval, default 24 hour
canal.instance.tsdb.snapshot.interval = 24
# purge snapshot expire , default 360 hour(15 days)
canal.instance.tsdb.snapshot.expire = 360

#################################################
#########       destinations        #############
#################################################

# 监听目标数据库，多个时用逗号 `,` 拼接，
# 对应 conf/databese_* 的名称
canal.destinations = example
# conf root dir
canal.conf.dir = ../conf
# 每隔 5s 自动扫描 conf/database_* 下配置的 instance
# auto scan instance dir add/remove and start/stop instance
canal.auto.scan = true
canal.auto.scan.interval = 5
# set this value to 'true' means that when binlog pos not found, skip to latest.
# WARN: pls keep 'false' in production env, or if you know what you want.
canal.auto.reset.latest.pos.mode = false

canal.instance.tsdb.spring.xml = classpath:spring/tsdb/h2-tsdb.xml
#canal.instance.tsdb.spring.xml = classpath:spring/tsdb/mysql-tsdb.xml

canal.instance.global.mode = spring
canal.instance.global.lazy = false
canal.instance.global.manager.address = ${canal.admin.manager}
#canal.instance.global.spring.xml = classpath:spring/memory-instance.xml
canal.instance.global.spring.xml = classpath:spring/file-instance.xml
#canal.instance.global.spring.xml = classpath:spring/default-instance.xml

##################################################
#########         MQ Properties      #############
##################################################
# aliyun ak/sk , support rds/mq
canal.aliyun.accessKey =
canal.aliyun.secretKey =
canal.aliyun.uid=

canal.mq.flatMessage = true
canal.mq.canalBatchSize = 50
canal.mq.canalGetTimeout = 100
# Set this value to "cloud", if you want open message trace feature in aliyun.
canal.mq.accessChannel = local

canal.mq.database.hash = true
canal.mq.send.thread.size = 30
canal.mq.build.thread.size = 8

##################################################
#########            Kafka           #############
##################################################
kafka.bootstrap.servers = 127.0.0.1:9092
kafka.acks = all
kafka.compression.type = none
kafka.batch.size = 16384
kafka.linger.ms = 1
kafka.max.request.size = 1048576
kafka.buffer.memory = 33554432
kafka.max.in.flight.requests.per.connection = 1
kafka.retries = 0

kafka.kerberos.enable = false
kafka.kerberos.krb5.file = ../conf/kerberos/krb5.conf
kafka.kerberos.jaas.file = ../conf/kerberos/jaas.conf

# sasl demo
# kafka.sasl.jaas.config = org.apache.kafka.common.security.scram.ScramLoginModule required \\n username=\"alice\" \\npassword="alice-secret\";
# kafka.sasl.mechanism = SCRAM-SHA-512
# kafka.security.protocol = SASL_PLAINTEXT

##################################################
#########           RocketMQ         #############
##################################################
rocketmq.producer.group = test
rocketmq.enable.message.trace = false
rocketmq.customized.trace.topic =
rocketmq.namespace =
rocketmq.namesrv.addr = 127.0.0.1:9876
rocketmq.retry.times.when.send.failed = 0
rocketmq.vip.channel.enabled = false
rocketmq.tag =

##################################################
#########           RabbitMQ         #############
##################################################
rabbitmq.host =
rabbitmq.virtual.host =
rabbitmq.exchange =
rabbitmq.username =
rabbitmq.password =
rabbitmq.deliveryMode =

##################################################
#########             Pulsar         #############
##################################################
pulsarmq.serverUrl =
pulsarmq.roleToken =
pulsarmq.topicTenantPrefix =
```

> conf/canal_local.properties
>
> 安装 Canal-Admin 后，启用这个配置，Canal-Server 会自动添加到 Admin 中，便于管理。
>
> 在启动 Canal-Server 时使用 local 参数指定配置：`sh bin/startup.sh local`。

```properties
# register ip
canal.register.ip =

# canal admin config
canal.admin.manager = 127.0.0.1:8089
canal.admin.port = 11110
canal.admin.user = admin
canal.admin.passwd = 4ACFE3202A5FF5CF467898FC58AAB1D615029441   # MD5(admin)
# admin auto register
canal.admin.register.auto = true
canal.admin.register.cluster =
canal.admin.register.name =
```

#### （3）配置 instance

安装 Canal-Admin 后，这里可以先不用配置，在 Canal-Admin 中的 Instance 管理页面下添加更方便。

> conf/example/instance.properties

```properties
#################################################
## mysql serverId , v1.0.26+ will autoGen
# canal.instance.mysql.slaveId=0

# enable gtid use true/false
canal.instance.gtidon=false

# position info
canal.instance.master.address=127.0.0.1:3306    # 监听目标 MySQL 的位置
canal.instance.master.journal.name=
canal.instance.master.position=
canal.instance.master.timestamp=
canal.instance.master.gtid=

# rds oss binlog
canal.instance.rds.accesskey=
canal.instance.rds.secretkey=
canal.instance.rds.instanceId=

# table meta tsdb info
canal.instance.tsdb.enable=true
#canal.instance.tsdb.url=jdbc:mysql://127.0.0.1:3306/canal_tsdb
#canal.instance.tsdb.dbUsername=canal
#canal.instance.tsdb.dbPassword=canal

#canal.instance.standby.address =
#canal.instance.standby.journal.name =
#canal.instance.standby.position =
#canal.instance.standby.timestamp =
#canal.instance.standby.gtid=

# username/password
# MySQL 的账号密码（需要 slave 权限）
canal.instance.dbUsername=canal
canal.instance.dbPassword=canal
canal.instance.connectionCharset = UTF-8
# enable druid Decrypt database password
canal.instance.enableDruid=false
#canal.instance.pwdPublicKey=MFwwDQYJKoZIhvcNAQEBBQADSwAwSAJBALK4BUxdDltRRE5/zXpVEVPUgunvscYFtEip3pmLlhrWpacX7y7GCMo2/JM6LeHmiiNdH1FWgGCpUfircSwlWKUCAwEAAQ==

# table regex
########## 配置要扫描的数据库及表名 ##########
# 注意 database 与 table 之间的连接符为： `\\.`
# canal.instance.filter.regex=database_*\\.table_*
canal.instance.filter.regex=.*\\..*
# table black regex
canal.instance.filter.black.regex=mysql\\.slave_.*
# table field filter(format: schema1.tableName1:field1/field2,schema2.tableName2:field1/field2)
#canal.instance.filter.field=test1.t_product:id/subject/keywords,test2.t_company:id/name/contact/ch
# table field black filter(format: schema1.tableName1:field1/field2,schema2.tableName2:field1/field2)
#canal.instance.filter.black.field=test1.t_product:subject/product_image,test2.t_company:id/name/contact/ch

# mq config
canal.mq.topic=example
# dynamic topic route by schema or table regex
#canal.mq.dynamicTopic=mytest1.user,topic2:mytest2\\..*,.*\\..*
canal.mq.partition=0
# hash partition config
#canal.mq.enableDynamicQueuePartition=false
#canal.mq.partitionsNum=3
#canal.mq.dynamicTopicPartitionNum=test.*:4,mycanal:6
#canal.mq.partitionHash=test.table:id^name,.*\\..*
#
# multi stream for polardbx
canal.instance.multi.stream.on=false
#################################################
```

#### （4）启动

```sh
# 没有 Canal-Admin 时
$ ./bin/startup.sh

# 有 Canal-Admin 时
$ ./bin/startup.sh local

$ jps
368705 CanalLauncher
368265 CanalAdminApplication
368906 Jps

# 查看 server 日志
$ cat logs/canal/canal.log
2024-08-22 16:26:03.503 [main] INFO  com.alibaba.otter.canal.deployer.CanalLauncher - ## set default uncaught exception handler
2024-08-22 16:26:03.506 [main] INFO  com.alibaba.otter.canal.deployer.CanalLauncher - ## load canal configurations
2024-08-22 16:26:03.511 [main] INFO  com.alibaba.otter.canal.deployer.CanalStarter - ## start the canal server.
2024-08-22 16:26:03.528 [main] INFO  com.alibaba.otter.canal.deployer.CanalController - ## start the canal server[172.17.0.1(172.17.0.1):11111]
2024-08-22 16:26:04.134 [main] INFO  com.alibaba.otter.canal.deployer.CanalStarter - ## the canal server is running now ......


# 查看 instance 日志
$ cat logs/example/example.log
2024-08-22 16:26:03.780 [main] INFO  c.a.otter.canal.instance.spring.CanalInstanceWithSpring - start CannalInstance for 1-example
2024-08-22 16:26:04.116 [main] WARN  c.a.o.canal.parse.inbound.mysql.dbsync.LogEventConvert - --> init table filter : ^.*\..*$
2024-08-22 16:26:04.116 [main] WARN  c.a.o.canal.parse.inbound.mysql.dbsync.LogEventConvert - --> init table black filter : ^mysql\.slave_.*$
2024-08-22 16:26:04.118 [main] INFO  c.a.otter.canal.instance.core.AbstractCanalInstance - start successful....
2024-08-22 16:26:14.174 [destination = example , address = /127.0.0.1:3306 , EventParser] WARN  c.a.o.c.p.inbound.mysql.rds.RdsBinlogEventParserProxy - ---> begin to find start position, it will be long time for reset or first position
2024-08-22 16:26:14.174 [destination = example , address = /127.0.0.1:3306 , EventParser] WARN  c.a.o.c.p.inbound.mysql.rds.RdsBinlogEventParserProxy - prepare to find start position just show master status
2024-08-22 16:26:14.606 [destination = example , address = /127.0.0.1:3306 , EventParser] WARN  c.a.o.c.p.inbound.mysql.rds.RdsBinlogEventParserProxy - ---> find start position successfully, EntryPosition[included=false,journalName=mysql-bin.000001,position=1761,serverId=1,gtid=,timestamp=1724312575000] cost : 428ms , the next step is binlog dump
```

#### （5）Docker（推荐）

```sh
# Server 会自动添加到 Admin 中
# 在 Instance 管理界面中配置 Instance 更方便
$ docker run -d \
  --restart=always \
  --name=canal-server \
  -p 9100:9100 \
  -p 11110:11110 \
  -p 11111:11111 \
  -p 11112:11112 \
  -v /data/docker-run/canal-server/logs:/home/admin/canal-server/logs \
  -e canal.admin.manager=172.16.56.53:8089 \
  -e canal.admin.port=11110 \
  -e canal.admin.user=admin \
  -e canal.admin.passwd=4ACFE3202A5FF5CF467898FC58AAB1D615029441 \
  canal/canal-server
```



### 5、Canal Adapter

Adapter 提供了数据落地到各种客户端的适配器，并支持以下功能：

- 数据同步管理的 API 接口。
- 可作为调试 DEMO 的 logger 适配器。
- 同步到关系型数据库（MySQL 同步到 MySQL），并提供全量数据同步功能（ETL）。
- 同步到 HBase（MySQL 同步到 HBase），并提供全量数据同步功能（ETL）。
- 同步到 ES（MySQL 同步到 ES），并提供全量数据同步功能（ETL）。

#### （1）下载

```sh
$ wget https://github.com/alibaba/canal/releases/download/canal-1.1.7/canal.adapter-1.1.7.tar.gz
$ mkdir /data/canal-adapter
$ tar -zxvf canal.adapter-1.1.7.tar.gz -C /data/canal-adapter/
$ cd /data/canal-adapter
$ tree
.
├── bin                         # 可执行文件目录
│   ├── restart.sh
│   ├── startup.bat
│   ├── startup.sh
│   └── stop.sh
├── conf                        # 配置目录
│   ├── application.yml         # Adapter 主配置文件
│   ├── bootstrap.yml
│   ├── es6
│   │   ├── biz_order.yml
│   │   ├── customer.yml
│   │   └── mytest_user.yml
│   ├── es7
│   │   ├── biz_order.yml
│   │   ├── customer.yml
│   │   └── mytest_user.yml
│   ├── es8                      # ES8：MySQL_table 到 ES_index 映射配置目录
│   │   ├── biz_order.yml          # 每个文件代表一个要同步的表映射
│   │   ├── customer.yml
│   │   └── mytest_user.yml
│   ├── hbase
│   │   └── mytest_person2.yml
│   ├── kudu
│   │   └── kudutest_user.yml
│   ├── logback.xml
│   ├── META-INF
│   │   └── spring.factories
│   ├── rdb
│   │   └── mytest_user.yml
│   └── tablestore
│       └── test.yml
├── lib
│   └── *.jar
├── logs
└── plugin
    ├── client-adapter.es6x-1.1.7-jar-with-dependencies.jar
    ├── client-adapter.es7x-1.1.7-jar-with-dependencies.jar
    ├── client-adapter.es8x-1.1.7-jar-with-dependencies.jar
    ├── client-adapter.hbase-1.1.7-jar-with-dependencies.jar
    ├── client-adapter.logger-1.1.7-jar-with-dependencies.jar
    ├── client-adapter.rdb-1.1.7-jar-with-dependencies.jar
    ├── client-adapter.tablestore-1.1.7-jar-with-dependencies.jar
    ├── connector.kafka-1.1.7-jar-with-dependencies.jar
    ├── connector.pulsarmq-1.1.7-jar-with-dependencies.jar
    ├── connector.rabbitmq-1.1.7-jar-with-dependencies.jar
    ├── connector.rocketmq-1.1.7-jar-with-dependencies.jar
    └── connector.tcp-1.1.7-jar-with-dependencies.jar
```

#### （2）配置

> conf/application.yml

```yaml
server:
  port: 8081
spring:
  jackson:
    date-format: yyyy-MM-dd HH:mm:ss
    time-zone: GMT+8
    default-property-inclusion: non_null

canal.conf:
  # ClientAdapter 数据订阅的方式有两种，
  #  - 通过 TCP 直连 canal-server
  #  - 订阅 kafka|rocketMQ|rabbitMQ 的消息
  mode: tcp          # tcp kafka rocketMQ rabbitMQ
  flatMessage: true
  zookeeperHosts:
  syncBatchSize: 1000
  retries: -1
  timeout:
  accessKey:
  secretKey:
  # 根据 mode 修改对应配置
  consumerProperties:
    # canal tcp consumer
    canal.tcp.server.host: 127.0.0.1:11111   # 选 TCP mode 时修改这里
    canal.tcp.zookeeper.hosts:
    canal.tcp.batch.size: 500
    canal.tcp.username:
    canal.tcp.password:
    # kafka consumer
    kafka.bootstrap.servers: 127.0.0.1:9092
    kafka.enable.auto.commit: false
    kafka.auto.commit.interval.ms: 1000
    kafka.auto.offset.reset: latest
    kafka.request.timeout.ms: 40000
    kafka.session.timeout.ms: 30000
    kafka.isolation.level: read_committed
    kafka.max.poll.records: 1000
    # rocketMQ consumer
    rocketmq.namespace:
    rocketmq.namesrv.addr: 127.0.0.1:9876
    rocketmq.batch.size: 1000
    rocketmq.enable.message.trace: false
    rocketmq.customized.trace.topic:
    rocketmq.access.channel:
    rocketmq.subscribe.filter:
    # rabbitMQ consumer
    rabbitmq.host:
    rabbitmq.virtual.host:
    rabbitmq.username:
    rabbitmq.password:
    rabbitmq.resource.ownerId:

# 使用远程（MySQL）的配置，
# 配置项已迁移至 conf/bootstrap.yml
#  srcDataSources:
#    defaultDS:
#      url: jdbc:mysql://127.0.0.1:3306/mytest?useUnicode=true # 需要修改
#      username: root
#      password: 121212
  canalAdapters:
    # ==========================================================================
    # 这里的 instance_name 一定要用 Canal-Admin 中 Instance 管理列表下【启动】状态的实例
    # ==========================================================================
  - instance: example # canal instance Name or mq topic name
    # 一份数据可以被多个 group 同时消费，多个 group 之间是并行执行的
    groups:
    # 一个 group 内部是串行执行多个 outerAdapters 的
    - groupId: g1
      outerAdapters:
      # 第 1 个适配器：logger 适配器
      # 将收到的变更事件输出到日志文件：`logs/adapter/adapter.log`
      - name: logger
      
      # 第 N 个适配器...
#      - name: rdb
#        key: mysql1
#        properties:
#          jdbc.driverClassName: com.mysql.jdbc.Driver
#          jdbc.url: jdbc:mysql://127.0.0.1:3306/mytest2?useUnicode=true
#          jdbc.username: root
#          jdbc.password: 121212
#          druid.stat.enable: false
#          druid.stat.slowSqlMillis: 1000
#      - name: rdb
#        key: oracle1
#        properties:
#          jdbc.driverClassName: oracle.jdbc.OracleDriver
#          jdbc.url: jdbc:oracle:thin:@localhost:49161:XE
#          jdbc.username: mytest
#          jdbc.password: m121212
#      - name: rdb
#        key: postgres1
#        properties:
#          jdbc.driverClassName: org.postgresql.Driver
#          jdbc.url: jdbc:postgresql://localhost:5432/postgres
#          jdbc.username: postgres
#          jdbc.password: 121212
#          threads: 1
#          commitSize: 3000
#      - name: hbase
#        properties:
#          hbase.zookeeper.quorum: 127.0.0.1
#          hbase.zookeeper.property.clientPort: 2181
#          zookeeper.znode.parent: /hbase
#      - name: es
#        hosts: 127.0.0.1:9300 # 127.0.0.1:9200 for rest mode
#        properties:
#          mode: transport # or rest
#          # security.ca.path: /etc/es8/ca.crt
#          # security.auth: test:123456 #  only used for rest mode
#          cluster.name: elasticsearch
#      - name: kudu
#        key: kudu
#        properties:
#          kudu.master.address: 127.0.0.1 # ',' split multi address
#      - name: phoenix
#        key: phoenix
#        properties:
#          jdbc.driverClassName: org.apache.phoenix.jdbc.PhoenixDriver
#          jdbc.url: jdbc:phoenix:127.0.0.1:2181:/hbase/db
#          jdbc.username:
#          jdbc.password:
#      - name: clickhouse
#        key: clickhouse1
#        properties:
#          jdbc.driverClassName: ru.yandex.clickhouse.ClickHouseDriver
#          jdbc.url: jdbc:clickhouse://127.0.0.1:8123/default
#          jdbc.username: default
#          jdbc.password: 123456
#          batchSize: 3000
#          scheduleTime: 600   # second unit
#          threads: 3          # parallel threads
```

> conf/bootstrap.yml

```properties
canal:
  manager:
    jdbc:
      url: jdbc:mysql://127.0.0.1:3306/canal_manager?useUnicode=true&characterEncoding=UTF-8
      username: canal
      password: canal
```

使用 [canal_manager.sql](https://raw.githubusercontent.com/alibaba/canal/master/admin/admin-web/src/main/resources/canal_manager.sql) 脚本建表并初始化 Demo 数据，其中 canal_config 表 id=2 的数据对应 adapter 下的application.yml 文件，canal_adapter_config 表对应每个 adapter 的子配置文件。

可以将本地 application.yml 文件和其他子配置文件删除或清空， 启动工程将自动从远程加载配置。

修改 mysql 中的配置信息后会自动刷新到本地动态加载相应的实例或者应用。

#### （3）数据同步管理 API

```sh
# 查询所有订阅同步的 canal instance 或 MQ topic
$ curl http://127.0.0.1:8081/destinations
[{"destination":"example","status":"on"}]

# [PUT] 修改实例 example 的数据同步开关：on|off
$ curl http://127.0.0.1:8081/syncSwitch/example/on -X PUT
{"code":20000,"message":"实例: example 开启同步成功"}

# [GET] 查看实例 example 的数据同步开关状态
#  on   开启同步
#  off  关闭同步
$ curl http://127.0.0.1:8081/syncSwitch/example
{"stauts":"on"}

# [POST] 手动导入数据到指定类型的库（ETL）
# 如果 params 参数为空则全表导入, 参数对应的查询条件在配置中的 etlCondition 指定
# API: /etl/<outerAdapters_name>/<table_name>
$ curl http://127.0.0.1:8081/etl/hbase/mytest_person2.yml -X POST -d "params=2018-10-21 00:00:00"

# [GET] 查看数据库/表的总数据
# API: /count/<outerAdapters_name>/<table_name>
$ curl http://127.0.0.1:8081/count/hbase/mytest_person2.yml
```

### 6、Logger 适配器

#### （1）配置 Adapter

> conf/application.yml

```yaml
server:
  port: 8081
spring:
  jackson:
    date-format: yyyy-MM-dd HH:mm:ss
    time-zone: GMT+8
    default-property-inclusion: non_null

canal.conf:
  # ClientAdapter 数据订阅的方式有两种，
  #  - 通过 TCP 直连 canal-server
  #  - 订阅 kafka|rocketMQ|rabbitMQ 的消息
  mode: tcp          # tcp kafka rocketMQ rabbitMQ
  flatMessage: true
  zookeeperHosts:
  syncBatchSize: 1000
  retries: -1
  timeout:
  accessKey:
  secretKey:
  # 根据 mode 修改对应配置
  consumerProperties:
    # canal tcp consumer
    canal.tcp.server.host: 127.0.0.1:11111   # 选 TCP mode 时修改这里
    canal.tcp.zookeeper.hosts:
    canal.tcp.batch.size: 500
    canal.tcp.username:
    canal.tcp.password:

  canalAdapters:
    # ==========================================================================
    # 这里的 instance_name 一定要用 Canal-Admin 中 Instance 管理列表下【启动】状态的实例
    # ==========================================================================
  - instance: 172.16.56.53_3306 # canal instance Name or mq topic name
    # 一份数据可以被多个 group 同时消费，多个 group 之间是并行执行的
    groups:
    # 一个 group 内部是串行执行多个 outerAdapters 的
    - groupId: g1
      outerAdapters:
      # 第 1 个适配器：logger 适配器
      # 将收到的变更事件输出到日志文件：`logs/adapter/adapter.log`
      - name: logger
```

#### （2）启动适配器

```sh
# 启动
$ ./bin/startup.sh

$ jps
371558 Jps
369666 CanalLauncher
369519 CanalAdminApplication
370584 CanalAdapterApplication

# 查看日志
$ cat logs/adapter/adapter.log
2024-08-22 16:45:55.256 [main] INFO  c.a.otter.canal.adapter.launcher.CanalAdapterApplication - Starting CanalAdapterApplication using Java 1.8.0_422 on 172.16.56.53 with PID 370584 (/data/canal-adapter/lib/client-adapter.launcher-1.1.7.jar started by dev in /data/canal-adapter/bin)
2024-08-22 16:45:55.260 [main] INFO  c.a.otter.canal.adapter.launcher.CanalAdapterApplication - No active profile set, falling back to default profiles: default
2024-08-22 16:45:55.605 [main] INFO  org.springframework.cloud.context.scope.GenericScope - BeanFactory id=a6cee8d1-48d1-3e64-a5f9-a1d1e90caee9
2024-08-22 16:45:55.757 [main] INFO  o.s.boot.web.embedded.tomcat.TomcatWebServer - Tomcat initialized with port(s): 8081 (http)
2024-08-22 16:45:55.763 [main] INFO  org.apache.coyote.http11.Http11NioProtocol - Initializing ProtocolHandler ["http-nio-8081"]
2024-08-22 16:45:55.764 [main] INFO  org.apache.catalina.core.StandardService - Starting service [Tomcat]
2024-08-22 16:45:55.764 [main] INFO  org.apache.catalina.core.StandardEngine - Starting Servlet engine: [Apache Tomcat/9.0.52]
2024-08-22 16:45:55.800 [main] INFO  o.a.catalina.core.ContainerBase.[Tomcat].[localhost].[/] - Initializing Spring embedded WebApplicationContext
2024-08-22 16:45:55.801 [main] INFO  o.s.b.w.s.context.ServletWebServerApplicationContext - Root WebApplicationContext: initialization completed in 487 ms
2024-08-22 16:45:56.152 [main] INFO  org.apache.coyote.http11.Http11NioProtocol - Starting ProtocolHandler ["http-nio-8081"]
2024-08-22 16:45:56.164 [main] INFO  o.s.boot.web.embedded.tomcat.TomcatWebServer - Tomcat started on port(s): 8081 (http) with context path ''
2024-08-22 16:45:56.169 [main] INFO  c.a.o.canal.adapter.launcher.loader.CanalAdapterService - ## syncSwitch refreshed.
2024-08-22 16:45:56.169 [main] INFO  c.a.o.canal.adapter.launcher.loader.CanalAdapterService - ## start the canal client adapters.
2024-08-22 16:45:56.174 [main] INFO  c.a.otter.canal.client.adapter.support.ExtensionLoader - extension classpath dir: /data/canal-adapter/plugin
2024-08-22 16:45:56.208 [main] INFO  c.a.o.canal.adapter.launcher.loader.CanalAdapterLoader - Load canal adapter: logger succeed
2024-08-22 16:45:56.218 [main] INFO  c.alibaba.otter.canal.connector.core.spi.ExtensionLoader - extension classpath dir: /data/canal-adapter/plugin
2024-08-22 16:45:56.237 [main] INFO  c.a.o.canal.adapter.launcher.loader.CanalAdapterLoader - Start adapter for canal-client mq topic: 172.16.56.53_3306-g1 succeed
2024-08-22 16:45:56.238 [Thread-3] INFO  c.a.otter.canal.adapter.launcher.loader.AdapterProcessor - =============> Start to connect destination: 172.16.56.53_3306 <=============
2024-08-22 16:45:56.238 [main] INFO  c.a.o.canal.adapter.launcher.loader.CanalAdapterService - ## the canal client adapters are running now ......
2024-08-22 16:45:56.247 [main] INFO  c.a.otter.canal.adapter.launcher.CanalAdapterApplication - Started CanalAdapterApplication in 11.23 seconds (JVM running for 11.507)
2024-08-22 16:45:56.311 [Thread-3] INFO  c.a.otter.canal.adapter.launcher.loader.AdapterProcessor - =============> Subscribe destination: 172.16.56.53_3306 succeed <=============
```

#### （3）变更验证

```sh
# 1. 监听 adapter.log
$ tail -n 0 -f logs/adapter/adapter.log

# 2. 往 user 表中插入一条数据
mysql> INSERT INTO `canal_sync`.`user`(`id`, `user`, `passwd`, `role`, `created_at`) VALUES (2, 'ycz', '123456', 'test', '2024-08-22 08:46:37');

# 3. 监听的 adapter.log 收到消息
2024-08-22 16:46:37.628 [pool-2-thread-1] INFO  c.a.o.canal.client.adapter.logger.LoggerAdapterExample - DML: {"data":[{"id":2,"user":"ycz","passwd":"123456","role":"test","created_at":1724316397000}],"database":"canal_sync","destination":"172.16.56.53_3306","es":1724316397000,"groupId":"g1","isDdl":false,"old":null,"pkNames":["id"],"sql":"","table":"user","ts":1724316397531,"type":"INSERT"}
```

### 7、ES 适配器

#### （1）配置 Adapter

```yml
server:
  port: 8081
spring:
  jackson:
    date-format: yyyy-MM-dd HH:mm:ss
    time-zone: GMT+8
    default-property-inclusion: non_null

canal.conf:
  # ClientAdapter 数据订阅的方式有两种，
  #  - 通过 TCP 直连 canal-server
  #  - 订阅 kafka|rocketMQ|rabbitMQ 的消息
  mode: tcp          # tcp kafka rocketMQ rabbitMQ
  flatMessage: true
  zookeeperHosts:
  syncBatchSize: 1000
  retries: -1
  timeout:
  accessKey:
  secretKey:
  # 根据 mode 修改对应配置
  consumerProperties:
    # canal tcp consumer
    canal.tcp.server.host: 127.0.0.1:11111   # 选 TCP mode 时修改这里
    canal.tcp.zookeeper.hosts:
    canal.tcp.batch.size: 500
    canal.tcp.username:
    canal.tcp.password:

  # 数据源配置
  srcDataSources:
    defaultDS:
      url: jdbc:mysql://127.0.0.1:3306/canal_sync?useUnicode=true&useSSL=false
      username: canal
      password: canal
      
  canalAdapters:
    # ==========================================================================
    # 这里的 instance_name 一定要用 Canal-Admin 中 Instance 管理列表下【启动】状态的实例
    # ==========================================================================
  - instance: 172.16.56.53_3306 # canal instance Name or mq topic name
    # 一份数据可以被多个 group 同时消费，多个 group 之间是并行执行的
    groups:
    # 一个 group 内部是串行执行多个 outerAdapters 的
    - groupId: g1
      outerAdapters:
      # 第 1 个适配器：logger 适配器
      # 将收到的变更事件输出到日志文件：`logs/adapter/adapter.log`
      # - name: logger
      
      # ---------- 以上配置与 logger adapter 一致 ---------- #
      
      # 第 2 个适配器：ES 适配器
      - name: es8              # 根据 ES 的版本选择，可选项 conf/: es6|es7|es8
        hosts: http://127.0.0.1:9200  # 127.0.0.1:9200 for rest mode
        properties:
          mode: rest           # or transport
          # security.ca.path: /etc/es8/ca.crt
          # security.auth: test:123456   #  only used for rest mode
          cluster.name: docker-cluster   # 在 elasticsearch.yml 配置文件中查看
```

#### （2）配置 es*/ 映射

```sh
# conf/es* 下的每个文件对应一个 Table 到 ES 的映射
$ tree conf/es8/
conf/es8/
├── biz_order.yml
├── customer.yml
└── mytest_user.yml

# 删除多余表映射
$ rm -f conf/es8/*

# 新建 user 表映射
$ vim conf/es8/user.yml
```

> conf/es8/user.yml

```yaml
dataSourceKey: defaultDS        # 数据源，对应上面配置的 srcDataSources 中的值
destination: 172.16.56.53_3306  # 与 adapter 配置的一致
groupId: g1                     # 只会同步对应 groupId 的数据
esMapping:
  # 注意：index_name 必须在 ES 中已存在，且字段也能对应上
  _index: canal_user          # ES 的索引
  _id: _id                    # ES 的 _id
  #  upsert: true          
  #  pk: id                   # 如果不需要 _id，则指定一个字段为主键
  # SQL 映射，注意事项：
  #  - 表名必须用别名，否则 Update 不成功
  #  - 表名不能用反引号括起来，如 FROM `user` u 无法同步
  sql: "
    SELECT
      u.id AS _id,
      u.id,
      u.user,
      u.passwd,
      u.role,
      u.created_at
    FROM
      user u"
    # 当 MySQL 中字段类型为 Json 时
  #  objFields:
  #    _labels: array:;   # 数组或者对象属性, array:; 代表字段里面是以;分隔的
  #    _obj: object       # Json 对象
  etlCondition: "WHERE created_at>={}"  # ETL 的条件参数，软删除时有用
  commitBatch: 3000
```

#### （3）创建 ES 索引

```json
// 在 Kibana 的控制台执行的
POST /canal_user/_mapping
{
    "properties": {
        "id": {
            "type": "integer"
        },
        "user": {
            // text 类型会被分词处理，然后可通过分词匹配
            "type": "text"
        },
        "passwd": {
            // keyword 类型，他是一个关键词不能被分词匹配，必须完整匹配
            "type": "keyword",
            "index": false  // index: false 不能被索引查询
        },
        "role": {
            "type": "text"
        },
        "created_at": {
          "type": "date"
        }
    }
}
```

#### （4）启动 Adapter

```sh
# 启动
$ ./bin/startup.sh
```

#### （5）验证

```sh
# 执行 user 全表数据导入到 ES
$ curl http://127.0.0.1:8081/etl/es8/user.yml -X POST
{"succeeded":true,"resultMessage":"导入ES 数据：7 条"}

# logs/adapter/adapter.log
2024-08-23 10:26:48.328 [http-nio-8081-exec-1] INFO  o.a.catalina.core.ContainerBase.[Tomcat].[localhost].[/] - Initializing Spring DispatcherServlet 'dispatcherServlet'
2024-08-23 10:26:48.329 [http-nio-8081-exec-1] INFO  org.springframework.web.servlet.DispatcherServlet - Initializing Servlet 'dispatcherServlet'
2024-08-23 10:26:48.329 [http-nio-8081-exec-1] INFO  org.springframework.web.servlet.DispatcherServlet - Completed initialization in 0 ms
2024-08-23 10:26:48.347 [http-nio-8081-exec-1] INFO  c.a.otter.canal.client.adapter.es8x.etl.ESEtlService - start etl to import data to index: canal_user
2024-08-23 10:26:48.589 [http-nio-8081-exec-1] INFO  c.a.otter.canal.client.adapter.es8x.etl.ESEtlService - 数据全量导入完成, 一共导入 7 条数据, 耗时: 242


# 查看 user 表的总数据
$ curl http://127.0.0.1:8081/count/es8/user.yml
{"esIndex":"canal_user","count":7}

# 监听 logs/adapter/adapter.log 日志文件

# 1. INSERT（同时去 ES 验证数据）
2024-08-23 10:38:14.541 [pool-3-thread-1] DEBUG c.a.o.canal.client.adapter.es.core.service.ESSyncService - DML: {"data":[{"id":8,"user":"888","passwd":"888","role":"888","created_at":1724380694000}],"database":"canal_sync","destination":"172.16.56.53_3306","es":1724380694000,"groupId":"g1","isDdl":false,"old":null,"pkNames":["id"],"sql":"","table":"user","ts":1724380694344,"type":"INSERT"}
Affected indexes: canal_user

# 2. UPDATE（同时去 ES 验证数据）
2024-08-23 10:39:03.222 [pool-3-thread-1] DEBUG c.a.o.canal.client.adapter.es.core.service.ESSyncService - sync error, es index: canal_user, DML : Dml{destination='172.16.56.53_3306', database='canal_sync', table='user', type='UPDATE', es=1724380734000, ts=1724380743221, sql='', data=[{id=8, user=888999, passwd=888, role=888, created_at=2024-08-23 10:38:14.0}], old=[{user=888}]}
Affected indexes: canal_user

# 3. DELETE（同时去 ES 验证数据）
2024-08-23 10:48:08.702 [pool-3-thread-1] DEBUG c.a.o.canal.client.adapter.es.core.service.ESSyncService - DML: {"data":[{"id":8,"user":"888999","passwd":"888","role":"888","created_at":1724380694000}],"database":"canal_sync","destination":"172.16.56.53_3306","es":1724381288000,"groupId":"g1","isDdl":false,"old":null,"pkNames":["id"],"sql":"","table":"user","ts":1724381288699,"type":"DELETE"}
Affected indexes: canal_user
```

> 当映射文件中定义的 ES 索引不存在时

```sh
# 当 canal_role 索引不存在时，报错
$ curl http://127.0.0.1:8081/etl/es8/role.yml -X POST
{"succeeded":false,"resultMessage":"导入ES 数据：0 条","errorMessage":"canal_role etl failed! ==>Elasticsearch exception [type=index_not_found_exception, reason=no such index [canal_role]]"}
```

