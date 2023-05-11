# Elastic Search 7.*

> 搜索引擎的特点：
>
> - 快速
> - 分词
> - 高亮

## 1、安装

### 1.1 安装 ElasticSearch

- 官网：https://www.elastic.co/cn

- 启动 Elasticsearch： `elasticsearch\bin\elasticsearch.bat` 。

- 访问： `localhost:9200` 。

### 1.2 安装 Elasticsearch - head 插件（ES 图形化界面) [下载]( https://github.com/mobz/elasticsearch-head )

1、安装 node.js [官网](https://nodejs.org/en/download/)

2、安装 grunt

```
npm install -g grunt-cli
```

3、将下载的 `elasticsearch-head-master` 文件解压到 `Elasticsearch` 下面。

4、在 `Elasticsearch\elasticsearch-head-master` 下执行 `npm install` 。

5、运行head插件：执行 `grunt server` 或者 `npm run start` 。

6、访问：`http://localhost:9100` 。

7、head 插件连接到 Elastic Search 服务

> 修改配置文件：`elasticsearch\config\elasticsearch.yml`

```
# 在文件末尾加上以下代码：
http.cors.enabled: true 
http.cors.allow-origin: "*"
```

> 重启：`elasticsearch.bat` 后点击连接

### 1.3 获取 Kibana （ES 可视化工具）

1、启动 Kibana： `kibana\bin\kibana.bat` 。

2、访问： `localhost:5601` 。

3、配置：`kibana\config\kibana.yml` 

```
i18n.locale: "zh-CN"	# 中文界面
```



## 2、ElasticSearch 基础

### 2.1 相关术语

- **与关系型数据库对比**

```
DB -> database -> table -> row -> column
ES -> index -> type -> document -> field
```

注：7.* 弃用 type，用内置字段 _doc 代替。原因：同名字段的冲突。

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

- IK Analyzer：中文分词器 [下载](https://github.com/medcl/elasticsearch-analysis-ik/releases)

  ```
  # 使用
  "properties": {
  	"content": {
  		"type": "text",
  		"analyzer": "ik_max_word",
  		"search_analyzer": "ik_smart"
  	}
  }
  
  # 分词参数：（一般分词时用细粒度，查询时用粗粒度）
  ik_smart：粗粒度分词
  ik_max_word：细粒度分词
  ```

  

### 2.2 数据类型

![img](https://img2020.cnblogs.com/blog/1853022/202009/1853022-20200920102635101-1577990369.png) 

说明：

- 字符串类型
  - text：支持分词，分词后索引，支持模糊、精确查询，不支持聚合；
  - keyword：不支持分词，直接索引，只支持精确查询，支持聚合；
  - string：5.* 后弃用，使用 text、keyword代替；

- 整数型

- 浮点型

  

## 3、ES 操作 

### 3.1 索引操作

> 创建索引

```
PUT http://localhost:9200/{index_name}

{
    // 设置
    "settings": {
        "number_of_shards" :   1,  // 分片数
        "number_of_replicas" : 0   // 副本数
    },
    // 映射
    "mappings": {
        // 属性
        "properties": {
            "id": {
                "type": "integer",
                "store": true,
                "index": true
            },
            "name": {
                "type": "text",
                "store": true,
                "index": true
            }
        }
    }
}
```

> mapping 映射说明：

- 字段要包含在 `properties` 属性中定义。
- 字段类型有：text、long



> 修改索引 - 设置

```
PUT http://localhost:9200/{index_name}/_settings

{
    "number_of_replicas": 1
}
```

> 修改索引 - 映射（补充映射）

```
PUT http://localhost:9200/{index_name}/_mappings

{
    // 属性
    "properties": {
        "name": {
            "type": "text",
            "store": true,
            "index": true
        }
    }
}
```

### 3.2 文档操作

> 添加单个文档

```
POST http://localhost:9200/{index_name}/_doc/{id}

{
  "@timestamp": "2099-05-06T16:21:15.000Z",
  "event": {
    "original": "192.0.2.42 - - [06/May/2099:16:21:15 +0000] \"GET /images/bg.jpg HTTP/1.0\" 200 24736"
  }
}
```

说明：当ID不存在时，将创建一条新的文档；当ID存在时，做更新操作；若不指定ID则随机生成。

> 返回值说明：
>
> - _version - 写操作数值会加1（增删改）
> - _shards - 存入分片总数、成功数、失败数

> 添加多个文档

```
PUT logs-my_app-default/_bulk
{ "create": { } }
{ "@timestamp": "2099-05-07T16:24:32.000Z", "event": { "original": "192.0.2.242 - - [07/May/2020:16:24:32 -0500] \"GET /images/hm_nbg.jpg HTTP/1.0\" 304 0" } }
{ "create": { } }
{ "@timestamp": "2099-05-08T16:25:42.000Z", "event": { "original": "192.0.2.255 - - [08/May/2099:16:25:42 +0000] \"GET /favicon.ico HTTP/1.0\" 200 3638" } }

```

> 获取文档

```
GET http://localhost:9200/{index_name}/_doc/{doc_id}
```

> 更新文档

```
PUT http://localhost:9200/{index_name}/_doc/{doc_id}
```

> 删除文档

```
DELETE http://localhost:9200/{index_name}/_doc/{doc_id}
```



### 3.3 搜索

```
GET http://127.0.0.1:9200/{index_name}/_search
```

#### （1）『match_all』匹配所有查询

```json
{
    "query": {
        "match_all": {}
    },
    // 排序
    "sort": [
        { "_id": "asc" }
    ],
    // 查询条数
	"from": 0,
	"size": 10
}
```

> 返回值

```json
{
    "took": 5,
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
        "max_score": 1.0,
        "hits": [
            {
                "_index": "ycz",
                "_type": "_doc",
                "_id": "1",
                "_score": 1.0,
                "_source": {
                    "name": "YCZ-1",
                    "age": 20,
                    "address": "湖北·荆州"
                },
                "sort": [
                    "1"
                ]
            },
            {
                "_index": "ycz",
                "_type": "_doc",
                "_id": "2",
                "_score": 1.0,
                "_source": {
                    "name": "YCZ-2",
                    "age": 21,
                    "address": "湖北·荆州"
                },
                "sort": [
                    "2"
                ]
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

#### （2）获取特定字段

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

#### （3）范围搜索

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

#### （4）『match』查询

> match 会先对字段进行分词操作，然后查询文档。
>
> 大小写不敏感

#### match 匹配一个关键词

> 匹配 `address` 中包含 “湖北” 的文档

```json
{
    "query": {
        "match": {
            "address": "湖北"
        }
    }
}
```

#### mutil_match 指定多个字段

> 任意一个字段匹配就行
>
> 查询所有字段："fields" : ["_all"]

```json
{
	"query": {
    	"multi_match": {
      		"query": "H型",
      		"fields": [
                "artworketd_title",
                "artworketd_tagnames",
                "artworketd_serialnum"
      		]
    	}
  	}
}
```

####  match_phrase 完全匹配

> 必须匹配短语中的所有分词，且位置也要一致

```
{
    "query": {
        "match_phrase": {
        	"artworketd_tagnames": "卫衣 女"
        }
    }
}
```

####  match_phrase_prefix 前缀匹配

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



#### （5） term查询和terms查询 

> 不执行分词器，大写字母无法匹配，适合keyword 、numeric、date 数据类型。

> term 匹配一个关键词

```
{
    "query": {
        "term": {
            "created_time": "2020-12-30"
        }
    }
}
```

> terms 匹配多个关键词

```
{
    "query": {
        "terms": {
            "created_time": ["2020-12-30", "2021-12-30"]
        }
    }
}
```

####  wildcard 通配符查询

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

#### fuzzy 模糊查询

> 模糊查询可以在 match 和 multi_match 查询中使用以便解决拼写的错误
>
> fuzzy 与 term 查询的模糊等价
>
> 匹配包含，不执行分词器，大写字母无法匹配

```
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

#### 高亮显示

```
{
    "query": {
    	"match": {
    		"artworketd_title": "衬衫"
    	}
    },
    "highlight": {
    	"pre_tags": ["<font color=red>"], 
    	"post_tags": ["</font>"],
        "fields": {
        	"artworketd_title": {}
        }
    }
}
```



#### 6、bool 查询（组合查询）

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

#### 7、filter 过滤查询

>  filter是不计算相关性的，同时可以cache。因此，filter速度要快于query 。

```json
{
	"query": {
        "bool": {
            "filter": {
                
            }
        }
    }
}
```



#### 跟踪总点击次数

```
"track_total_hits" : true ,
"track_total_hits" : 100 ,
```



### 3.4 聚合查询