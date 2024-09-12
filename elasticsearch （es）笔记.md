## elasticsearch （es）笔记

#### docker安装

```sh
#新建es的config配置文件夹
mkdir -p /data/elasticsearch/config
#新建es的data目录
mkdir -p /data/elasticsearch/data
#新建es的plugins目录
mkdir -p /data/elasticsearch/plugins
#给目录设置权限
chmod 777 -R /data/elasticsearch
#写入配置到elasticsearch.yml中， 下面的 > 表示覆盖的方式写入， >>表示追加的方式写入，但是要确保http.host: 0.0.0.0不能被写入多次
echo "http.host: 0.0.0.0" >> /data/elasticsearch/config/elasticsearch.yml
#安装es
docker run --name elasticsearch -p 9200:9200 -p 9300:9300 \
    -e "discovery.type=single-node" \
  -e ES_JAVA_OPTS="-Xms128m -Xmx256m" \
  -v /data/elasticsearch/config/elasticsearch.yml:/usr/share/elasticsearch/config/elasticsearch.yml \
  -v /data/elasticsearch/data:/usr/share/elasticsearch/data \
  -v /data/elasticsearch/plugins:/usr/share/elasticsearch/plugins \
  -d elasticsearch:7.10.1
# 安装kibana ip与虚拟机一致 版本与es一致
docker run -d --name kibana -e ELASTICSEARCH_HOSTS="http://192.168.65.131:9200" -p 5601:5601 kibana:7.10.1
```

#### 基础概念

|          | mysql    | es                                                  |
| -------- | -------- | --------------------------------------------------- |
| 库       | database | 新版无此概念                                        |
| 表       | table    | index (新版type为固定值_doc)                        |
| 行(记录) | row      | document (json构成)                                 |
| 列(字段) | column   | field                                               |
| 容器     | schema   | mapping                                             |
| 语句     | sql      | dsl(描述性结构化语言 descriptor structure language) |

#### 文档API

##### PUT

`PUT`修改数据 不存在时创建 需指定id 修改是直接覆盖

`_doc` 是文档的默认类型标识符,表示一个存储的单个文档

```
PUT demo/_doc/1
{
	"name":"test",
	"age":15,
	"favorite":[
		"ball","badminton"
	]
}
```

response

```json
{
  "_index" : "demo", //所在索引
  "_type" : "_doc", //文档
  "_id" : "1",  //指定的id
  "_version" : 1, //版本号 有可能重复 
  "result" : "created",  //存在是为updated
  "_shards" : { //分布式切片
    "total" : 2,
    "successful" : 1,
    "failed" : 0
  },
  "_seq_no" : 0, //实现乐观锁
  "_primary_term" : 1
}

```



##### POST

`post` id由es自动生成，不需要手动指定, 也可手动指定 存在也会更新 但是是**直接替换**

```
POST demo/_doc
{
	"name":"demo"
}
```

```json	
{
  "_index" : "demo",
  "_type" : "_doc",
  "_id" : "_wKr35EBVWJ1fyqAl5ZP", //自动生成
  "_version" : 1,
  "result" : "created",
  "_shards" : {
    "total" : 2,
    "successful" : 1,
    "failed" : 0
  },
  "_seq_no" : 2,
  "_primary_term" : 1
}

```

post+ _create 没有则创建 有则报错

```
POST demo/_create/1
{
	"name":"demo"
}
```

```json
{
  "error" : {
    "root_cause" : [
      {
        "type" : "version_conflict_engine_exception",
        "reason" : "[1]: version conflict, document already exists (current version [2])",
        "index_uuid" : "on-aEnb9QEWpGe7PxG9gxA",
        "shard" : "0",
        "index" : "demo"
      }
    ],
    "type" : "version_conflict_engine_exception",
    "reason" : "[1]: version conflict, document already exists (current version [2])",
    "index_uuid" : "on-aEnb9QEWpGe7PxG9gxA",
    "shard" : "0",
    "index" : "demo"
  },
  "status" : 409
}

```



使用`_updated`在已有的字段上新增

```
POST demo/_update/1
{
  "doc": {  //必须有，会检查是否发生变化 无变化并不会执行 version和seq_no不会变
    	"demo":"sss"
  }

}
```





##### GET

```
GET _cat/indices //查询索引
GET demo //具体某一个索引
```

```
GET demo/_doc/1 	//获取id为1的文档信息
GET demo/_source/1  //直接获取源数据
```

```json
{
  "_index" : "demo",
  "_type" : "_doc",
  "_id" : "1",
  "_version" : 2,
  "_seq_no" : 1,
  "_primary_term" : 1,
  "found" : true,  //是否找到
  "_source" : {    //数据
    "name" : "test",
    "age" : 16,
    "favorite" : [
      "ball",
      "badminton"
    ]
  }
}
---
{
  "name" : "test",
  "age" : 16,
  "favorite" : [
    "ball",
    "badminton"
  ]
}
```



##### DELETE

```
DELETE demo/_doc/1  //删除id为1的
DELETE demo //删除索引
```

```json
{
  "_index" : "demo",
  "_type" : "_doc",
  "_id" : "1",
  "_version" : 8,
  "result" : "deleted",
  "_shards" : {
    "total" : 2,
    "successful" : 1,
    "failed" : 0
  },
  "_seq_no" : 10,
  "_primary_term" : 1
}
----
{
  "acknowledged" : true
}

```





#### 查询方法

##### URI简单查询

```
GET _search?q=test  //不加上索引则查询全部索引
GET demo/_search?q=test 
```

```json
{
  "took" : 3,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 1,
      "relation" : "eq" //等值查询
    },
    "max_score" : 0.87546873,
    "hits" : [
      {
        "_index" : "demo",
        "_type" : "_doc",
        "_id" : "1",
        "_score" : 0.87546873,
        "_source" : {
          "name" : "test",
          "age" : 16,
          "favorite" : [
            "ball",
            "badminton"
          ]
        }
      }
    ]
  }
}
```

##### JSON结构体查询 

###### 全文查询

`match_all` 匹配全部

`match` 匹配 会自动分词 然后通过倒排索引进行匹配

`match_phrase` 短语匹配 会分词 但要求全部匹配 且分词后顺序也需保持一致

`multi_match`  多匹配查询 即多个字段的match查询，可以通过`^`提升权重

`query_string`  可以在所有字段中查询 也可以使用`and` `or` 等实现复杂查询

```
GET demo/_search
{
	"query": {
		"match_all": {} //匹配所有
	}
}
----
GET user/_search
{
  "query":{
    "match":{
      "address": "Hutchinson Court" //
    }
  }
}
----
GET user/_search
{
  "query": {
    "match_phrase": {
      "address": "Hutchinson the Court" //
    }
  }
}
----
GET resume/_search
{
  "query": {
    "multi_match": {
      "query": "go",   //查询关键字go
      "fields": ["title^2", "desc"] //从title和desc两个字段中查询,title权重*2
    }
  }
}
----
GET user/_search
{
  "query":{
    "query_string": {
      "default_field": "address", //不指定则所有字段查询
      "query": "Hutchinson AND Court" //等价于 match_phrase
    }
  }
}

```

######  术语查询

`term`不会对查询的信息进行分词处理 要求直接匹配

**es会对写入的数据和查询数据进行分词+归一化**，此时的term如果查询的是会被分词的内容，那么是**没有结果**的，因为term会直接去es的表里查，不经过分词处理 但是表里是分过词的

可以对有keyword类型的字段进行查询(解释在mapping映射)



```sh
GET user/_search
{
  "query": {
    "term": {
       //es将这个存储为了 "671" "bristol" "street" 三个词，并不存在"671 bristol street"
      "address": "671 bristol street"
      
      "address": "Street" //es存储的只有street 即归一化后的词，不存在Street
      "address": "street" //与es存储的一致
    }
  }
}

GET user/_search
{
  "query": {
    "term": {
      "address.keyword": "641 Royce Street"
    }
  }
}
```

`range` 范围查询

```sh
GET user/_search
{
  "query":{
    "range": {
      "age": {
        "gte": 20,
        "lte": 30
      }
    }
  }
}
```

`exists`存在查询，是否存在某个字段

```
GET user/_search
{
  "query": {
    "exists": {
      "field": "test_field"
    }
  }
}
```

`fuzzy`模糊查询，查询在设定字符串距离内的所有内容

```go
GET user/_search
{
  "query": {
    "match": {
      "address": {
        "query": "Midison streat",  // Midison street 相差1  midison strea也相差1
        "fuzziness": 1	//字符串相差距离为1
      }
    }
  }
}
```

###### 复合查询

`bool`查询 包含`must` `should` `must_not`  `filter`四类

> 1. must: 必须匹配,查询上下文,加分
> 2. should: 应该匹配,查询上下文,加分
> 3. must_not: 必须不匹配,过滤上下文,过滤
> 4. filter: 必须匹配,过滤上下文,过滤

```sh
GET user/_search
{
  "query": {
    "bool": {
      "must": [
        {
          "term": {
            "state" : "va"
          }
        }
      ],
      "should": [
        {
          "range": {
            "age": {
              "gte":20,
              "lte": 25
            }
          }
        }
      ],
      "must_not": [
        {
          "term": {
             "city" : "salix"
          }
        }
      ],
      "filter": [
        {
          "term": {
            "address": "street"
          }
        }
      ]
    }
  }
}
```



#### mapping 映射

类型`text` 和 `keyword`的区别：

>1. es自动推断时会为每个是text类型的field添加一个keyword类型
>2. text类型会进行分词存储，而keyword类型不会进行分词，直接原文存储
>3. 查询时可以通过field.keyword来完成一句话的搜索



##### 倒排索引

倒排索引，即在对文本进行分词后得到对应分词集合中的id 查询时直接获取对应词的集合的信息

```Go
package main

import (
	"fmt"
	"sort"
	"strings"
)

// 实现倒排索引
type node struct {
	Id  int
	str string
}

func main() {

	idx := make(map[string][]int)

	natural := []node{
		{1, "hello world"},
		{2, "hello test"},
		{3, "good world"},
	}
	for _, v := range natural {
		//分词
		s := strings.Split(v.str, " ")
		for _, vv := range s {
			//词被添加过
            if _, ok := idx[vv]; ok {
				//当前id已经存过
                if i := sort.SearchInts(idx[vv], v.Id); i != len(idx[vv]) {
					continue
				}
			}
			idx[vv] = append(idx[vv], v.Id)
		}

	}
	for k, v := range idx {
		fmt.Println(k, v)
	}

}

```



#### 批量添加 & 批量查询

批量添加

```
POST _bulk //路径中有索引可以省略索引
{ "index" : { "_index" : "test", "_id" : "1" } }
{ "field1" : "value1" }
{ "delete" : { "_index" : "test", "_id" : "2" } }
{ "create" : { "_index" : "test", "_id" : "3" } }
{ "field1" : "value3" }
{ "update" : {"_id" : "1", "_index" : "test"} }
{ "doc" : {"field2" : "value2"} }

//路径中有索引可以省略索引
POST indexName/_bulk
{"index":{"_id":"1"}}
{"field":"val1"}
```

批量查询

```
POST  _mget
{
	"docs":[
		{
			"_index":"demo",
			"id":"1"
		}
	]

}
```

