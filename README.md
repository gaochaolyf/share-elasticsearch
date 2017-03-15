# ElasticSearch

@(安装配置)[搜索, Filter]

>ElasticSearch是一个开源的分布式搜索引擎，具备高可靠性，支持非常多的企业级搜索用例。像Solr4一样，是基于Lucene构建的。支持时间时间索引和全文检索。官网：http://www.elasticsearch.org
它对外提供一系列基于java和http的api，用于索引、检索、修改大多数配置。


-------------------

[TOC]

## 安装配置
### Composer相关
将composer.phar下载到当前目录
curl -sS https://getcomposer.org/installer | php

composer require "elasticsearch/elasticsearch:~5.0"

### 安装elasticsearch
brew install elasticsearch

启动 sudo elasticsearch 是错的 elasticsearch是不能用root启动的 要用普通账号

确保装了java jdk 
```dos
gaochaodeMacBook-Pro:~ gaochaolyf$ java -version
java version "1.8.0_101"
Java(TM) SE Runtime Environment (build 1.8.0_101-b13)
Java HotSpot(TM) 64-Bit Server VM (build 25.101-b13, mixed mode)
```

### elasticsearch启动
elasticsearch （后台启动方式为elasticsearch -d）

但是 max file descriptors 最好提高到64000 这个设置看了很久还是没找到
```
gaochaodeMacBook-Pro:~ gaochaolyf$ elasticsearch
[2016-08-04 15:11:09,920][INFO ][node                     ] [Sunstroke] version[2.3.4], pid[2282], build[e455fd0/2016-06-30T11:24:31Z]
[2016-08-04 15:11:09,920][INFO ][node                     ] [Sunstroke] initializing ...
[2016-08-04 15:11:10,372][INFO ][plugins                  ] [Sunstroke] modules [reindex, lang-expression, lang-groovy], plugins [], sites []
[2016-08-04 15:11:10,388][INFO ][env                      ] [Sunstroke] using [1] data paths, mounts [[/ (/dev/disk1)]], net usable_space [179.5gb], net total_space [232.6gb], spins? [unknown], types [hfs]
[2016-08-04 15:11:10,389][INFO ][env                      ] [Sunstroke] heap size [989.8mb], compressed ordinary object pointers [true]
[2016-08-04 15:11:10,389][WARN ][env                      ] [Sunstroke] max file descriptors [10240] for elasticsearch process likely too low, consider increasing to at least [65536]
[2016-08-04 15:11:11,556][INFO ][node                     ] [Sunstroke] initialized
[2016-08-04 15:11:11,556][INFO ][node                     ] [Sunstroke] starting ...
[2016-08-04 15:11:11,616][INFO ][transport                ] [Sunstroke] publish_address {127.0.0.1:9300}, bound_addresses {[fe80::1]:9300}, {[::1]:9300}, {127.0.0.1:9300}
[2016-08-04 15:11:11,620][INFO ][discovery                ] [Sunstroke] elasticsearch_gaochaolyf/MESO2-btRJqTjNCeniyVOw
[2016-08-04 15:11:14,656][INFO ][cluster.service          ] [Sunstroke] new_master {Sunstroke}{MESO2-btRJqTjNCeniyVOw}{127.0.0.1}{127.0.0.1:9300}, reason: zen-disco-join(elected_as_master, [0] joins received)
[2016-08-04 15:11:14,669][INFO ][http                     ] [Sunstroke] publish_address {127.0.0.1:9200}, bound_addresses {[fe80::1]:9200}, {[::1]:9200}, {127.0.0.1:9200}
[2016-08-04 15:11:14,669][INFO ][node                     ] [Sunstroke] started
[2016-08-04 15:11:14,682][INFO ][gateway                  ] [Sunstroke] recovered [0] indices into cluster_state
```

浏览器请求 http://127.0.0.1:9200
```json
{
  "name" : "Sunstroke",
  "cluster_name" : "elasticsearch_gaochaolyf",
  "version" : {
    "number" : "2.3.4",
    "build_hash" : "e455fd0c13dceca8dbbdbb1665d068ae55dabe3f",
    "build_timestamp" : "2016-06-30T11:24:31Z",
    "build_snapshot" : false,
    "lucene_version" : "5.5.0"
  },
  "tagline" : "You Know, for Search”
}
```

### 安装maven
```dos
gaochaodeMacBook-Pro:~ gaochaolyf$ brew install maven
gaochaodeMacBook-Pro:~ gaochaolyf$ mvn -v
Apache Maven 3.3.9 (bb52d8502b132ec0a5a3f4c09453c07478323dc5; 2015-11-11T00:41:47+08:00)
Maven home: /usr/local/Cellar/maven/3.3.9/libexec
Java version: 1.8.0_101, vendor: Oracle Corporation
Java home: /Library/Java/JavaVirtualMachines/jdk1.8.0_101.jdk/Contents/Home/jre
Default locale: zh_CN, platform encoding: UTF-8
OS name: "mac os x", version: "10.11.5", arch: "x86_64", family: "mac"
```

### 下载中分文词库
- 下载：https://github.com/medcl/elasticsearch-analysis-mmseg
- 解压： tar xvf elasticsearch-analysis-mmseg＊＊gz
- cd elasticsearch-analysis-mmseg
- mvn package (编译打包)
编译完成，会在elasticsearch-analysis-mmseg文件夹目录下生成一个target文件夹，
/Users/gaochaolyf/Downloads/elasticsearch-analysis-mmseg-master/target/releases
下会有个编译好的zip压缩包
elasticsearch-analysis-mmseg-1.9.4.zip
### 配置中文搜索
- 进入elasticsearch目录：/usr/local/Cellar/elasticsearch/2.3.4/libexec/plugins文件夹
- 如果没有上面这个文件夹，mkdir plugins生成完成，在plugins文件夹下新建：mkdir  analysis-mmseg
- 将编译好的elasticsearch-analysis-mmseg目录下的打包文件
/Users/gaochaolyf/Downloads/elasticsearch-analysis-mmseg-master/target/releases/elasticsearch-analysis-mmseg-1.9.4.zip
解压并拷贝到elasticsearch的plugins目录下
- 编辑elasticsearch配置文件，进行简单配置：
sudo vim /usr/local/Cellar/elasticsearch/2.3.4/libexec/config/elasticsearch.yml

复制这段代码到此文件下：
```
index:
  analysis: 
    analyzer:
      mmseg_maxword:
        type: custom
        filter:
        - lowercase
        tokenizer: mmseg_maxword
      mmseg_maxword_with_cut_letter_digi:
        type: custom
        filter:
        - lowercase
        - cut_letter_digit
        tokenizer: mmseg_maxword 
```

## 基本概念

### 1.计算集群中的文档数量
http://localhost:9200/_count?pretty

### 2.索引 类型 文档 字段概念描述
>在Elasticsearch中，文档归属于一种类型(type),而这些类型存在于索引(index)中，我们可以画一些简单的对比图来类比传统关系型数据库：
>
Relational DB -> Databases -> Tables -> Rows -> Columns

Elasticsearch -> Indices   -> Types  -> Documents -> Fields

Elasticsearch集群可以包含多个**索引(indices)**（数据库），每一个索引可以包含多个**类型(types)**（表），每一个类型包含多个**文档(documents)**（行），然后每个文档包含多个**字段(Fields)**（列）。

### 3.查询一个type下的所有document
```dos
curl -XPOST http://localhost:9200/index/fulltext/_search  -d'{

    "query" : {

        "match_all" : []

    }

}'
```

## 简单的实例演示

### 1.创建一个索引
curl -XPUT http://localhost:9200/index

### 2.创建一个映射
```
curl -XPOST http://localhost:9200/index/fulltext/_mapping -d'
{
    "fulltext": {
             "_all": {
            "analyzer": "mmseg_maxword",
            "search_analyzer": "mmseg_maxword",
            "term_vector": "no",
            "store": "false"
        },
        "properties": {
            "content": {
                "type": "string",
                "store": "no",
                "term_vector": "with_positions_offsets",
                "analyzer": "mmseg_maxword",
                "search_analyzer": "mmseg_maxword",
                "include_in_all": "true",
                "boost": 8
            }
        }
    }
}'
```
### 3.Indexing一些文档
```
curl -XPOST http://localhost:9200/index/fulltext/1 -d'
{"content":"美国留给伊拉克的是个烂摊子吗"}
'

curl -XPOST http://localhost:9200/index/fulltext/2 -d'
{"content":"公安部：各地校车将享最高路权"}
'

curl -XPOST http://localhost:9200/index/fulltext/3 -d'
{"content":"中韩渔警冲突调查：韩警平均每天扣1艘中国渔船"}
'

curl -XPOST http://localhost:9200/index/fulltext/4 -d'
{"content:"中国驻洛杉矶领事馆遭亚裔男子枪击 嫌犯已自首"}
'
```
### 4.查询及高亮显示 Content：中国
```
curl -XPOST http://localhost:9200/index/fulltext/_search  -d'
{
    "query" : { "term" : { "content" : "中国" }},
    "highlight" : {
        "pre_tags" : ["<tag1>", "<tag2>"],
        "post_tags" : ["</tag1>", "</tag2>"],
        "fields" : {
            "content" : {}
        }
    }
}'
```
下面是查询结果
```json
{
    "took": 14,
    "timed_out": false,
    "_shards": {
        "total": 5,
        "successful": 5,
        "failed": 0
    },
    "hits": {
        "total": 2,
        "max_score": 2,
        "hits": [
            {
                "_index": "index",
                "_type": "fulltext",
                "_id": "4",
                "_score": 2,
                "_source": {
                    "content": "中国驻洛杉矶领事馆遭亚裔男子枪击 嫌犯已自首"
                },
                "highlight": {
                    "content": [
                        "<tag1>中国</tag1>驻洛杉矶领事馆遭亚裔男子枪击 嫌犯已自首 "
                    ]
                }
            },
            {
                "_index": "index",
                "_type": "fulltext",
                "_id": "3",
                "_score": 2,
                "_source": {
                    "content": "中韩渔警冲突调查：韩警平均每天扣1艘中国渔船"
                },
                "highlight": {
                    "content": [
                        "均每天扣1艘<tag1>中国</tag1>渔船 "
                    ]
                }
            }
        ]
    }
}
```

## 遇到的问题

1.连接Client方法的时候 官方文档是这样的

![image](http://note.youdao.com/yws/public/resource/5af0dc5e634038b95922d8f55b58a8b1/xmlnote/19A010D0CAA6455897923CA83E3185A4/7765)

设置了Client客户端的ip和端口 能获取elasticsearch实例
但是操作都报错具体错误为

![image](http://note.youdao.com/yws/public/resource/5af0dc5e634038b95922d8f55b58a8b1/xmlnote/D3A18BD06DBB42DA9DD5A818E7AB97C7/7767)

以为是要设置连接池的问题 然后看了连接池相关的文档 花了比较长时间 最后
```php
use Elasticsearch\ClientBuilder;
// elasticsearch.host: ['192.168.5.51:9200', '192.168.5.84:9200']
$host_url = config_get('elasticsearch.host');
$clientBuilder = ClientBuilder::create();   // Instantiate a new ClientBuilder
$clientBuilder->setHosts($host_url);           // Set the hosts
$this->_client = $clientBuilder->build();   // Build the client object
```
这样写就可以了 就能正常创建索引等

2.进行查询的时候报错 具体的错误是json解析错误 命令行界面能正确获取数据

为什么php调api无法正确获取数据呢

带着这个问题 找到json解析前的数据并打印出来发现为
```json
{
    "took": 2,
    "timed_out": false,
    "_shards": {
        "total": 5,
        "successful": 5,
        "failed": 0
    },
    "hits": {
        "total": 2,
        "max_score": 2.0,
        "hits": [
            {
                "_index": "index",
                "_type": "fulltext",
                "_id": "4",
                "_score": 2.0,
                "_source": {
                    content: "中国驻洛杉矶领事馆遭亚裔男子枪击 嫌犯已自首"
                },
                "highlight": {
                    "content": [
                        "中国驻洛杉矶领事馆遭亚裔男子枪击 嫌犯已自首"
                    ]
                }
            },
            {
                "_index": "index",
                "_type": "fulltext",
                "_id": "3",
                "_score": 0.61370564,
                "_source": {
                    content: "中韩渔警冲突调查：韩警平均每天扣1艘中国渔船"
                },
                "highlight": {
                    "content": [
                        "中韩渔警冲突调查：韩警平均每天扣1艘中国渔船"
                    ]
                }
            }
        ]
    }
}
```
有数据 但是确实是json解析错误 格式不对 找到不对的地方发现

content: "中国驻洛杉矶领事馆遭亚裔男子枪击 嫌犯已自首”

上面这一行的content没引号 正确的为

"content": "中国驻洛杉矶领事馆遭亚裔男子枪击 嫌犯已自首”

因为这个document插入的时候是采用命令行的方式插入的具体为

https://github.com/medcl/elasticsearch-analysis-mmseg 分词库的例子
```dos
curl -XPOST http://localhost:9200/index/fulltext/1 -d'
{content:"美国留给伊拉克的是个烂摊子吗"}
'

curl -XPOST http://localhost:9200/index/fulltext/2 -d'
{content:"公安部：各地校车将享最高路权"}
'

curl -XPOST http://localhost:9200/index/fulltext/3 -d'
{content:"中韩渔警冲突调查：韩警平均每天扣1艘中国渔船"}
'

curl -XPOST http://localhost:9200/index/fulltext/4 -d'
{content:"中国驻洛杉矶领事馆遭亚裔男子枪击 嫌犯已自首"}
'
```

以上的json格式都不对 content都没引号 但是却可以执行并且能插入document，然而从api获取的却是非法的document解析不了从而报错，不知道这个算不算elasticsearch的bug，非法的json格式可以插入，并且不做转化，然而又查询不了，正确的应该是
```dos
curl -XPOST http://localhost:9200/index/fulltext/1 -d'
{"content":"美国留给伊拉克的是个烂摊子吗"}
'

curl -XPOST http://localhost:9200/index/fulltext/2 -d'
{"content":"公安部：各地校车将享最高路权"}
'

curl -XPOST http://localhost:9200/index/fulltext/3 -d'
{"content":"中韩渔警冲突调查：韩警平均每天扣1艘中国渔船"}
'

curl -XPOST http://localhost:9200/index/fulltext/4 -d'
{"content":"中国驻洛杉矶领事馆遭亚裔男子枪击 嫌犯已自首"}
'
```

3.composer依赖注入elasticsearch的时候注意要github账号登录与授予权限，也有可能composer不稳定很慢

4.启动elasticsearch命令是
elasticsearch -d 后台方式启动 elasticsearch也可以，不可以使用sudo elasticsearch 因为elasticsearch无法用root账号启动
```bash
http://localhost:9200/index/fulltext -d'{

    "query" : {

        "match" : {"_all":"中国"}

    }

}'
```

## 深入研究及高级搜索

1.bulk 通过data.json文件批量导入数据

http://www.cnblogs.com/xing901022/p/5339419.html

2.elasticsearch的查询器query与过滤器filter的区别

 filter查询速度比query确实快不少，也证明filter走了cache缓存。 但是如果我们对比下命中的数目，query要比filter要多一点，换句话说，更加的精准。
 http://www.tuicool.com/articles/7rqAFne

3.range
```
$body = '{
            "query": {
                "bool": {
                    "must": [
                        {
                            "term": {
                                "Seat": 6
                            }
                           
                        },
                        {
                            "range": {
                                "PurchasePrice": {
                                    "gte":  128000
                                }
                            }
                        }
                    ]
                }
            }
         }';
```

4.terms的情况匹配多个
```
'{
    "filter": {
        "bool": {
            "must": [
                {
                    "terms": {
                        "Seat": [ "4", "6" ]
                    }
                   
                },
                {
                    "range": {
                        "PurchasePrice": {
                            "gte":  128000
                        }
                    }
                }
            ]
        }
    }
 }';
```
gt :: 大于

gte:: 大于等于

lt :: 小于

lte:: 小于等于

term 主要用于精确匹配哪些值，比如数字，日期，布尔值或 not_analyzed的字符串(未经分析的文本数据类型)

5.mapping相关

查看一个索引的mapping http://127.0.0.1:9200/insurance/_mapping?pretty

pretty顾名思义，使其以美观的形式打印出JSON数据

ES的mapping非常类似于静态语言中的数据类型：声明一个变量为int类型的变量， 以后这个变量都只能存储int类型的数据。同样的， 一个number类型的mapping字段只能存储number类型的数据。

同语言的数据类型相比，mapping还有一些其他的含义，mapping不仅告诉ES一个field中是什么类型的值， 它还告诉ES如何索引数据以及数据是否能被搜索到。

当你的查询没有返回相应的数据， 你的mapping很有可能有问题。当你拿不准的时候， 直接检查你的mapping。
一个type（表）的mapping，各个字段的类型:
```
mappings: {  
    item: {  
        properties: {  
            description: {  
                type: string  
            }  
            name: {  
                type: string  
            }  
        }  
    }  
}  
```
6.分词查询

我们可以在做查询的时候键入_analyze关键字查看分析的过程
```
curl -X GET "http://localhost:9200/insurance/_analyze?analyzer=standard&pretty=true" -d "凯迪拉克"  
// 结果：
{
  "tokens" : [ {
    "token" : "凯",
    "start_offset" : 0,
    "end_offset" : 1,
    "type" : "<IDEOGRAPHIC>",
    "position" : 0
  }, {
    "token" : "迪",
    "start_offset" : 1,
    "end_offset" : 2,
    "type" : "<IDEOGRAPHIC>",
    "position" : 1
  }, {
    "token" : "拉",
    "start_offset" : 2,
    "end_offset" : 3,
    "type" : "<IDEOGRAPHIC>",
    "position" : 2
  }, {
    "token" : "克",
    "start_offset" : 3,
    "end_offset" : 4,
    "type" : "<IDEOGRAPHIC>",
    "position" : 3
  } ]
}
```
7.重要的查询过滤语句

http://es.xiaoleilu.com/054_Query_DSL/70_Important_clauses.html

8.bool 查询

>bool 查询与 bool 过滤相似，用于合并多个查询子句。不同的是，bool 过滤可以直接给出是否匹配成功， 而bool 查询要计算每一个查询子句的 _score （相关性分值）。

must:: 查询指定文档一定要被包含。

must_not:: 查询指定文档一定不要被包含。

should:: 查询指定文档，有则可以为文档相关性加分。

以下查询将会找到 title 字段中包含 "how to make millions"，并且 "tag" 字段没有被标为 spam。 如果有标识为 "starred" 或者发布日期为2014年之前，那么这些匹配的文档将比同类网站等级高：

```
{

    "bool": {

        "must":     { "match": { "title": "how to make millions" }},

        "must_not": { "match": { "tag":   "spam" }},

        "should": [

            { "match": { "tag": "starred" }},

            { "range": { "date": { "gte": "2014-01-01" }}}

        ]

    }

}
```
提示： 如果bool 查询下没有must子句，那至少应该有一个should子句。但是 如果有must子句，那么没有should子句也可以进行查询。

9.验证查询 

http://es.xiaoleilu.com/054_Query_DSL/80_Validating_queries.html

只支持查询语句的验证 explain还返回错误，比如我解析一个filter的语句是不支持的
```
curl -X GET http://localhost:9200/_validate/query?explain -d'{ "filter": { "bool": { "must": [ { "terms": { "Seat": [ "4", "6" ] } }, { "range": { "PurchasePrice": { "gte": 128000 } } } ] } } }’
// 结果报错：
{
    "valid": false,
    "_shards": {
        "total": 3,
        "successful": 3,
        "failed": 0
    },
    "explanations": [
        {
            "index": "index",
            "valid": false,
            "error": "org.elasticsearch.index.query.QueryParsingException: request does not support [filter]"
        },
        {
            "index": "insurance",
            "valid": false,
            "error": "org.elasticsearch.index.query.QueryParsingException: request does not support [filter]"
        },
        {
            "index": "my_index",
            "valid": false,
            "error": "org.elasticsearch.index.query.QueryParsingException: request does not support [filter]"
        }
    ]
}
// 请求不支持filter
request does not support [filter]
curl -X GET http://localhost:9200/_validate/query?explain -d'{
   "query": {
      "match" : {
         "tweet" : "really powerful"
      }
   }
}'
```
10.排序是默认_score浮点型数据 从大到小相关性排列的

>过滤语句与 _score 没有关系，但是有隐含的查询条件 match_all 为所有的文档的 _score 设值为 1。 也就相当于所有的文档相关性是相同的。

- 10.1 多级排序
```
        $body = [
            "query" => [
                "match" => ["_all" => ["query" => "途观", "operator" => "and"]]
            ],
//            "sort"  => [
//                "PurchasePrice" => ["order" => "desc"]
//            ]
//        先根据座位数从高到低排序,座位数相同的按照价格从高到低
            "sort"  => [
                ["Seat" => ["order" => "desc"]],
                ["PurchasePrice" => ["order" => "desc"]]
            ]
        ];
```
- 10.2 为多值字段排序
在为一个字段的多个值进行排序的时候， 其实这些值本来是没有固定的排序的
一个拥有多值的字段就是一个集合，
你准备以哪一个作为排序依据呢？
对于数字和日期，你可以从多个值中取出一个来进行排序，你可以使用min, max, avg 或 sum这些模式。 比说你可以在 dates 字段中用最早的日期来进行排序：
```
"sort": {
    "dates": {
        "order": "asc",
        "mode":  "min"
    }
}
```

11._source 相当于SQL语句中的筛选field字段

在搜索请求中你可以通过限定 _source 字段来请求指定字段：
```
GET /_search
{
    "query":   { "match_all": {}},
    "_source": [ "title", "created" ]
}
```
这些字段会从 _source 中提取出来，而不是返回整个 _source 字段。

12.ik分词替换掉标准的中文分词
```
curl -X GET "http://localhost:9200/insurance/_analyze?analyzer=standard&pretty=true" -d "凯迪拉克"  
{
  "tokens" : [ {
    "token" : "凯",
    "start_offset" : 0,
    "end_offset" : 1,
    "type" : "<IDEOGRAPHIC>",
    "position" : 0
  }, {
    "token" : "迪",
    "start_offset" : 1,
    "end_offset" : 2,
    "type" : "<IDEOGRAPHIC>",
    "position" : 1
  }, {
    "token" : "拉",
    "start_offset" : 2,
    "end_offset" : 3,
    "type" : "<IDEOGRAPHIC>",
    "position" : 2
  }, {
    "token" : "克",
    "start_offset" : 3,
    "end_offset" : 4,
    "type" : "<IDEOGRAPHIC>",
    "position" : 3
  } ]
}

$body = '{
            "query" : {
                "filtered" : {
                    "query" : {
                        "match" : {"BrandName" : {"query": "凯迪拉克", "operator" : "and"}}
                    },
                    "filter" : {
                        "and":[
                            {
                                "range": {
                                  "PurchasePrice": {
                                    "gte": 200000,
                                    "lte": 400000
                                  }
                                }
                            },
                            {
                                "terms" : {
                                    "Seat" : ["4", "5"]
                                }
                            }
                        ]
                    }
                }
            },
            "sort" : {
                "Seat" : {"order" : "desc"},
                "PurchasePrice" : {"order" : "desc"}
            }
        }';
```
上面这个相对复杂的查询，因为分词原因凯迪拉克会搜索出奥迪，别克车型，虽然加上operator => and会解决这一问题，但如果我相同时搜索凯迪拉克和大众呢？

{"query": "凯迪拉克 大众", "operator" : "and”} 无法搜索结果因为框里的已经全部归为一体，

{"query": "凯迪拉克 大众”} 也不行，会搜索出奥迪，别克等多余车型，故而需要ik分词，使凯迪拉克作为一个独立的词语

跟之前安装中文分词过程一样，教程 http://keenwon.com/1404.html
https://github.com/medcl/elasticsearch-analysis-ik

但是这次重新启动，报错了信息，Plugin [analysis-ik] is incompatible with Elasticsearch [2.3.4]. Was designed for version [2.3.5]
版本不对，我下载的ik分词是支持Elasticsearch 2.3.5的而自己电脑中的是2.3.4的，在git找到2.3.4的分支，下载重新安装一遍，重启成功。
```
curl -X GET "http://localhost:9200/insurance/_analyze?analyzer=ik&pretty=true" -d "凯迪拉克 大众"
{
  "tokens" : [ {
    "token" : "凯迪拉克",
    "start_offset" : 0,
    "end_offset" : 4,
    "type" : "CN_WORD",
    "position" : 0
  }, {
    "token" : "凯",
    "start_offset" : 0,
    "end_offset" : 1,
    "type" : "CN_WORD",
    "position" : 1
  }, {
    "token" : "迪",
    "start_offset" : 1,
    "end_offset" : 2,
    "type" : "CN_WORD",
    "position" : 2
  }, {
    "token" : "拉",
    "start_offset" : 2,
    "end_offset" : 3,
    "type" : "CN_CHAR",
    "position" : 3
  }, {
    "token" : "克",
    "start_offset" : 3,
    "end_offset" : 4,
    "type" : "CN_WORD",
    "position" : 4
  }, {
    "token" : "大众",
    "start_offset" : 5,
    "end_offset" : 7,
    "type" : "CN_WORD",
    "position" : 5
  } ]
}
```
又引申出问题是，因为还是存在单个分词，只是权重下降，_score匹配度凯迪拉克的在前，如果针对搜索的结果，进行其他字段的排序而非_score的排序，可能会出现不够精确的可能，除非把多级排序_score置位最优先也可。

13.文档是如何被匹配到的

Explain Api
文档是如何被匹配到的
当explain选项加到某一文档上时，它会告诉你为何这个文档会被匹配，以及一个文档为何没有被匹配。
请求路径为 /index/type/id/_explain, 如下所示：
```
curl -X GET "http://localhost:9200/insurance/base_car/24433/_explain?pretty=ture" -d'{
    "query" : {
        "filtered" : {
            "filter" : {
                "bool" : {
                    "should" : { "term" : {"BrandName" : "宝马"}}
                }
            }
        }
    }
}'
```
除了上面我们看到的完整描述外，我们还可以看到这样的描述：
"Failure to meet condition(s) of required/prohibited clause(s)"

14.日期数学操作

当用于日期字段时，range 过滤器支持日期数学操作。例如，我们想找到所有最近一个小时的文档：
```json
"range" : {
    "timestamp" : {
        "gt" : "now-1h"
    }
}
```
这个过滤器将始终能找出所有时间戳大于当前时间减 1 小时的文档，让这个过滤器像移窗一样通过你的文档。
日期计算也能用于实际的日期，而不是仅仅是一个像 now 一样的占位符。只要在日期后加上双竖线 ||，就能使用日期数学表达式了。
```json
"range" : {
    "timestamp" : {
        "gt" : "2014-01-01 00:00:00",
        "lt" : "2014-01-01 00:00:00||+1M"
    }
}
```
15.处理null的两个关键字和用法

- exists => not null
- missing => null

http://es.xiaoleilu.com/080_Structured_Search/30_existsmissing.html

16.多重查询字符串

http://es.xiaoleilu.com/110_Multi_Field_Search/05_Multiple_query_strings.html
```json
{
  "query": {
    "bool": {
      "should": [
        { "match": { "title":  "War and Peace" }},
        { "match": { "author": "Leo Tolstoy"   }},
        { "bool":  {
          "should": [
            { "match": { "translator": "Constance Garnett" }},
            { "match": { "translator": "Louise Maude"      }}
          ]
        }}
      ]
    }
  }
}
```
17.boost参数 提高查询得分

http://es.xiaoleilu.com/100_Full_Text_Search/25_Boosting_clauses.html

## 参考链接
http://www.thinkphp.cn/code/1290.html

https://www.elastic.co/guide/en/elasticsearch/client/php-api/2.0/index.html

http://www.cnblogs.com/zhangchenliang/p/4215807.html

http://www.tuicool.com/articles/ayiUrqM 
