##1. 安装 
输入地址：http://localhost:9200，测试结果
```
{
  "name" : "LAPTOP-TI7HRI2R",
  "cluster_name" : "elasticsearch",
  "cluster_uuid" : "tfoiv_YgTDKbtuGCQPhgQA",
  "version" : {
    "number" : "7.9.3",
    "build_flavor" : "default",
    "build_type" : "zip",
    "build_hash" : "c4138e51121ef06a6404866cddc601906fe5c868",
    "build_date" : "2020-10-16T10:36:16.141335Z",
    "build_snapshot" : false,
    "lucene_version" : "8.6.2",
    "minimum_wire_compatibility_version" : "6.8.0",
    "minimum_index_compatibility_version" : "6.0.0-beta1"
  },
  "tagline" : "You Know, for Search"
}
```

#####问题的解决
1. Elasticsearch 是使用 java 开发的，且 7.X 版本的 ES 需要 JDK 版本 1.8 以上，默认安装包带有 jdk 环境，如果系统配置 JAVA_HOME，那么使用系统默认的 JDK，如果没有配置使用自带的 JDK，一般建议使用系统配置的 JDK。
2. 双击启动窗口闪退，通过路径访问追踪错误，如果是“空间不足”，请修改
config/jvm.options 配置文件
```
# 设置 JVM 初始内存为 1G。此值可以设置与-Xmx 相同，以避免每次垃圾回收完成后 JVM 重新分配内存
# Xms represents the initial size of total heap space
# 设置 JVM 最大可用内存为 1G
# Xmx represents the maximum size of total heap space
-Xms1g
-Xmx1g

```
##2. 数据格式
![es和mysql对比](https://upload-images.jianshu.io/upload_images/9049859-fe1734934f416803.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

ES 里的 Index 可以看做一个库，而 Types 相当于表，Documents 则相当于表的行。
这里 Types 的概念已经被逐渐弱化，Elasticsearch 6.X 中，一个 index 下已经只能包含一个type，Elasticsearch 7.X 中, Type 的概念已经被删除了。

##3. HTTP 操作
#####注意事项:
put操作是幂等性操作
post操作是非幂等性

#####创建索引
PUT 请求 ：http://127.0.0.1:9200/shopping
#####查看所有索引
GET 请求 ：http://127.0.0.1:9200/_cat/indices?v
![返回参数说明](https://upload-images.jianshu.io/upload_images/9049859-c39b346002db1a2b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#####创建文档
POST 请求 ：http://127.0.0.1:9200/shopping/_doc
此处发送请求的方式必须为 POST，不能是 PUT，否则会发生错误

如果想要自定义唯一性标识，需要在创建时指定：http://127.0.0.1:9200/shopping/_doc/1
#####查看文档
查看文档时，需要指明文档的唯一性标识，类似于 MySQL 中数据的主键查询

GET 请求 ：http://127.0.0.1:9200/shopping/_doc/1
#####修改文档
POST 请求 ：http://127.0.0.1:9200/shopping/_doc/1

#####修改字段
POST 请求 ：http://127.0.0.1:9200/shopping/_update/1

#####删除文档
删除一个文档不会立即从磁盘上移除，它只是被标记成已删除（逻辑删除）
DELETE 请求 ：http://127.0.0.1:9200/shopping/_doc/1

#####条件删除文档
POST 请求 ：http://127.0.0.1:9200/shopping/_delete_by_query

#####查询语法
1. 分页查询
```
GET test/doc/_search
{
  "query": {
    "match_phrase_prefix": {
      "name": "wang"
    }
  },
  "from": 0,
  "size": 1
  "_score":["title"]
}
```
2. 多条件查询
```
GET test/doc/_search
{
  "query": {
    "bool": {
      "must": [
        {
          "match": {
            "name": "wanggfei"
          }
        },{
          "match": {
            "age": 25
          }
        }
      ]
    }
  }
}
```
3. 范围查询
filter(条件过滤查询，过滤条件的范围用range表示gt表示大于、lt表示小于、gte表示大于等于、lte表示小于等于)
```
GET test/doc/_search
{
  "query": {
    "bool": {
      "must": [
        {
          "match": {
            "name": "wangjifei"
          }
        }
      ],
      "filter": {
        "range": {
          "age": {
            "gte": 10,
            "lt": 27
          }
        }
      }
    }
  }
}
```
4. 查询结果过滤
```
GET test3/doc/_search
{
  "query": {
    "match": {
      "name": "顾"
    }
  },
  "_source": ["name","age"]
}
```
5. 全文检索
使用match是把中文进行拆词,使用的是倒排索引,要想使用完全匹配就应该使用match_phrase
```
GET test1/doc/_search
{
  "query":{
    "match_phrase": {
      "title": "中国"
    }
  }
}
```
```
GET test1/doc/_search
{
  "query":{
    "match_phrase": {
      "title": {
        "query": "中国世界",
        "slop":2
      }
    }
  }
}
```
我们搜索中国和世界这两个指定词组时，但又不清楚两个词组之间有多少别的词间隔。那么在搜的时候就要留有一些余地。这时就要用到了slop了。相当于正则中的中国.*?世界。这个间隔默认为0

6. 高亮显示
```
GET test3/doc/_search
{
  "query": {
    "match": {
      "name": "顾老二"
    }
  },
  "highlight": {
    "fields": {
      "name": {}
    }
  }
}
```
7. 聚合操作
```
GET zhifou/doc/_search
{
  "query": {
    "match": {
      "from": "gu"
    }
  },
  "aggs": {
    "my_avg": { //取的别名
      "avg": {  //分组操作
        "field": "age"  //需要分组的字段
      }
    }
  },
  "_source": ["name", "age"]  //显示想要的数据
}
```
```
GET zhifou/doc/_search
{
  "query": {
    "match": {
      "from": "gu"
    }
  },
  "aggs": {
    "my_avg": {
      "avg": {
        "field": "age"
      }
    }
  },
"size" :0 // 不显示明细数据了
}
```
```
GET zhifou/doc/_search
{
  "size": 0, 
  "query": {
    "match_all": {}
  },
  "aggs": {
    "age_group": {
      "range": {   //分组
        "field": "age",
        "ranges": [
          {
            "from": 15,
            "to": 20
          },
          {
            "from": 20,
            "to": 25
          },
          {
            "from": 25,
            "to": 30
          }
        ]
      }
    }
  }
}
```
```
GET zhifou/doc/_search
{
  "size": 0, 
  "query": {
    "match_all": {}
  },
  "aggs": {
    "age_group": {
      "range": {
        "field": "age",
        "ranges": [
          {
            "from": 15,
            "to": 20
          },
          {
            "from": 20,
            "to": 25
          },
          {
            "from": 25,
            "to": 30
          }
        ]
      },
      "aggs": {
        "my_avg": {
          "avg": {  //在aggs的自定义别名age_group中，使用range来做分组，field是以age为分组，分组使用ranges来做，from和to是范围,对每个小组内的数据做平均年龄处理
            "field": "age"
          }
        }
      }
    }
  }
}
```
8. Mappings
映射就是在创建索引的时候，有更多定制的内容，更加的贴合业务场景。
用来定义一个文档及其包含的字段如何存储和索引的过程。
index属性默认为true，如果该属性设置为false，那么，elasticsearch不会为该属性创建索引，也就是说无法当做主查询条件。

>参考文档:https://www.cnblogs.com/xiohao/p/12970224.html
感觉es的查询语法挺多的,但是我们不需要去刻意的记忆,要用的时候直接去查文档即可,前提是你得知道有哪些查询语法,以及你想要的数据是什么
