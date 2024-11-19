# 一、概述

## 1.1、Install & Deploy

### 1.1.1、ElasticSearch

官网：https://www.elastic.co/cn

```sh
# 目录结构
/usr/share/elasticsearch/
├── bin
│   ├── elasticsearch
│   ├── elasticsearch-plugin
│   ├── elasticsearch-reset-password
│   └── ...
├── config
│   ├── elasticsearch.yml
│   ├── role_mapping.yml
│   ├── roles.yml
│   ├── users
│   ├── users_roles.yml
│   └── ...
├── data
│   └── ...
├── jdk
│   └── ...
├── lib
│   └── *.jar
├── logs
│   └── ...
├── modules
│   └── ...
└── plugins
    └── ...
```

#### （1）单节点

```sh
# 禁用安全功能并开启跨域（否则 elasticsearch-head 无法连接）
docker run --name ES -d \
  -m 4G \
  -p 9200:9200 \
  -v /data/docker-run/es/plugins/:/usr/share/elasticsearch/plugins/ \
  -v es_data:/usr/share/elasticsearch/data/ \
  -e "discovery.type=single-node" \
  -e "xpack.security.enabled=false" \
  -e "http.cors.enabled=true" \
  -e "http.cors.allow-origin='*'" \
  docker.elastic.co/elasticsearch/elasticsearch:8.15.0

# 【验证】访问：`http://localhost:9200`
{
  "name" : "5e3de75a65ea",
  "cluster_name" : "docker-cluster",
  "cluster_uuid" : "hh8UqUflTOiw9tUOhRhpyQ",
  "version" : {
    "number" : "8.15.0",
    "build_flavor" : "default",
    "build_type" : "docker",
    "build_hash" : "1a77947f34deddb41af25e6f0ddb8e830159c179",
    "build_date" : "2024-08-05T10:05:34.233336849Z",
    "build_snapshot" : false,
    "lucene_version" : "9.11.1",
    "minimum_wire_compatibility_version" : "7.17.0",
    "minimum_index_compatibility_version" : "7.0.0"
  },
  "tagline" : "You Know, for Search"
} 

# 启动安全功能时，用环境变量指定 elastic 用户的密码
docker run --name ES -d \
  -m 4G \
  -p 9200:9200 \
  -v /data/docker-run/es/plugins/:/usr/share/elasticsearch/plugins/ \
  -v es_data:/usr/share/elasticsearch/data/ \
  -e ELASTIC_PASSWORD=admin \
  -e "discovery.type=single-node" \
  docker.elastic.co/elasticsearch/elasticsearch:8.15.0
```

> ES 配置文件：elasticsearch.yml

```yaml
cluster.name: "docker-cluster"
network.host: 0.0.0.0

#----------------------- BEGIN SECURITY AUTO CONFIGURATION -----------------------
#
# The following settings, TLS certificates, and keys have been automatically
# generated to configure Elasticsearch security features on 20-08-2024 01:17:15
#
# --------------------------------------------------------------------------------

# Enable security features
xpack.security.enabled: true

# cors
http.cors.enabled: true
http.cors.allow-origin: '*'

xpack.security.enrollment.enabled: true

# Enable encryption for HTTP API client connections, such as Kibana, Logstash, and Agents
xpack.security.http.ssl:
  enabled: true
  keystore.path: certs/http.p12

# Enable encryption and mutual authentication between cluster nodes
xpack.security.transport.ssl:
  enabled: true
  verification_mode: certificate
  keystore.path: certs/transport.p12
  truststore.path: certs/transport.p12
#----------------------- END SECURITY AUTO CONFIGURATION -------------------------
```

#### （2）ES 集群

```yaml
version: '3'

services:
  es01:
    image: docker.elastic.co/elasticsearch/elasticsearch:8.15.0
    container_name: es01
    environment:
      - node.name=es01
      - cluster.name=es-docker-cluster
      - discovery.seed_hosts=es02,es03
      - bootstrap.memory_lock=true
      - "ES_JAVA_OPTS=-Xms512m -Xmx512m"
      - xpack.security.enabled=false
      - http.cors.enabled=true
      - http.cors.allow-origin="*"
    ulimits:
      memlock:
        soft: -1
        hard: -1
    volumes:
      - esdata01:/usr/share/elasticsearch/data
      - /data/docker/ES/plugins/:/usr/share/elasticsearch/plugins/
    ports:
      - 9201:9200
    networks:
      - esnet
 
  es02:
    image: docker.elastic.co/elasticsearch/elasticsearch:8.15.0
    container_name: es02
    environment:
      - node.name=es02
      - cluster.name=es-docker-cluster
      - discovery.seed_hosts=es01,es03
      - bootstrap.memory_lock=true
      - "ES_JAVA_OPTS=-Xms512m -Xmx512m"
      - xpack.security.enabled=false
      - http.cors.enabled=true 
      - http.cors.allow-origin="*"
    ulimits:
      memlock:
        soft: -1
        hard: -1
    volumes:
      - esdata02:/usr/share/elasticsearch/data
      - /data/docker/ES/plugins/:/usr/share/elasticsearch/plugins/
    ports:
      - 9202:9200
    networks:
      - esnet

  es03:
    image: docker.elastic.co/elasticsearch/elasticsearch:8.15.0
    container_name: es03
    environment:
      - node.name=es03
      - cluster.name=es-docker-cluster
      - discovery.seed_hosts=es01,es02
      - bootstrap.memory_lock=true
      - "ES_JAVA_OPTS=-Xms512m -Xmx512m"
      - xpack.security.enabled=false
      - http.cors.enabled=true 
      - http.cors.allow-origin="*"
    ulimits:
      memlock:
        soft: -1
        hard: -1
    volumes:
      - esdata03:/usr/share/elasticsearch/data
      - /data/docker/ES/plugins/:/usr/share/elasticsearch/plugins/
    ports:
      - 9203:9200
    networks:
      - esnet

volumes:
  esdata01:
    driver: local
  esdata02:
    driver: local
  esdata03:
    driver: local

networks:
  esnet:
    driver: bridge
```

> 验证

```
$ docker compose up -d

http://172.16.2.13:9200/_cat/nodes?v
ip         heap.percent ram.percent cpu load_1m load_5m load_15m node.role   master name
172.20.0.2           65           3   1    1.58    0.54     0.51 cdfhilmrstw -      es03
172.20.0.3           23           3   1    1.58    0.54     0.51 cdfhilmrstw *      es01
172.20.0.4           63           3   1    1.58    0.54     0.51 cdfhilmrstw -      es02
```



### 1.1.2、Elasticsearch - head 

[Elasticsearch - head](https://github.com/mobz/elasticsearch-head) 是 ES 的图形化管理界面。需要先安装 [Node.js](https://nodejs.org/en/download/)。

> 安装 Node.js

[nodejs 清华镜像站](https://mirrors.tuna.tsinghua.edu.cn/nodejs-release/v20.16.0/)

```sh
# 查看系统的 CPU 架构
$ lscpu
架构/Architecture:	x86_64

# 说明
AMD：结果中包含 x86_64 或 i686。
ARM：结果中包含 armv7l、aarch64、arm64。

# 根据 CPU 架构选择对应安装包
$ wget  https://nodejs.org/dist/v10.8.0/node-v10.8.0-linux-x64.tar.xz
$ tar -xvf node-v10.8.0-linux-x64.tar.xz
$ mv node-v10.8.0-linux-x64 /usr/local/nodejs-v10.8
$ vim /etc/profile
`
# Nodejs
export PATH=$PATH:/usr/local/nodejs-v10.8/bin
`

$ source /etc/profile

# 查看 node 版本
$ node -v
v10.8.0
$ npm -v
6.2.0
```

>  安装 elasticsearch-head

```sh
$ git clone https://github.com/mobz/elasticsearch-head.git
$ cd elasticsearch-head-master
$ ll
drwxrwxr-x  2 dev dev 4096 11月  6  2020 crx/
-rw-rw-r--  1 dev dev  248 11月  6  2020 Dockerfile
-rw-rw-r--  1 dev dev  221 11月  6  2020 Dockerfile-alpine
-rw-rw-r--  1 dev dev   13 11月  6  2020 .dockerignore
-rw-rw-r--  1 dev dev  104 11月  6  2020 elasticsearch-head.sublime-project
-rw-rw-r--  1 dev dev   62 11月  6  2020 .gitignore
-rw-rw-r--  1 dev dev 2240 11月  6  2020 Gruntfile.js
-rw-rw-r--  1 dev dev 3482 11月  6  2020 grunt_fileSets.js
-rw-rw-r--  1 dev dev 1100 11月  6  2020 index.html
-rw-rw-r--  1 dev dev  546 11月  6  2020 .jshintrc
-rw-rw-r--  1 dev dev  559 11月  6  2020 LICENCE
-rw-rw-r--  1 dev dev  886 11月  6  2020 package.json
-rw-rw-r--  1 dev dev  100 11月  6  2020 plugin-descriptor.properties
drwxrwxr-x  4 dev dev 4096 11月  6  2020 proxy/
-rw-rw-r--  1 dev dev 7243 11月  6  2020 README.textile
drwxrwxr-x  5 dev dev 4096 11月  6  2020 _site/
drwxrwxr-x  5 dev dev 4096 11月  6  2020 src/
drwxrwxr-x  4 dev dev 4096 11月  6  2020 test/

# 安装 grunt
# $ npm install -g grunt-cli

# 安装 & 运行
$ npm install
# grunt server 或 npm run start
$ npm run start

> elasticsearch-head@0.0.0 start /data/es-head
> grunt server

(node:130201) ExperimentalWarning: The http2 module is an experimental API.
Running "connect:server" (connect) task
Waiting forever...
Started connect web server on http://localhost:9100


# 访问：`http://localhost:9100`


# 修改 ES 配置文件（允许跨域）
$ elasticsearch\config\elasticsearch.yml
`
# 追加配置
http.cors.enabled: true 
http.cors.allow-origin: "*"
`

# 重启 ES 后，即可用 ES-head 连接到 ES
```



### 1.1.3、Kibana 

**（1）Docker 安装 Kibana**

```sh
# 创建容器卷
$ docker volume create kibana-data

# 配置文件（内容见下）
$ vim config/kibana.yml

# 运行容器
$ docker run --name kib -d \
  --net esn \
  -p 5601:5601 \
  -v /data/docker-run/kibana/config/kibana.yml:/usr/share/kibana/config/kibana.yml \
  -v kibana-data:/usr/share/kibana/data \
  docker.elastic.co/kibana/kibana:8.15.0
  
# 访问：`http://localhost:5601`
```

> kibana.yml

```yaml
### >>>>>>> BACKUP START: Kibana interactive setup (2024-08-14T03:23:29.515Z)

#
# ** THIS IS AN AUTO-GENERATED FILE **
#

# Default Kibana configuration for docker target
#server.host: "0.0.0.0"
#server.shutdownTimeout: "5s"
#elasticsearch.hosts: [ "http://elasticsearch:9200" ]
#monitoring.ui.container.elasticsearch.enabled: true

### >>>>>>> BACKUP END: Kibana interactive setup (2024-08-14T03:23:29.515Z)

# 默认 localhost:5601，除本机外无法访问
server.host: 0.0.0.0
server.shutdownTimeout: 5s
# 设置中文
i18n.locale: "zh-CN"

# 这是在 Kibana Web 配置 Elastic 后自动生成的
# This section was automatically generated during setup.
elasticsearch.hosts: ['http://172.16.56.53:9200']
# 这套账号只能 kibana System 身份认证使用，没有 Web 访问权限
elasticsearch.username: kibana
elasticsearch.password: GEC27a00fGr9iLe1p9ba
```

**（2）配置 Elastic 以开始**

- 选择手动配置

- 输入地址：http://172.16.56.53:9200

- 连接到 ES（http://172.16.56.53:9200）：需要 kibana 专用账户名和密码

  ```sh
  # 重置一个账号和密码
  $ docker exec -it ES /usr/share/elasticsearch/bin/elasticsearch-reset-password -u kibana
  Password for the [kibana] user successfully reset.
  New value: GEC27a00fGr9iLe1p9ba
  
  # username：kibana
  # password：GEC27a00fGr9iLe1p9ba
  ```

- 验证 Kibana

  ```sh
  $ docker exec -it kib /usr/share/kibana/bin/kibana-verification-code
  Kibana is currently running with legacy OpenSSL providers enabled! For details and instructions on how to disable see https://www.elastic.co/guide/en/kibana/8.15/production.html#openssl-legacy-provider
  Your verification code is:  288 406
  
  # code: 288 406
  ```

- 登录 Kibana（注意：kibana 用户无法访问）

  ```
  username: elastic
  password: admin
  ```



## 1.2、Description

Elasticsearch 是一个基于 Apache Lucene 构建的分布式搜索和分析引擎、可扩展的数据存储和矢量数据库。

### 1.2.1 术语

- 索引（index）：是 ES 中的基本存储单元，索引是由名称或别名唯一标识的文档集合。

- 文档（document）：ES 以 JSON 文档的形式序列化和存储数据。

- 元数据字段（field）：是存储文档有关信息的系统字段，如：`_index` 索引名称，`_id` 文档 ID。

- 映射（mapping）：每个索引都有一个 mapping 或 schema，用于如何为文档中的字段编制索引。mapping 定义每个字段的数据类型、字段的索引方式、以及应该如何存储。

  - 动态映射：让 ES 自动检测数据类型并创建索引。
  - 显式映射：通过为每个字段指定数据类型来预定义映射。（建议用于生产环境）

- 全文搜索：使用倒排索引、分词元化和文本分析构建快速、相关的全文搜索解决方案。

- 分析（analysis）：可将字符串字段转换为单个术语，用于：

  - 将字符串字段分词，并添加到倒排索引中，使文档可搜索。
  - 用于高级查询（如：match 查询）以生成搜索词。

  

（2）映射（mapping）

mapping 是处理数据的方式和规则方面做一些限制，如：某个字段的数据类型、默认值、分析器、是否被索引等。这些都是映射里面可以设置的，其它处理 ES 里面数据的一些使用规则设置也叫映射，按最优规则处理数据对性能提高很大，因此才需要建立映射，并且需要思考如何建立映射才能对性能更好。

### 1.2.2、ES & NoSQL & SQL

|               | 数据库   | 结构    | 表（集合） | 文档（行）       | 字段（列） |
| ------------- | -------- | ------- | ---------- | ---------------- | ---------- |
| ElasticSearch | node     | mapping | index      | document（Json） | field      |
| MongoDB       | database |         | collection | document（Bson） | field      |
| MySQL         | database | scheme  | table      | row              | column     |

注：7.* 弃用 type，用内置字段 _doc 代替。原因：同名字段的冲突。

### 1.2.3、核心概念

- 正排索引

| ID   | Content |
| ---- | ------- |
| 1    | ......  |
| 2    | ......  |
| 3    |         |

- **倒排索引**

| 标签 | ID      |
| ---- | ------- |
| 电脑 | 1、2、3 |
| 手机 | 3、5    |
| ...  | ...     |

每个ID的存储格式：

（DocID，TF，<POS>）=>（文档ID，出现次数，出现位置）

- 节点（node）：存放分片和副本。

- 分片（shards）：存放数据，分片存在节点上。

- 副本（replication）：副本数量应等于分片*副本数。

- 健康值：
  - 绿色：主分片和副本都正常，能拿到全部分片数据；
  - 黄色：主分片正常，副本未分配，部分分片损坏能从副本拿到数据；
  - 红色：分片和副本都损坏，无法拿到完整数据。



- 分片（shards）：

- 集群健康值：

  - green：主分片和副本都正常
  - yellow：主分片正常，副本未分配
  - red：不可用。

- 倒排索引：

  - 词条：索引中最小存储和查询单元

  - 词典：词条的集合，B+Tree、Hash

  - 倒排表：



## 1.4、优化

### 1.4.1、硬件选择

- 使用 SSD。固态硬盘比机械硬盘快。
- 使用 RAID 0。条带化 RAID 会提高磁盘 I/O，代价就是当一块磁盘故障时整个磁盘都故障。不要使用镜像或者奇偶校验 RAID 因为副本已经提供了这个功能。
- 使用多块硬盘，允许 ES 通过多个 path.data 目录配置把数据条带化分配到它们上面。
- 不要使用远程挂载的存储，比如 NFS 或 SMB/CIFS。这个引入的延迟会影响性能。

### 1.4.2、分片&副本策略

分片和副本的设计为 ES 提供了分布式和故障转移的特性。但并不意味着分片和副本是可以无限分配的。而且索引的分片完成分配后由于索引的路由机制，我们不能再次修改分片数量。

分片&副本过多的代价：

- 一个分片的底层是一个 Lucene 索引，会消耗一定的文件句柄、内存、以及 CPU 运转。
- 每一个搜索请求都需要命中索引中的一个分片，如果多个分片都在同一个节点上，会竞争使用相同的资源。
- 计算相关度的词项统计信息是基于分片的。分片过多时，每个分片都只有很少的数据会导致相关度变低。

分片策略：

- 控制每个分片占用的硬盘容量不超过 ES 的最大 JVM 的堆空间设置（32G）。例如索引的总容量在 500G 左右时，分片数量为 500/32 ≈ 16 个。
- 考虑节点数量，一般一个节点有时候就是一台物理机，如果分片数过多，大大超过了节点数，很可能会导致一个节点上存在多个分片，一旦节点故障，即使保持了一个以上的副本，同样有可能会导致数据丢失。所以，一般设置分片数不超过节点数的 3 倍。
- 主节点，副本和节点最大数之间数量关系：节点数 <= 主分片数*(副本数+1)

### 1.4.3、ES 集群

多个节点和分片使 ES 具有分布式和可扩展性。

- 自我管理的 ES：你负责设置和管理节点、集群、分片和副本。这包括管理底层基础架构、扩展，以及通过故障转移和备份策略确保高可用性。
- ES 能够通过将索引细分为分片来跨节点分发数据。ES 中的每个索引都是一组（包含一个或多个）物理分片，其中每个分片都是一个自包含的 Lucene 索引，包含索引中文档的子集。通过将文档分布在多个分片的索引中，并将这些分片分布在多个节点上，ES 提高了索引和查询能力。
- 有两种类型的分片：主分片和副本分片。索引中的每个文档都属于一个主分片。副本 shard 是主分片的副本。副本在集群中的节点之间维护数据的冗余副本。 这可以防止硬件故障，并增加处理读取请求（如搜索或检索文档）的能力。
- 在创建索引时，索引中的主分片数量是固定的，但副本分片的数量可以 随时更改，而不会中断索引或查询操作。
- 集群中的分片副本会在节点之间自动平衡，以提供可扩展性和高可用性。所有节点都是 了解集群中的所有其他节点，并且可以将客户端请求转发到相应的节点。这允许 Elasticsearch 在集群中分配索引和查询负载。

# 二、索引（index）

## 2.1、集群操作

### 2.1.1、查看集群节点数

```json
> GET /_cat/nodes?v

# Master: es01
ip         heap.percent ram.percent cpu load_1m load_5m load_15m node.role   master name
172.20.0.3           23           3   1    1.58    0.54     0.51 cdfhilmrstw *      es01
172.20.0.4           63           3   1    1.58    0.54     0.51 cdfhilmrstw -      es02
172.20.0.2           65           3   1    1.58    0.54     0.51 cdfhilmrstw -      es03
```



## 2.1、索引操作

可以按索引设置索引级别的 `settings`：

- 静态的（static）：只能在索引创建时或在已关闭的索引上设置。
- 动态的（dynamic）：使用 ES 默认的配置值。

```json
{
    "index": {
        // ==============
        //  静态的索引设置
        // ==============
        
        // 主分片数，默认：1
        // 只能在创建索引时设置。不能在已关闭的索引上更改它。
        "number_of_shards": 1,
        
        // index.sort 是静态的索引设置，只能在创建时设置（可选）
        "sort": {
            // 只允许：boolean numeric date keyword doc_values 类型
            "field": [ "field_1", "field_2" ],

            // asc 
            // desc
            "order": [ "desc", "asc" ],

            // ES 支持按多值字段排序，mode 选项控制选择什么值来对文档进行排序（可选）
            // min: 选择最低值
            // max: 选择最高值
            "mode": "min|max",

            // missing 参数指定应如何处理缺少该字段的文档（可选）
            // _last:  字段没有值的文档将最后排序
            // _first: 首先对字段没有值的文档进行排序
            "missing": "_last|_first"
        },
        
        // ...


        // ==============
        //  动态的索引设置
        // ==============
        
        // 每个主分片具有的副本数，默认：1
        "number_of_replicas": 1
        
        // ...
    }
}
```

### 2.1.1、[PUT] 创建索引

```json
> PUT /<index_name>
{
    // 定义别名
    "aliases": {},
    
    // 定义映射（表结构）
    "mappings": {
        "properties": {
            "content": {
                "type": "text"
            },
            "created_at": {
                "type": "long"
            }
        }
    },
    
    // 索引设置
    "settings": {
        "index": {
            // 定义主分片及副本
            "number_of_shards": 3,
            "number_of_replicas": 1,
            
            // 定义排序
            "sort": {
                "field": "created_at",
                "order": "desc"
            }
        }
    }
}

// Response
{
    "acknowledged": true,
    "shards_acknowledged": true,
    "index": "index_name"
}
```

### 2.1.2、[GET] 索引列表

```json
// 索引列表
> GET /_cat/indices?v

// Response
health status index      uuid                   pri rep docs.count docs.deleted store.size pri.store.size dataset.size
yellow open   index_name xO8L2TSARaii_gkb0wQiFw   1   1          0            0             249b           249b         249b
```

### 2.1.3、[GET] 索引详情

```json
// 根据 <index_name> 查询单条索引详情
> GET /<index_name>


// Response
{
    "index_name": {
        "aliases": {},
        "mappings": {},
        "settings": {
            "index": {
                "routing": {
                    "allocation": {
                        "include": {
                            "_tier_preference": "data_content"
                        }
                    }
                },
                "number_of_shards": "1",
                "provided_name": "<index_name>",
                "creation_date": "1723604838298",
                "number_of_replicas": "1",
                "uuid": "xO8L2TSARaii_gkb0wQiFw",
                "version": {
                    "created": "8512000"
                }
            }
        }
    }
}



```

### 2.1.4、[PUT] 修改索引

```json
// 修改 settings 属性
> PUT /<index_name>/_settings

// Request
{
    // 主分片不可修改
    // "number_of_shards": 1,
    
    // 副本数，可调整
    "number_of_replicas": 0   
}

// Response
{
    "acknowledged": true
}

// 再次查看结果
> GET /<index_name>

{
    "index_name": {
        "aliases": {},
        "mappings": {},
        "settings": {
            "index": {
                "routing": {
                    "allocation": {
                        "include": {
                            "_tier_preference": "data_content"
                        }
                    }
                },
                "number_of_shards": "1",
                "provided_name": "<index_name>",
                "creation_date": "1723618307628",
                "number_of_replicas": "0",
                "uuid": "GJr7aa2eQSGyp4AE8vl6-g",
                "version": {
                    "created": "8512000"
                }
            }
        }
    }
}
```

### 2.1.5、[DELETE] 删除索引

```json
> DELETE /<index_name>

// Response
{
    "acknowledged": true
}
```

### 2.1.6、索引别名（alias）

```json
// [HEAD] 检查别名是否存在
> HEAD /<index_name>/_alias/<alias_n>

Status Code:
  200  所有指定的别名都存在。
  404  一个或多个指定的别名不存在。


// [PUT/POST] 创建或更新别名
> PUT /<index_name>/_alias/<alias_n>

// Response
{
    "acknowledged": true,
    "errors": false
}


// [GET] 获取别名（不指定 alias_n 时，默认获取全部别名，等价于 _all）
> GET /<index_name>/_alias/<alias_n>

// Response
{
    "index_name": {
        "aliases": {
            "alias_n": {}
        }
    }
}


// [DELETE] 删除别名
> DELETE /<index_name>/_alias/<alias_n>

// Response
{
    "acknowledged": true,
    "errors": false
}
```

### 2.1.7、索引模板

```json
// 设置模板
PUT _template/<template_name>
{
  "index_patterns": [
    "my*" // 以 my 开头的 index 会自动应用该模板
  ],
  "settings": {
    "index": {
      "number_of_shards": 2,
      "number_of_replicas" : 1
    }
  },
  "mappings": {
    "properties": {
      "created_at": {
        "type": "date",
        "format": ["yyyy-MM-dd hh:mm:ss"]
      }
    }
  }
}

// 查看模板
GET _template/<template_name>

// 删除模板
DELETE _template/<template_name>
```

### 2.1.8、索引排序

在 ES 中创建新索引时，可以配置 Segments 将对每个 Shard 进行排序。默认情况下，Lucene 不应用任何排序。

```
PUT /<index_name>

{
  "settings": {
    "index": {
    
      "sort.field": [ "created_at" ], 
      "sort.order": [ "desc" ]       
    }
  }
}
```



# 三、映射（mapping）

## 3.1、数据类型

![img](https://img2020.cnblogs.com/blog/1853022/202009/1853022-20200920102635101-1577990369.png)

### 3.1.1 数值型

```json
# 整型
byte      1 Byte：-128 ~ 127
short     2 Byte：-32768 ~ 32767
integer   4 Byte：-2^31 ~ 2^31 - 1
long      8 Byte: -2^63 ~ 2^63 -1

// 例：定义四个字段：int8 int16 int32 int64
// 分别对应 Golang 中的四种 int 类型
{
    "int8": {
        "type": "byte"
    },
    "int16": {
        "type": "short"
    },
    "int32": {
        "type": "interger"
    },
    "int64": {
        "type": "long"
    }
}

# 浮点型
double          64 位双精度浮点型
float           32 位单精度浮点型
half_float      16 位半精度浮点型
scaled_float    64 缩放浮点型（定点型）
                一个由 long 支持的浮点数，缩放由固定的 double 缩放因子。

// 例：定义四个字段：float64 float32 float16 price
// 其中 double 和 float 类型对应 Golang 中的 float64 和 float32
{
    "float64": {
        "type": "double"
    },
    "float32": {
        "type": "float"
    },
    "float16": {
        "type": "half_float"
    },
    
    "price": {
        // 缩放浮点型
        "type": "scaled_float",
        
        // 编码值时使用的缩放因子。
        // 值将在索引时乘以此因子，并四舍五入到最接近的多头值。
        // 如价格 price 为 57.34，
        // 缩放因子为 100，则存储为 5734
        // 缩放因子为 10，则存储为 573
        "scaling_factor": 100
    }
}

unsigned_long  8 Byte: 0 ~ 2^64 -1
```

### 3.1.2 字符型

- string：从 ElasticSearch 5.x 起弃用

#### 1、keyword

keyword：精确匹配字符串

- 用于结构化内容（如：ID, Email, HostName, StatusCode, tags）。
- 非结构化机器生成内容的通配符查询（wildcard），该类型针对具有较大值或较高基数的字段进行了优化。
- Keyword 字段通常用于排序、聚合和术语级（term、terms）查询。

```json
{
    "tags": {
        "type": "keyword"
        
        // Options...
        
        // 字段是否应该以列跨步方式存储在磁盘上，以便以后可以用于排序、聚合或脚本编写？
        // 默认为：true
        "doc_values": true,
        
        // 不要索引任何超过此值的字符串。
        // 默认值为2147483647，以便接受所有值。
        "ignore_above": 0,
        
        // 该字段是否应该快速搜索？
        // 默认为：true
        // 只启用了 doc_values 的 keyword 字段仍然可以查询，尽管速度较慢。
        "index": true,
        
        // 接受替换任何显式空值的字符串值。
        // 默认为 null，这意味着该字段被视为缺失。
        // 请注意，如果使用脚本值，则无法设置此值。
        "null_value": null,
        
        // 字段值是否应与_source字段分开存储和检索。
        // 默认：false
        "store": false
    }
}
```

并非所有数值数据都应映射为数值类型。ES 为范围查询优化了数字类型（如 integer, long）。在以下情况下，考虑将数字标识符映射为 keyword ：

- 不打算使用范围查询来搜索标识符数据。
- 快速检索很重要。keyword 字段上的 term 搜索通常比 numeric 字段上的 term 搜索更快。

如果不确定使用哪个，可以使用多字段将数据映射为 keyword 和 numeric 数据类型。

**（1）constant_keyword**

constant_keyword 是 keyword 字段的专门化，用于索引中所有文档具有相同值的情况。用于始终包含相同值的关键字字段。

```json
{
    // 允许提交没有 level 字段或 `level: debug` 的文档。
    // 不允许提供与配置（debug）不同的值。
    "level": {
        "type": "constant_keyword",
        
        // 与索引中的所有文档关联的值。
        // 若不提供，则根据第一个被索引的文档进行设置。
        "value": "debug"
    }
}
```

constant_keyword 支持与 keyword 字段相同的查询和聚合，但利用了所有文档每个索引都有相同值的事实来更有效地执行查询。

**（2）wildcard**

wildcard 类型是一个专门用于非结构化机器生成内容的keyword 字段，计划使用类似 grep 的通配符和正则表达式查询进行搜索。wildcard 类型针对具有大值或高基数的字段进行了优化。

以下情况使用 text 类型：

- 内容是人类可读的，例如：电子邮件正文或产品描述。
- 计划在该字段中搜索单个单词或短语。

以下情况使用 keyword 类型：

- 内容是机器生成的，例如：日志消息或 HTTP 请求信息。
- 计划使用术语级（term）查询在字段中搜索精确的完整值（如：`org.foo.bar`），或部分字符序列（如：`org.foo.*`）。
- 如果计划使用 wildcard 或 regexp 查询定期搜索字段并满足以下条件之一，则使用 wildcard 类型：
  - 该字段包含超过 100 万个唯一值，且计划使用带有前导通配符的模式（如：`*.foo`）定期搜索字段。
  - 该字段包含大于 32KB 的值，且计划使用任何通配符模式定期搜索字段。

#### 2、text

text 用于全文内容的传统字段类型，最合适于非结构化但人类可读的内容（如：电子邮件正文或产品描述）。

text 字段会被分析（analysis），在被索引之前会通过分析器将字符串转换为单个术语的列表。

text 字段不能用于排序，也很少用于聚合。

```json
{
    "content": {
        "type": "text",
        
        // Options...
        
        // 应在索引时和搜索时（除非被 search_analyzer 覆盖）用于 text 字段的分析器
        // 默认为标准分析器（standard）
        "analyzer": "standard",
        
        // 应在搜索时使用的分析器
        // 默认为 analyzer 设置的分析器
        "search_analyzer": "",
        
        // Multi-fields 允许以多种方式对同一字符串值进行索引，以用于不同的目的，
        // 例如一个字段用于搜索，多字段用于排序和聚合，或者由不同的分析器分析相同的字符串值。
        "fields": {},
        
        // 该字段是否应可搜索
        "index": true
        
        // ...
    }
}
```

**（1）text & keyword**

```json
{
    // text 类型字段用于全文搜索，不支持排序、分组、前缀查询
    // 定义一个 keyword 子字段用于排序
    // 这样既可以用于全文搜索，也可以通过 title.keyword 进行排序、分组、前缀。
    "title": {
        "type": "text",
        "fields": {
            "keyword": {
                "type": "keyword",
                // 不要索引任何超过此值的字符串。
                "ignore_above": 256
            }
        }
    }
}
```

**（2）token_count**

token_count 类型实际上是一个整数字段，它接受字符串值，对其进行分析（analyze），然后对字符串中的令牌（token）数量进行索引。

```json
"name": { 
    "type": "text",
    "fields": {
        "length": { 
            "type":     "token_count",
            
            // 用于分析字符串值的分析器。【必须】。
            // 为了获得最佳性能，请使用不带令牌过滤器的分析器。
            "analyzer": "standard"
        }
    }
}
```



### 3.1.3 布尔型（boolean）

```json
{
    "bool": {
        "type": "boolean",

        // Options...
        
        // 字段是否应该以列跨步方式存储在磁盘上，以便以后可以用于排序、聚合或脚本编写？
        "doc_values": false,

        // 该字段是否应该快速搜索，默认：true。
        "index": true,

        // 默认情况下，错误的数据类型写入索引会引发异常，并拒绝整个文档。
        // 忽略异常时，错误字段不会被索引，其他字段正常处理。
        "ignore_malformed": false,

        "null_value": null,

        "script": null,
        
        "on_script": null,

        // 字段值是否与 _source 字段分开存储和检索，默认：false。
        "store": false
    }
}
```

### 3.1.4 二进制（binary）

接受二进制值作为 Base64 编码的字符串（如：图像）。

默认情况下，该字段不存储，也不可搜索。

```json
{
    "blob": {
        "type": "binary",
        
        // 字段是否应该以列跨步方式存储在磁盘上，以便以后可以用于排序、聚合或脚本编写？
        // 对于 TSDB 索引（index.mode设置为time_series的索引）时，默认为 true
        "doc_values": false,
        
        // 字段值是否应与 _source 字段分开存储和检索，默认：false。
        "store": false
    }
}
```

### 3.1.5 对象和关系型

#### 1、数组 array

在 ES 中，没有专用的 array 数据类型。默认情况下，任何字段都可以包含零个或多个值，但是数组中的所有值必须具有相同的数据类型。

```
# 例如：
[]string{"one", "two"}
[]int{1, 2}
```

#### 2、对象 object

```json
{
    "manager": {
        // 不需要将字段类型显式设置为 object，因为这是默认值。
        "type": "object",
        
        // 对象内的字段，可以是任何数据类型，包括对象。
        // 新属性可以添加到现有对象中。
        "properties": {
            "age": {
                "type": "integer"
            },
            "name": {
                "properties": {
                    "first": {
                        "type": "text"
                    },
                    "last": {
                        "type": "text"
                    }
                }
            }
        },
        
        // Options...
        
        // 是否应将新属性动态添加到现有对象中。
        "dynamic": true,
        
        // 是否应解析和索引为对象字段给定的 JSON 值
        "enabled": false,
        
        // 对象是否可以容纳子对象
        "subobjects": true
    }
}
```

> 扁平化

```json
PUT index/_doc/1
{ 
  "region": "US",
  "manager": { 
    "age":     30,
    "name": { 
      "first": "John",
      "last":  "Smith"
    }
  }
}

// 此文档被索引为简单、扁平的 key-value 列表 pairs
{
  "region":             "US",
  "manager.age":        30,
  "manager.name.first": "John",
  "manager.name.last":  "Smith"
}
```



#### 3、嵌套 nested

nested 是 object 类型的一个专门版本，它允许以一种可以彼此独立查询的方式对对象数组进行索引。嵌套文档的查询通常很昂贵。

```json
{
    "user_nested": {
        "type": "nested",
        
        // Options
        
        // 是否应将新属性动态添加到现有嵌套对象中
        // 默认：true
        // 可选：false, strict
        "dynamic": "true",
        
        // 嵌套对象中的字段，可以是任何数据类型，包括嵌套。
        // 新属性可以添加到现有的嵌套对象中。
        "properties": {},
        
        // 嵌套对象中的所有字段也将作为标准（平面）字段添加到父文档中。
        "include_in_parent": false,
        
        // 嵌套对象中的所有字段也将作为标准（平面）字段添加到根文档中。
        "include_in_root": false
    }
}
```

> 扁平对象数组：将整个对象映射为单个字段，并允许对其内容进行简单搜索。

```json
PUT index/_doc/1
{
  "group" : "fans",
  // user 字段将动态添加为 object 类型
  "user" : [ 
    {
      "first" : "John",
      "last" :  "Smith"
    },
    {
      "first" : "Alice",
      "last" :  "White"
    }
  ]
}

// 内部转换为
{
  "group" :        "fans",
  "user.first" : [ "Alice", "John" ],
  "user.last" :  [ "Smith", "White" ]
}

// user.first 和 user.last 字段被展平为多值字段，alice 和 john 之间的关联丢失。
// 错误地匹配 alice AND smith 的查询（能查到结果）。
GET index/_search
{
  "query": {
    "bool": {
      "must": [
        { "match": { "user.first": "Alice" }},
        { "match": { "user.last":  "Smith" }}
      ]
    }
  }
}
```

> 嵌套对象数组

```json
// 显式定义映射
PUT index/_mappgins
{
    "properties": {
        "user_nested": {
            "type": "nested"
        }
    }
}

// 写入数据
PUT index/_doc/2
{
  "group" : "fans",
  "user_nested" : [
    {
      "first" : "John",
      "last" :  "Smith"
    },
    {
      "first" : "Alice",
      "last" :  "White"
    }
  ]
}

// 错误的查询条件（无法匹配）
GET index/_search
{
    "query": {
        "nested": {
            "path": "user_nested",
            "query": {
                "bool": {
                    "must": [
                        {
                            "match": {
                                "user_nested.first": "Alice"
                            }
                        },
                        {
                            "match": {
                                "user_nested.last": "Smith"
                            }
                        }
                    ]
                }
            }
        }
    }
}

// 匹配 ID：2
GET index/_search
{
    "query": {
        "nested": {
            "path": "user_nested",
            "query": {
                "bool": {
                    "must": [
                        {
                            "match": {
                                "user_nested.first": "Alice"
                            }
                        },
                        {
                            "match": {
                                "user_nested.last": "White"
                            }
                        }
                    ]
                }
            },
            // 高亮显示匹配的嵌套文档
            "inner_hits": {
                "highlight": {
                    "fields": {
                        "user_nested.first": {}
                    }
                }
            }
        }
    }
}
```

嵌套映射和对象的限制

```json
{
    // 索引中不同嵌套映射的最大数量。
    "index.mapping.nested_fields.limit": 50,
    // 单个文档在所有嵌套类型中可以包含的嵌套JSON对象的最大数量。
    // 防止内存不足错误。
    "index.mapping.nested_objects.limit": 10000
}
```

#### 4、连接 join

```json
# join: 是一个特殊的字段类型，用于在同一索引的文档中创建父子关系。关系部分定义了文档中的一组可能关系，每个关系都是父名称和子名称。
# 不建议使用多级关系来复制关系模型。每一级关系都会在查询时增加内存和计算方面的开销。为了获得更好的搜索性能，请对数据进行非规范化处理。
{
    "properties": {
        "my_id": {
            "type": "keyword"
        },
        // 字段名称：my_join_field
        "my_join_field": {
            "type": "join",
            // 定义单个关系，其中 question 是 answer 的父级
            "relations": {
                "question": "answer" 
            }
        }
    }
}

```

### 3.1.6 日期类型（date|date_nanos）

JSON 没有日期数据类型，因此 ES 中的日期可以是：

- 包含格式化日期的字符串（如：`2015-01-01`，`2015/01/01 12:10:30`）
- 时间戳（10位）
- 毫秒时间戳（13位）
- 如果分辨率为纳秒，请使用 date_nanos

```json
{
    "created_at": {
        "type": "date",

        // 支持的格式，不指定时使用默认值
        //  string: 2006-01-02 || 2006/01/02 15:04:05
        //  long: 10位秒级时间戳 || 13位毫秒时间戳
        "format": "yyyy-MM-dd HH:mm:ss||yyyy-MM-dd||epoch_millis",

        // 该字段是否应该快速搜索，默认：true。
        "index": true,

        // 字段值是否与 _source 字段分开存储和检索，默认：false。
        "store": false
    },

    // 纳秒级时间戳
    "created_nanos": {
        "type": "date_nanos",
    }
}
```

### 3.1.7 别名（alias）

别名：映射定义索引中字段的备用名称，可用于代替搜索请求中的目标字段。

别名的目标的限制：

- 字段别名只能有一个目标。
- 目标必须是具体字段，而不是对象或其它字段别名。
- 创建别名时，目标字段必须存在。
- 如果定义了嵌套对象，则字段别名必须具有与其目标相同的嵌套范围。

```json
{
    "created_at": {
        "type": "long"
    },
    
    "created_date": {
        "type": "alias",
        "path": "created_at"
    }
}
```

### 3.1.8 其它

```
# 地理
geo_point
geo_shape

# ip

# version
```

## 3.2 映射操作

映射是定义文档及其包含的字段如何存储和索引的过程。

- 动态映射（Dynamic）：ES 会自动检测文档中字段的数据类型，只需要将数据添加到索引中即可。ES 会自动添加包括顶级映射、内部对象、嵌套字段。
- 显式映射（Explicit）：精确定义数据类型，可以实现：
  - 定义哪些字符串字段为全文字段。
  - 定义哪些字段包含数字、日期、地理位置。
  - 使用无法自动检测的数据类型（geo_point、geo_shape）。
  - 选择日期格式，包括自定义日期格式。
  - 创建自定义规则来控制动态添加字段的映射。
  - 优化字段以进行部分匹配。
  - 执行特有语言的文本分析（analysis）。

### 2.2.1、[PUT] 显式映射

```json
> PUT /<index_name>/_mappings

// Request
{
    "properties": {
        "id": {
            "type": "integer",
            "index": false  // index 不能被索引查询
        },
        "title": {
            // text 类型会被分词处理，然后可通过分词匹配
            "type": "text",
            "index": true  // index 可以被索引查询
        },
        "author": {
            // keyword 类型，他是一个关键词不能被分词匹配，必须完整匹配
            "type": "keyword",
            "index": true  // index 可以被索引查询
        },
        "content": {
            "type": "text",
            "index": true
        }
    }
}
```

### 2.2.2、[GET] 映射详情

```json
> GET /<index_name>/_mappings

// Response
{
    "index_name": {
        "mappings": {
            "properties": {
                "id": {
                    "type": "integer",
                    "index": false
                },
                "title": {
                    "type": "text",
                    "index": true
                },
                "author": {
                    "type": "keyword",
                    "index": true
                },
                "content": {
                    "type": "text",
                    "index": true
                }
            }
        }
    }
}
```

### 2.2.3、[PUT] 修改映射

在大多数情况下，无法更改已映射的字段的映射。 这些更改需要重新编制索引。

```json
// 修改 mappings 属性
> PUT /<index_name>/_mappings

// Request
{
    "properties": {
        // 这里只能新增一些字段
        
        // 修改原字段类型时：
        // mapper [created_at] cannot be changed from type [long] to [text]
        "created_at": {
            "type": "text"
        }
    }
}

// Response
{
    "error": {
        "root_cause": [
            {
                "type": "illegal_argument_exception",
                "reason": ""
            }
        ],
        "type": "illegal_argument_exception",
        "reason": "mapper [created_at] cannot be changed from type [long] to [text]"
    },
    "status": 400
}

// 修改 mappings 属性
> PUT /<index_name>/_mappings

// Request
{
    "properties": {
        "content": {
            // 字段类型不可修改
            "type": "text",
            
            // 分析器不可修改
            // "analyzer": "ik_max_word",
            
            // 可增加子字段
            "fields": {
                "keyword": {
                    "type": "keyword"
                }
            }
        }
    }
}

// Response
{
    "acknowledged": true
}
```

### 2.2.4、[GET] 获取映射字段

```json
> GET /<index_name>/_mapping/field/<field_name>

// Response
{
    "index_name": {
        "mappings": {
            "created_at": {
                "full_name": "created_at",
                "mapping": {
                    "created_at": {
                        "type": "long"
                    }
                }
            }
        }
    }
}
```

# 四、文本分析（analysis）

文本分析是将非结构化文本（如：电子邮件正文或产品描述）转换为针对搜索进行优化的结构化格式的过程。

ES 在索引或搜索文本字段时执行文本分析。

概述：

文本分析使 ES 能够执行全文搜索，其中搜索返回所有相关结果，而不仅仅是精确匹配。

Tokenization：

分析通过标记化（Tokenization）使全文搜索成为可能：将文本分解为更小的块，称为 token。在大多数情况下，这些 token 是单个单词。

Normalization：

标记化允许在单个术语上进行匹配，但每个标记任然是字面上匹配的。如大小写和同义词无法匹配，文本分析可以将这些标记标准化为标准格式，这允许匹配与搜索不完全相同但足够相似以任然相关的标记。

## 4.1、内置分词器

标准分词器（standard）：根据 Unicode 文本分割算法定义的单词边界将文本划分为术语。它删除了大多数标点符号，小写术语，并支持删除停用词。

简单分词器（simple）：在遇到非字母字符时将文本划分为术语。它使所有术语小写。

空白分词器（whitespace）：每当遇到任何空白字符时将文本划分为术语。它不小写术语。

停止分词器（stop）：与简单分词器类似，支持删除停用字。

关键字分词器（keyword）：是一个“noop”分析器，它接受给定的任何文本，并输出与单个术语完全相同的文本。

模式分词器（pattern）：使用正则表达式将文本拆分为术语。它支持小写和停用词。

语言分词器（language）：ES 提供了许多特定语言的分析器，如英语或法语。

指纹分词器（fingerprint）：是一种专业分析仪，可以创建可用于重复检测的指纹。



## 4.2、标准分词器 

```json
// 英文分词规则：按空格分
GET _analyze
{
    "text": "hello world"
    // "analyzer": "standard"
}

// Response
{
    "tokens": [
        {
            "token": "hello",
            "start_offset": 0,
            "end_offset": 5,
            "type": "<ALPHANUM>",
            "position": 0
        },
        {
            "token": "world",
            "start_offset": 6,
            "end_offset": 11,
            "type": "<ALPHANUM>",
            "position": 1
        }
    ]
}

// 中文分词规则：按单个汉字分（标点符号忽略）
GET _analyze
{
    "text": "白日依山尽，黄河入海流。"
    // "analyzer": "standard"
}

// Response
{
  "tokens": [
    {
      "token": "白",
      "start_offset": 0,
      "end_offset": 1,
      "type": "<IDEOGRAPHIC>",
      "position": 0
    },
    {
      "token": "日",
      "start_offset": 1,
      "end_offset": 2,
      "type": "<IDEOGRAPHIC>",
      "position": 1
    },
    {
      "token": "依",
      "start_offset": 2,
      "end_offset": 3,
      "type": "<IDEOGRAPHIC>",
      "position": 2
    },
    {
      "token": "山",
      "start_offset": 3,
      "end_offset": 4,
      "type": "<IDEOGRAPHIC>",
      "position": 3
    },
    {
      "token": "尽",
      "start_offset": 4,
      "end_offset": 5,
      "type": "<IDEOGRAPHIC>",
      "position": 4
    },
    {
      "token": "黄",
      "start_offset": 6,
      "end_offset": 7,
      "type": "<IDEOGRAPHIC>",
      "position": 5
    },
    {
      "token": "河",
      "start_offset": 7,
      "end_offset": 8,
      "type": "<IDEOGRAPHIC>",
      "position": 6
    },
    {
      "token": "入",
      "start_offset": 8,
      "end_offset": 9,
      "type": "<IDEOGRAPHIC>",
      "position": 7
    },
    {
      "token": "海",
      "start_offset": 9,
      "end_offset": 10,
      "type": "<IDEOGRAPHIC>",
      "position": 8
    },
    {
      "token": "流",
      "start_offset": 10,
      "end_offset": 11,
      "type": "<IDEOGRAPHIC>",
      "position": 9
    }
  ]
}
```

自定义词组

## 4.3、IK 中文分词器

### 4.3.1 下载

 [GitHub](https://github.com/medcl/elasticsearch-analysis-ik/releases)

[手动下载打包的插件](https://release.infinilabs.com/analysis-ik/stable/)

```sh
# 1 手动下载打包的插件，放入 elasticsearch/plugins/... 并重启 ES
$ wget https://release.infinilabs.com/analysis-ik/stable/elasticsearch-analysis-ik-8.15.0.zip
$ unzip elasticsearch-analysis-ik-8.15.0.zip -d plugins/elasticsearch-analysis-ik-8.15.0
$ ls plugins/
elasticsearch-analysis-ik-8.15.0

# 重启 ES
$ docker restart ES

# 2 使用 plugin-cli 安装插件（失败）
$ ./bin/elasticsearch-plugin install https://get.infini.cloud/elasticsearch/analysis-ik/8.15.0
```

### 4.3.2 验证

```json
// 智能分词：ik_smark
GET _analyze
{
    "text": "白日依山尽，黄河入海流。",
    "analyzer": "ik_smart"
}

// Response
{
  "tokens": [
    {
      "token": "白日",
      "start_offset": 0,
      "end_offset": 2,
      "type": "CN_WORD",
      "position": 0
    },
    {
      "token": "依",
      "start_offset": 2,
      "end_offset": 3,
      "type": "CN_CHAR",
      "position": 1
    },
    {
      "token": "山",
      "start_offset": 3,
      "end_offset": 4,
      "type": "CN_CHAR",
      "position": 2
    },
    {
      "token": "尽",
      "start_offset": 4,
      "end_offset": 5,
      "type": "CN_CHAR",
      "position": 3
    },
    {
      "token": "黄河",
      "start_offset": 6,
      "end_offset": 8,
      "type": "CN_WORD",
      "position": 4
    },
    {
      "token": "入海流",
      "start_offset": 8,
      "end_offset": 11,
      "type": "CN_WORD",
      "position": 5
    }
  ]
}

// 细粒度分词：ik_max_word
GET _analyze
{
  "text": "白日依山尽，黄河入海流。",
  "analyzer": "ik_max_word"
}

// Response
{
  "tokens": [
    {
      "token": "白日",
      "start_offset": 0,
      "end_offset": 2,
      "type": "CN_WORD",
      "position": 0
    },
    {
      "token": "依",
      "start_offset": 2,
      "end_offset": 3,
      "type": "CN_CHAR",
      "position": 1
    },
    {
      "token": "山",
      "start_offset": 3,
      "end_offset": 4,
      "type": "CN_CHAR",
      "position": 2
    },
    {
      "token": "尽",
      "start_offset": 4,
      "end_offset": 5,
      "type": "CN_CHAR",
      "position": 3
    },
    {
      "token": "黄河",
      "start_offset": 6,
      "end_offset": 8,
      "type": "CN_WORD",
      "position": 4
    },
    {
      "token": "入海流",
      "start_offset": 8,
      "end_offset": 11,
      "type": "CN_WORD",
      "position": 5
    },
    {
      "token": "入海",
      "start_offset": 8,
      "end_offset": 10,
      "type": "CN_WORD",
      "position": 6
    },
    {
      "token": "海流",
      "start_offset": 9,
      "end_offset": 11,
      "type": "CN_WORD",
      "position": 7
    }
  ]
}
```

### 4.3.3、使用

> IK 分词器使用原则：分词时用细粒度，查询时用粗粒度
>
> - ik_smart：将文本按照粗粒度进行拆分，适合短语查询。
> - ik_max_word：将文本按照最细粒度进行拆分，适合术语查询。

```json
// 定义字段时，指定分词器（不指定时使用默认分词器）
"properties": {
	"content": {
		"type": "text",
		"analyzer": "ik_max_word",
		"search_analyzer": "ik_smart"
	}
}
```





# 五、文档操作（_doc）

## 5.1、[POST] 创建文档

```json
// 创建文档，使用默认生成的主键 _id
> POST /<index_name>/_doc

// Request
{
    "id": 1001,
    "title": "《静夜思》",
    "author": "唐·李白",
    "content": "床前明月光，疑是地上霜。举头望明月，低头思故乡。"
}

// Response
{
    "_index": "index_name",
    "_id": "hpDYT5EBDedEgQc-yYlt",  // 生成的随机 ID
    "_version": 1,                  // 写操作（增删改）数值会加1
    "result": "created",            // 本次操作是：created|updated
    "_shards": {                    // 存入分片总数、成功数、失败数
        "total": 2,
        "successful": 1,
        "failed": 0
    },
    "_seq_no": 0,
    "_primary_term": 1
}
```

## 5.2、[PUT/POST] 创建/更新文档

```json
// 创建或更新文档，指定主键 _id
// 当 ID 不存在时，created
// 当 ID 存在时，updated
> POST /<index_name>/_doc/1002

// Request
{
    "id": 1002,
    "title": "《春晓》",
    "author": "唐·孟浩然",
    "content": "春眠不觉晓，处处闻啼鸟。夜来风雨声，花落知多少。"
}

// Response
{
    "_index": "index_name",
    "_id": "1002",
    "_version": 1,
    "result": "created",
    "_shards": {
        "total": 2,
        "successful": 1,
        "failed": 0
    },
    "_seq_no": 1,
    "_primary_term": 1
}
```

## 5.3、[GET] 获取文档

```json
> GET /<index_name>/_doc/1001

// Response
{
    "_index": "index_name",
    "_id": "1001",
    "found": false
}

> GET /<index_name>/_doc/1002

// Response
{
    "_index": "index_name",
    "_id": "1002",
    "_version": 1,
    "_seq_no": 1,
    "_primary_term": 1,
    "found": true,
    "_source": {
        "title": "《春晓》",
        "author": "唐·孟浩然",
        "content": "春眠不觉晓，处处闻啼鸟。夜来风雨声，花落知多少。"
    }
}
```

## 5.4、[POST] 更新文档

```json
// 注意：这里用 _doc 操作的话，文档内容会被覆盖为以下内容，而不是仅修改字段
> POST /<index_name>/_update/1002

// Request
{
    "doc": {
        "title": "春晓~"
    }
}

// Response
{
    "_index": "index_name",
    "_id": "1002",
    "_version": 2,
    "result": "updated",
    "_shards": {
        "total": 2,
        "successful": 1,
        "failed": 0
    },
    "_seq_no": 2,
    "_primary_term": 1
}
```

## 5.5、[DELETE] 删除文档

```json
> DELETE /<index_name>/_doc/1002

// Response
{
    "_index": "index_name",
    "_id": "1002",
    "_version": 2,
    "result": "deleted",
    "_shards": {
        "total": 2,
        "successful": 1,
        "failed": 0
    },
    "_seq_no": 2,
    "_primary_term": 1
}
```

## 5.6、[POST] 批量操作文档

```json
PUT /index_name/_bulk

// Request（最后一行需要 \n 结尾）

{"index":{"_index":"index_name","_id":1001}}
{"id":1001,"title":"《静夜思》","author":"唐·李白","content":"床前明月光，疑是地上霜。举头望明月，低头思故乡。"}
{"index":{"_index":"index_name","_id":1002}}
{"id":1002,"title":"《春晓》","author":"唐·孟浩然","content":"春眠不觉晓，处处闻啼鸟。夜来风雨声，花落知多少。"}
{"index":{"_index":"index_name","_id":1003}}
{"id":1003,"title":"《寻隐者不遇》","author":"唐·贾岛","content":"松下问童子，言师采药去。只在此山中，云深不知处。"}
{"index":{"_index":"index_name","_id":1004}}
{"id":1004,"title":"《江雪》","author":"唐·柳宗元","content":"千山鸟飞绝，万径人踪灭。孤舟蓑笠翁，独钓寒江雪。"}
{"index":{"_index":"index_name","_id":1005}}
{"id":1005,"title":"《登鹳雀楼》","author":"唐·王之涣","content":"白日依山尽，黄河入海流。欲穷千里目，更上一层楼。"}
{"index":{"_index":"index_name","_id":1006}}
{"id":1006,"author":"唐·白居易","title":"《草 / 赋得古原草送别》","content":"离离原上草，一岁一枯荣。野火烧不尽，春风吹又生。远芳侵古道，晴翠接荒城。又送王孙去，萋萋满别情。"}

```

> Response

```json
{
    "errors": false,
    "took": 3200575433,
    "items": [
        {
            "index": {
                "_index": "index_name",
                "_id": "1001",
                "_version": 1,
                "result": "created",
                "_shards": {
                    "total": 2,
                    "successful": 1,
                    "failed": 0
                },
                "_seq_no": 0,
                "_primary_term": 1,
                "status": 201
            }
        },
        {
            "index": {
                "_index": "index_name",
                "_id": "1002",
                "_version": 1,
                "result": "created",
                "_shards": {
                    "total": 2,
                    "successful": 1,
                    "failed": 0
                },
                "_seq_no": 1,
                "_primary_term": 1,
                "status": 201
            }
        },
        {
            "index": {
                "_index": "index_name",
                "_id": "1003",
                "_version": 1,
                "result": "created",
                "_shards": {
                    "total": 2,
                    "successful": 1,
                    "failed": 0
                },
                "_seq_no": 2,
                "_primary_term": 1,
                "status": 201
            }
        },
        {
            "index": {
                "_index": "index_name",
                "_id": "1004",
                "_version": 1,
                "result": "created",
                "_shards": {
                    "total": 2,
                    "successful": 1,
                    "failed": 0
                },
                "_seq_no": 3,
                "_primary_term": 1,
                "status": 201
            }
        },
        {
            "index": {
                "_index": "index_name",
                "_id": "1005",
                "_version": 1,
                "result": "created",
                "_shards": {
                    "total": 2,
                    "successful": 1,
                    "failed": 0
                },
                "_seq_no": 4,
                "_primary_term": 1,
                "status": 201
            }
        }
    ]
}
```



# 六、Query DSL

Query DSL 是一种功能齐全的 JSON 样式查询语言，支持复杂的搜索、筛选和聚合。 它是当今 ES 的原始且最强大的查询语言。`_search` 端点接受用Query DSL语法编写的查询。

Query DSL 支持多种搜索技术，包括：

- 全文搜索：搜索经过分析（analyzed）和索引（indexed）的文本，以支持短语（`phrase`）或邻近（`proximity`）或模糊（`fuzzy`）匹配等。
- 关键字搜索：使用 `keyword` 字段搜索精确匹配项。
- 语义搜索：在 ES 集群中生成的嵌入上使用密集或稀疏向量搜索来搜索 semantic_text 字段。
- 向量搜索：使用 kNN 算法搜索类似的密集向量，以查找在 Elasticsearch 外部生成的嵌入。
- 地理空间搜索：使用地理空间查询搜索位置并计算空间关系。
- 过滤数据：filter 使您能够通过检索与特定字段级条件匹配的文档来包含或排除文档。



## 6.1、match_all

（1）匹配所有索引的所有文档，所有文档的 `_source: 1.0`

```json
> [GET|POST] /_search

{
    "query": {
        "match_all": {}
    }
}
```

（2）提高得分

```json
> [GET|POST] /_search

{
    "query": {
        "match_all": {
            "boost": 1.2
        }
    }
}
```

（3）匹配指定索引的所有文档

```json
> [GET|POST] /<index_name>/_search

{
    "query": {
        "match_all": {}
    }
}
```



## 6.2、全文查询

全文查询能够搜索分析过的 text 字段，如电子邮件正文。查询字符串使用与索引期间应用于字段的分析器相同的分析器进行处理。

### 6.2.1 intervals

一种全文查询，允许对匹配术语的顺序和接近度进行细粒度控制。

### 6.2.1 match

match 是用于执行全文查询的标准查询，包括模糊（fuzzy）匹配的选项。

match 查询在执行搜索之前分析任何提供的文本，这意味着 match 查询可以在 text 字段中搜索已分析的标记，而不是精确的术语。

支持类型：text、number、date、boolean。

```json
{
    "query": {
        "match": {
            "<field>": {
                // Required
                // 期望在提供的 <field> 中找到的 text、number、boolean、data 类型值
                "query": "<query_value>",
                
                // Options...
                
                // 用于对查询值中的文本进行分析
                // 默认为映射中 field 定义的 search_analyzer
                "analyzer": "",
                
                // 自动生成同义词短语查询
                "auto_generate_synonyms_phrase_query": true,
                
                // 相关性分数权重，默认：1.0
                // 0 ~ 1.0 会降低相关性得分
                // 大于 1.0 则会增加相关性得分
                "boost": 1.0,
                
                // 匹配允许的最大编辑距离。
                "fuzziness": "",
                
                // 查询将扩展到的最大术语数。
                "max_expansions": 50,
                
                // 模糊匹配中保持不变的起始字符数
                "prefix_length": 0,
                
                // 模糊匹配的编辑包括两个相邻字符的转置（ab→ba）
                "fuzzy_transpositions": true,
                
                // 用于重写查询的方法
                "fuzzy_rewrite": "",
                
                // 忽略基于格式的错误，
                // 例如为数字字段提供文本查询值。
                "lenient": false,
                
                // 用于解释查询值中文本的布尔逻辑。
                // OR (Default): `capital of Hungary` 被解释为 `capital OR of OR Hungary`.
                // AND: `capital of Hungary` 被解释为 `capital AND of AND Hungary`.
                "operator": "OR",
                
                // 文档必须匹配的最小子句数
                "minimum_should_match": "",
                
                // 指示 analyzer 在删除所有标记时（如：使用 stop 过滤器）是否返回文档
                // none (Default): 不会返回任何文档
                // all: 返回所有文档，类似于match_all查询
                "zero_terms_query": "none"
            }
        }
    }
}
```

> minimum_shold_match 的可选值

| Example     | Description                                                  |
| ----------- | ------------------------------------------------------------ |
| 3           | 表示一个固定值，与可选子句的数量无关。                       |
| -2          | 表示可选子句的总数                                           |
| 75%         | 表示可选子句总数的这一百分比是必需的。根据百分比计算的数字被四舍五入并用作最小值。 |
| -25%        | 表示可选子句总数的这一百分比可能缺失。根据百分比计算的数字被四舍五入，然后从总数中减去以确定最小值。 |
| 3<90%       | 如果可选子句的数量等于（或小于）整数，则它们都是必需的，但如果它大于整数，则适用规范。在这个例子中：如果有1到3个条款，它们都是必需的，但对于4个或更多条款，只需要90%。 |
| 2<-25% 9<-3 | 多个条件规范可以用空格分隔，每个条件规范仅对大于其前一个的数字有效。在这个例子中：如果有1或2个子句，则都需要，如果有3-9个子句，除了25%之外都需要，并且如果有9个以上的子句，则除了3个之外都需要。 |

> 例

```json
// 匹配文档中包含 this 或 is 或 test
// 不区分大小写
{
    "query": {
        "match": {
            "message": "this is test"
        }
    }
}

// 匹配文档中同时包含 this is test 的文档（顺序无关、位置无关）
{
    "query": {
        "match": {
            "message": "this is test",
            "operator": "AND"
        }
    }
}

// 模糊（拼写错误、同义词等）
{
    "query": {
        "match": {
            "message": {
                "query": "this is a testt",
                "fuzziness": "AUTO"
            }
        }
    }
}

// 当没有文档匹配时，返回所有（类似 match_all 查询）
{
    "query": {
        "match": {
            "message": {
                "query": "to be or not to be",
                "operator": "and",
                "zero_terms_query": "all"
            }
        }
    }
}
```



### 6.2.3 match_bool_prefix

match_bool_prefix 查询分析其输入，并根据术语构造 bool 查询。除最后一个术语外，每个术语都用 term 查询。最后一个术语用 prefix 查询。

```json
{
    "query": {
        "match_bool_prefix": {
            "<field>": {
                // Required
                // 期望在提供的 <field> 中找到的 text、number、boolean、data 类型值
                "query": "<query_value>",
                
                // Options...
                
                // 用于对查询值中的文本进行分析，生成的术语数量即为 bool 查询的子句数量
                // 默认为映射中 field 定义的 search_analyzer
                "analyzer": "",
                
                // 相关性分数权重，默认：1.0
                // 0 ~ 1.0 会降低相关性得分
                // 大于 1.0 则会增加相关性得分
                "boost": 1.0,
                
                // --------------------------------------------
                // fuzziness, prefix_length, max_expansions, 
                // fuzzy_transpositions, fuzzy_rewrite 
                // 可以应用于除最后一个项之外的所有项构造的 term 子查询。
                
                // 匹配允许的最大编辑距离。
                "fuzziness": "",
                
                // 查询将扩展到的最大术语数。
                "max_expansions": 50,
                
                // 模糊匹配中保持不变的起始字符数
                "prefix_length": 0,
                
                // 模糊匹配的编辑包括两个相邻字符的转置（ab→ba）
                "fuzzy_transpositions": true,
                
                // 用于重写查询的方法
                "fuzzy_rewrite": "",
                
                
                // -----------------------------------
                // minimum_should_match, operator
                // 将该设置应用于构造的全部 bool 查询
                
                // 用于解释查询值中文本的布尔逻辑。
                // OR (Default): `capital of Hungary` 被解释为 `capital OR of OR Hungary`.
                // AND: `capital of Hungary` 被解释为 `capital AND of AND Hungary`.
                "operator": "OR",
                
                // 文档必须匹配的最小子句数
                "minimum_should_match": ""
            }
        }
    }
}
```

> 案例

```json
{
    "query": {
        "match_bool_prefix" : {
            "message" : "quick brown f"
        }
    }
}

// 生成类似的 bool 查询
{
    "query": {
        "bool" : {
            "should": [
                { "term": { "message": "quick" }},
                { "term": { "message": "brown" }},
                { "prefix": { "message": "f"}}
            ]
        }
    }
}
```

match_bool_prefix 查询和 match_phrase_prefix 之间的一个重要区别是，match_phrase _prefix 请求将其术语作为短语进行匹配，但 match_bool_prefix 查询可以在任何位置匹配其术语。

### 6.2.4 match_phrase

类似于 match 查询，但用于匹配精确的短语或单词邻近匹配。

match_phrase 查询分析文本，并从分析的文本中创建 phrase 查询。

```json
{
    "query": {
        "match_phrase": {
            "<field>": {
                // Required
                // 期望在提供的 <field> 中找到的 text、number、boolean、data 类型值
                "query": "<query_value>",
                
                // Options...
                
                // 用于对查询值中的文本进行分析
                // 默认为映射中 field 定义的 search_analyzer
                "analyzer": "",
               
                // 相关性分数权重，默认：1.0
                // 0 ~ 1.0 会降低相关性得分
                // 大于 1.0 则会增加相关性得分
                "boost": 1.0,
                
                // 指示 analyzer 在删除所有标记时（如：使用 stop 过滤器）是否返回文档
                // none (Default): 不会返回任何文档
                // all: 返回所有文档，类似于match_all查询
                "zero_terms_query": "none",
                
                // 指定词项之间可以间隔的最大距离
                // 默认为 0（词项必须连续）
                "slop": 0
            }
        }
    }
}
```

> 案例

```json
{
    "query": {
        "match_phrase": {
            // 未指定分词器，默认将中文分词成单个字
            // 无法匹配到结果：黄河入海流
        	"content": "黄河海流"
        }
    }
}

{
    "query": {
        "match_phrase": {
            "content": {
                "query": "黄河海流",
                // 设置 slop 后，可以匹配：黄河入海流
                "slop": 1
            }
        }
    }
}

// 注意：索引分词器与搜索分词器不同时，可能导致精确匹配失败。
// "analyzer": "ik_max_word"
// "search_analyzer": "ik_smart"
```

### 6.2.5 match_phrase_prefix

与 match_phrase 查询类似，但对最后一个单词进行 prefix 通配符搜索。

```json
{
    "query": {
        "match_phrase_prefix": {
            "<field>": {
                // Required
                // 期望在提供的 <field> 中找到的 text、number、boolean、data 类型值
                "query": "<query_value>",

                // Options...

                // 用于对查询值中的文本进行分析
                // 默认为映射中 field 定义的 search_analyzer
                "analyzer": "",

                // 相关性分数权重，默认：1.0
                // 0 ~ 1.0 会降低相关性得分
                // 大于 1.0 则会增加相关性得分
                "boost": 1.0,
                
                // 查询将扩展到的最大术语数。
                "max_expansions": 50,

                // 指示 analyzer 在删除所有标记时（如：使用 stop 过滤器）是否返回文档
                // none (Default): 不会返回任何文档
                // all: 返回所有文档，类似于match_all查询
                "zero_terms_query": "none",

                // 指定词项之间可以间隔的最大距离
                // 默认为 0（词项必须连续）
                "slop": 0
            }
        }
    }
}
```

> 案例

```json
{
    "query": {
        "match_phrase_prefix": {
            "message": {
                // 匹配短语 `quick brown f*`
                "query": "quick brown f"
            }
        }
    }
}
```

### 6.2.6 multi_match

multi_match 查询基于 match 查询构建，以允许多字段查询。

```json
{
    "query": {
        "multi_match" : {
            // Required...
            
            // 期望在提供的 <fields> 中找到的 text、number、boolean、data 类型值
            "query": "<query_value>",

            // 字段名可以用通配符指定（如：`*_name` 可以匹配 first_name 和 last_name 字段）
            // 可以用 ^ 增加字段的得分（如：`title^3` 将 title 字段的得分乘以 3）
            // fields 为空时，将从所有字段中匹配
            "fields": [ "subject", "*_name", "title^3" ],
            
            // Options...
            
            // multi_match 查询在内部执行的方式取决于 type 参数
            "type": "best_fields",
            
            // 
        }
    }
}
```

> multi_match 的 type 参数的可选值：

| type                  | 说明                                                         |
| --------------------- | ------------------------------------------------------------ |
| best_fields (Default) | 查找与任何字段匹配的文档，但使用最佳字段中的 _score。        |
| most_fields           | 查找与任何字段匹配的文档，并组合每个字段的 _score。          |
| cross_fields          | 用相同的分析器处理字段，就像它们是一个大字段一样，查找任何字段中的每个单词。 |
| phrase                | 对每个字段运行 match_prase 查询，并使用最佳字段中的 _score。 |
| phrase_prefix         | 对每个字段运行 match_prash_prefix 查询，并使用最佳字段中的 _score。 |
| bool_prefix           | 在每个字段上创建 match_bool_prefix 查询，并组合每个字段的 _score。 |

**（1）best_fields**

在同一字段中搜索多个最佳单词时，best_fields 类型最有用。例如一个字段中的 brown fox 比一个字段的 brown 和另一个字段的 fox 更有意义。

best_fields 类型为每个字段生成一个匹配查询，并将其包装在 dis_max 查询中，以找到单个最佳匹配字段。

通常，best_fields 类型使用单个最佳匹配字段的分数，如果指定了 tie_breaker，则会按如下方式计算得分：最佳匹配字段的得分 + （所有其他匹配字段的得分 * tie_breaker）。

```json
{
    "query": {
        "multi_match" : {
            "query":      "brown fox",
            "type":       "best_fields",
            "fields":     [ "subject", "content" ],
            "tie_breaker": 0.3

            // [match] Options...
            // analyzer,
            // boost,
            // operator,
            // minimum_should_match,
            // fuzziness,
            // lenient,
            // prefix_length,
            // max_expansions,
            // fuzzy_rewrite,
            // zero_terms_query,
            // auto_generate_synonyms_phrase_query,
            // fuzzy_transpositions,
        }
    }
}

// 将按以下方式执行
// best_fields 和 most_fields 类型是以字段为中心的（为每个字段生成一个匹配查询）

{
    "query": {
        "dis_max": {
            "queries": [
                { "match": { "subject": "brown fox" }},
                { "match": { "content": "brown fox" }}
            ],
            "tie_breaker": 0.3
        }
    }
}
```

> tie_breaker

| tie_breaker | 说明                                                     |
| ----------- | -------------------------------------------------------- |
| 0.0（默认） | 获取单个最佳匹配字段的分数                               |
| 1.0         | 每个 match 子句的分数相加                                |
| 0.0~1.0     | 取单个最佳分数 + tie_breaker * 其他匹配字段/组的每个分数 |

**（2）most_fields**

当查询包含以不同方式分析的相同文本的多个字段时，most_fields 类型最有用。

每个 match 子句的分数加在一起，就像 bool 查询一样。

```json
{
    "query": {
        "multi_match" : {
            "query":      "quick brown fox",
            "type":       "most_fields",
            "fields":     [ "title", "title.original", "title.shingles" ]
            
            // [match] Options...
            // analyzer,
            // boost,
            // operator,
            // minimum_should_match,
            // fuzziness,
            // lenient,
            // prefix_length,
            // max_expansions,
            // fuzzy_rewrite,
            // zero_terms_query
        }
    }
}

// 将按以下方式执行
// best_fields 和 most_fields 类型是以字段为中心的（为每个字段生成一个匹配查询）

{
    "query": {
        "bool": {
            "should": [
                { "match": { "title":          "quick brown fox" }},
                { "match": { "title.original": "quick brown fox" }},
                { "match": { "title.shingles": "quick brown fox" }}
            ]
        }
    }
}
```

**（3）cross_fields**

cross_fields 类型对于需要匹配多个字段的结构化文档特别有用。

cross_field 类型采取以术语为中心的方法。它首先将查询字符串分析为单个术语，然后在任何字段中查找每个术语，就像它们是一个大字段一样。换句话说，所有术语必须至少出现在一个字段中，文档才能匹配。

cross_field 类型只能在具有相同分析器的字段上以术语为中心的模式工作。具有相同分析器的字段被分组在一起，如果有多个组，查询将使用任何组中的最佳分数。

```json
{
    "query": {
        "multi_match" : {
            "query":      "Will Smith",
            "type":       "cross_fields",
            "fields":     [ "first_name", "last_name" ],
            
            // 所有术语必须同时出现在一个字段中，文档才能匹配
            "operator":   "and",
            
            // 通过在查询中指定分析器参数，可以将所有字段强制归入同一组。
            "analyzer":   "standard"
        }
    }
}


```

**（4）phrase & phrase_prefix**

```json
{
    "query": {
        "multi_match" : {
            "query":      "quick brown f",
            "type":       "phrase_prefix",
            "fields":     [ "subject", "message" ]

            // [match] Options...
            // analyzer, boost, lenient and zero_terms_query

            // [match_prash] Options...
            // slop

            // [match_prefix] Options...
            // max_expansions
        }
    }
}

// 将按以下方式执行（行为与 best_fields 一样）

{
    "query": {
        "dis_max": {
            "queries": [
                { "match_phrase_prefix": { "subject": "quick brown f" }},
                { "match_phrase_prefix": { "message": "quick brown f" }}
            ]
        }
    }
}
```

**（5）bool_prefix**

```json
{
    "query": {
        "multi_match" : {
            "query":      "quick brown f",
            "type":       "bool_prefix",
            "fields":     [ "subject", "message" ]
            
            // [match] Options...
            // analyzer, boost, operator, minimum_should_match, lenient, zero_terms_query, and auto_generate_synonyms_phrase_query
            
            // [term] Options...
            // fuzziness, prefix_length, max_expansions, fuzzy_rewrite, fuzzy_transpositions
        }
    }
}
```

### 6.2.7 combined_fields

combined_fields 查询提供了一种以术语为中心的方法，可以按术语处理 operator 和 minimum_shold_match。

combined_fields 查询支持搜索多个 text 字段，就像它们的内容已被索引到一个组合字段中一样。

查询采用以术语为中心的输入字符串视图：首先，它将查询字符串分析为单个术语，然后在任何字段中查找每个术语。当匹配可以跨越多个 text 字段时，例如文章的标题、摘要和正文，此查询特别有用。

```json
{
    "query": {
        "combined_fields": {
            // Required...

            // 在提供的 <fields> 中搜索的文本
            "query": "<query_value>",

            // 要搜索的字段列表。
            // 允许使用字段通配符模式，允许字段提分。
            // 仅支持文本字段，并且它们必须具有相同的搜索分析器。
            // 字段数量限制，默认 4096
            "fields": [ "field_1", "field_2", "..." ],

            // Options...

            // 自动生成同义词短语查询
            "auto_generate_synonyms_phrase_query": true,

            // 用于解释查询值中文本的布尔逻辑。
            // OR (Default): `capital of Hungary` 被解释为 `capital OR of OR Hungary`.
            // AND: `capital of Hungary` 被解释为 `capital AND of AND Hungary`.
            "operator": "OR",

            // 文档必须匹配的最小子句数
            "minimum_should_match": "",

            // 指示 analyzer 在删除所有标记时（如：使用 stop 过滤器）是否返回文档
            // none (Default): 不会返回任何文档
            // all: 返回所有文档，类似于match_all查询
            "zero_terms_query": "none"
        }
    }
}
```

> 示例

```json
{
    "query": {
        "combined_fields" : {
            "query":      "database systems",
            "fields":     [ "title^2", "abstract", "body"],
            "operator":   "and"
        }
    }
}
```

> 与 multi_match 查询的比较

- combined_fields 查询提供了一种跨多个文本字段进行匹配和评分的原则性方法，他要求所有字段都具有相同的搜索分析器。
- multi_match 查询更适合处理不同类型字段（如 keyword 或 number）的单一查询，他支持文本和非文本字段，并接受不同分析器的文本字段。
- combined_fields 是以术语为中心的：operator 和 minimum_shold_match 是按术语应用的，而不是按字段应用的。换句话说，每个术语必须至少出现在一个字段中，文档才能匹配。
- combined_fields 相比 multi_field 的 cross_fields 模式的主要优势是其基于 BM25F 算法的鲁棒性和可解释的评分方法。

### query_string

支持简洁的 Lucene 查询字符串语法，允许您在单个查询字符串中指定 AND|OR|NOT 条件和多字段搜索。仅限专家用户。

### simple_query_string

一个更简单、更健壮的 query_string 语法版本，适合直接向用户公开。



## 6.3、术语级查询

使用术语级查询根据结构化数据中的精确值查找文档，结构化数据包括：日期范围、IP地址、价格、产品ID。

术语级查询不分析搜索词，仍然会使用 normalizer 属性对 keyword 字段的搜索词规范化。

### 6.3.1 exists

返回字段存在索引值的文档。

```json
{
    "query": {
        // Required
        // 如果 JSON 值为 null 或 [], 则认为字段不存在
        "field": "<field_name>"
    }
}
```

由于多种原因，文档字段的索引值可能不存在：

- 源 JSON 中的字段为 null 或 []。
- 该字段在映射中设置了 `"index"：false, "doc_values"：false`。
- 字段值的长度超过了映射中的 ignore_abover 设置。
- 字段值格式不正确，映射中定义了 ignore_malformed。

```json
// 返回缺少索引值的文档
{
    "query": {
        "bool": {
            "must_not": {
                "exists": {
                    "field": "user.id"
                }
            }
        }
    }
}
```

### 6.3.2 fuzzy

返回包含与搜索词相似的词的文档，以 Levenshtein 编辑距离衡量。编辑距离是将一个术语转换为另一个术语所需的字符更改的数量。这些变化可能包括：

- 更改字符（box→fox）
- 删除字符（black→lack）
- 插入字符（sic→sick）
- 转换两个相邻的字符（act→cat）

为了找到相似的词，模糊查询会在指定的编辑距离内创建一组搜索词的所有可能变体或扩展。然后，查询将返回每个扩展的精确匹配。

```json
{
    "query": {
        "fuzzy": {
            "<field>": {
                // Required
                // 期望在提供的 <field> 中找到的术语
                "value": "<query_value>",
                
                // Options...
                
                // 相关性分数权重，默认：1.0
                // 0 ~ 1.0 会降低相关性得分
                // 大于 1.0 则会增加相关性得分
                "boost": 1.0,
                
                // 匹配允许的最大编辑距离。
                "fuzziness": "",
                
                // 创建的最大变体数（默认：50）。
                "max_expansions": 50,
                
                // 创建扩展时保持不变的开头字符数（默认：0）。
                "prefix_length": 0,
                
                // 指示编辑是否包括两个相邻字符的转置（ab→ba）
                "transpositions": true,
                
                // 用于重写查询的方法
                "rewrite": ""
            }
        }
    }
}
```

> 示例

```json
{
    "query": {
        "fuzzy": {
            "name": {
                // 匹配 ki 的近视词，如：kib, ik, 
                "value": "ki"
            }
        }
    }
}

// 使用高级参数
{
    "query": {
        "fuzzy": {
            "name": {
                "value": "ki",
                "fuzziness": "AUTO",
                "max_expansions": 50,
                "prefix_length": 0,
                "transpositions": true,
                "rewrite": "constant_score_blended"
            }
        }
    }
}
```

### 6.3.3 ids

根据文档 IDs 返回文档，此查询使用存储在 `_id` 字段中的文档 IDs。

```json
{
    "query": {
        "ids" : {
            // Required，文档 ID 数组
            "values" : [1, 2, 3]
        }
    }
}
```

### 6.3.4 prefix

返回在提供的字段中包含特定前缀的文档。

```json
{
    "query": {
        "prefix": {
            "<field>": {
                // Required
                // 希望在提供的 <field> 中找到的术语的开头字符
                "value": "<prefix_value>",
                
                // Options...
                
                // 用于重写查询的方法
                "rewrite": "",
                
                // 允许将值与索引字段值进行 ASCII 不区分大小写的匹配（默认：false）
                "case_insensitive": false
                
            }
        }
    }
}
```

> 示例

```json
{
    "query": {
        "prefix": {
            "name": {
                // 匹配 name：ki*
                "value": "ki"
            }
        }
    }
}
```

加快前缀查询：可以使用 index_prefixes 映射参数来加速前缀查询。如果启用，ES 将根据配置设置在单独的字段中索引前缀。这使得 ES 能够以更大的索引为代价更有效地运行前缀查询。
允许昂贵的查询：如果 search.allow_expensive_queries 设置为 false，则不会执行前缀查询。但是，如果启用了index_prefixes，则会构建一个优化的查询，该查询不会被认为很慢，并且尽管有此设置，它仍将被执行。

### 6.3.5 range

返回包含所提供范围内的术语的文档。

```json
{
    "query": {
        "range": {
            "<field>": {
                
                // Options...
                
                // date, numeric 数据类型可用
                "gt": 0,
                "gte": 0,
                "lt": 0,
                "lte": 0,
                
                // 用于转换查询中的 date 值的日期格式
                // 默认使用映射中 <field> 提供的日期格式。
                "format": "",
                
                // 指示范围查询如何匹配范围字段的值
                // INTERSECTS 匹配具有与查询范围相交的范围字段值的文档
                // CONTAINS 将文档与完全包含查询范围的范围字段值进行匹配。
                // WITHIN 匹配具有完全在查询范围内的范围字段值的文档。
                "relation": "INTERSECTS",
                
                // 时区，用于将查询中的日期值转换为 UTC 的协调世界时
                // 如：+08:00 或 America/Los_Angeles
                "time_zone": "",
                
                // 相关性分数权重，默认：1.0
                // 0 ~ 1.0 会降低相关性得分
                // 大于 1.0 则会增加相关性得分
                "boost": 1.0
            }
        }
    }
}
```

> 示例
>
> [Date Math](https://www.elastic.co/guide/en/elasticsearch/reference/current/common-options.html#date-math)

```json
// 匹配 timestamp 介于昨天到今天的文档
{
    "query": {
        "range": {
            "timestamp": {
                "gte": "now-1d/d",
                "lte": "now/d"
            }
        }
    }
}

// 对于范围查询和日期范围聚合，ES 用以下值替换缺失的日期组件，缺失的年份部件不予更换。
// MONTH_OF_YEAR:    01
// DAY_OF_MONTH:     01
// HOUR_OF_DAY:      23
// MINUTE_OF_HOUR:   59
// SECOND_OF_MINUTE: 59
// NANO_OF_SECOND:   999_999_999

// 缺失的日期组件
{
    "query": {
        "range": {
            // ES 会将 gt 值转换为：
            // 2099-12-01T23:59:59.999_999_999Z
            "timestamp": {
                "gt": "2024-12"
            }
        }
    }
}
```

如果 search.allow_expensive_queries 设置为 false，则不会对 text 或 keyword 字段执行范围查询。

### 6.3.6 regexp

返回包含与正则表达式匹配的术语的文档。

```json
{
    "query": {
        "regexp": {
            "<field>": {
                // Required，默认正则表达式限制为 1000 个字符
                // 可以匹配：ky key kimchy
                "value": "k.*y",
                
                // Options...
                
                // 为正则表达式启用可选运算符。
                "flags": "ALL",
                
                // 允许正则表达式值和索引字段值进行不区分大小写的匹配（默认：false）
                "case_insensitive": false,
                
                // 查询所需的自动机状态的最大数量（默认：10000）
                "max_determinized_states": 10000,
                
                // 用于重写查询的方法。
                "rewrite": "constant_score_blended"
            }
        }
    }
}
```

### 6.3.7 term

可以使用 term 查询根据精确值（如价格、产品ID或用户名）查找文档。

```json
{
    "query": {
        "term": {
            "<field>": {
                // Required
                // 希望在提供的 <field> 中找到的术语
                "value": "kimchy",
                
                // Options...
                
                // 相关性分数权重，默认：1.0
                // 0 ~ 1.0 会降低相关性得分
                // 大于 1.0 则会增加相关性得分
                "boost": 1.0,
                
                // 允许将值与索引字段值进行 ASCII 不区分大小写的匹配（默认：false）
                "case_insensitive": false
            }
        }
    }
}
```

避免对 text 字段使用 term 查询，改用 match 查询。

默认情况下，ES 会在分析过程中更改 text 字段的值，默认 standard 分析器按如下方式更改文本字段值：

- 删除大部分标点符号。
- 将剩余内容划分为单独的单词，称为标记。
- 将标记小写

为了更好地搜索 text 字段，match 查询还会在执行搜索之前分析您提供的搜索词。这意味着 match 查询可以在 text 字段中搜索已分析的标记，而不是精确的术语。
term 查询不分析搜索词。term 查询仅搜索您提供的确切术语。这意味着在搜索 text 字段时，term 查询可能会返回较差的结果或没有结果。

> 示例

```json
// term 查询不执行分词器，大写字母无法匹配，适合keyword 、numeric、date 数据类型。

{
    "query": {
        // term 匹配一个关键词
        // 由于 term 不执行分词器，而目标文档被分成了单汉字的词组，故无法匹配到目标
        "term": {
            "content": "黄河"
        }
    }
}
```



### 6.3.8 terms

返回在提供的字段中包含一个或多个精确术语的文档。

terms 查询与 term 查询相同，只是可以搜索多个值。如果文档中至少包含一个术语，则该文档将匹配。

要搜索包含多个匹配词的文档，请使用 terms_set 查询。

```json
{
    "query": {
        "terms": {
            // Required
            // 希望在提供的 <field> 中找到的术语数组（类似 IN 查询）
            "<field>": ["term_value1", "term_value2", "..."],

            // Options...

            // 相关性分数权重，默认：1.0
            // 0 ~ 1.0 会降低相关性得分
            // 大于 1.0 则会增加相关性得分
            "boost": 1.0
        }
    }
}
```

> 示例

```json
{
    "query": {
        // terms 匹配多个关键词
        "terms": {
            "content": ["黄", "河"]
        }
    }
}
```



> Terms lookup

terms lookup 获取现有文档的字段值，ES 将这些值用作搜索词。在搜索大量术语时，这可能会很有帮助。

要运行术语查找，必须启用字段的 _source。您不能使用跨集群搜索在远程索引上运行术语查找。

默认情况下，ES 将 terms lookup 限制为最多 65536 个术语。这包括使用 terms lookup 获取的术语。您可以使用 index.max_terms_count 设置更改此限制。

为了减少网络流量，如果可能的话，术语查找将从本地数据节点上的分片中获取文档的值。如果您的术语数据不大，请考虑使用具有单个主分片的索引，该索引在所有适用的数据节点上完全复制，以最大限度地减少网络流量。

```json
{
    "query": {
        "terms": {
            "color" : {
                // Required...
                
                // 要获取字段值的索引名
                "index" : "<index_name>",
                
                // 要获取字段的文档 ID
                "id" : "<id>",
                
                // 要获取的字段
                "path" : "<field>",
                
                // Options...
                
                // 从中获取术语值的文档的自定义路由值。
                "routing": ""
            }
        }
    }
}
```

> 示例

```json
// 使用带有术语查找参数的 terms 查询来查找包含一个或多个与文档 ID: 2 相同的术语的文档。

// 包含 pretty 参数，使响应更具可读性。
> GET /<index_name>/_search?pretty
{
    "query": {
        "terms": {
            "color" : {
                "index" : "my-index-000001",
                "id" : "2",
                "path" : "color"
            }
        }
    }
}
```



### 6.3.9 terms_set

返回在提供的字段中包含最少数量的精确术语的文档。

```json
{
    "query": {
        "terms_set": {
            "<field>": {
                // Required...
                // 希望在提供的 <field> 中找到的术语数组（类似 IN 查询）
                "terms": ["term_value1", "term_value2", "..."],

                // Options...

                // 相关性分数权重，默认：1.0
                // 0 ~ 1.0 会降低相关性得分
                // 大于 1.0 则会增加相关性得分
                "boost": 1.0,

                // 以下参数三选一，定义匹配术语的数量

                // 匹配术语的数量
                "minimum_should_match": 0,

                // 匹配术语数量的字段（number 型）
                "minimum_should_match_field": "",

                // 匹配术语数量的自定义脚本
                "minimum_should_match_script": ""
            }
        }
    }
}
```

### 6.3.10 wildcard

返回包含与通配符模式匹配的术语的文档。

如果 `search.allow_expensive_queries` 设置为 false，则不会执行通配符查询。

```json
{
    "query": {
        "wildcard": {
            "<field>": {
                // Required...
                
                // 希望在提供的 <field> 中找到的通配符模式（类似 LIKE 查询）
                // 支持两种通配符：
                //  ? 匹配任何单个字符
                //  * 匹配零或多个字符
                "value": "<wildcard_value>",
                
                // value 参数的别名
                // 如果同时指定 value 和 wildcard，则最后一个参数生效
                "wildcard": "wildcard_value",

                // Options...

                // 相关性分数权重，默认：1.0
                // 0 ~ 1.0 会降低相关性得分
                // 大于 1.0 则会增加相关性得分
                "boost": 1.0,

                // 允许模式和索引字段值进行不区分大小写的匹配（默认：false）
                "case_insensitive": false,
                
                // 用于重写查询的方法
                "rewrite": ""
            }
        }
    }
}
```

> 示例

```json
{
    "query": {
        "wildcard": {
            "name": {
                "value": "ki*y",
                "boost": 1.0,
                "rewrite": "constant_score_blended"
            }
        }
    }
}
```



## 6.4、复合查询

复合查询包裹其他复合或叶子查询，以组合其结果和分数、改变其行为、或从查询切换到 filter 上下文。

### 6.4.1、Query & Filter

默认情况下 ES 按相关性得分对匹配的搜索结果进行排序，相关性得分衡量每个文档与查询的匹配程度。

在 query 上下文中，查询子句除了决定文档是否匹配外，还在 _source 元数据字段中计算相关性得分。

在 filter 上下文中，筛选器子句根据是/否标准决定文档匹配，而无需计算得分。filter 有以下好处：

- 简单二进制逻辑：是或否。
- 性能：filter 不计算相关性得分，因此 filter 条件的执行速度比 query 快。
- 缓存：ES 会自动缓存常用的筛选条件，从而加快后续的搜索性能。
- 资源效率：与全文查询相比， filter 条件消耗的 CPU 资源更少。
- 查询组合：filter 条件可以与评分查询结合使用，以有效的优化结果集。



filter 对于查询结构化数据和在复杂搜索中实现 `must have` 条件特别有效。结构化数据是指以预定义的方式高度组织和格式化的信息。（包括：numeric, date, boolean, keyword）是精确过滤操作的理想类型。

常见的 filter 应用：

- 日期范围检查：`{ "range": { "publish_date": { "gte": "2015-01-01" }}}`
- 特定字段值检查：`{ "term":  { "status": "published" }}`



### 6.4.2 Boolean 查询

用于将多个叶子或复合查询子句组合的默认查询，如 must，should，must_not，filter 子句。

must 子句和 should 子句的分数合并在一起，匹配的子句越多越好；而 must_not 子句和 filter 子句是在 filter 上下文中执行的。

must：子句（查询）必须出现在匹配的文档中，并将有助于得分。

should：子句（查询）可能出现在匹配的文档中。

must_not：子句（查询）不得出现在匹配的文档中。子句在 filter 上下文中执行，这会忽略评分（_score: 0），并考虑缓存子句。

filter：子句（查询）必须出现在匹配的文档中。会忽略评分（_score: 0），并考虑缓存子句。

```json
{
    "query": {
        "bool": {
            // 类似 AND 查询
            "must": [
                {
                    "match_query": {}
                },
                {
                    "terms_query": {}
                }
                // query: {...}
                // 这是一个 []query 数组，每个元素都适用 query 查询语法
            ],

            // 类似 OR 查询
            "should": [
                {
                    "match_query": {}
                },
                {
                    "terms_query": {}
                }
                // {...}
            ],

            // 类似 NOT 查询（不评分，有缓存）
            "must_not": [
                {
                    "match_query": {}
                },
                {
                    "terms_query": {}
                }
                // {...}
            ],

            // 不评分，有缓存
            "filter": [
                {
                    "match_query": {}
                },
                {
                    "terms_query": {}
                }
                // {...}
            ],

            // 指定返回文档的 should 子句必须匹配的数量或百分比
            // 如果 bool 查询至少包含一个 should 子句，并且没有 must 或 filter 子句，则默认值为 1。否则，默认值为 0。
            "minimum_should_match": 1,
            "boost": 1.0
        }
    }
}
```

> 示例

```json
// 以下三个查询都返回 status 包含 active 术语的所有文档

// filter 查询，_score: 0
{
    "query": {
        "bool": {
            "filter": {
                "term": {
                    "status": "active"
                }
            }
        }
    }
}

// match_all 查询，_score: 1.0
{
    "query": {
        "bool": {
            "must": {
                "match_all": {}
            },
            "filter": {
                "term": {
                    "status": "active"
                }
            }
        }
    }
}

// constant_score 查询，_score: 1.0
{
    "query": {
        "constant_score": {
            "filter": {
                "term": {
                    "status": "active"
                }
            }
        }
    }
}
```



### 6.4.3 Boosting

返回与正查询（positive）匹配的文档，同时降低与负查询（negative）也匹配的文档的相关性得分。

可以使用 boosting 查询来降级某些文档，而不会将其从搜索结果中排除。

```json
{
    "query": {
        "boosting": {
            // Required
            // 要运行的查询，返回与此查询匹配的文档
            "positive": {
                "term": {
                    "text": "apple"
                }
            },
            
            // Required
            // 查询用于降低匹配文档的相关性得分
            // 最终得分 = _score * negative_boost 
            "negative": {
                "term": {
                    "text": "pie tart fruit crumble tree"
                }
            },
            
            // Required
            // 用于降低匹配文档的相关得分，取值：0 ~ 1.0 
            "negative_boost": 0.5
        }
    }
}
```



### 6.4.4 Constant score

包装一个 filter 查询，并返回每个匹配的文档，其相关性得分等于 boost 参数值。

```json
{
    "query": {
        "constant_score": {
            // Required
            // 筛选要运行的查询，返回与此查询匹配的文档
            "filter": {
                "term": { "name": "kimchy" }
            },
            
            // 与 filter 查询匹配的每个文档的常量相关性分数（默认：1.0）
            "boost": 1.0
        }
    }
}
```



## 6.5、查询场景示例

### 6.5.1 全量查询

> SELECT * FROM table;

```json
> [GET|POST] /<index_name>/_search

// Request
{
    "query": {
        "match_all": {}
    }
}

// Response
{
    "took": 2,
    "timed_out": false,
    "_shards": {
        "total": 1,
        "successful": 1,
        "skipped": 0,
        "failed": 0
    },
    "hits": {
        "total": {
            "value": 5,
            "relation": "eq"
        },
        "max_score": 1.0,
        "hits": [
            {
                "_index": "index_name",
                "_id": "1003",
                "_score": 1.0,
                "_source": {
                    "id": 1003,
                    "title": "《寻隐者不遇》",
                    "author": "唐·贾岛",
                    "content": "松下问童子，言师采药去。只在此山中，云深不知处。"
                }
            },
            {
                "_index": "index_name",
                "_id": "1004",
                "_score": 1.0,
                "_source": {
                    "id": 1004,
                    "title": "《江雪》",
                    "author": "唐·柳宗元",
                    "content": "千山鸟飞绝，万径人踪灭。孤舟蓑笠翁，独钓寒江雪。"
                }
            },
            {
                "_index": "index_name",
                "_id": "1005",
                "_score": 1.0,
                "_source": {
                    "id": 1005,
                    "title": "《登鹳雀楼》",
                    "author": "唐·王之涣",
                    "content": "白日依山尽，黄河入海流。欲穷千里目，更上一层楼。"
                }
            },
            {
                "_index": "index_name",
                "_id": "1001",
                "_score": 1.0,
                "_source": {
                    "id": 1001,
                    "title": "《静夜思》",
                    "author": "唐·李白",
                    "content": "床前明月光，疑是地上霜。举头望明月，低头思故乡。"
                }
            },
            {
                "_index": "index_name",
                "_id": "1002",
                "_score": 1.0,
                "_source": {
                    "id": 1002,
                    "title": "《春晓》",
                    "author": "唐·孟浩然",
                    "content": "春眠不觉晓，处处闻啼鸟。夜来风雨声，花落知多少。"
                }
            }
        ]
    }
}
```

> 返回值说明

| 参数                              | 类型   | 说明                                                    |
| --------------------------------- | ------ | ------------------------------------------------------- |
| took: 2                           | int    | 查询花费时长（ms）                                      |
| timed_out: false                  | bool   | 请求是否超时                                            |
| _shards: {}                       | object | 搜索的分片信息                                          |
| _shards.total: 1                  | int    | 总数                                                    |
| _shards.sucessful: 1              | int    | 成功                                                    |
| _shards.skipped: 0                | int    | 跳过                                                    |
| _shards.failed: 0                 | int    | 失败                                                    |
| hits: {}                          | object |                                                         |
| hits.total: {}                    | object | 匹配的文档总数                                          |
| hits.total.value: 5               | int    | 匹配的文档数（可能不是准确值）                          |
| hits.total.relation: "eq"         | string | 当文档总数大于 10000 时，此处可能是 gte                 |
| hits.max_score: 1.0               | float  | 最相关的文档评分（最高得分）                            |
| hits.hits: []                     | array  | 数据列表，默认包含前10条                                |
| hits.hits[0]._index: "index_name" | string | 文档所在索引                                            |
| hits.hits[0]._id: "1001"          | string | 文档 ID                                                 |
| hits.hits[0]._score: 1.0          | float  | 文档得分（match_all，默认：1.0）                        |
| hits.hits[0]._source: {}          | object | 文档的源数据                                            |
| hits.hits[0].sort: []             | array  | 文档排序方式 （指定 sort 时才有，默认按相关性分数排序） |

### 6.5.2 返回指定字段（_source）

> ```mysql
> SELECT a, b, c FROM `table`;
> ```

```json
// 返回指定字段
{
    "query": {
        "match_all": {},
    },
    "_source": ["name", "age"]
}

// 包含需要的字段，排除不需要的字段，可以使用通配符 *
{
    "query": {
        "match_all": {},
        "_source": {
            "includes": ["age", "name"],
            "excludes": ["addr*", "*time"]
        }
    }
}

// 返回值中不包含 _source ，包含每个命中的 fields 的数组。
{
    "query": {
        "match_all": {}
    },
    "_source": false,
    "fields": [
        "name", "age"
    ]
}
```

### 6.5.3 范围搜索（range）

> ```mysql
> SELECT * FROM `table` WHERE created_time BETWEEN '2021-01-01' AND '2021-12-31';
> ```

```json
// gt、lt 形式
{
    "query": {
        "range": {
            "created_time": {
                "gte": "2021-01-01",
                "lte": "2021-12-31"
            }
        }
    }
}

// from、to 形式，默认包含两端的值
{
    "query": {
        "range": {
            "created_time": {
                "from": "2021-01-01",
                "to": "2021-12-31",
                "include_lower": true,
                "include_upper": false
            }
        }
    }
}

// 搜索昨天的数据
{
    "query": {
        "range": {
            "created_time": {
                "gte": "now-1d/d",
                "lt": "now/d"
            }
        }
    }
}
```

### 6.5.4 IN 查询（terms）

> ```mysql
> SELECT * FROM `table` WHERE `status` IN (1, 2, 3);
> ```

```json
{
    "query": {
        "terms": {
            "status": [1, 2, 3]
        }
    }
}
```

### 6.5.5 LIKE 查询（wildcard）

仅 keyword 字段。

> ```mysql
> SELECT * FROM `table` WHERE name LIKE 'YCZ_?';
> ```

```json
{
	"query": {
        // 支持通配符：? 和 *
    	"wildcard": {
            "name": "YCZ_?"
        }
	}
}
```

### 6.5.6 统计总数（COUNT）

ES 默认限制索引最多只能查询，当匹配的总数大于 10000 时，无法得到精确值。

**（1）默认总数的最大值（10000）**

```json
// Request
{
    "query": {
        "bool": {
            // ...
        }
    }
}

// Response
{
    "took": 717,
    "timed_out": false,
    "_shards": {
        "total": 3,
        "successful": 3,
        "skipped": 0,
        "failed": 0
    },
    "hits": {
        // 默认的最大值，total >= 10000
        "total": {
            "value": 10000,
            "relation": "gte"
        },
        "max_score": null,
        "hits": [
            {
                // ...
            }
        ]
    }
}
```

**（2）指定总数的最大值**

```json
// Request
{
    "query": {
        "bool": {
            // ...
        }
    },
    "track_total_hits" : 20000
}

// Response
{
    "took": 785,
    "timed_out": false,
    "_shards": {
        "total": 3,
        "successful": 3,
        "skipped": 0,
        "failed": 0
    },
    "hits": {
        // 指定的最大值，total >= 20000
        "total": {
            "value": 20000,
            "relation": "gte"
        },
        "max_score": null,
        "hits": [
            {
                // ...
            }
        ]
    }
}
```

**（3）返回精确的总数**

> ```mysql
> SELECT COUNT(*) FROM `table`;
> ```

```json
// Request
{
    "query": {
        "bool": {
            // ...
        }
    },
    // 为 false时，ES 不会尝试计算文档数量
    "track_total_hits" : true
}

// Response
{
    "took": 785,
    "timed_out": false,
    "_shards": {
        "total": 3,
        "successful": 3,
        "skipped": 0,
        "failed": 0
    },
    "hits": {
        // 精确返回
        "total": {
            "value": 64000000,
            "relation": "eq"
        },
        "max_score": null,
        "hits": [
            {
                // ...
            }
        ]
    }
}
```

### 6.5.7 排序（sort）

> ```mysql
> SELECT * FROM `table` ORDER BY field1 ASC, field2 DESC;
> ```

```json
{
    "query": {
        "match_all": {}
    },
    
    // 排序
    "sort": {
        "field1": "asc",
        // or
        "field2": {
            "order": "desc"
        },
    }
}
```

### 6.5.8 分页

> ```mysql
> SELECT * FROM `table` LIMIT 10 OFFSET 0;
> ```

```json
{
    "query": {
        "match_all": {}
    },

    // 分页
    "from": 0,
    "size": 10
}
```

### 6.5.9 批量删除

> ```mysql
> DELETE FROM `table` WHERE created_at > 0;
> ```

```json
> POST /<index_name>/_delete_by_query

{
    "query": {
        "range": {
            "created_at": {
                "gt": 0
            }
        }
    }
}
```

### 6.5.10 高亮

```json
// Request
{
    "query": {
        "match": {
            "content": "江河海"
        }
    },
    "highlight": {
        // 默认 tags 为：<em>*</em>
        "pre_tags": ["<font color=red>"], 
    	"post_tags": ["</font>"],
        "fields": {
            "content": {}
        }
    }
}

// Response
{
    "took": 6,
    "timed_out": false,
    "_shards": {
        "total": 1,
        "successful": 1,
        "skipped": 0,
        "failed": 0
    },
    "hits": {
        "total": {
            "value": 2,
            "relation": "eq"
        },
        "max_score": 2.7725885,
        "hits": [
            {
                "_index": "index_name",
                "_id": "1005",
                "_score": 2.7725885,
                "_source": {
                    "title": "《登鹳雀楼》",
                    "author": "唐·王之涣",
                    "content": "白日依山尽，黄河入海流。欲穷千里目，更上一层楼。"
                },
                "highlight": {
                    "content": [
                        "白日依山尽，黄<font color=red>河</font>入<font color=red>海</font>流。欲穷千里目，更上一层楼。"
                    ]
                }
            },
            {
                "_index": "index_name",
                "_id": "1004",
                "_score": 1.3862942,
                "_source": {
                    "title": "《江雪》",
                    "author": "唐·柳宗元",
                    "content": "千山鸟飞绝，万径人踪灭。孤舟蓑笠翁，独钓寒江雪。"
                },
                "highlight": {
                    "content": [
                        "孤舟蓑笠翁，独钓寒<font color=red>江</font>雪。"
                    ]
                }
            }
        ]
    }
}
```



### Query 语法汇总

```json
> [GET|POST] /<index_name>/_search

// Request
{
    "query": {
        // bool 查询
        "bool": {
            "must": [],
            "should": [],
            "must_not": [],
            "filter": []
        },
        
        // 各种查询方法（match, terms, ...）
        "query_method": {}
    },
    
    // 限制返回字段
    "_source": null,
    
    // 跟踪总数
    "track_total_hits": true,
    
    // 排序
    "sort": [],
    
    // 分页
	"from": 0,
	"size": 0,
    
    // 高亮
    "highlight": {},
    
    // 聚合
    "aggs": {},
    
    // 执行计划
    "explain": false
}
```





# 七、聚合查询

聚合将您的数据汇总为指标、统计数据或其他分析。

ES 将聚合分为三类：

- 根据字段值计算度量（如总和或平均值）的指标聚合。
- 桶聚合，根据字段值、范围或其他条件将文档分组到桶中，也称为桶。
- 从其他聚合而不是文档或字段中获取输入的管道聚合。

> 聚合语法：

```json
> POST /<index_name>/_search

// Request
{
    // 全局限制聚合的文档集
    "query": {
        // Query
    },

    // 仅返回聚合结果
    "size": 0,

    // 聚合语法（aggregations）
    "aggs": {
        // 限制聚合的文档集
        "filter": {
            // Query
        },

        // 第一个聚合：aggregation
        // 聚合结果名称
        "<group_name>": {
            // 聚合方法：terms, avg, histogram
            "<aggregation_name>": {
                // 每种聚合方法的参数不同
            }
        },

        // 第二个聚合：aggregation
        "<group_name_2>": {
            // 限制聚合的文档集
            "filter": {
                // Query
            },

            // 第二个聚合的子聚合
            "aggs": {
                // aggregations
            }
        }
    }
}
```

## 7.1 指标（metrics）聚合

metrics 系列聚合基于以某种方式从正在聚合的文档中提取的值来计算度量。这些值通常从文档的字段中提取（使用字段数据），但也可以使用脚本生成。

数字指标聚合是一种特殊类型的指标聚合，它输出数值。一些聚合输出单个数字指标（例如平均值），称为单值数字指标聚合，另一些聚合生成多个指标（例如统计数据），也称为多值数字指标聚集。当这些聚合充当某些 bucket 聚合的直接子聚合时，单值和多值数值指标聚合之间的区别起着重要作用（一些 bucket 聚合使您能够根据每个 bucket 中的数值指标对返回的 bucket 进行排序）。

### 7.1.1 单值指标集合

```json
// 语法
{
    "aggs": {
        // 聚合结果名称
        "group_name": {
            // 聚合方法：avg|min|max|sum|stats
            "aggregation_name": {
                // 从文档中的特定数字或直方图字段中提取数据
                "field": "<field_name>",
                
                "fotmat": "",
                
                // 参数定义如何处理缺少值的文档。
                // 默认情况下，它们将被忽略，但也可以将其视为有值。
                // 字段中没有值的文档将与值为 10 的文档归入同一个桶中。
                "missing": 10,
                
                // 从提供的脚本（运行时字段）中提取数据
                "script": {}
            }
        }
    }
}
```

#### (1) value_count

单值指标聚合，统计从聚合文档中提取的值的数量。

value_count 不会消除重复值，因此即使一个字段有重复值，每个值也会单独计数。

> ```mysql
> SELECT COUNT(score) AS `count` FROM `table`;
> ```

```json
// Request
{
    "aggs": {
        // 平均分
        "count": {
            "value_count": {
                // SELECT COUNT(score) FROM table;
                // 注意：当 score 缺失时，无法统计到该条数据（类似 MySQL 对 NULL 的处理）
                "field": "score",
                
                // 对缺失值赋予任意值时，即可统计到该数据（类似 MySQL 的 COUNT(*)）
                "missing": 0
            }
        }
    }
}

// Response
{
    // ...
    "aggregations": {
        "count": {
            "value": 2
        }
    }
}
```

#### (2) min

单值指标聚合，跟踪并返回从聚合文档中提取的数值中的最小值。

> ```mysql
> SELECT MIN(score) AS min_score FROM `table`;
> ```

```json
// Request
{
    "aggs": {
        // 最低分
        "min_score": {
            "min": {
                "field": "score"
            }
        }
    }
}

// Response
{
    // ...
    "aggregations": {
        "min_score": {
            "value": 50.0
        }
    }
}
```

#### (3) max

单值指标集合，跟踪并返回从聚合文档中提取的数值中的最大值。

> ```mysql
> SELECT MAX(score) AS max_score FROM `table`;
> ```

```json
// Request
{
    "aggs": {
        // 最高分
        "max_score": {
            "max": {
                "field": "score"
            }
        }
    }
}

// Response
{
    // ...
    "aggregations": {
        "max_score": {
            "value": 100.0
        }
    }
}
```

#### (4) avg

单值指标聚合，计算从聚合文档中提取的数值的平均值。

> ```mysql
> SELECT AVG(score) AS avg_score FROM `table`;
> ```

```json
// Request
{
    "aggs": {
        // 平均分
        "average_score": {
            "avg": {
                "field": "score"
            }
        }
    }
}

// Response
{
    // ...
    "aggregations": {
        "average_score": {
            "value": 75.0
        }
    }
}
```

#### (5) sum

单值指标聚合，该聚合对从聚合文档中提取的数值进行求和。

> ```mysql
> SELECT SUM(score) AS sum_score FROM `table`;
> ```

```json
// Request
{
    "aggs": {
        // 总分
        "sum_score": {
            "sum": {
                "field": "score"
            }
        }
    }
}

// Response
{
    // ...
    "aggregations": {
        "sum_score": {
            "value": 150.0
        }
    }
}
```

#### (6) stats

对上述单值指标（avg|max|min|sum）进行汇总的多值指标聚合，用于计算从聚合文档中提取的数值的统计数据。

```json
// Request
{
    "aggs": {
        // 分数统计
        "score_stats": {
            "stats": {
                "field": "score"
            }
        }
    }
}

// Response
{
    // ...
    "aggregations": {
        "score_stats": {
            "count": 2,
            "min": 50.0,
            "max": 100.0,
            "avg": 75.0,
            "sum": 150.0
        }
    }
}
```





## 7.2 桶（bucket）聚合

bucket 聚合不像 metrics 聚合那样计算字段上的指标，而是创建文档桶。每个 bucket 都与一个标准（取决于聚合类型）相关联，该标准决定了当前上下文中的文档是否“落入”其中。换句话说，bucket 有效地定义了文档集。除了 bucket 本身，bucket 聚合还计算并返回“落入”每个 bucket 的文档数量。

与 metrics 聚合不同，bucket 聚合可以容纳子聚合。这些子聚合将针对由其“父” bucket 聚合创建的桶进行聚合。

每个不同的 bucket 聚合器都有不同的 “bucketing” 策略。有些定义了单个桶，有些定义了固定数量的多个桶，还有一些在聚合过程中动态创建桶。

### filter

单桶聚合，将文档集缩小到与查询匹配的文档集。

```json
// Request
{
    "aggs": {
        // 所有商品平均价格
        "avg_price": { 
            "avg": { "field": "price" } 
        },
        "t_shirts": {
            // t-shirt 类商品的平均价格
            "filter": { 
                "term": { "type": "t-shirt" } 
            },
            "aggs": {
                "avg_price": { 
                    "avg": { "field": "price" } 
                }
            }
        }
    }
}

// Response
{
    "aggregations": {
        "avg_price": { 
            "value": 140.71428571428572 
        },
        "t_shirts": {
            "doc_count": 3,
            "avg_price": { "value": 128.33333333333334 }
        }
    }
}
```



### filters

```json
// Request
{
    "size": 0,
    "aggs" : {
        "messages" : {
            "filters" : {
                "filters" : {
                    "errors" :   { "match" : { "body" : "error"   }},
                    "warnings" : { "match" : { "body" : "warning" }}
                },

                // 匿名过滤器
                // "filters" : [
                //    { "match" : { "body" : "error"   }},
                //    { "match" : { "body" : "warning" }}
                // ]
            }
        }
    }
}

// Response
{
    // ...
    "aggregations": {
        "messages": {
            "buckets": {
                "errors": {
                    "doc_count": 1
                },
                "warnings": {
                    "doc_count": 2
                }
            },

            // 匿名过滤器的桶
            // "buckets": [
            //     {
            //         "doc_count": 1
            //     },
            //     {
            //         "doc_count": 2
            //     }
            // ]
        }
    }
}
```



### terms

一种基于来源的多桶值聚合，其中桶是动态构建的——每个唯一值一个。

```json
{
    "aggs": {
        "<group_name>": {
            "terms": {
                // field 可以是 keyword, numeric, ip, boolean, binary
                "field": "<field>",
                
                // 可以筛选将为其创建 bucket 的值，使用基于正则表达式字符串或精确值数组。
                // include 子句可以使用 partition 表达式进行过滤。
                // 当两者都被定义时，exclude 具有优先级，这意味着先计算 include 再计算 exclude。
                // 因此，标签 water_sports 将不会被聚合。
                "include": ".*sport.*",
                "exclude": "water_.*",
                // 使用精确值筛选
                // "include": [ "mazda", "honda" ],
                // "exclude": [ "rover", "jensen" ],
                
                // 默认按文档 _count 降序排列术语。
                // 可以指定不同的排序顺序，但不建议这样做，因为可能会导致结果不准确。
                // 特别要避免使用：{ "_count": "asc" }
                "order": { "_key": "desc" },
                
                // 最小文档计数（默认：1）
                // 计数小于此配置的桶不返回
                "min_doc_count": 1,
                
                // 从全部术语列表中返回的桶数（默认：10）。
                "size": 10,
                
                // 如何处理缺失值的文档，默认被忽略。
                "missing": "N/A",
                
                // 显示每桶的文档计数错误
                "show_term_doc_count_error": false,
                
                "script": null
            }
        }
    }
}
```

> 示例：
>
> ```mysql
> SELECT base AS `key`, COUNT(*) AS doc_count FROM `table` GROUP BY base;
> ```

```json
// Request
{
    "size": 0,
    "aggs": {
        "base_terms": {
            "terms": {
                "field": "base"
            }
        }
    }
}

// Response
{
    // ...
    "aggregations": {
        "base_terms": {
            // 文档计数错误上限
            "doc_count_error_upper_bound": 0,
            // 其它文档计数总数，
            // 当用 size 控制返回桶数量时，其它桶没法显示的会汇总到这里
            "sum_other_doc_count": 0,
            "buckets": [
                {
                    "key": "16",
                    "doc_count": 67368400
                },
                {
                    "key": "1",
                    "doc_count": 39661
                },
                {
                    "key": "128",
                    "doc_count": 8751
                },
                {
                    "key": "2",
                    "doc_count": 6172
                },
                {
                    "key": "4",
                    "doc_count": 3882
                },
                {
                    "key": "8",
                    "doc_count": 44
                }
            ]
        }
    }
}
```



### rare_terms

一种基于来源的多桶值聚合，可以找到“罕见”的术语——位于分布长尾且不频繁的术语。从概念上讲，这就像一个按 _count 升序排序的 terms 聚合。

> 语法：

```json
{
    "aggs": {
        "<group_name>": {
            "rare_terms": {
                // field 可以是 keyword, numeric, ip, boolean, binary
                "field": "<field>",
                
                // 可以筛选将为其创建 bucket 的值，使用基于正则表达式字符串或精确值数组。
                // include 子句可以使用 partition 表达式进行过滤。
                // 当两者都被定义时，exclude 具有优先级，这意味着先计算 include 再计算 exclude。
                // 因此，标签 water_sports 将不会被聚合。
                "include": ".*sport.*",
                "exclude": "water_.*",
                // 使用精确值筛选
                // "include": [ "mazda", "honda" ],
                // "exclude": [ "rover", "jensen" ],
                
                // 最大文档计数（默认：1，最大为：100）
                // 控制术语可以具有的文档计数的上限
                // 计数大于此配置的桶不返回
                "max_doc_count": 1,
                
                // 如何处理缺失值的文档，默认被忽略。
                "missing": "N/A",
                
                // 精度（默认：0.001，最小：0.00001）
                "precision": 0.001
            }
        }
    }
}
```

> 示例：

```json
// Request
{
    "size": 0,
    "aggs": {
        "base_terms": {
            "rare_terms": {
                "field": "base",
                "max_doc_count": 100
            }
        }
    }
}

// Response
{
    // ...
    "aggregations": {
        "base_terms": {
            // 只能返回文档计数：44 的桶，其余桶的计数都已超过 100.
            "buckets": [
                {
                    "key": "8",
                    "doc_count": 44
                }
            ]
        }
    }
}
```



### multi_terms

一种基于来源的多桶值聚合，其中桶是动态构建的——每组值唯一。 multi_terms 聚合与 terms 聚合非常相似，但在大多数情况下，它会比 terms 聚合慢，并且会消耗更多的内存。因此，如果经常使用同一组字段，则将此字段的组合键作为单独的字段进行索引，并在该字段上使用 terms 聚合会更有效。

当您需要按多个文档或复合键上的度量聚合进行排序并获得前 N 个结果时，multi_terms 聚合是最有用的。如果不需要排序，并且所有值都是预期的，使用嵌套 terms 聚合或 composite 聚合检索将是一种更快、更节省内存的解决方案。

> 语法：

```json
{
    "aggs": {
        "<group_name>": {
            "multi_terms": {
                // 从全部术语列表中返回的桶数（默认：10）。
                "size": 10,

                // 默认按文档 _count 降序排列术语。
                // 可以指定不同的排序顺序，但不建议这样做，因为可能会导致结果不准确。
                // 特别要避免使用：{ "_count": "asc" }
                "order": { "_key": "desc" },

                // 最小文档计数（默认：1）
                // 计数小于此配置的桶不返回
                "min_doc_count": 1,

                // 显示每桶的文档计数错误
                "show_term_doc_count_error": false,

                // 定义术语集（至少 2 个）
                "terms": [
                    {
                        // field 可以是 keyword, numeric, ip, boolean, binary
                        "field": "<field>",

                        // 如何处理缺失值的文档，默认被忽略。
                        "missing": "N/A"
                    },
                    {
                        "field": "<field>",
                        "missing": "N/A"
                    }
                ]
            }
        }
    }
}
```

> 示例：
>
> ```mysql
> SELECT 
>   CONCAT(base, '|', length) AS `key`, COUNT(*) AS doc_count 
> FROM `table` 
> WHERE base = 16 
> GROUP BY base, length;
> ```

```json
// Request
{
    "query": {
        "term": {
            "base": 16
        }
    },
    "size": 0,
    "aggs": {
        "base_length_terms": {

            "multi_terms": {
                "terms": [
                    {
                        "field": "base"
                    },
                    {
                        "field": "length"
                    }
                ]
            }
        }
    }
}

// Response
{
    "took": 22965,      // 22.97 s
    "timed_out": false,
    "_shards": {
        "total": 1,
        "successful": 1,
        "skipped": 0,
        "failed": 0
    },
    "hits": {
        "total": {
            "value": 10000,
            "relation": "gte"
        },
        "max_score": null,
        "hits": []
    },
    "aggregations": {
        "base_length_terms": {
            "doc_count_error_upper_bound": 0,
            "sum_other_doc_count": 0,
            "buckets": [
                {
                    "key": [
                        "16",
                        6
                    ],
                    "key_as_string": "16|6",
                    "doc_count": 64000000
                },
                {
                    "key": [
                        "16",
                        5
                    ],
                    "key_as_string": "16|5",
                    "doc_count": 3200000
                },
                {
                    "key": [
                        "16",
                        4
                    ],
                    "key_as_string": "16|4",
                    "doc_count": 160000
                },
                {
                    "key": [
                        "16",
                        3
                    ],
                    "key_as_string": "16|3",
                    "doc_count": 8000
                },
                {
                    "key": [
                        "16",
                        2
                    ],
                    "key_as_string": "16|2",
                    "doc_count": 400
                }
            ]
        }
    }
}
```

> 【推荐】用嵌套 terms 查询实现（性能更好）。

```json
// Request
{
    "size": 0,
    "aggs": {
        "base_terms": {
            "terms": {
                "field": "base"
            }
        },
        "base16_length_terms": {
            "filter": {
                "term": {
                    "base": 16
                }
            },
            "aggs": {
                "length_terms": {
                    "terms": {
                        "field": "length"
                    }
                }
            }
        }
    }
}

// Response
{
    "took": 4244,       // 4.24 s
    "timed_out": false,
    "_shards": {
        "total": 1,
        "successful": 1,
        "skipped": 0,
        "failed": 0
    },
    "hits": {
        "total": {
            "value": 10000,
            "relation": "gte"
        },
        "max_score": null,
        "hits": []
    },
    "aggregations": {
        "base_terms": {
            "doc_count_error_upper_bound": 0,
            "sum_other_doc_count": 0,
            "buckets": [
                {
                    "key": "16",
                    "doc_count": 67368400
                },
                {
                    "key": "1",
                    "doc_count": 39661
                },
                {
                    "key": "128",
                    "doc_count": 8751
                },
                {
                    "key": "2",
                    "doc_count": 6172
                },
                {
                    "key": "4",
                    "doc_count": 3882
                },
                {
                    "key": "8",
                    "doc_count": 44
                }
            ]
        },
        "base16_length_terms": {
            "doc_count": 67368400,
            "length_terms": {
                "doc_count_error_upper_bound": 0,
                "sum_other_doc_count": 0,
                "buckets": [
                    {
                        "key": 6,
                        "doc_count": 64000000
                    },
                    {
                        "key": 5,
                        "doc_count": 3200000
                    },
                    {
                        "key": 4,
                        "doc_count": 160000
                    },
                    {
                        "key": 3,
                        "doc_count": 8000
                    },
                    {
                        "key": 2,
                        "doc_count": 400
                    }
                ]
            }
        }
    }
}
```



### Composite





### histogram

一种基于来源的多桶值聚合，可以应用于从文档中提取的数值或数值范围值。它在值上动态构建固定大小（间隔）的桶。

```sh
# 落桶规则计算
bucket_key = Math.floor((value - offset) / interval) * interval + offset

# 假设
interval= 5
offset= 0

value= 32
bucket_key= Math.floor(32/5) * 5 = 30
# 故 {value: 32} 会落入与 {key: 30} 关联的桶中：[30, 34)
```

> 语法

```json
{
    "aggs": {
        "<group_name>": {
            "histogram": {
                "field": "<field>",
                
                // 默认情况下，bucket 的初始边界从 0 开始，以 interval 为步长生成桶：
                //  [0, 5) 
                //  [5, 10) 
                //  [10, 15)
                //   ...
                "interval": 5,
                
                // 可以使用 offset 移动 bucket 的初始边界
                // 当 offset 为 2 时，则生成桶：[2, 7) [7, 12) ...
                "offset": 0,
                
                // 默认情况下，bucket 作为有序数组返回。
                // 将 keyed 设置为 true 时，
                // 将为每个 bucket 关联一个唯一的字符串 key，并以哈希的形式返回范围桶。
                "keyed": false,
                
                // 默认情况下，返回的 bucket 按其键升序，
                // 可以使用顺序设置控制顺序行为，同 terms 聚合。
                "order": {},
                
                // 如何处理缺失值的文档，默认被忽略。
                "missing": "N/A",
                
                "script": null
            }
        }
    }
}
```

> 示例

```json
// Request
{
    "size": 0,
    "aggs": {
        "histogram": {
            "histogram": {
                "field": "length",
                "interval": 500
            }
        }
    }
}

// Response
{
    // ...
    "aggregations": {
        "histogram": {
            "buckets": [
                {
                    "key": 0.0,
                    "doc_count": 67426354
                },
                {
                    "key": 500.0,
                    "doc_count": 498
                },
                {
                    "key": 1000.0,
                    "doc_count": 33
                },
                {
                    "key": 1500.0,
                    "doc_count": 16
                },
                {
                    "key": 2000.0,
                    "doc_count": 3
                },
                {
                    "key": 2500.0,
                    "doc_count": 4
                },
                {
                    "key": 3000.0,
                    "doc_count": 2
                }
            ]
        }
    }
}
```



### range

一种基于来源的多桶值聚合，使用户能够定义一组范围，每个范围代表一个桶。在聚合过程中，将根据每个 bucket 范围和相关/匹配文档的 “bucket” 检查从每个文档中提取的值。

请注意，此聚合的范围是左闭右开区间：[from, to)。

> 语法：

```json
{
    "aggs": {
        "<group_name>": {
            "range": {
                "field": "<field>",

                // 默认情况下，bucket 作为有序数组返回。
                // 将 keyed 设置为 true 时，
                // 将为每个 bucket 关联一个唯一的字符串 key，并以哈希的形式返回范围桶。
                "keyed": false,

                // 如何处理缺失值的文档，默认被忽略。
                "missing": "N/A",

                // 定义范围
                "ranges": [
                    {
                        // 自定义每个范围的 key
                        // 默认为：from-to（"*-100.0", "100.0-200.0"）
                        "key": "",
                        "from": 0.0,
                        "to": 0.0
                    }
                ],

                "script": null
            }
        }
    }
}
```

> 示例：

```json
// Request
{
    "size": 0,
    "aggs": {
        "length_ranges": {
            "range": {
                "field": "length",
                "ranges": [
                    // 注：聚合范围为左闭右开区间：[from, to)
                    { "key": "NoSeq", "to": 1 },  
                    { "key": "2-5", "from": 2, "to": 6 },
                    { "key": "6-10", "from": 6, "to": 11 },
                    { "key": "11-15", "from": 11, "to": 16 },
                    { "key": "16-20", "from": 16, "to": 21 },
                    { "key": "21-25", "from": 21, "to": 26 },
                    { "key": "26-30", "from": 26, "to": 31 },
                    { "key": "31-35", "from": 31, "to": 36 },
                    { "key": "36-40", "from": 36, "to": 41 },
                    { "key": "41-45", "from": 41, "to": 46 },
                    { "key": "46-50", "from": 46, "to": 51 },
                    { "key": ">50", "from": 51 }
                ]
            }
        }
    }
}

// Response
{
    // ...
    "aggregations": {
        "length_ranges": {
            "buckets": [
                {
                    "key": "NoSeq",
                    "to": 1.0,
                    "doc_count": 43985
                },
                {
                    "key": "2-5",
                    "from": 2.0,
                    "to": 6.0,
                    "doc_count": 3370041
                },
                {
                    "key": "6-10",
                    "from": 6.0,
                    "to": 11.0,
                    "doc_count": 64002376
                },
                {
                    "key": "11-15",
                    "from": 11.0,
                    "to": 16.0,
                    "doc_count": 1312
                },
                {
                    "key": "16-20",
                    "from": 16.0,
                    "to": 21.0,
                    "doc_count": 1153
                },
                {
                    "key": "21-25",
                    "from": 21.0,
                    "to": 26.0,
                    "doc_count": 432
                },
                {
                    "key": "26-30",
                    "from": 26.0,
                    "to": 31.0,
                    "doc_count": 1756
                },
                {
                    "key": "31-35",
                    "from": 31.0,
                    "to": 36.0,
                    "doc_count": 1140
                },
                {
                    "key": "36-40",
                    "from": 36.0,
                    "to": 41.0,
                    "doc_count": 455
                },
                {
                    "key": "41-45",
                    "from": 41.0,
                    "to": 46.0,
                    "doc_count": 267
                },
                {
                    "key": "46-50",
                    "from": 46.0,
                    "to": 51.0,
                    "doc_count": 372
                },
                {
                    "key": ">50",
                    "from": 51.0,
                    "doc_count": 3618
                }
            ]
        }
    }
}
```





## 7.3 管道（pipeline）聚合

> SELECT COUNT(*) AS group_name FROM table GROUP BY id;

```json
> POST /<index_name>/_search

// Request
{
    // 按 ID 分组
    "aggs":{
        "group_name": {
            "terms": {
                "field": "id"
            }
        }
    },
    
    // aggs 时，不需要原始数据
    "size": 0
}

// Response
{
    "took": 3,
    "timed_out": false,
    "_shards": {
        "total": 1,
        "successful": 1,
        "skipped": 0,
        "failed": 0
    },
    "hits": {
        "total": {
            "value": 5,
            "relation": "eq"
        },
        "max_score": null,
        "hits": []
    },
    "aggregations": {
        "group_name": {
            "doc_count_error_upper_bound": 0,
            "sum_other_doc_count": 0,
            "buckets": [
                {
                    "key": 1001,
                    "doc_count": 1
                },
                {
                    "key": 1002,
                    "doc_count": 1
                },
                {
                    "key": 1003,
                    "doc_count": 1
                },
                {
                    "key": 1004,
                    "doc_count": 1
                },
                {
                    "key": 1005,
                    "doc_count": 1
                }
            ]
        }
    }
}
```







# 八、SQL 语法

```sql
# format 格式化结果样式
POST _sql?format=txt
{
    "query": """
    SELECT * FROM "index_name"
    """
}

# 将 SQL 翻译成 DSL
POST _sql/translate
{
    "query": """
    SELECT * FROM "index_name"
    """
}
```



## 2.6 文档评分机制

```
# 查看评分计划
GET <index_name>/_search?explain=true
```

```
score = boost * idf * tf 
      = 权重 * 逆文档评率 * 词频
```

ES 的评分机制是一个基于词频（TF）和逆文档频率（IDF）的公式，简称 TF-IDF 公式。

- TF：Term Frequency，搜索文本中的各个词条（term）在查询文本中出现的次数，出现次数越多，就越相关，得分越高。
- IDF：Inverse Document Frequency：搜索文本中的各个词条（term）在整个索引的所有文档中出现的次数，出现的次数越多，说明越不相关，得分越低。

> 权重

```
{
    "match": {
        "title": {
            "query": "value",
            // 查询权重，匹配到 value 的文档权重会变高
            "boost": 2
        }
    }
}
```



## 2.7 慢查询
