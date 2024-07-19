## 一、入门

### 1、概述

NoSQL 的特点：非关系型的、分布式的、开源的、水平可扩展的。

MongoDB 是一个高性能、开源、无模式的文档型数据库，是当前 NoSQL 数据库产品中最热门的一种。它在许多场景下可用于替代传统的关系型数据库或 key/value 存储方式，MongoDB 使用 C++开发。[官网](https://www.mongodb.com/)

MongoDB 是一个介于关系数据库和非关系数据库之间的产品，是非关系数据库当中功能最
丰富，最像关系数据库的。

MongoDB 最大的特点是他支持的查询语言非常强大，其语法有点类似于面向对象的查询语言，几乎可以实现类似关系数据库单表查询的绝大部分功能，而且还支持对数据建立索引。

它是一个面向集合的、模式自由的文档型数据库。

> 数据类型

MongoDB 支持的数据结构非常松散，是类似 JSON 的 BSON 格式，因此可以存储比较复杂的数据类型。

存储的数据是键/值对的集合，键是字符串，值可以是数据类型集合里的任意类型，包括数组和文档。

BSON 是一种二进制序列化格式，用于在 MongoDB 中存储文档和进行远程过程调用。

> 三高：

- 高并发（High Performance）：对数据库高并发读写的需求。
- 高效存储（Huge Storage）：对海量数据的高效率存储和访问的需求。
- 高可扩展 & 高可用（High Scalability & High Availability）：对数据库的高可扩展性和高可用性的需求。

> 特点：

- 面向集合的存储：适合存储对象及 JSON 形式的数据。
- 动态查询：MongoDB 支持丰富的查询表达式。查询指令使用 JSON 形式的标记，可轻易查询文档中内嵌的对象及数组。
- 完整的索引支持：包括文档内嵌对象及数组。MongoDB 的查询优化器会分析查询表达式，并生成一个高效的查询计划。
- 查询监视：MongoDB 包含一系列监视工具用于分析数据库操作的性能。
- 复制及自动故障转移：MongoDB 数据库支持服务器之间的数据复制，支持主-从模式及服务器之间的相互复制。复制的主要目标是提供冗余及自动故障转移。
- 高效的传统存储方式：支持二进制数据及大型对象（如照片或图片）。
- 自动分片以支持云级别的伸缩性：自动分片功能支持水平的数据库集群，可动态添加额外的机器。

> 适用场景：

- 网站数据：MongoDB 非常适合实时的插入，更新与查询，并具备网站实时数据存储所需的复制及高度伸缩性。
- 缓存：由于性能很高，MongoDB 也适合作为信息基础设施的缓存层。在系统重启之后，由 MongoDB 搭建的持久化缓存层可以避免下层的数据源过载。
- 大尺寸，低价值的数据：使用传统的关系型数据库存储一些数据时可能会比较昂贵，在此之前，很多时候程序员往往会选择传统的文件进行存储。
- 高伸缩性的场景：MongoDB 非常适合由数十或数百台服务器组成的数据库。MongoDB 的路线图中已经包含对 MapReduce 引擎的内置支持。
- 用于对象及 JSON 数据的存储：MongoDB 的 BSON 数据格式非常适合文档化格式的存储及查询。

### 2、[安装](https://www.mongodb.com/docs/manual/administration/install-community/)

- 下载：[MongoDB 社区服务器下载](https://www.mongodb.com/try/download/community)

- 解压到：D:\MongoDB-5.0
- 创建数据文件及日志文件的存放目录：
  - D:\MongoDB-5.0\data 
  - D:\MongoDB-5.0\log

- 配置环境变量

- 启动 MongoDB 服务：

  ```shell
  # 启动 MongoDB 服务
  > mongod --dbpath=D:\MongoDB-5.0\data
  
  # 配置文件方式启动
  > mongod -f /etc/mongodb.cfg
  
  # Daemon 方式启动
  > mongod -f /etc/mongodb.cfg --fork
  ```

- 将 MongoDB 作为 Windows 服务

  ```shell
  # 将 MongoDB 作为 Windows 服务随机启动
  > mongod --dbpath=D:\MongoDB-5.0\data --logpath=D:\MongoDB-5.0\log\mongod.log --install
  
  > net start mongodb
  ```

- 客户端连接验证

  ```shell
  # 连接到 MongoDB 服务（mongod.exe）
  > mongo
  MongoDB shell version v5.0.15
  connecting to: mongodb://127.0.0.1:27017/?compressors=disabled&gssapiServiceName=mongodb
  Implicit session: session { "id" : UUID("3d81ac46-77e3-4b41-95d6-9a9529fc6bd7") }
  MongoDB server version: 5.0.15
  ......
  
  # 新版中没有 mongo 
  # 需要额外安装 mongosh
  ```



> 配置参数说明

```
 dbpath:
数据文件存放路径，每个数据库会在其中创建一个子目录，用于防止同一个实例多次运行的 mongod.lock 也保存在此目录中。
 logpath
错误日志文件
 logappend
错误日志采用追加模式（默认是覆写模式）
 bind_ip
对外服务的绑定 ip，一般设置为空，及绑定在本机所有可用 ip 上，如有需要可以单独指定
 port
对外服务端口。Web 管理端口在这个 port 的基础上+1000
 fork
以后台 Daemon 形式运行服务
 journal
开启日志功能，通过保存操作日志来降低单机故障的恢复时间，在 1.8 版本后正式加入，取代在 1.7.5 版本中的 dur 参数。
 syncdelay
系统同步刷新磁盘的时间，单位为秒，默认是 60 秒。
 directoryperdb
每个 db 存放在单独的目录中，建议设置该参数。与 MySQL 的独立表空间类似
 maxConns
最大连接数
 repairpath
执行 repair 时的临时目录。在如果没有开启 journal，异常 down 机后重启，必须执行 repair 操作。
```



### 3、常用命令

```
# 查看帮助信息
> help
    db.help()                    help on db methods
    db.mycoll.help()             help on collection methods
    sh.help()                    sharding helpers
    rs.help()                    replica set helpers
    help admin                   administrative help
    help connect                 connecting to a db help
    help keys                    key shortcuts
    help misc                    misc things to know
    help mr                      mapreduce
    ...
    exit                         quit the mongo shell
    
# 停止 MongoDB 服务
> use admin
> db.shutdownServer()

# 查看实例运行状态
db.serverStatus()
```



### 4、SQL 到 MongoDB 映射表

#### （1）术语和概念


| SQL 术语/概念                                             | MongoDB 术语/概念                                            |
| :-------------------------------------------------------- | :----------------------------------------------------------- |
| 数据库（databases）                                       | 数据库（databases）                                          |
| 表（tables）                                              | 集合（collections）                                          |
| 行（rows）                                                | 文档（BSON document）                                        |
| 列（columns）                                             | 字段（fields）                                               |
| 索引（index）                                             | 索引（index）                                                |
| 表连接（table joins）                                     | 嵌套文档（$lookup, embedded documents）                      |
| 主键（Primary Key）<br>指定任何唯一的列或列组合作为主键。 | 主键（Primary Key）<br>在 MongoDB 中，主键自动设置为 `_id` 字段。 |
| 聚合（GROUP BY)                                           | 聚合管道（Aggregation Pipeline）                             |
| SELECT INTO NEW_TABLE                                     | `$out`                                                       |
| MERGE INTO TABLE                                          | `$merge`                                                     |
| 联合（UNION ALL）                                         | `$unionWith`                                                 |
| 事务（Transactions）                                      | 事务（transactions）                                         |

#### （2）可执行文件

|                 | MongoDB   | MySQL    | Oracle    | Informix    | DB2          |
| :-------------- | :-------- | :------- | :-------- | :---------- | :----------- |
| Database Server | `mongod`  | `mysqld` | `oracle`  | `IDS`       | `DB2 Server` |
| Database Client | `mongosh` | `mysql`  | `sqlplus` | `DB-Access` | `DB2 Client` |

注：`mongos`：分片路由，如果使用了 sharding 功能，则应用程序连接的是 mongos 而不是 mongod。

#### （3）聚合管道

| SQL Term, Functions, Concepts | MongoDB Aggregate Operators |
| ----------------------------- | --------------------------- |
| WHERE                         | $match                      |
| GROUP BY                      | $group                      |
| HAVING                        | $match                      |
| SELECT                        | $project                    |
| ORDER BY                      | $sort                       |
| LIMIT                         | $limit                      |
| SUM()                         | $sum                        |
| COUNT()                       | $sum <br>$sortrByCount      |
| JOIN                          | $lookup                     |
| SELECT INTO NEW_TABLE         | $out                        |
| MEREG INTO TABLE              | $merge                      |
| UNION ALL                     | $unionWith                  |



## 二、MongoDB DDL

### 1、数据库方法

```shell
# 查看帮助
> db.help()
DB methods:
    ...

# 显示所有数据库
> show dbs
admin   0.000GB
config  0.000GB
local   0.000GB

# 设置当前数据库（不存在时自动创建）
# 新创建的 DB 因为没有数据，所以 show dbs 时不会显示
> use <database>
switched to db <database>

# 查看数据库统计数据
> db.stats()
{
    "db" : "test",
    "collections" : 1,
    "views" : 0,
    "objects" : 7,
    "avgObjSize" : 118.14285714285714,
    "dataSize" : 827,
    "storageSize" : 36864,
    "indexes" : 1,
    "indexSize" : 36864,
    "totalSize" : 73728,
    "scaleFactor" : 1,
    "fsUsedSize" : 22371524608,
    "fsTotalSize" : 926641287168,
    "ok" : 1
}

# MongoDB 版本
> db.version()
5.0.15
```

### 2、集合方法

```shell
# 查看帮助
> db.mycoll.help()
DBCollection methods:
    ...

# 显示当前数据库中的集合
> show collections

# 创建集合
#  1 手动创建
> db.createCollection("collection")
#  2 插入数据时，若 collection 不存在，则会自动创建
> db.collection.insertOne( {"data": "test"} )

# 新增字段 
#  更新数据时，若字段不存在，则自动新增
> db.collection.updateMany(
	{ },
	{ $set: { create_at: new Date() } }
)
{ "acknowledged" : true, "matchedCount" : 1, "modifiedCount" : 1 }

# 删除字段
> db.collection.updateMany(
	{ },
	{ $unset: { create_at: "" } }
)
{ "acknowledged" : true, "matchedCount" : 1, "modifiedCount" : 1 }

# 删除集合
> db.collection.drop()
true
```



## 三、MongoDB DML（CRUD）

### 1、Insert

```shell
# ============
#  插入单个文档
# ============
db.collection.insertOne( obj, <optional params> )

 - 可选参数：w, wtimeout, j

# ============
#  插入多个文档
# ============
db.collection.insertMany( [objects], <optional params> )

 - 可选参数：w, wtimeout, j
```

插入行为：

- 创建 `collection`：当该集合不存在时，插入操作将创建该集合。
- `_id` 字段：在 MongoDB 中，存储在 Collection 中的每个 Document 都需要一个唯一的 `_id` 字段作为主键。如果插入文档是省略了 `_id` 字段，MongoDB 会自动生成一个 ObjectId。
- 原子性：MongoDB 中的所有写操作在单个文档级别上都是原子的。
- 写确认：可以指定从 MongoDB 请求的写入操作的确认级别。

### 2、Query

> 填充数据

```json
db.collection.insertMany([
    {
        "item": "circular",
        "status": 0,
        "colors": ["red", "green", "blue"],
        "position": {
            "x": 10.0,
            "y": 0.0,
            "radius": 5
        }
    },
    {
        "item": "square",
        "status": 1,
        "colors": ["green", "blue", "red"],
        "position": {
            "x": 0.0,
            "y": 0.0,
            "w": 10,
            "h": 10
        }
    },
    {
        "item": "oblong",
        "status": 2,
        "colors": ["pink", "green", "blue"],
        "position": {
            "x": 10.0,
            "y": 10.0,
            "w": 3,
            "h": 4
        }
    },
    {
        "item": "oblong",
        "status": 3,
        "colors": ["purple", "write", "blue"],
        "position": {
            "x": 0.0,
            "y": 10.0,
            "w": 6,
            "h": 8
        }
    }
])

// result
{
    "acknowledged" : true,
    "insertedIds" : [
        ObjectId("642b83dbb4a8e4ecd1d6c410"),
        ObjectId("642b83dbb4a8e4ecd1d6c411"),
        ObjectId("642b83dbb4a8e4ecd1d6c412"),
        ObjectId("642b83dbb4a8e4ecd1d6c413")
    ]
}
```

#### （1）[查询文档](https://www.mongodb.com/docs/v5.0/tutorial/query-documents/)

```shell
# find 语法
db.collection.find([query],[fields])

 - query   是一个可选的查询过滤器。
 - fields  是要返回的可选字段集。

# ===========================
#  SELECT * FROM collection
# ===========================
db.collection.find( {} )

# ============
#  相等条件查询
# ============

# ... WHERE item = "oblong"
db.collection.find( { item: "oblong" } )

# ==========
#  AND 查询
# ==========

# ... WHERE item = "oblong" AND status = 2
db.collection.find( { item: "oblong", status: 2 } )

# ==============
#  $or  OR 查询
# ==============

# ... WHERE item = "oblong" OR status = 0
db.collection.find( { $or: [ { item: "oblong" }, { status: 0 } ] } } )

# ===========================
#  $eq $ne $gt $lt $gte $lte
#           范围查询
# ===========================

# ... WHERE status = 1
db.collection.find( { status: { $eq: 1 } } )
# ... WHERE status <> 1
db.collection.find( { status: { $ne: 1 } } )
# ... WHERE status > 1
db.collection.find( { status: { $gt: 1 } } )
# ... WHERE status < 1
db.collection.find( { status: { $lt: 1 } } )
# ... WHERE status >= 1
db.collection.find( { status: { $gte: 1 } } )
# ... WHERE status <= 1
db.collection.find( { status: { $lte: 1 } } )

# ==================
#  $in   IN 查询
#  $nin  NOT IN 查询
# ==================

# 注：对同一字段进行相等性检查时，使用 $in 运算而不是 $or 运算
# ... WHERE status IN (1, 2, 3)
db.collection.find( { status: { $in: [ 1, 2, 3 ] } } )

# ===============
#  $mod  取模运算
# ===============
# 查询 status % 3 == 1 的数据
db.collection.find( { status: { $mod: [ 3, 1 ] } } )


# ===================
#  $regex  LIKE 查询
# ===================

# ... WHERE item LIKE "%cir%"
db.collection.find( { item: /cir/ } )
# ... WHERE item LIKE "cir%"
db.collection.find( { item: /^cir/ } )
# ... WHERE item LIKE "%cir"
db.collection.find( { item: /cir$/ } )
# or
db.collection.find( { item: { $regex: "cir" } } )
db.collection.find( { item: { $regex: "^cir" } } )
db.collection.find( { item: { $regex: "cir$" } } )

# ==================
#  AND + OR 混合查询
# ==================
# ... WHERE status > 0 AND ( item LIKE "c%" OR item LIKE "s%" ) 
db.collection.find( {
    status: { $gt: 0 },
    $or: [ { item: /^c/ }, { item: /^s/ } ]
} )

# ======================================================================== #
# ===================== db.collection.find().修饰语 ===================== #
# ======================================================================== #

# 查看帮助
db.mycoll.find().help()

# ==================================================
#                      COUNT
#  db.mycoll.count( query = {}, <optional params> ) 
#   - 可选参数：limit, skip, hint, maxTimeMS
#  db.mycoll.find(...).count()
# ==================================================

# SELECT COUNT(*) FROM <colletion>
db.collection.count()

# SELECT COUNT(name) FROM collection
db.collection.count( { name: { $exists: true } } )

# SELECT COUNT(*) FROM <colletion> WHERE status > 0
db.collection.count( { status: { $gt: 0 } } )

# ----- UNCLEAR -----
# db.mycoll.countDocuments( query = {}, <optional params> ) 
# - 可选参数：limit, skip, hint, maxTimeMS

# ==============================
#        LIMIT & OFFSET
#  db.mycoll.find(...).limit(n)
#  db.mycoll.find(...).skip(n)
# ==============================

# 限制条数
# SELECT * FROM collection LIMIT 1 
db.collection.find().limit(1)
# or
db.collection.findOne()

# 分页（page=2 & size=5）
# SELECT * FROM collection OFFSET 5 LIMIT 1 
db.collection.find().skip(5).limit(1)

# ===============================
#            ORDER BY
#  db.mycoll.find(...).sort(...)
# ===============================

# 升序
# SELECT * FROM collection ORDER BY id ASC
db.collection.find().sort( { _id: 1 } )

# 降序
# SELECT * FROM collection ORDER BY id DESC
db.collection.find().sort( { _id: -1 } )

# 多个排序条件
# SELECT * FROM collection ORDER BY status ASC, id DESC
db.collection.find().sort( { status: 1, _id: -1 } )

# ==========
#  DISTINCT
# ==========
db.mycoll.distinct( key, query, <optional params> )

# SELECT DISTINCT(status) FROM collection
> db.collection.distinct("status")
[ 0, 1, 2, 3 ]

# SELECT DISTINCT(status) FROM collection
> db.collection.aggregate( [ { $group : { _id : "$status" } } ] )
{ "_id" : 2 }
{ "_id" : 1 }
{ "_id" : 3 }
{ "_id" : 0 }

# =========
#  EXPLAIN 
# =========

# EXPLAIN SELECT * FROM collection
> db.collection.find().explain()
{
    "explainVersion" : "1",
    "queryPlanner" : {
        "namespace" : "test.collection",
        "indexFilterSet" : false,
        "parsedQuery" : {

        },
        "queryHash" : "8B3D4AB8",
        "planCacheKey" : "D542626C",
        "maxIndexedOrSolutionsReached" : false,
        "maxIndexedAndSolutionsReached" : false,
        "maxScansToExplodeReached" : false,
        "winningPlan" : {
            "stage" : "COLLSCAN",
            "direction" : "forward"
        },
        "rejectedPlans" : [ ]
    },
    "command" : {
        "find" : "collection",
        "filter" : {

        },
        "$db" : "test"
    },
    "serverInfo" : {
        "host" : "DESKTOP-JRJQPP2",
        "port" : 27017,
        "version" : "5.0.15",
        "gitVersion" : "935639beed3d0c19c2551c93854b831107c0b118"
    },
    "serverParameters" : {
        "internalQueryFacetBufferSizeBytes" : 104857600,
        "internalQueryFacetMaxOutputDocSizeBytes" : 104857600,
        "internalLookupStageIntermediateDocumentMaxSizeBytes" : 104857600,
        "internalDocumentSourceGroupMaxMemoryBytes" : 104857600,
        "internalQueryMaxBlockingSortMemoryUsageBytes" : 104857600,
        "internalQueryProhibitBlockingMergeOnMongoS" : 0,
        "internalQueryMaxAddToSetBytes" : 104857600,
        "internalDocumentSourceSetWindowFieldsMaxMemoryBytes" : 104857600
    },
    "ok" : 1
}
```

#### （2）[查询嵌套文档](https://www.mongodb.com/docs/v5.0/tutorial/query-embedded-documents/)

```shell
# =====================
#  匹配嵌套文档（相等匹配）
# =====================
# 注：整个嵌套文档的相等匹配包括字段顺序
db.collection.find({
	"position": {
		"x": 0.0,
		"y": 0.0,
		"w": 10,
		"h": 10
	}
})

# 顺序不同或缺少字段都无法匹配
db.collection.find({
	"position": {
		"y": 0.0,
		"x": 0.0,
		"w": 10,
		"h": 10
	}
})

# ====================
#  查询嵌套字段（点符号）
# ====================
# 在嵌套字段上指定相等匹配（使用点号查询时，字段和嵌套字段必须在引号内。）
db.collection.find( { "positoin.h": { $gte: 10 } } )
```

#### （3）[查询数组](https://www.mongodb.com/docs/v5.0/tutorial/query-arrays/)

```shell
# =====================
#  匹配整个数组（相等匹配）
# =====================
# 强制匹配所有元素和顺序
db.collection.find( { "colors": ["red", "green", "blue"] } )

# ===============
#  $all  包含匹配
# ===============
# 不考虑数组中的顺序或其他元素
db.collection.find( { "colors": { $all: ["green", "blue"] } } )

# ==============
#  查询元素的数组
# ==============
# 查询的数组字段中至少包含一个指定值
db.collection.find( { "colors": "red" } )

# 在数组元素上查询具有复合过滤条件的数组
#  一个元素 > 15 & 另一个元素 < 20
#  或单个元素同时满足两个条件
db.inventory.find( { dim_cm: { $gt: 15, $lt: 20 } } )

# $elemMatch  查询满足多个条件的数组元素
#  至少有一个数组元素同时满足所有条件
db.inventory.find( { dim_cm: { $elemMatch: { $gt: 22, $lt: 30 } } } )

# ====================
#  按数组索引位置查询元素
# ====================
db.collection.find( { "colors.0": "red" } )

# ========================
#  $size  按数组长度查询数组
# ========================
db.collection.find( { "colors": { $size: 3 } } )
```

> [查询嵌套文档数组](https://www.mongodb.com/docs/v5.0/tutorial/query-array-of-documents/)

#### （4）[指定查询字段](https://www.mongodb.com/docs/v5.0/tutorial/project-fields-from-query-results/)

```shell
# =======================
#  inclusion 包含指定字段
# =======================
#  默认情况下 `_id` 字段会在匹配文档中返回

# SELECT _id, item, status FROM collection
db.collection.find( { }, { item: 1, status: 1 } )

# 排除 `_id` 字段
db.collection.find( { }, { item: 1, status: 1, _id: 0 } )

# 返回嵌套文档中的特定字段
db.collection.find( { }, { item: 1, "position.x": 1, "position.y": 1 } )

# ==========================
#  $slice  返回数组中的特定元素
# ==========================
# 0：不返回元素，1：第一个元素，...，-1：最后一个元素
db.collection.find( { }, { "colors": { $slice: 0 } } )

# =======================
#  exclusion 排除指定字段
# =======================
db.collection.find( { }, { position: 0 } )

# 排除嵌套文档中的特定字段
db.collection.find( { }, { "positoin.x": 0, "positoin.y": 0 } )


# ==================================================
#  注意：除 `_id` 字段外，不能在投影文档中组合包含和排除语句。
# ==================================================
> db.collection.find( { }, { positoin: 0, colors: 1 } )
Error: error: {
        "ok" : 0,
        "errmsg" : "Cannot do inclusion on field colors in exclusion projection",
        "code" : 31253,
        "codeName" : "Location31253"
}

> db.collection.find( { }, { colors: 1, positoin: 0 } )
Error: error: {
        "ok" : 0,
        "errmsg" : "Cannot do exclusion on field positoin in inclusion projection",
        "code" : 31254,
        "codeName" : "Location31254"
}
```

#### （5）[查询 Null 字段或缺失字段](https://www.mongodb.com/docs/v5.0/tutorial/query-for-null-fields/)

```shell
# 插入数据
db.collection.insertOne( { "data": null } )

# ===========
#  相等过滤器
# ===========
# 匹配字段值为 null 或者不包含该字段的文档
db.collection.find( { "data": null } )

# ================
#  $type  类型检查
# ================
# 仅匹配字段值为 null 的文档（BSON 类型中的 Null 类型编号为 10）
db.collection.find( { "data": { $type: 10 } } )

# ==================
#  $exists  存在检查
# ==================
# true  返回包含指定字段的文档
# false 返回不包含指定字段的文档
db.collection.find( { "data": { $exists: true } } )
```

#### （6）[迭代游标](https://www.mongodb.com/docs/v5.0/tutorial/iterate-a-cursor/#manually-iterate-the-cursor)

```shell
# 将查询结果存入 cursor
> var cursor = db.collection.find()

# 迭代游标
> cursor

# 转为数组
> cursor.toArray()

# 输出指定索引
> cursor.toArray()[0]
```

### 3、Update

#### （1）更新文档

```shell
# ===================
#  更新第一个匹配的文档
# ===================
db.collection.updateOne( filter, <update object or pipeline>, <optional params> )

 - 可选参数：upsert, w, wtimeout, j, hint, let

# $set          更新字段值
# $currentDate  将 `lastModified` 字段的值更新为当前日期（字段不存在则自动创建）
db.collection.updateOne( 
    { item: "oblong" },
    {
        $set: { "positoin.x": 11, status: "M" },
        $currentDate: { lastModified: true }
    } 
)

# =================
#  更新所有匹配的文档
# =================
db.collection.updateMany( filter, <update object or pipeline>, <optional params> )

 - 可选参数：upsert, w, wtimeout, j, hint, let
 
db.collection.updateMany( 
    { status: { $gte: 0 } },
    {
        $set: { status: "M" },
        $currentDate: { lastModified: true }
    } 
)

# 删除指定字段
db.collection.updateMany({}, {
    $unset: {<field1>: 1, <field2>: 1, ...}
})


#  替换第一个匹配的文档（旧版）
db.collection.replaceOne( filter, replacement, <optional params> )

 - 可选参数：upsert, w, wtimeout, j
```

更新行为：

- 原子性：MongoDB 中的所有写操作在单个文档级别上都是原子的。
- `_id` 字段：一旦设置，就不能更新字段 `_id` 的值。
- 字段顺序：对于写操作，MongoDB 保留文档字段的顺序，但以下情况除外：
  - `_id` 字段始终是文档中的第一个字段；
  - 字段名称的更新可能会导致文档中的字段重新排序。
- 写确认：可以指定从 MongoDB 请求的写入操作的确认级别。

#### （2）[使用聚合管道进行更新](https://www.mongodb.com/docs/v5.0/tutorial/update-documents-with-aggregation-pipeline/)

#### （3）批量处理

```
// 将毫秒时间戳转换为 Date 对象
db.system_log.find().forEach(function(doc) {
    doc.visible_time = new Date(doc.created_at);
    db.system_log.save(doc);
});
```



### 4、Delete

```shell
# ===================
#  删除第一个匹配的文档
# ===================
db.mycoll.deleteOne( filter, <optional params> )

 - 可选参数：w, wtimeout, j

db.collection.deleteOne( { status: "M" } )

# =================
#  删除所有匹配的文档
# =================
db.mycoll.deleteMany( filter, <optional params> )

 - 可选参数：w, wtimeout, j
 
db.collection.deleteMany( { status: "M" } ) 


# 删除所有文档
db.collection.deleteMany( { } ) 


# 删除匹配的单个文档或所有文档（旧版）
db.collection.remove(query)
```

删除行为：

- 索引：删除操作不会删除索引，即使删除集合中的所有文档也是如此。
- 原子性：MongoDB 中的所有写操作在单个文档级别上都是原子的。
- 写确认：可以指定从 MongoDB 请求的写入操作的确认级别。



### 5、bulkWrite [批量写入操作](mongodb.com/docs/v5.0/core/bulk-write-operations/)

```
db.mycoll.bulkWrite( operations, <optional params> )
```

### 6、文本搜索

> 填充数据

```shell
db.collection.insertMany([
    {
        "rank": 1,
        "title": "Python",
        "content": "Mar 2023 programming language Python ranks 1rd in TIOBE, it is backend interpretive script language."
    },
    {
        "rank": 2,
        "title": "C",
        "content": "Mar 2023 programming language C ranks 2rd in TIOBE, it is backend compiled language."
    },
    {
        "rank": 3,
        "title": "Java",
        "content": "Mar 2023 programming language Java ranks 3rd in TIOBE, it is backend interpretive script language."
    },
    {
        "rank": 4,
        "title": "C++",
        "content": "Mar 2023 programming language C++ ranks 4rd in TIOBE, it is backend compiled language."
    },
    {
        "rank": 5,
        "title": "C#",
        "content": "Mar 2023 programming language C# ranks 5rd in TIOBE, it is backend interpretive script language."
    },
    {
        "rank": 6,
        "title": "Visual Basic",
        "content": "Mar 2023 programming language VB ranks 6rd in TIOBE, it is backend compiled language."
    },
    {
        "rank": 7,
        "title": "JavaScript",
        "content": "Mar 2023 programming language JavaScript ranks 7rd in TIOBE, it is frontend script language."
    },
    {
        "rank": 8,
        "title": "SQL",
        "content": "Mar 2023 programming language SQL ranks 8rd in TIOBE, it is SQL script language."
    },
    {
        "rank": 9,
        "title": "PHP",
        "content": "Mar 2023 programming language PHP ranks 9rd in TIOBE, it is backend interpretive script language."
    },
    {
        "rank": 10,
        "title": "Go",
        "content": "Mar 2023 programming language Go ranks 10rd in TIOBE, it is backend compiled language."
    },
])
```

> 执行文本搜索

```shell
# 执行文本搜索查询的集合中必须有一个文本索引
> db.collection.find( { $text: { $search: "\"script language\""} } )
Error: error: {
        "ok" : 0,
        "errmsg" : "text index required for $text query",
        "code" : 27,
        "codeName" : "IndexNotFound"
}

# 创建文本索引
# 允许在 title 和 content 字段上进行文本搜索
> db.collection.createIndex( { title: "text", content: "text" } )
{
    "numIndexesBefore" : 1,
    "numIndexesAfter" : 2,
    "createdCollectionAutomatically" : false,
    "ok" : 1
}

# 查看集合中的所有索引
> db.collection.getIndexes()
[
    {
        "v": 2,
        "key": {
            "_id": 1
        },
        "name": "_id_"
    },
    {
        "v": 2,
        "key": {
            "_fts": "text",
            "_ftsx": 1
        },
        "name": "title_text_content_text",
        "weights": {
            "content": 1,
            "title": 1
        },
        "default_language": "english",
        "language_override": "language",
        "textIndexVersion": 3
    }
]

# ==============================================
#  $text 查询运算符对具有 text 索引的集合执行文本搜索。
# ==============================================
# $text 将使用空格和大多数标点符号作为分隔符来标记搜索字符串，并对搜索字符串中的所有标记执行 OR 逻辑运算。

# 执行文本搜索
#  搜索包含 SQL 或 script 或 language 短语的文档
db.collection.find( { $text: { $search: "SQL script language" } } )

# 搜索确切的短语 ""
#  通过用双引号将它们括起来来搜索确切的短语
db.collection.find( { $text: { $search: "\"SQL script language\"" } } )

# 排除术语 -
#  通过 `-` 排除一个词
db.collection.find( { $text: { $search: "Go C -C#" } } )      # Go
db.collection.find( { $text: { $search: "Go C -\"C#\"" } } )  # Go C C++

# =======================================
#  $meta 查询运算符获取每个匹配文档的相关性得分
# =======================================

# 获取相关性得分
db.collection.find(
    { $text: { $search: "Go C" } },
    { score: { $meta: "textScore" } }
)

# 按相关顺序排列
db.collection.find(
    { $text: { $search: "Go C" } },
    { score: { $meta: "textScore" } }
).sort( { score: { $meta: "textScore" } } )

# ==================
#  聚合管道中的文本搜索
# ==================
# 聚合管道中的文本搜索限制：
# - 包含 $text 的 $match 阶段必须是管道中的第一个阶段。
# - 一个 $text 运算符在阶段中只能出现一次。
# - $text 运算符表达式不能出现在 $or 或 $not 表达式中。
# - 要按匹配分数排序，在阶段 $meta 中使用聚合表达式 $sort。

# 计算包含一个词的文章的总浏览量
db.articles.aggregate(
   [
     { $match: { $text: { $search: "cake" } } },
     { $group: { _id: null, views: { $sum: "$views" } } }
   ]
)

# 返回按文本搜索分数排序的结果
db.articles.aggregate(
   [
     { $match: { $text: { $search: "cake tea" } } },
     { $sort: { score: { $meta: "textScore" } } },
     { $project: { title: 1, _id: 0 } }
   ]
)

# 匹配文本分数
db.articles.aggregate(
   [
     { $match: { $text: { $search: "cake tea" } } },
     { $project: { title: 1, _id: 0, score: { $meta: "textScore" } } },
     { $match: { score: { $gt: 1.0 } } }
   ]
)

# 指定文本搜索的语言
db.articles.aggregate(
   [
     { $match: { $text: { $search: "saber -claro", $language: "es" } } },
     { $group: { _id: null, views: { $sum: "$views" } } }
   ]
)
```

> 文本搜索语言

| 语言名称     | ISO 639-1（双字母代码） |
| :----------- | :---------------------- |
| `danish`     | `da`                    |
| `dutch`      | `nl`                    |
| `english`    | `en`                    |
| `finnish`    | `fi`                    |
| `french`     | `fr`                    |
| `german`     | `de`                    |
| `hungarian`  | `hu`                    |
| `italian`    | `it`                    |
| `norwegian`  | `nb`                    |
| `portuguese` | `pt`                    |
| `romanian`   | `ro`                    |
| `russian`    | `ru`                    |
| `spanish`    | `es`                    |
| `swedish`    | `sv`                    |
| `turkish`    | `tr`                    |

### 7、[地理空间查询](https://www.mongodb.com/docs/manual/geospatial-queries/)

### 8、[Read 隔离（Read Concern）](https://www.mongodb.com/docs/manual/reference/read-concern/)

> ReadConcern 选项允许您控制从副本集和副本集分片读取的数据的一致性和隔离性。

通过写关注点和读关注点的有效使用，可以适当调整一致性和可用性保证的级别。

未显式指定读关注点的操作会继承全局默认读关注的设置。

```shell
# 设置全局默认读写关注点
db.adminCommand(
    {
        setDefaultRWConcern : 1,
        defaultReadConcern: { <read concern> },
        defaultWriteConcern: { <write concern> },
        writeConcern: { <write concern> },
        comment: <any>
    }
)
```

> 读关注点的级别

| Level                                                        | 描述                                                         |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| [`local`](https://www.mongodb.com/docs/v5.0/reference/read-concern-local/#mongodb-readconcern-readconcern.-local-) | 查询从实例返回数据，但不保证数据已写入大多数副本集成员（即可能回滚）。<br />针对主要和次要读取的默认值。 |
| [`available`](https://www.mongodb.com/docs/v5.0/reference/read-concern-available/#mongodb-readconcern-readconcern.-available-) | 查询从实例返回数据，但不保证数据已写入大多数副本集成员（即可能回滚）。 |
| [`majority`](https://www.mongodb.com/docs/v5.0/reference/read-concern-majority/#mongodb-readconcern-readconcern.-majority-) | 查询返回已被大多数副本集成员确认的数据。                     |
| [`linearizable`](https://www.mongodb.com/docs/v5.0/reference/read-concern-linearizable/#mongodb-readconcern-readconcern.-linearizable-) | 查询返回的数据反映了在读取操作开始之前完成的所有成功的多数确认写入。在返回结果之前，查询可能会等待并发执行的写入传播到大多数副本集成员。 |
| [`snapshot`](https://www.mongodb.com/docs/v5.0/reference/read-concern-snapshot/#mongodb-readconcern-readconcern.-snapshot-) | 具有读取关注的查询`"snapshot"`返回多数提交的数据，因为它出现在最近的特定单个时间点的分片中。 |



### 9、[Write 确认（Write Concern）](https://www.mongodb.com/docs/manual/reference/write-concern/)

> Write Concern 描述了 MongoDB 对独立mongod、副本集或分片集群的写操作请求的确认级别。在分片集群中，mongos 实例将把写关注传递给分片。

未指定显式写关注的操作继承全局默认写关注设置。

```shell
# 设置全局默认读写关注点
db.adminCommand(
    {
        setDefaultRWConcern : 1,
        defaultReadConcern: { <read concern> },
        defaultWriteConcern: { <write concern> },
        writeConcern: { <write concern> },
        comment: <any>
    }
)
```

> 写关注规范

```shell
{ w: <value>, j: <boolean>, wtimeout: <number> }
```

- `w`：用于确认写操作已传播到指定数量的mongod 实例或具有指定标记的 mongod 实例。

  | w：<取值>                     | 描述 |
  | ----------------------------- | ---- |
  | `majority`                    |      |
  | `<number>`                    |      |
  | `<custom write concern name>` |      |

- `j`：用于确认写操作已写入磁盘日志。
- `wtimeout`：指定写操作的时间限制。wtimeout 仅适用于 `w > 1` 的值。



## 四、CRUD 概念

### 1、原子性和事务

#### （1）原子性

在 MongoDB 中，写操作在**单个文档**级别上是原子的，即使该操作修改了单个文档中的多个嵌套文档。

#### （2）多文档事务

当单个写操作（如：db.collection.updateMany()）修改多个文档时，每个文档的修改时原子的，但整个操作不是原子的。

当执行多文档写入操作时，无论是通过单个写入操作还是通过多个写入操作，其他操作都可能交错。

对于需要对多个文档（在单个或多个集合中）进行原子化读写的情况，MongoDB支持分布式事务，包括副本集和分片集群上的事务。

#### （3）并发控制

并发控制允许多个应用程序同时运行不会导致数据不一致或冲突。

findAndModify 对文档的操作是原子的，如果查找条件与文档匹配，则对该文档执行更新。在当前更新完成之前，对该文档的并发查询和其他更新不会受到影响。

### 2、读取隔离、一致性和新近度

### 3、分布式查询

### 4、查询计划

### 5、查询优化

### 6、写操作性能

### 7、可尾游标

### 8、带 . 和 $ 的字段名称



## 五、聚合操作

> 聚合操作处理多个文档返回计算结果，可以使用聚合操作实现：
>
> - 将来自多个文档的值组合在一起；
> - 对分组数据执行操作以返回单个结果；
> - 分析数据随时间的变化。

### 1、聚合方法

```shell
# 返回集合或视图中文档的近似计数
db.collection.estimatedDocumentCount()

# 返回集合或视图中的文档数
db.collection.count()

# 返回指定字段具有不同值的文档数组
db.collection.distinct()

# 对大型数据集执行映射缩减聚合（MongoDB 5.0 起弃用）
db.collection.mapReduce()

# 提供对聚合管道的访问权限
db.collection.aggregate()
```



### 2、聚合管道

#### （1）阶段

聚合管道由一个或多个处理文档的阶段组成：

- 每个阶段对输入文档执行操作。
- 从一个阶段输出的文档被传递到下一个阶段。
- 聚合管道可以返回文档组的结果。

| 阶段             | 描述                                                         | 语法                                                         |
| ---------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| $addField        | 向文档中添加新字段。与$project类似，$addFields对流中的每个文档进行重新整形；具体地说，通过向包含输入文档中的现有字段和新添加的字段的输出文档添加新字段。<br/>$set是$addFields的别名。 | { $addFields: { <newField>: <expression>, ... } }            |
| $bucket          | 根据指定的表达式和bucket边界，将传入的文档分类到称为bucket的组中。 |                                                              |
| $bucketAuto      | 根据指定的表达式，将传入文档分类为特定数量的组，称为bucket。Bucket边界是自动确定的，目的是将文档平均分配到指定数量的Bucket中。 |                                                              |
| $changeStream    | 返回集合的“更改流”光标。此阶段在聚合管道中只能出现一次，并且必须作为第一个阶段出现。 |                                                              |
| $collStats       | 返回有关集合或视图的统计信息。                               |                                                              |
| $count           | 返回聚合管道的此阶段的文档数的计数。<br/>与$count聚合累加器不同。 | { $count: <string> }                                         |
| $facet           | 在同一组输入文档的单个阶段内处理多个聚合管道。允许创建多方面聚合，这些聚合能够在单个阶段中跨多个维度或方面表征数据。 |                                                              |
| $geoNear         | 根据与地理空间点的接近程度返回有序的文档流。结合了地理空间数据的$match、$sort和$limit功能。输出文档包括附加的距离字段，并且可以包括位置标识符字段。 |                                                              |
| $graphLookup     | 对集合执行递归搜索。向每个输出文档添加一个新的数组字段，该字段包含该文档的递归搜索的遍历结果。 |                                                              |
| $group           | 根据指定的标识符表达式对输入文档进行分组，并将累加器表达式（如果指定）应用于每个组。使用所有输入文档，并为每个不同的组输出一个文档。输出文档仅包含标识符字段，如果指定，还包含累积字段。 | {<br/> $group:<br/>   {<br/>     _id: <expression>, // Group key<br/>     <field1>: { <accumulator1> : <expression1> },<br/>     ...<br/>   }<br/> } |
| $indexStats      | 返回有关集合中每个索引的使用情况的统计信息。                 |                                                              |
| $limit           | 将未修改的前n个文档传递到管道，其中n是指定的限制。对于每个输入文档，输出一个文档（前n个文档）或零个文档（在前n个文件之后）。 | { $limit: <positive 64-bit integer> }                        |
| $listSessions    | 列出活动时间足以传播到system.sessions集合的所有会话。        |                                                              |
| $lookup          | 对同一数据库中的另一个集合执行左外部联接，以筛选“联接”集合中的文档进行处理。 |                                                              |
| $match           | 过滤文档流，只允许匹配的文档未经修改地传递到下一个管道阶段$match使用标准的MongoDB查询。对于每个输入文档，输出一个文档（匹配）或零个文档（不匹配）。 | { $match: { <query> } }                                      |
| $merge           | 将聚合管道的结果文档写入集合。该阶段可以将结果合并（插入新文档、合并文档、替换文档、保留现有文档、操作失败、使用自定义更新管道处理文档）到输出集合中。若要使用$merge阶段，它必须是管道中的最后一个阶段。 |                                                              |
| $out             | 将聚合管道的结果文档写入集合。要使用$out阶段，它必须是管道中的最后一个阶段。 |                                                              |
| $planCacheStats  | 返回集合的计划缓存信息。                                     |                                                              |
| $project         | 重塑流中的每个文档，例如通过添加新字段或删除现有字段。对于每个输入文档，输出一个文档。 |                                                              |
| $redact          | 通过基于存储在文档本身中的信息限制每个文档的内容来重塑流中的每个文档。包含$project和$match的功能。可用于实现字段级编校。对于每个输入文档，输出一个或零个文档。 |                                                              |
| $replaceRoot     | 用指定的嵌入文档替换文档。该操作将替换输入文档中的所有现有字段，包括_id字段。指定嵌入在输入文档中的文档，以将嵌入的文档提升到顶级。 |                                                              |
| $replaceWith     | $replaceWith是$replaceRoot阶段的别名。                       |                                                              |
| $sample          | 从其输入中随机选择指定数量的文档。                           |                                                              |
| $search          | 对Atlas集合中的一个或多个字段执行全文搜索。                  |                                                              |
| $set             | 向文档中添加新字段。与$project类似，$set对流中的每个文档进行整形；具体地说，通过向包含输入文档中的现有字段和新添加的字段的输出文档添加新字段。<br/>$set是$addFields阶段的别名。 |                                                              |
| $setWindowFields | 将文档分组到窗口中，并将一个或多个运算符应用于每个窗口中的文档。 |                                                              |
| $skip            | 跳过前n个文档，其中n是指定的跳过编号，并将未修改的其余文档传递到管道。对于每个输入文档，输出零个文档（对于前n个文档）或一个文档（如果在前n个文件之后）。 |                                                              |
| $sort            | 按指定的排序键重新排序文档流。只有订单发生变化；这些文档保持不变。对于每个输入文档，输出一个文档。 |                                                              |
| $sortByCount     | 根据指定表达式的值对传入文档进行分组，然后计算每个不同组中的文档数。 |                                                              |
| $unionWith       | 执行两个集合的并集；即将来自两个集合的流水线结果组合成单个结果集。 |                                                              |
| $unset           | 从文档中删除/排除字段。                                      |                                                              |
| $unwind          | 从输入文档中解构一个数组字段，为每个元素输出一个文档。每个输出文档都用一个元素值替换数组。对于每个输入文档，输出n个文档，其中n是数组元素的数量，对于空数组可以为零。 |                                                              |



#### （2）变量

聚合表达式既可以使用用户定义的变量，也可以使用系统变量。

变量可以保存任何 BSON 类型的数据，若要访问变量值，请在变量名称前加上前缀双美元符号（`$$<variable>.<field>`）。

**用户变量：**

用户变量名称可以包含 ASCII 字符和任何非 ASCII 字符，必须以小写 ASCII 字母或非 ASCII 字符开头。

**系统变量：**

| 变量           | 描述                                                         |
| -------------- | ------------------------------------------------------------ |
| $$NOW          | 返回当前日期时间值的变量。<br/>NOW为部署的所有成员返回相同的值，并在聚合管道的所有阶段保持不变。 |
| $$CLUSTER_TIME | 返回当前时间戳值的变量。<br/>CLUSTER_TIME 仅在副本集和分片集群上可用 。<br/>CLUSTER_TIME 为部署的所有成员返回相同的值，并在管道的所有阶段保持不变。 |
| $$ROOT         | 引用当前正在聚合管道阶段处理的根文档，即顶级文档。           |
| $$CURRENT      | 引用聚合管道阶段中正在处理的字段路径的开头。除非另有说明，否则所有阶段都以 CURRENT 开头，与 ROOT 相同。<br/>CURRENT 是可修改的。但是，由于`$<field>` 相当于 `$$CURRENT<field>`，重新绑定 CURRENT 将更改 `$access` 的含义。 |
| $$REMOVE       | 计算结果为缺失值的变量。允许有条件地排除字段。在 $project 中，设置为变量 REMOVE 的字段将从输出中排除。 |
| $$DESCEND      | $redact 表达式允许的结果之一。                               |
| $$PRUNE        | $redact 表达式允许的结果之一。                               |
| $$KEEP         | $redact 表达式允许的结果之一。                               |





#### （3）运算符

运算符表达式类似于接受参数的函数。

```
# 运算符接受单个参数
{ <operator>: <argument> }

# 运算符接受一组参数
{ <operator>: [ <argument1>, <argument2>, ...] }
```

##### 算术运算符

| 名称      | 描述                                                         |
| --------- | ------------------------------------------------------------ |
| $abs      | 返回一个数字的绝对值。                                       |
| $add      | 添加数字以返回总和，或添加数字和日期以返回新日期。如果将数字和日期相加，则将数字视为毫秒。接受任意数量的参数表达式，但最多只能有一个表达式解析为一个日期。 |
| $ceil     | 返回大于或等于指定数字的最小整数。                           |
| $divide   | 返回第一个数字除以第二个数字的结果。接受两个参数表达式。     |
| $exp      | 将e提升到指定的指数。                                        |
| $floor    | 返回小于或等于指定数字的最大整数。                           |
| $ln       | 计算一个数字的自然对数。                                     |
| $log      | 计算以指定基数表示的数字的对数。                             |
| $log10    | 计算一个数字的对数基数10。                                   |
| $mod      | 返回第一个数字除以第二个数字的余数。接受两个参数表达式。     |
| $multiply | 将数字相乘以返回乘积。接受任意数量的参数表达式。             |
| $pow      | 将数字提升到指定的指数。                                     |
| $round    | 将数字舍入为整整数或指定的小数位数。                         |
| $sqrt     | 计算平方根。                                                 |
| $subtract | 返回第一个值减去第二个值的结果。如果这两个值是数字，则返回差值。如果这两个值是日期，则返回以毫秒为单位的差值。如果这两个值是一个日期和一个以毫秒为单位的数字，则返回结果日期。接受两个参数表达式。如果这两个值是日期和数字，请先指定日期参数，因为从数字中减去日期没有意义。 |
| $trunc    | 将数字截断为整整数或指定的小数位数。                         |

##### 数组运算符

| 名称           | 描述                                                         |
| -------------- | ------------------------------------------------------------ |
| $arrayElemAt   | 返回指定数组索引处的元素。                                   |
| $arrayToObject | 将键值对数组转换为文档。                                     |
| $concatArrays  | 串联数组以返回串联的数组。                                   |
| $filter        | 选择数组的子集以返回仅包含与筛选条件匹配的元素的数组。       |
| $first         | 返回第一个数组元素。与$first累加器不同。                     |
| $in            | 返回一个布尔值，指示指定值是否在数组中。                     |
| $indexOfArray  | 在数组中搜索指定值的引用，并返回第一个引用的数组索引。如果未找到子字符串，则返回-1。 |
| $isArray       | 确定操作数是否为数组。返回布尔值。                           |
| $last          | 返回最后一个数组元素。与$last累加器不同。                    |
| $map           | 将子表达式应用于数组的每个元素，并按顺序返回结果值的数组。接受命名参数。 |
| $objectToArray | 将文档转换为表示键值对的文档数组。                           |
| $range         | 根据用户定义的输入输出一个包含整数序列的数组。               |
| $reduce        | 将表达式应用于数组中的每个元素，并将它们组合为一个值。       |
| $reverseArray  | 返回元素顺序相反的数组。                                     |
| $size          | 返回数组中的元素数。接受单个表达式作为参数。                 |
| $slice         | 返回数组的子集。                                             |
| $zip           | 将两个数组合并在一起。                                       |

##### 布尔运算符

| 名称 | 描述 |
| ---- | ---- |
|      |      |
|      |      |
|      |      |



##### 比较运算符



##### 条件运算符



##### 自定义运算符



##### 数据大小运算符



##### 日期运算符



##### 文本运算符



##### 其他运算符



##### 对象表达式运算符



##### 设置运算符



##### 字符串运算符



##### 三角运算符



##### 类型运算符





```shell
# 填充数据
db.collection.insertMany([
    {
        "grade": "高一年级",
        "class": "高一（1）班",
        "student_num": 50,
        "score": 84
    },
    {
        "grade": "高一年级",
        "class": "高一（2）班",
        "student_num": 55,
        "score": 97
    },
    {
        "grade": "高一年级",
        "class": "高一（3）班",
        "student_num": 58,
        "score": 91
    },
    {
        "grade": "高二年级",
        "class": "高二（1）班",
        "student_num": 62,
        "score": 90
    },
    {
        "grade": "高二年级",
        "class": "高二（2）班",
        "student_num": 65,
        "score": 92
    },
    {
        "grade": "高二年级",
        "class": "高二（3）班",
        "student_num": 66,
        "score": 93
    },
    {
        "grade": "高三年级",
        "class": "高三（1）班",
        "student_num": 51,
        "score": 86
    },
    {
        "grade": "高三年级",
        "class": "高三（2）班",
        "student_num": 50,
        "score": 79
    },
    {
        "grade": "高三年级",
        "class": "高三（3）班",
        "student_num": 56,
        "score": 75
    }
])
```

#### （4）聚合管道查询示例

```shell
# 查询各年级的总人数
db.collection.aggregate([

    // Stage 1: 匹配 grade = "高二年级" 的所有文档，并传到下一阶段
    // { $match: { grade: "高二年级" } },
    
    // Stage 2: 按 grade 进行分组，计算各年级的学生总数
    { $group: { _id: "$grade", studentTotal: { $sum: "$student_num" } } },
    
    // Stage 3: 按总人数降序排列
    { $sort: { studentTotal: -1 } }
])

{ "_id" : "高二年级", "studentTotal" : 193 }
{ "_id" : "高一年级", "studentTotal" : 163 }
{ "_id" : "高三年级", "studentTotal" : 157 }

# 查询高二年级的最高分及平均分
db.collection.aggregate([

    // Stage 1: 匹配 grade = "高二年级" 的所有文档，并传到下一阶段
    { $match: { grade: "高二年级" } },

    // Stage 2: 按 grade 进行分组，计算学生总人数
    { $group: { _id: "$grade", maxScore: { $max: "$score" }, avgScore: { $avg: "$score" } } }

])

{ "_id" : "高二年级", "maxScore" : 93, "avgScore" : 91.66666666666667 }

```

> 聚合管道阶段的注意事项：
>
> - 一个阶段不必为每个输入文档输出一个文档。
> - 同一阶段可以在管道中出现多次，但 $out、$merge、$geoNear 阶段除外。
> - 要计算平均值并在阶段中执行其他计算，请使用 指定 聚合运算符的聚合表达式。

#### （5）[聚合管道优化](https://www.mongodb.com/docs/v5.0/core/aggregation-pipeline-optimization/)

```

```

#### （6）聚合管道限制

- 结果大小限制：结果集中的每个文档都受 16MB （BSON 文档大小）限制。
- 阶段数限制：单个管道中允许的聚合管道阶段数限制为 1000。（V 5.0 更改）
- 内存限制：每个单独的管道阶段都有 100MB 的 RAM 限制。

#### 

#### 



## 六、数据模型



## 七、事务



## 八、索引

MongoDB 索引使用 B-Tree 数据结构。

### 1、操作索引

```shell
# 创建索引
db.collection.createIndex( <key and index type specification>, <options> )

# 例：在字段 created_at 上创建一个键降序索引 
db.collection.createIndex( { created_at: -1 } )

# 索引排序顺序
# 对于单字段索引，排序顺序（升序或降序）对索引键无关紧要，因为 MongoDB 可以遍历索引在任一方向。

# 指定索引名称
db.collection.createIndex(
   { <field>: <value> },
   { name: "<indexName>" }
)

# 在指定索引名称之前，请考虑以下事项：
#  索引名称必须唯一，创建同名索引时返回错误
#  不能重命名索引，须先删除后创建

# 获取索引
db.collection.getIndexes()

# 删除索引
# 删除单个索引
db.collection.dropIndex("<indexName>")
# 删除多个索引
db.collection.dropIndexes("<index1>", "<index2>", "<index3>")
# 删除除 _id 索引之外的所有索引
db.collection.dropIndexes()

```

### 2、索引类型

#### （1）单字段索引（Single Field）

单字段索引是 B-Tree 索引，又叫排序索引。

默认情况下，所有集合中都有一个单字段索引：_id。您可以在文档中的任何字段上创建单字段索引，包括：

- 顶级文档字段
- 嵌入式文档
- 嵌入文档中的字段

```sh
# 语法
# field：要在其上创建索引的字段
# type：索引值的排序顺序（1 升序，-1 降序）
db.collection.createIndex( { <field>: <type>} ）

# 例
db.collection.createIndex({created_at: -1})
```

优点：适用于在单个字段上快速查询和排序，支持高效的范围查询。

缺点：不能处理多个字段之间的查询关系，索引占用空间较大，维护开销大

使用场景：适用于对单个字段进行频繁查询和排序的场景，如按创建时间倒序。

#### （2）复合索引（Compound）

> 限制：单个复合索引最多可以包含 32 个字段。
>

```sh
# 语法
db.collection.createIndex( { <field1>: <type>, <field2>: <type2>, ... } )

# 例
db.collection.createIndex({"item": 1, "location": 1, "stock": 1})

# 最左前缀特性，具有以下索引前缀
{ item: 1 }
{ item: 1, location: 1 }

# 支持这些字段的组合查询
item
item + location
item + location + stock
```

> 复合索引的排序顺序

复合索引支持与索引的排序顺序或索引的反向排序顺序匹配的排序操作。

```shell
# 创建复合索引
db.collection.createIndex( { score: -1, username: 1 } )

# 有效的索引查询
# 正向排序
db.collection.find().sort({ score: -1, username: 1 })
# 反向排序
db.collection.find().sort({ score: 1, username: -1 })
```

优点：可以同时对多个文本字段进行搜索和排序。

缺点：索引占用空间较大，维护开销大。

使用场景：适用于需要对多个文本字段进行搜索和排序的场景。

#### （3）多键（数组）索引（Multikey）

[多键索引](https://www.mongodb.com/docs/manual/core/indexes/index-types/index-multikey/)

```
db.collection.createIndex({ "field.$": 1 })
```

优点：可以提高查询数组元素的速度。

缺点：索引占用空间较大，更新数组字段会导致写操作性能下降。

使用场景：对数组字段进行频繁的查询和排序操作；需要使用数组操作符（$elemMatch, $all）进行数组元素的匹配查询；数组中的元素数量过多时。

#### （4）文本索引（Text）

文本索引支持对包含字符串内容的字段进行文本搜索查询。

一个集合只能有一个文本索引，但该索引可以涵盖多个字段。

```sh
db.collection.createIndex(
   {
      <field1>: "text",
      <field2>: "text",
      ...
   },
   # 指定文本索引的语言
   {
       default_language: "spanish"
   },
   # 指定索引的名称
   {
       name: "MyTextIndex",
       # 指定字段权重（默认 1），可以使用权重控制搜索结果：$meta
       weights: {
           <field1>: 10,
           <field2>: 5
       },
   }
)
```

> 文本搜索支持

文本索引支持本地部署上的 $text 查询操作。若要执行文本搜索，必须创建文本索引并使用查询运算符 $text。

```
db.collection.find(
   {
      $text: { $search: "coffee" }
   }
)
```

文本索引可能会占用大量 RAM。

> 创建通配符文本索引

您可以创建一个包含每个文档字段的文本索引

> 限制扫描的条目数

```sh
# 要将文本搜索限制为仅扫描特定 dept 中的那些文档，需创建一个复合索引
# dept 的升序/降序索引，description 的文本索引
db.inventory.createIndex(
   {
     dept: 1,
     description: "text"
   }
)

# 特定部门内的文本搜索将限制对索引文档的扫描。
# 例如，以下查询只扫描 dept=kitchen 的那些文档：
db.inventory.find( { dept: "kitchen", $text: { $search: "green" } } )
```

#### （5）通配符索引（Wildcard）

通配符索引：对子字段未知的文档创建的索引。

```sh
# 例如，以下用户数据需要支持查询
{ "userMetadata" : { "likes" : [ "dogs", "cats" ] } }
{ "userMetadata" : { "dislikes" : "pickles" } }
{ "userMetadata" : { "age" : 45 } }
{ "userMetadata" : "inactive" }

# 在所有字段上创建通配符索引
db.collection.createIndex( { "userMetadata.$**" : 1 } )

# 以上通配符索引支持的单字段查询
userMetadata
userMetadata.likes
userMetadata.dislikes
userMetadata.age

# 在指定字段上创建
db.collection.createIndex(
  { "$**" : 1 },
  { "wildcardProjection" :
    { "fieldA" : 1, "fieldB.fieldC" : 1 }
  }
)

# 排除指定字段创建
db.collection.createIndex(
  { "$**" : 1 },
  { "wildcardProjection" :
    { "fieldA" : 0, "fieldB.fieldC" : 0 }
  }
)
```

#### （6）二维球体索引（2dsphere）

#### （7）地理空间索引（geoHaystack）

#### （8）哈希索引（Hashed）

MongoDB 不支持在哈希索引上指定唯一约束。

```sh
# 创建
db.collection.createIndex( { <field>: "hashed" } )
```

优点：可以快速查找索引键值。

缺点：无法按照索引键的顺序进行排序和范围查询。

使用场景：适用于均匀分布的键值和快速查找索引键值的场景。

> 复合哈希索引
>
> 从 MongoDB 4.4 开始，支持创建包含单个散列字段的复合索引。

```
db.collection.createIndex( { "fieldA" : 1, "fieldB" : "hashed", "fieldC" : -1 } )
```

### 3、索引属性

#### （1）TTL 索引

TTL 索引是一种特殊的单字段索引，MongoDB 可以使用它在一定时间后或特定的时钟时间自动从集合中删除文档。数据过期对于某些类型的信息很有用，如机器生成的事件数据、日志和会话信息。

> 注意：

- TTL 索引支持查询的方式与非 TTL 索引相同。
- TTL 索引是单字段索引。复合索引不支持 TTL 并忽略 `expireAfterSeconds` 选项。
- `_id` 字段不支持 TTL 索引。
- 不能在 capped 集合上创建 TTL 索引。
- 不能在时间序列集合上创建 TTL 索引。
- 不能将非 TTL 索引字段直接更改为 TTL 索引；不能更改现有 TTL 索引的 `expireAfterSeconds` 选项值，先删除。

```sh
# 创建 TTL 索引
db.collection.createIndex( { <field>: 1 }, { expireAfterSeconds: <second> } )

# 例：已删除数据 7天后自动过期
# deleted_at 字段必须为 Date 类型
db.request_log.createIndex( { "deleted_at": 1 }, { expireAfterSeconds: 7 * 24 * 3600 } )
```

> 数据过期

- TTL 索引使文档在从索引字段值起经过指定秒数后过期，即过期阈值是索引字段值加上指定的秒数。
- 如果字段时一个数组，并且索引中有多个日期值，MongoDB 会使用数组中最低（最早）的日期值来计算过期阈值。
- 如果文档中的索引字段不是日期或包含一个或多个日期值的数组，则文档不会过期。

#### （2）唯一索引

唯一索引可确保索引字段不会存储重复的值；即强制索引字段的唯一性。默认情况下，MongoDB在创建集合的过程中会在_id字段上创建一个唯一的索引。

```sh
# 创建唯一索引
db.collection.createIndex( <key and index type specification>, { unique: true } )

# 单字段索引和复合索引都可以创建唯一索引
```

- 如果集合中已经包含违反索引唯一约束的数据，MongoDB就无法在指定的索引字段上创建唯一索引。

- 不能在哈希索引上指定唯一约束。

#### （3）部分索引

部分索引应优先于稀疏索引

#### （4）不区分大小写的索引

#### （5）稀疏索引

稀疏索引只包含具有索引字段的文档的条目，即使索引字段包含 null 也是如此。索引跳过任何缺少索引字段的文档。索引是“稀疏”的，因为它不包括集合的所有文档。相比之下，非稀疏索引包含集合中的所有文档，为不包含索引字段的文档存储null 值。

```sh
db.collection.createIndex({ field: 1 }, { sparse: true })
```

优点：只为存在索引字段的文档创建索引，减少了索引的大小。
缺点：查询性能可能会受到影响。
使用场景：适用于索引字段不是所有文档都存在的场景，例如文档的某个可选字段。

### 4、索引策略

（1）ESR（相等、排序、范围）

### 5、强制使用索引

### 6、查询执行计划

> [Explain 结果]([Explain Results — MongoDB Manual](https://www.mongodb.com/docs/v5.0/reference/explain-results/))

```sh
db.collection.find().explain([verbosity])

verbosity:
    queryPlanner  默认模式
    executionStats
    allPlansExecution
```

> 已创建索引

```sh
> db.getCollection("request_log").createIndex({
    created_at: - 1,
    server: 1
})
```

#### （1）queryPlanner

> MongoDB 运行查询优化器来为正在评估的操作选择获胜计划。

```sh
> db.getCollection("request_log").find({
    server: "ucs"
}).sort({
    created_at: - 1
}).explain("queryPlanner")

# output
{
    "queryPlanner": {
        "plannerVersion": 1,
        "namespace": "readline_mns.request_log",
        "indexFilterSet": false,
        "parsedQuery": {
            "server": {
                "$eq": "ucs"
            }
        },
        "queryHash": "FEB85418",
        "planCacheKey": "F84E21BA",
        "winningPlan": {
            "stage": "FETCH",
            "filter": {
                "server": {
                    "$eq": "ucs"
                }
            },
            "inputStage": {
                "stage": "IXSCAN",
                "keyPattern": {
                    "created_at": -1
                },
                "indexName": "created_at_-1",
                "isMultiKey": false,
                "multiKeyPaths": {
                    "created_at": [ ]
                },
                "isUnique": false,
                "isSparse": false,
                "isPartial": false,
                "indexVersion": 2,
                "direction": "forward",
                "indexBounds": {
                    "created_at": [
                        "[MaxKey, MinKey]"
                    ]
                }
            }
        },
        "rejectedPlans": [
            {
                "stage": "SORT",
                "sortPattern": {
                    "created_at": -1
                },
                "memLimit": 104857600,
                "type": "simple",
                "inputStage": {
                    "stage": "FETCH",
                    "inputStage": {
                        "stage": "IXSCAN",
                        "keyPattern": {
                            "server": 1
                        },
                        "indexName": "server_1",
                        "isMultiKey": false,
                        "multiKeyPaths": {
                            "server": [ ]
                        },
                        "isUnique": false,
                        "isSparse": false,
                        "isPartial": false,
                        "indexVersion": 2,
                        "direction": "forward",
                        "indexBounds": {
                            "server": [
                                "[\"ucs\", \"ucs\"]"
                            ]
                        }
                    }
                }
            }
        ]
    },
    "serverInfo": {
        "host": "centos-248",
        "port": 27017,
        "version": "4.4.14",
        "gitVersion": "0b0843af97c3ec9d2c0995152d96d2aad725aab7"
    },
    "ok": 1
}
```

> 参数说明

| 参数                                   | 说明                                                         |
| -------------------------------------- | ------------------------------------------------------------ |
| queryPlanner{}                         | 查询优化器                                                   |
| queryPlanner.plannerVersion            | 查询计划输出格式版本                                         |
| queryPlanner.namespace                 | 命名空间，格式为：`<database>.<collection>`                  |
| queryPlanner.indexFilterSet            | MongoDB 是否为查询形状应用了索引过滤器                       |
| queryPlanner.parsedQuery               | 查询条件                                                     |
| queryPlanner.queryHash                 | 查询形状的哈希（缓存）                                       |
| queryPlanner.planCacheKey              | 与查询关联的计划缓存项的密钥的哈希                           |
| queryPlanner.winningPlan{}             | 详细说明查询优化器选择的计划                                 |
| queryPlanner.winningPlan.stage         | **阶段名称：**<br />COLLSCAN：集合扫描（全表扫描）<br />FETCH：检索文档<br />IXSCAN：索引扫描<br />SHARD_MERGE：合并分片结果<br />IDHACK：使用 _id 查询<br />SHARDING_FILTER：从分片中过滤掉孤立文档 |
| queryPlanner.winningPlan.filter        | 过滤条件                                                     |
| queryPlanner.winningPlan.inputStage{}  | 子阶段（父阶段只有一个子阶段），为其父阶段提供文档或索引键   |
| queryPlanner.winningPlan.inputStages[] | 子阶段（父阶段具有多个子阶段），为其父阶段提供文档或索引键   |
| queryPlanner.winningPlan.direction     | 查询方向                                                     |
| queryPlanner.winningPlan.indexBounds   | 索引边界                                                     |
| ...                                    |                                                              |
| queryPlanner.rejectedPlans[]           | 查询优化器考虑和拒绝的候选计划                               |
| queryPlanner.rejectedPlans.stage       | SORT 排序阶段                                                |
| queryPlanner.rejectedPlans.sortPattern | 排序模式                                                     |
| queryPlanner.rejectedPlans.memLimit    | 内存限制                                                     |
| ...                                    |                                                              |
| serverInfo{}                           | MongoDB 服务器信息                                           |

#### （2）executionStats

> MongoDB 运行查询优化器来选择获胜计划，执行获胜计划直到完成，并返回描述获胜计划执行情况的统计信息，不提供被拒绝计划的查询执行信息。

```sh
> db.getCollection("request_log").find({
    server: "ucs"
}).sort({
    created_at: - 1
}).explain("executionStats")

# output
{
    "queryPlanner": {},
    "executionStats": {
        "executionSuccess": true,
        "nReturned": 105894,
        "executionTimeMillis": 1436,
        "totalKeysExamined": 128746,
        "totalDocsExamined": 128746,
        "executionStages": {
            "stage": "FETCH",
            "filter": {
                "server": {
                    "$eq": "ucs"
                }
            },
            "nReturned": 105894,
            "executionTimeMillisEstimate": 1075,
            "works": 128747,
            "advanced": 105894,
            "needTime": 22852,
            "needYield": 0,
            "saveState": 169,
            "restoreState": 169,
            "isEOF": 1,
            "docsExamined": 128746,
            "alreadyHasObj": 0,
            "inputStage": {
                "stage": "IXSCAN",
                "nReturned": 128746,
                "executionTimeMillisEstimate": 8,
                "works": 128747,
                "advanced": 128746,
                "needTime": 0,
                "needYield": 0,
                "saveState": 169,
                "restoreState": 169,
                "isEOF": 1,
                "keyPattern": {
                    "created_at": -1
                },
                "indexName": "created_at_-1",
                "isMultiKey": false,
                "multiKeyPaths": {
                    "created_at": [ ]
                },
                "isUnique": false,
                "isSparse": false,
                "isPartial": false,
                "indexVersion": 2,
                "direction": "forward",
                "indexBounds": {
                    "created_at": [
                        "[MaxKey, MinKey]"
                    ]
                },
                "keysExamined": 128746,
                "seeks": 1,
                "dupsTested": 0,
                "dupsDropped": 0
            }
        }
    },
    "serverInfo": {},
    "ok": 1
}
```

> 参数说明

| 参数                                                       | 说明                                                         |
| ---------------------------------------------------------- | ------------------------------------------------------------ |
| executionStats                                             | 包含描述获胜计划的已完成查询执行情况的统计信息。对于写操作，已完成的查询执行是指将要执行的修改，但不将这些修改应用于数据库。 |
| executionStats.executionSuccess                            | 是否执行成功                                                 |
| executionStats.nReturned                                   | 与查询条件匹配的文档数                                       |
| executionStats.executionTimeMillis                         | 选择查询计划和执行查询所需的总时间（ms）                     |
| executionStats.totalKeysExamined                           | 扫描的索引条目数                                             |
| executionStats.totalDocsExamined                           | 执行查询期间检查的文档数。检查文档的常见查询执行阶段是 COLLSCAN 和 FETCH。 |
| executionStats.executionStages{}                           | 以阶段树的形式详细说明已完成的获胜计划执行情况；<br/>每个阶段都包含特定于该阶段的执行信息。 |
| executionStats.executionStages.stage                       | 查询方式                                                     |
| executionStats.executionStages.filter                      | 过滤条件                                                     |
| executionStats.executionStages.nReturned                   | 返回结果数                                                   |
| executionStats.executionStages.executionTimeMillisEstimate | 查询执行的估计时间量（ms）                                   |
| executionStats.executionStages.works                       | 指定查询执行阶段执行的工作单位数。，一个查询会分解成小的工作单元 |
| executionStats.executionStages.advanced                    | 此阶段返回或推进到其父阶段的中间结果数                       |
| executionStats.executionStages.needTime                    | 没有将中间结果推进到其父阶段的工作周期数                     |
| executionStats.executionStages.needYield                   | 存储层请求查询阶段挂起处理并产生锁的次数                     |
| executionStats.executionStages.saveState                   | 查询阶段挂起处理并保存其当前执行状态的次数，例如为生成其锁做准备的次数 |
| executionStats.executionStages.restoreState                | 查询阶段恢复已保存的执行状态的次数，例如在恢复以前产生的锁之后 |
| executionStats.executionStages.isEOF                       | 执行阶段是否已到达 Steam 的结尾：<br>true 或 1，则表示已到达流的结尾<br>false 或 0，则该阶段可能仍有结果要返回 |
| executionStats.executionStages.docsExamined                | 文档检查数                                                   |
| executionStats.executionStages.alreadyHasObj               |                                                              |
| executionStats.executionStages.inputStage{}                | 输入阶段                                                     |
| executionStats.executionStages.inputStage.keysExamined     | 对于扫描索引的查询执行阶段（IXSCAN），keysExamined 是在索引扫描过程中检查的 in-bounds 和 out-of-bounds 键的总数。如果索引扫描由单个连续的键组成，则只需要检查边界内的键。如果索引边界由多个键范围组成，则索引扫描执行过程可以检查越界键，以便从一个范围的末尾跳到下一个区域的开头。<br>`db.keys.find( { x : { $in : [ 3, 4, 50, 74, 75, 90 ] } } ).explain( "executionStats" )`<br>查询扫描键 3、4、5、50、51、74、75、76、90、91。键 5、51、76 和 91 是仍在检查的越界键。keysExamined的值为10。 |
| executionStats.executionStages.inputStage.docsExamined     | 指定在查询执行阶段扫描的文档数<br/>用于COLLSCAN阶段以及从集合中检索文档的阶段（FETCH） |
| executionStats.executionStages.inputStage.seeks            | 仅用于索引扫描（IXSCAN）阶段。<br/>为了完成索引扫描，我们必须将索引光标搜索到新位置的次数。 |
| executionStats.allPlansExecution                           | 包含在计划选择阶段为获胜计划和被拒绝计划捕获的部分执行信息。仅当解释在 allPlansExecution 详细模式下运行时，该字段才存在。 |

#### （3）allPlansExecution

> MongoDB 运行查询优化器来选择获胜计划，并执行获胜计划直至完成。在该模式中，MongoDB 返回描述获胜计划执行情况的统计信息，以及在 allPlansExecution 计划选择过程中捕获的其他候选计划的统计信息。

```sh
> db.getCollection("request_log").find({
    server: "ucs"
}).sort({
    created_at: - 1
}).explain("allPlansExecution")

# output
{
    "queryPlanner": {},
    "executionStats": {},
    	...,
        "allPlansExecution": [
            {
                "nReturned": 101,
                "executionTimeMillisEstimate": 0,
                "totalKeysExamined": 168,
                "totalDocsExamined": 168,
                "executionStages": {
                    "stage": "FETCH",
                    "filter": {
                        "server": {
                            "$eq": "ucs"
                        }
                    },
                    "nReturned": 101,
                    "executionTimeMillisEstimate": 0,
                    "works": 168,
                    "advanced": 101,
                    "needTime": 67,
                    "needYield": 0,
                    "saveState": 0,
                    "restoreState": 0,
                    "isEOF": 0,
                    "docsExamined": 168,
                    "alreadyHasObj": 0,
                    "inputStage": {
                        "stage": "IXSCAN",
                        "nReturned": 168,
                        "executionTimeMillisEstimate": 0,
                        "works": 168,
                        "advanced": 168,
                        "needTime": 0,
                        "needYield": 0,
                        "saveState": 0,
                        "restoreState": 0,
                        "isEOF": 0,
                        "keyPattern": {
                            "created_at": -1
                        },
                        "indexName": "created_at_-1",
                        "isMultiKey": false,
                        "multiKeyPaths": {
                            "created_at": [ ]
                        },
                        "isUnique": false,
                        "isSparse": false,
                        "isPartial": false,
                        "indexVersion": 2,
                        "direction": "forward",
                        "indexBounds": {
                            "created_at": [
                                "[MaxKey, MinKey]"
                            ]
                        },
                        "keysExamined": 168,
                        "seeks": 1,
                        "dupsTested": 0,
                        "dupsDropped": 0
                    }
                }
            },
            {
                "nReturned": 0,
                "executionTimeMillisEstimate": 0,
                "totalKeysExamined": 168,
                "totalDocsExamined": 168,
                "executionStages": {
                    "stage": "SORT",
                    "nReturned": 0,
                    "executionTimeMillisEstimate": 0,
                    "works": 168,
                    "advanced": 0,
                    "needTime": 168,
                    "needYield": 0,
                    "saveState": 131,
                    "restoreState": 131,
                    "isEOF": 0,
                    "sortPattern": {
                        "created_at": -1
                    },
                    "memLimit": 104857600,
                    "type": "simple",
                    "totalDataSizeSorted": 26441633,
                    "usedDisk": false,
                    "inputStage": {
                        "stage": "FETCH",
                        "nReturned": 168,
                        "executionTimeMillisEstimate": 0,
                        "works": 168,
                        "advanced": 168,
                        "needTime": 0,
                        "needYield": 0,
                        "saveState": 131,
                        "restoreState": 131,
                        "isEOF": 0,
                        "docsExamined": 168,
                        "alreadyHasObj": 0,
                        "inputStage": {
                            "stage": "IXSCAN",
                            "nReturned": 168,
                            "executionTimeMillisEstimate": 0,
                            "works": 168,
                            "advanced": 168,
                            "needTime": 0,
                            "needYield": 0,
                            "saveState": 131,
                            "restoreState": 131,
                            "isEOF": 0,
                            "keyPattern": {
                                "server": 1
                            },
                            "indexName": "server_1",
                            "isMultiKey": false,
                            "multiKeyPaths": {
                                "server": [ ]
                            },
                            "isUnique": false,
                            "isSparse": false,
                            "isPartial": false,
                            "indexVersion": 2,
                            "direction": "forward",
                            "indexBounds": {
                                "server": [
                                    "[\"ucs\", \"ucs\"]"
                                ]
                            },
                            "keysExamined": 168,
                            "seeks": 1,
                            "dupsTested": 0,
                            "dupsDropped": 0
                        }
                    }
                }
            }
        ]
    },
    "serverInfo": {},
    "ok": 1
}
```



## 十、管理篇

### 1、数据导出与导入

### 2、数据备份与恢复

### 3、访问控制

### 4、命令行操作

### 5、进程控制

```
# 查看活动的进程
> db.currentOp()

参数说明：
    Opid: 操作进程号
    Op: 操作类型(查询，更新等)
    Ns: 命名空间, 指操作的是哪个对象
    Query: 如果操作类型是查询的话，这里将显示具体的查询内容
    lockType: 锁的类型，指明是读锁还是写锁
    
# 结束进程
> db.killOp()
```



MongoDB 权威指南：

- 存储过程
- Capped Collection
- GridFS
- MapReduce



性能：

- explain 执行计划
- 优化器 profile



架构：

- Replica Sets 复制集
- Sharding 分片
- Replica Sets + Sharding
