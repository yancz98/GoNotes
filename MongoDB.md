## 一、入门

### 1、概述

### 2、安装

[MongoDB 社区服务器下载](https://www.mongodb.com/try/download/community)

- 配置环境变量
- 安装成功

```shell 
# 连接到 MongoDB 服务（mongod.exe）
> mongo.exe
MongoDB shell version v5.0.15
connecting to: mongodb://127.0.0.1:27017/?compressors=disabled&gssapiServiceName=mongodb
Implicit session: session { "id" : UUID("3d81ac46-77e3-4b41-95d6-9a9529fc6bd7") }
MongoDB server version: 5.0.15
......

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
```

### 3、数据类型

BSON 是一种二进制序列化格式，用于在 MongoDB 中存储文档和进行远程过程调用。

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



## 三、MongoDB DML（CURD）

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

# ==============
#  $in  IN 查询
# ==============

# 注：对同一字段进行相等性检查时，使用 $in 运算而不是 $or 运算
# ... WHERE status IN (1, 2, 3)
db.<collection>.find( { status: { $in: [ 1, 2, 3 ] } } )

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

# 聚合管道中的文本搜索
...
```



## 四、聚合操作



## 五、数据模型



## 六、事务



## 七、索引
