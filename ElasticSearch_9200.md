## 一、概述

### 1.1、Install & Deploy

#### 1、ElasticSearch

官网：https://www.elastic.co/cn

```sh
# 启动一个默认容器，从里面复制出配置文件（单节点）
$ docker run -d --rm \
  --name ES0 \
  -m 1G \
  -e "discovery.type=single-node" \
  docker.elastic.co/elasticsearch/elasticsearch:8.15.0

# cp from docker
$ docker cp ES0:/usr/share/elasticsearch/config/elasticsearch.yml .

$ docker stop ES0

# 配置文件详情见下
$ vim config/elasticsearch.yml

`
cluster.name: "docker-cluster"
network.host: 0.0.0.0

# 安全功能，默认启动，这里需禁用（否则 elasticsearch-head 无法连接）
# Enable security features
xpack.security.enabled: false
`

# 运行单节点 ES
$ docker run --name ES -d \
  -m 4G \
  -p 9200:9200 \
  -v /data/docker-run/es/config/elasticsearch.yml:/usr/share/elasticsearch/config/elasticsearch.yml \
  -v elasticsearch-data:/usr/share/elasticsearch/data \
  -e "discovery.type=single-node" \
  docker.elastic.co/elasticsearch/elasticsearch:8.15.0
  
# 访问：`http://localhost:9200`
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
  -v /data/docker-run/es/config/elasticsearch.yml:/usr/share/elasticsearch/config/elasticsearch.yml \
  -v elasticsearch-data:/usr/share/elasticsearch/data \
  -e ELASTIC_PASSWORD=admin \
  -e "discovery.type=single-node" \
  docker.elastic.co/elasticsearch/elasticsearch:8.15.0
```

> elasticsearch.yml

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

#### 2、Elasticsearch - head 

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

#### 3、Kibana 

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




### 1.2、特点

- 快速
- 分词
- 高亮

#### 1、ES 与 SQL 对应关系

|               | 数据库   | 结构    | 表（集合） | 文档（行）       | 字段（列） |
| ------------- | -------- | ------- | ---------- | ---------------- | ---------- |
| ElasticSearch | node     | mapping | index      | document（Json） | field      |
| MongoDB       | database |         | collection | document（Bson） | field      |
| MySQL         | database | scheme  | table      | row              | column     |

注：7.* 弃用 type，用内置字段 _doc 代替。原因：同名字段的冲突。

（2）映射（mapping）

mapping 是处理数据的方式和规则方面做一些限制，如：某个字段发数据类型、默认值、分析器、是否被索引等。这些都是映射里面可以设置的，其它处理 ES 里面数据的一些使用规则设置也叫映射，按最优规则处理数据对性能提高很大，因此才需要建立映射，并且需要思考如何建立映射才能对性能更好。

（3）分片（shards）

（4）集群健康值

- green：主分片和副本都正常
- yellow：主分片正常，副本未分配
- red：不可用。

（5）倒排索引

词条：索引中最小存储和查询单元

词典：词条的集合，B+Tree、Hash

倒排表：

#### 2、核心概念

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

- **节点**：存放分片和副本。

- **分片**：存放数据，分片存在节点上。

- **副本**：副本数量应等于分片*副本数。

- **健康值**：

  - 绿色：能拿到全部分片数据；
  - 黄色：部分分片损坏能从副本拿到数据；
  - 红色：分片和副本都损坏，无法拿到完整数据。

  

### 1.3、数据类型

```
# 整型
byte      1 Byte
short     2 Byte
integer   4 Byte
long      8 Byte

# 浮点型
float
double
half_float
scaled_float

# 字符型
keyword
text

# 布尔型
boolean

# 二进制
binary

# 日期
date

# 对象类型（json）
object

# 嵌套类型
nested

# 地理
geo_point
geo_shape
```

#### 1、keyword

keyword 类型，不分词存储，整个关键词建立索引，只支持精确查询，支持聚合；

#### 2、text

text 类型，分词存储，按分词建立倒排索引，支持模糊、精确查询，不支持排序和聚合；可以通过子字段 keyword 排序。

```json
{
    "index_name": {
        "mappings": {
            "properties": {
                // text 类型字段用于全文搜索，不支持排序
                // 定义一个 keyword 子字段用于排序
                // 这样既可以用于全文搜索，也可以通过 title.keyword 进行排序。
                "title": {
                    "type": "text",
                    "fields": {
                        "keyword": {
                            "type": "keyword",
                            "ignore_above": 256
                        }
                    }
                }
            }
        }
    }
}
```




### 1.4 优化

#### 1、硬件选择

- 使用 SSD。固态硬盘比机械硬盘快。
- 使用 RAID 0。条带化 RAID 会提高磁盘 I/O，代价就是当一块磁盘故障时整个磁盘都故障。不要使用镜像或者奇偶校验 RAID 因为副本已经提供了这个功能。
- 使用多块硬盘，允许 ES 通过多个 path.data 目录配置把数据条带化分配到它们上面。
- 不要使用远程挂载的存储，比如 NFS 或 SMB/CIFS。这个引入的延迟会影响性能。

#### 2、分片&副本策略

分片和副本的设计为 ES 提供了分布式和故障转移的特性。但并不意味着分片和副本是可以无限分配的。而且索引的分片完成分配后由于索引的路由机制，我们不能再次修改分片数量。

分片&副本过多的代价：

- 一个分片的底层是一个 Lucene 索引，会消耗一定的文件句柄、内存、以及 CPU 运转。
- 每一个搜索请求都需要命中索引中的一个分片，如果多个分片都在同一个节点上，会竞争使用相同的资源。
- 计算相关度的词项统计信息是基于分片的。分片过多时，每个分片都只有很少的数据会导致相关度变低。

分片策略：

- 控制每个分片占用的硬盘容量不超过 ES 的最大 JVM 的堆空间设置（32G）。例如索引的总容量在 500G 左右时，分片数量为 500/32 ≈ 16 个。
- 考虑节点数量，一般一个节点有时候就是一台物理机，如果分片数过多，大大超过了节点数，很可能会导致一个节点上存在多个分片，一旦节点故障，即使保持了一个以上的副本，同样有可能会导致数据丢失。所以，一般设置分片数不超过节点数的 3 倍。
- 主节点，副本和节点最大数之间数量关系：节点数 <= 主分片数*(副本数+1)



## 二、ES|QL

### 2.1 索引操作

#### （1）[PUT] 创建索引

```json
> PUT http://localhost:9200/<index_name>


// Response
{
    "acknowledged": true,
    "shards_acknowledged": true,
    "index": "index_name"
}
```

#### （2）[GET] 查询索引

```json
// 根据 <index_name> 查询单条索引详情
> GET http://localhost:9200/<index_name>


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


// 索引列表
> GET http://localhost:9200/_cat/indices?v


// Response
health status index      uuid                   pri rep docs.count docs.deleted store.size pri.store.size dataset.size
yellow open   index_name xO8L2TSARaii_gkb0wQiFw   1   1          0            0             249b           249b         249b
```

#### （3）[PUT] 修改索引

```json
// 修改 settings 属性
> PUT http://localhost:9200/<index_name>/_settings

// Request
{
    "number_of_shards" :   1,  // 分片数
    "number_of_replicas" : 0   // 副本数
}

// Response
{
    "acknowledged": true
}

// 再次查看结果
> GET http://localhost:9200/<index_name>

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

#### （4）[DELETE] 删除索引

```json
> DELETE http://localhost:9200/<index_name>

// Response
{
    "acknowledged": true
}
```

#### （5）索引别名 alias

```json
// [HEAD] 检查别名是否存在
> HEAD http://localhost:9200/<index_name>/_alias/<alias_n>

Status Code:
  200  所有指定的别名都存在。
  404  一个或多个指定的别名不存在。


// [PUT/POST] 创建或更新别名
> PUT http://localhost:9200/<index_name>/_alias/<alias_n>

// Response
{
    "acknowledged": true,
    "errors": false
}


// [GET] 获取别名（不指定 alias_n 时，默认获取全部别名，等价于 _all）
> GET http://localhost:9200/<index_name>/_alias/<alias_n>

// Response
{
    "index_name": {
        "aliases": {
            "alias_n": {}
        }
    }
}


// [DELETE] 删除别名
> DELETE http://localhost:9200/<index_name>/_alias/<alias_n>

// Response
{
    "acknowledged": true,
    "errors": false
}
```

#### （6）定义表结构（映射）mapping

```json
// 修改 mappings 属性
> PUT http://localhost:9200/<index_name>/_mappings

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

#### （7）索引模板

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



### 2.2 文档操作（_doc）

#### （1）[POST] 创建单个文档

```json
// 创建文档，使用默认生成的主键 _id
> POST http://localhost:9200/<index_name>/_doc

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

#### （2）[PUT/POST] 创建或更新单个文档

```json
// 创建或更新文档，指定主键 _id
// 当 ID 不存在时，created
// 当 ID 存在时，updated
> POST http://localhost:9200/<index_name>/_doc/1002

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

#### （3）[GET] 获取单个文档

```json
> GET http://localhost:9200/<index_name>/_doc/1001

// Response
{
    "_index": "index_name",
    "_id": "1001",
    "found": false
}

> GET http://localhost:9200/<index_name>/_doc/1002

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

#### （4）[POST] 修改文档字段

```json
// 注意：这里用 _doc 操作的话，文档内容会被覆盖为以下内容，而不是仅修改字段
> POST http://localhost:9200/<index_name>/_update/1002

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

#### （5）[DELETE] 删除单个文档

```json
> DELETE http://localhost:9200/<index_name>/_doc/1002

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

#### （6）[POST] 批量操作文档

> 在 Kibana 的开发者工具中执行：
>
> [DevTool](http://localhost:5601/app/dev_tools#/console)

```json
PUT /index_name/_bulk
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



### 2.3 搜索（_search）

#### 1、全量查询

##### （1）无查询条件

```json
> GET http://127.0.0.1:9200/<index_name>/_search

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

- `took` – 查询花费时长（毫秒）
- `timed_out` – 请求是否超时
- `_shards` – 搜索了多少分片，成功、失败、跳过了多个分片（明细）
- `hits.total.value` - 找到的文档总数
- `hits.max_score` – 最相关的文档分数
- `hits._score` - 文档的相关性算分 (match_all 没有算分)
- `hits.hits` - 响应的数据，默认包含前10条，可用 from+size 限制
- `hits.hits._source` - 文档提交时的原始 JSON 对象

- `hits.hits.sort` - 文档排序方式 （如没有则按相关性分数排序）

##### （2）query 查询条件

```json
// 指定键值对查询
> GET http://127.0.0.1:9200/<index_name>/_search?q=title:春晓

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
            "value": 1,
            "relation": "eq"
        },
        "max_score": 3.2750041,
        "hits": [
            {
                "_index": "index_name",
                "_id": "1002",
                "_score": 3.2750041,
                "_source": {
                    "title": "《春晓》",
                    "author": "唐·孟浩然",
                    "content": "春眠不觉晓，处处闻啼鸟。夜来风雨声，花落知多少。"
                }
            }
        ]
    }
}


// 全文本搜索关键词（分词）
> GET http://127.0.0.1:9200/<index_name>/_search?q=江河海

// Response
{
    "took": 4,
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
                }
            },
            {
                "_index": "index_name",
                "_id": "1004",
                "_score": 1.6375021,
                "_source": {
                    "title": "《江雪》",
                    "author": "唐·柳宗元",
                    "content": "千山鸟飞绝，万径人踪灭。孤舟蓑笠翁，独钓寒江雪。"
                }
            }
        ]
    }
}
```

##### （3）request body 查询条件（推荐）

```json
> GET http://127.0.0.1:9200/<index_name>/_search

// Request
{
    "query": {
        "match_all": {}
    }
}

// Response
// 结果等价于 query 无查询条件
```

#### 2、query 语法（类比 SELECT）

```json
> [GET|POST] http://127.0.0.1:9200/<index_name>/_search

// Request
{
    "query": {
        // 无 WHERE 条件
        // SELECT * FROM <table> 
        "match_all": {},
        
        // match 只能有一个匹配字段（先分词再匹配）
        // WHERE field1 LIKE %values%
        "match": {
            "field": "values"
        },
        
        // multi_match 可以多个匹配字段，任意字段匹配即命中（先分词再匹配）
        // 当 fields 为空时，默认从所有字段匹配，等价于：URL?q=values
        // WHERE field1 LIKE %value2% OR field1 LIKE %value2%
        "multi_match": {
            "query": "values",
            "fields": [
                "field1",
                "field2"
            ]
        },
        
        // term & terms 词条匹配，不会分词，完整匹配
        "term": {
            "field": "value"
        },
        // 完整匹配多个选项
        "terms": {
            "field": ["value1", "value2"]
        },
        
        // wildcard 使用通配符匹配
        // * 匹配多个字符
		// ? 匹配一个字符
        "wildcard": {
            "field": "val_*"
        },
        
        // 范围查询
        // WHERE field1 BETWEEN (value1 AND value2)
        "range": {
            "field1": {
                "gte": "value1",
                "lte": "value2"
            }
        },
        
        // 组合条件 (AND|OR) 查询
        "bool": {
            // AND
            "must": [
                {
                    "match": {
                        "field1": "value1"
                    }
                },
                {
                    "match": {
                        "field2": "value2"
                    }
                }
            ],
            
            // OR
            "should": [
                {
                    "match": {
                        "field3": "value3"
                    }
                },
                {
                    "match": {
                        "field4": "value4"
                    }
                }
            ],
            
            // NOT
            "must_not": [
                {
                    "match": {
                        "field5": "value5"
                    }
                },
                {
                    "match": {
                        "field6": "value6"
                    }
                }
            ],
        }
    },
    
    // 选择返回字段
    // SELECT field1, field2, ... FROM <table>
    "_source": ["field1", "field2", "..."],
    
    "_source": {
        // 只返回 field1, field2 字段（可以使用通配符）
        "includes": ["field1", "field2"],
        // 排除 field1, field2 字段（可以使用通配符）
        "excludes": ["addr*", "*time"]
    },
    
    // 排序
    // ORDER BY field1 ASC, field2 DESC
    "sort": {
        "field1": "asc",
        "field2": "desc",
        // or
        "field2": {
            "order": "desc"
        },
    },
    
    // 分页
    // OFFSET 0 LIMIT 10
	"from": 0,
	"size": 10,
    
    // 聚合操作
    "aggs": {
        // 分组名称
        "group_name": {
            // terms 聚合操作，统计每组的条数
            "terms": {
                // 分组字段
                "field": "type"
            }
        },
        // 分组名称
        "group_name_avg": {
            // terms 聚合操作，统计每组的条数
            "avg": {
                // 分组字段
                "field": "type"
            }
        },
    }
    
    // aggs 时，不要原始数据
    // "size": 0
}
```

#### 3、获取指定字段（_source）

> 默认情况下，`_source` 为原始的 JSON 对象，有时显得冗余。

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

#### 4、范围搜索（range）

> 注意：range 中只能实现一个字段的范围查询。

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

#### 5、从一个字段匹配（match）

```json
{
    "query": {
        // match 会对查询字段分词，然后匹配文档。
        // 大小写不敏感
        "match": {
            // 匹配 `address` 中包含 “湖北” 的文档
            "address": "湖北"
        }
    }
}
```

#### 6、从多个字段匹配（mutil_match）

```json
{
	"query": {
        // 从任意字段匹配
    	"multi_match": {
      		"query": "query_key",
            // fields 为空时，从所有字段中匹配
      		"fields": [
                "artworketd_title",
                "artworketd_tagnames",
                "artworketd_serialnum"
      		]
    	}
  	}
}
```

####  7、匹配短语（match_phrase）

```json
{
    "query": {
        // 将检索关键词分词，分词结果必须在被检索字段中都包含，
        // 而且顺序必须相同，默认必须都是连续的
        "match_phrase": {
            // 未指定分词器，默认将中文分词成单个字
            // 无法匹配到结果：黄河入海流
        	"content": "黄河海流"
        }
    }
}
```

####  8、匹配短语前缀（match_phrase_prefix）

```json
{
    "query": {
        "match_phrase_prefix": {
            "artworketd_frontids": "s:159"
        }
    }
}

{
    "query": {
        "match_phrase_prefix": {
            "summary": {
                "query": "cli ro",
                "slop": 3,
                "max_expansions": 10
            }
        }
    }
}
```

#### 9、匹配词条（term & terms）

> 不执行分词器，大写字母无法匹配，适合keyword 、numeric、date 数据类型。

```json
{
    "query": {
        // term 匹配一个关键词
        // 由于 term 不执行分词器，而目标文档被分成了单汉字的词组，故无法匹配到目标
        "term": {
            "content": "黄河"
        }
    }
}

{
    "query": {
        // terms 匹配多个关键词
        "terms": {
            "content": ["黄", "河"]
        }
    }
}
```

####  10、通配符匹配（wildcard）

> `*` 代表多个
>
> `?` 代表一个

```json
{
	"query": {
    	"wildcard": {
            "name": "YCZ_?"
        }
	}
}
```

#### 11、模糊查询（fuzzy）

> 模糊查询可以在 match 和 multi_match 查询中使用以便解决拼写的错误
>
> fuzzy 与 term 查询的模糊等价
>
> 匹配包含，不执行分词器，大写字母无法匹配

```json
{
	"query": {
		"fuzzy": {
			"name": "YCZ"
		}
	}
}

{
    "query": {
        "multi_match" : {
            "query" : "ycz_",
            "fields": ["name"],
            "fuzziness": "AUTO"
        }
    }
}
```

#### 12、高亮显示

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



#### 13、bool 查询（组合查询）

> must			<=>    and 
>
> should		<=>    or
>
> must_not	<=>    not

```json
{
    "query": {
        "bool": {
            "should": [
                "match": {
                    "artworketd_title": "西装"
                },
                "match": {
                    "artworketd_title": "卫衣"
                }
            ],
            "must_not": [
                {
                    "match": {
                        "artworketd_tagnames": "卫衣"
                    }
                },
                {
                    "match": {
                        "artworketd_tagnames": "西装"
                    }
                }
            ]
        }
    }
}
```

#### 14、filter 过滤查询

>  filter 是不计算相关性（_score）的，同时可以 cache。因此，filter 速度要快于 query 。

```json
{
	"query": {
        "bool": {
            "filter": {
                // 这里使用 query 语法
            }
        }
    }
}
```



#### 12、跟踪总点击次数

```
"track_total_hits" : true ,
"track_total_hits" : 100 ,
```



### 2.4 聚合查询

```json
POST http://172.16.56.53:9200/poetry/_search
```

> Request

```json
{
    "aggs":{
        "group_name": {
            "terms": {
                "field": "id"
            }
        }
    },
    "size": 0
}
```

> Response

```json

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

### 2.5 分词

#### 1、分词器

```json
GET|POST http://172.16.56.53:9200/poetry/_analyze

{
    "text": "hello world"
    // "analyzer": "ik_max_word|ik_smart"
}

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
```

自定义词组

#### 2、IK 分词器

IK Analyzer：中文分词器 [下载](https://github.com/medcl/elasticsearch-analysis-ik/releases)

```json
// 定义字段时，指定分词器
"properties": {
	"content": {
		"type": "text",
		"analyzer": "ik_max_word",
		"search_analyzer": "ik_smart"
	}
}

// 分词参数：（一般分词时用细粒度，查询时用粗粒度）
// ik_smart：粗粒度分词
// ik_max_word：细粒度分词
```

### 2.6 文档评分机制

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

### 2.7 EQL

### 2.8 SQL 语法

```
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

