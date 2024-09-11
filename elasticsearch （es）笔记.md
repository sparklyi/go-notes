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
      "address": "Hutchinson Court" //会自动分词 然后通过倒排索引进行匹配
    }
  }
}
```

 采用的是倒排索引，即在对文本进行分词后得到对应分词集合中的id 

简单实现

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
POST demo/_bulk
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

