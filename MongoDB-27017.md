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
  > mongod --dapath=D:\MongoDB-5.0\data --logpath=D:\MongoDB-5.0\log\mongod.log --install
  
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
#  2 插入数据时，若 <collection> 不存在，则会自动创建
> db.<collection>.insertOne( {"data": "test"} )

# 新增字段 
#  更新数据时，若字段不存在，则自动新增
> db.<collection>.updateMany(
	{ },
	{ $set: { create_at: new Date() } }
)
{ "acknowledged" : true, "matchedCount" : 1, "modifiedCount" : 1 }

# 删除字段
> db.<collection>.updateMany(
	{ },
	{ $unset: { create_at: "" } }
)
{ "acknowledged" : true, "matchedCount" : 1, "modifiedCount" : 1 }

# 删除集合
> db.<collection>.drop()
true
```



## 三、MongoDB DML（CRUD）

### 1、Insert

```shell
# ============
#  插入单个文档
# ============
db.<collection>.insertOne( obj, <optional params> )

 - 可选参数：w, wtimeout, j

# ============
#  插入多个文档
# ============
db.<collection>.insertMany( [objects], <optional params> )

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
db.<collection>.insertMany([
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
db.<collection>.find([query],[fields])

 - query   是一个可选的查询过滤器。
 - fields  是要返回的可选字段集。

# ===========================
#  SELECT * FROM collection
# ===========================
db.<collection>.find( {} )

# ============
#  相等条件查询
# ============

# ... WHERE item = "oblong"
db.<collection>.find( { item: "oblong" } )

# ==========
#  AND 查询
# ==========

# ... WHERE item = "oblong" AND status = 2
db.<collection>.find( { item: "oblong", status: 2 } )

# ==============
#  $or  OR 查询
# ==============

# ... WHERE item = "oblong" OR status = 0
db.<collection>.find( { $or: [ { item: "oblong" }, { status: 0 } ] } } )

# ===========================
#  $eq $ne $gt $lt $gte $lte
#           范围查询
# ===========================

# ... WHERE status = 1
db.<collection>.find( { status: { $eq: 1 } } )
# ... WHERE status <> 1
db.<collection>.find( { status: { $ne: 1 } } )
# ... WHERE status > 1
db.<collection>.find( { status: { $gt: 1 } } )
# ... WHERE status < 1
db.<collection>.find( { status: { $lt: 1 } } )
# ... WHERE status >= 1
db.<collection>.find( { status: { $gte: 1 } } )
# ... WHERE status <= 1
db.<collection>.find( { status: { $lte: 1 } } )

# ==================
#  $in   IN 查询
#  $nin  NOT IN 查询
# ==================

# 注：对同一字段进行相等性检查时，使用 $in 运算而不是 $or 运算
# ... WHERE status IN (1, 2, 3)
db.<collection>.find( { status: { $in: [ 1, 2, 3 ] } } )

# ===============
#  $mod  取模运算
# ===============
# 查询 status % 3 == 1 的数据
db.<collection>.find( { status: { $mod: [ 3, 1 ] } } )


# ===================
#  $regex  LIKE 查询
# ===================

# ... WHERE item LIKE "%cir%"
db.<collection>.find( { item: /cir/ } )
# ... WHERE item LIKE "cir%"
db.<collection>.find( { item: /^cir/ } )
# ... WHERE item LIKE "%cir"
db.<collection>.find( { item: /cir$/ } )
# or
db.<collection>.find( { item: { $regex: "cir" } } )
db.<collection>.find( { item: { $regex: "^cir" } } )
db.<collection>.find( { item: { $regex: "cir$" } } )

# ==================
#  AND + OR 混合查询
# ==================
# ... WHERE status > 0 AND ( item LIKE "c%" OR item LIKE "s%" ) 
db.<collection>.find( {
    status: { $gt: 0 },
    $or: [ { item: /^c/ }, { item: /^s/ } ]
} )

# ======================================================================== #
# ===================== db.<collection>.find().修饰语 ===================== #
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
db.<collection>.count()

# SELECT COUNT(name) FROM <collection>
db.<collection>.count( { name: { $exists: true } } )

# SELECT COUNT(*) FROM <colletion> WHERE status > 0
db.<collection>.count( { status: { $gt: 0 } } )

# ----- UNCLEAR -----
# db.mycoll.countDocuments( query = {}, <optional params> ) 
# - 可选参数：limit, skip, hint, maxTimeMS

# ==============================
#        LIMIT & OFFSET
#  db.mycoll.find(...).limit(n)
#  db.mycoll.find(...).skip(n)
# ==============================

# 限制条数
# SELECT * FROM <collection> LIMIT 1 
db.<collection>.find().limit(1)
# or
db.<collection>.findOne()

# 分页（page=2 & size=5）
# SELECT * FROM <collection> OFFSET 5 LIMIT 1 
db.collection.find().skip(5).limit(1)

# ===============================
#            ORDER BY
#  db.mycoll.find(...).sort(...)
# ===============================

# 升序
# SELECT * FROM <collection> ORDER BY id ASC
db.<collection>.find().sort( { _id: 1 } )

# 降序
# SELECT * FROM <collection> ORDER BY id DESC
db.<collection>.find().sort( { _id: -1 } )

# 多个排序条件
# SELECT * FROM <collection> ORDER BY status ASC, id DESC
db.<collection>.find().sort( { status: 1, _id: -1 } )

# ==========
#  DISTINCT
# ==========
db.mycoll.distinct( key, query, <optional params> )

# SELECT DISTINCT(status) FROM <collection>
> db.<collection>.distinct("status")
[ 0, 1, 2, 3 ]

# SELECT DISTINCT(status) FROM <collection>
> db.<collection>.aggregate( [ { $group : { _id : "$status" } } ] )
{ "_id" : 2 }
{ "_id" : 1 }
{ "_id" : 3 }
{ "_id" : 0 }

# =========
#  EXPLAIN 
# =========

# EXPLAIN SELECT * FROM <collection>
> db.<collection>.find().explain()
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
db.<collection>.find({
	"position": {
		"x": 0.0,
		"y": 0.0,
		"w": 10,
		"h": 10
	}
})

# 顺序不同或缺少字段都无法匹配
db.<collection>.find({
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
db.<collection>.find( { "positoin.h": { $gte: 10 } } )
```

#### （3）[查询数组](https://www.mongodb.com/docs/v5.0/tutorial/query-arrays/)

```shell
# =====================
#  匹配整个数组（相等匹配）
# =====================
# 强制匹配所有元素和顺序
db.<collection>.find( { "colors": ["red", "green", "blue"] } )

# ===============
#  $all  包含匹配
# ===============
# 不考虑数组中的顺序或其他元素
db.<collection>.find( { "colors": { $all: ["green", "blue"] } } )

# ==============
#  查询元素的数组
# ==============
# 查询的数组字段中至少包含一个指定值
db.<collection>.find( { "colors": "red" } )

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
db.<collection>.find( { "colors.0": "red" } )

# ========================
#  $size  按数组长度查询数组
# ========================
db.<collection>.find( { "colors": { $size: 3 } } )
```

> [查询嵌套文档数组](https://www.mongodb.com/docs/v5.0/tutorial/query-array-of-documents/)

#### （4）[指定查询字段](https://www.mongodb.com/docs/v5.0/tutorial/project-fields-from-query-results/)

```shell
# =======================
#  inclusion 包含指定字段
# =======================
#  默认情况下 `_id` 字段会在匹配文档中返回

# SELECT _id, item, status FROM collection
db.<collection>.find( { }, { item: 1, status: 1 } )

# 排除 `_id` 字段
db.<collection>.find( { }, { item: 1, status: 1, _id: 0 } )

# 返回嵌套文档中的特定字段
db.<collection>.find( { }, { item: 1, "position.x": 1, "position.y": 1 } )

# ==========================
#  $slice  返回数组中的特定元素
# ==========================
# 0：不返回元素，1：第一个元素，...，-1：最后一个元素
db.<collection>.find( { }, { "colors": { $slice: 0 } } )

# =======================
#  exclusion 排除指定字段
# =======================
db.<collection>.find( { }, { position: 0 } )

# 排除嵌套文档中的特定字段
db.<collection>.find( { }, { "positoin.x": 0, "positoin.y": 0 } )


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
db.<collection>.insertOne( { "data": null } )

# ===========
#  相等过滤器
# ===========
# 匹配字段值为 null 或者不包含该字段的文档
db.<collection>.find( { "data": null } )

# ================
#  $type  类型检查
# ================
# 仅匹配字段值为 null 的文档（BSON 类型中的 Null 类型编号为 10）
db.<collection>.find( { "data": { $type: 10 } } )

# ==================
#  $exists  存在检查
# ==================
# true  返回包含指定字段的文档
# false 返回不包含指定字段的文档
db.<collection>.find( { "data": { $exists: true } } )
```

#### （6）[迭代游标](https://www.mongodb.com/docs/v5.0/tutorial/iterate-a-cursor/#manually-iterate-the-cursor)

```shell
# 将查询结果存入 cursor
> var cursor = db.<collection>.find()

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
db.<collection>.updateOne( filter, <update object or pipeline>, <optional params> )

 - 可选参数：upsert, w, wtimeout, j, hint, let

# $set          更新字段值
# $currentDate  将 `lastModified` 字段的值更新为当前日期（字段不存在则自动创建）
db.<collection>.updateOne( 
    { item: "oblong" },
    {
        $set: { "positoin.x": 11, status: "M" },
        $currentDate: { lastModified: true }
    } 
)

# =================
#  更新所有匹配的文档
# =================
db.<collection>.updateMany( filter, <update object or pipeline>, <optional params> )

 - 可选参数：upsert, w, wtimeout, j, hint, let
 
db.<collection>.updateMany( 
    { status: { $gte: 0 } },
    {
        $set: { status: "M" },
        $currentDate: { lastModified: true }
    } 
)


#  替换第一个匹配的文档（旧版）
db.<collection>.replaceOne( filter, replacement, <optional params> )

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

```
unclear
```



### 4、Delete

```shell
# ===================
#  删除第一个匹配的文档
# ===================
db.mycoll.deleteOne( filter, <optional params> )

 - 可选参数：w, wtimeout, j

db.<collection>.deleteOne( { status: "M" } )

# =================
#  删除所有匹配的文档
# =================
db.mycoll.deleteMany( filter, <optional params> )

 - 可选参数：w, wtimeout, j
 
db.<collection>.deleteMany( { status: "M" } ) 


# 删除所有文档
db.<collection>.deleteMany( { } ) 


# 删除匹配的单个文档或所有文档（旧版）
db.<collection>.remove(query)
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
db.<collection>.insertMany([
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
> db.<collection>.find( { $text: { $search: "\"script language\""} } )
Error: error: {
        "ok" : 0,
        "errmsg" : "text index required for $text query",
        "code" : 27,
        "codeName" : "IndexNotFound"
}

# 创建文本索引
# 允许在 title 和 content 字段上进行文本搜索
> db.<collection>.createIndex( { title: "text", content: "text" } )
{
    "numIndexesBefore" : 1,
    "numIndexesAfter" : 2,
    "createdCollectionAutomatically" : false,
    "ok" : 1
}

# 查看集合中的所有索引
> db.<collection>.getIndexes()
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
db.<collection>.find( { $text: { $search: "SQL script language" } } )

# 搜索确切的短语 ""
#  通过用双引号将它们括起来来搜索确切的短语
db.<collection>.find( { $text: { $search: "\"SQL script language\"" } } )

# 排除术语 -
#  通过 `-` 排除一个词
db.<collection>.find( { $text: { $search: "Go C -C#" } } )      # Go
db.<collection>.find( { $text: { $search: "Go C -\"C#\"" } } )  # Go C C++

# =======================================
#  $meta 查询运算符获取每个匹配文档的相关性得分
# =======================================

# 获取相关性得分
db.<collection>.find(
    { $text: { $search: "Go C" } },
    { score: { $meta: "textScore" } }
)

# 按相关顺序排列
db.<collection>.find(
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

原子性：在 MongoDB 中，写操作在单个文档级别上是原子的，即使该操作修改了单个文档中的多个嵌套文档。

多文档事务：当单个写操作（如：db.collection.updateMany()）修改多个文档时，每个文档的修改时原子的，但整个操作不是原子的。

因此，在需要对多个文档进行读写原子性的情况时，应使用 多文档事务。

并发控制：findAndModify 对文档的操作是原子的。

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

### 1、单一目的聚合方法

```shell
# 返回集合或视图中文档的近似计数
db.collection.estimatedDocumentCount()

# 返回集合或视图中的文档数
db.collection.count()

# 返回指定字段具有不同值的文档数组
db.collection.distinct()

```

### 2、集合管道

> 聚合管道由一个或多个处理文档的阶段组成：
>
> - 每个阶段对输入文档执行操作。
> - 从一个阶段输出的文档被传递到下一个阶段。
> - 聚合管道可以返回文档组的结果。

```shell
# 填充数据
db.<collection>.insertMany([
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

#### （1）聚合管道查询

```shell
# 查询各年级的总人数
db.<collection>.aggregate([

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
db.<collection>.aggregate([

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

#### （2）[聚合管道优化](https://www.mongodb.com/docs/v5.0/core/aggregation-pipeline-optimization/)

```

```

#### （3）聚合管道限制

- 结果大小限制：结果集中的每个文档都受 16MB （BSON 文档大小）限制。
- 阶段数限制：单个管道中允许的聚合管道阶段数限制为 1000。（V 5.0 更改）
- 内存限制：每个单独的管道阶段都有 100MB 的 RAM 限制。

#### （4）[聚合管道快速参考](https://www.mongodb.com/docs/v5.0/meta/aggregation-quick-reference/)





## 六、数据模型



## 七、事务



## 八、索引

1、基础索引

2、文档索引

3、组合索引

4、唯一索引

5、强制使用索引

6、删除索引



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
