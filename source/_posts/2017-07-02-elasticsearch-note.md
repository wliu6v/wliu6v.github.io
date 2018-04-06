---
title: 一些 Elasticsearch 知识点
date: 2017-07-02 12:56:43
tags: [elasticsearch]
---

> 本文面向的环境是 ElasticSearch 5 系列版本。列一些在应用中碰到的知识点以便于查阅

<!-- more -->

# Elasticsearch 教程

- [Elasticsearch: The Definitive Guide](https://www.elastic.co/guide/en/elasticsearch/guide/current/index.html)
	- 最主要的官方教程，从入门到各个知识点的概述都有。
	- 在教程中有很多代码可以直接点击在 SENSE 中查看，但是最新版的 ES 已经没有 SENSE 插件了（而是直接整合进了 Kibana），因此需要先修改 SENSE 地址的指向才能正常工作。  
		`http://localhost:5601/app/kibana#/dev_tools/console`
- [Elasticsearch: 权威指南（上面那个的中文版）](https://www.elastic.co/guide/cn/elasticsearch/guide/current/index.html)  
- [Elasticsearch Reference](https://www.elastic.co/guide/en/elasticsearch/reference/current/index.html)
	- 看上去更为详细的教程，对每个 API 都有讲解，适合用来查阅而不是用来学习

# 知识点

## 如何查看分析器的工作方式
Ref : [Elasticsearch: 权威指南 » 基础入门 » 映射和分析 » 分析与分析器](https://www.elastic.co/guide/cn/elasticsearch/guide/current/analysis-intro.html)

```
GET /_analyze
{
  "analyzer": "standard",
  "text": "Text to analyze"
}
```


## 如何查看一个搜索的条件是否合法
Ref : [Elasticsearch: 权威指南 » 基础入门 » 请求体查询 » 验证查询]( https://www.elastic.co/guide/cn/elasticsearch/guide/current/validating-queries.html)

使用 validate-query API 进行验证
```
GET /gb/tweet/_validate/query?explain
{
   "query": {
      "tweet" : {
         "match" : "really powerful"
      }
   }
}
```


## 如何同时对多个字段进行排序，如何对一个字段中的多个值进行排序
Ref : [Elasticsearch: 权威指南 » 基础入门 » 排序与相关性 » 排序](https://www.elastic.co/guide/cn/elasticsearch/guide/current/_Sorting.html)
```
GET /_search
{
    "query" : {
        "bool" : {
            "must":   { "match": { "tweet": "manage text search" }},
            "filter" : { "term" : { "user_id" : 2 }}
        }
    },
    "sort": [
        { "date":   { "order": "desc" }},
        { "_score": { "order": "desc" }}
    ]
}
```

## 如何将一个字段定义多种索引方式
Ref : [Elasticsearch: 权威指南 » 基础入门 » 排序与相关性 » 字符串排序与多字段](https://www.elastic.co/guide/cn/elasticsearch/guide/current/multi-fields.html)

一个简单的方法是用两种方式对同一个字符串进行索引，这将在文档中包括两个字段： analyzed 用于搜索， not\_analyzed 用于排序。

但是保存相同的字符串两次在 `_source` 字段是浪费空间的。 我们真正想要做的是传递一个 单字段 但是却用两种方式索引它。所有的 `_core_field` 类型 (strings, numbers, Booleans, dates) 接收一个 fields 参数。

该参数允许你转化一个简单的映射如：

```
"tweet": {
    "type":     "string",
    "analyzer": "english"
}
```
为一个多字段映射如：
```
"tweet": { 
    "type":     "string",
    "analyzer": "english",
    "fields": {
        "raw": { 
            "type":  "string",
            "index": "not_analyzed"
        }
    }
}
```


## 关于重建索引
https://www.elastic.co/guide/cn/elasticsearch/guide/current/root-object.html

Elasticsearch 会存储 _source 字段，因此当我们需要重建索引的时候，我们可以直接从 ES 如此做，而不需要从远程的数据库中重新获取。

重建索引时使用 [reindex API](https://www.elastic.co/guide/en/elasticsearch/reference/current/docs-reindex.html) 实现。该 API 只会重建所有的数据，但不会覆盖 setting。因此应该先创建新的 index 并调整好相关的设置，然后使用该 API 更新数据。 

需要注意的是，因为重建索引是将旧的索引放到新的不同的索引中，因此名字会发生变化。如果需要在运行中进行无缝切换，可以使用 [索引别名](https://www.elastic.co/guide/cn/elasticsearch/guide/current/index-aliases.html) 屏蔽掉背后的索引的变化。

索引别名的用法：
```
PUT index2/_alias/index
DELETE index2/_alias/index
```

索引别名的命名对象__不能__是一个已经存在的索引。如上例，如果已经存在了一个 index 索引，那么该别名操作会失败。

通常如果要做到零停机的切换索引，会使用原子式的修改：
```
POST _aliases
{
  "actions": [
    {
      "add": {
        "index": "index2",
        "alias": "index"
      }
    },{
      "remove": {
        "index": "index1",
        "alias": "index"
      }
    }
  ]
}
```


## 遇到新添加的字段时应抛出异常
Ref : [Elasticsearch: 权威指南 » 基础入门 » 索引管理 » 动态映射](https://www.elastic.co/guide/cn/elasticsearch/guide/current/dynamic-mapping.html)

Elasticsearch 会自动对新添加的字段进行分析并决定其类型，但是在大多数情况下这种做法都不是我们想要的。因此可以将新字段的处理方式改为严格模式，使其遇到新字段的时候直接抛出异常。
```
PUT /my_index
{
    "mappings": {
        "my_type": {
            "dynamic":      "strict"
        }
    }
}
```


## 在搜索结果中返回分数的计算方式
Ref : [Elasticsearch: 权威指南 » 基础入门 » 排序与相关性 » 什么是相关性?](https://www.elastic.co/guide/cn/elasticsearch/guide/current/relevance-intro.html)

通过在任意搜索语句后面添加 explain 参数以输出每个分数计算的方式。

如果想查看某个文档为什么没有被搜索到，使用 /index/type/id/_explain 的方式进行调用。


## 关于 inline script
Ref : [Elasticsearch Reference [5.5] » Modules » Scripting](https://www.elastic.co/guide/en/elasticsearch/reference/current/modules-scripting.html)

Elasticsearch 支持在某些请求中添加 inline 字段以调用脚本。目前 Elasticsearch 主要的支持脚本语言是 painless 语言，语法与 Groovy 类似。

这里有 painless 的 [基本语法文档](https://www.elastic.co/guide/en/elasticsearch/reference/current/modules-scripting-painless-syntax.html) 和 [API 文档](https://www.elastic.co/guide/en/elasticsearch/reference/current/painless-api-reference.html)。其中 API 文档大部分是来自于 Oracle 官网的 java8 doc

要注意 Elasticsearch 默认是关闭了 inline script 的。启用的话需要在配置文件中声明：

```
script.inline: true

# 需要正则表达式的话就顺便添加这句
script.painless.regex.enabled: true
```

这里是一个 inline script 的应用场景

```
POST _reindex
{
  "source": {
    "index": "index1"
  },
  "dest": {
    "index": "index2"
  },
  "script": {
    "inline": "def str = ctx._source.param1; def p1 = /<.*?>/; def p2 = /\\[(img|url|code).*?\\].*?\\[\\/\\1\\]/im; def filtered = p1.matcher(str).replaceAll(''); filtered = p2.matcher(filtered).replaceAll(''); ctx._source.param1 = filtered; ctx_source.param2 = str"
  }
}
```

## 注意配置 Elasticsearch 的 Java Heap Memory Size

Ref : [Elasticsearch: The Definitive Guide [2.x] » Administration, Monitoring, and Deployment » Production Deployment » Heap: Sizing and Swapping](https://www.elastic.co/guide/en/elasticsearch/guide/current/heap-sizing.html)


> 血的教训。。。一开始忘了配置了，跑了两个星期之后就 OOM 挂了。

ES 默认的 Heap 为 1G，通常我们需要将其改的大一些，有两种方式：
	
- 在环境变量中 `export ES_JAVA_OPTS="-Xms31g -Xmx31g"`
- 在启动 es 时 `ES_JAVA_OPTS="-Xms31g -Xmx31g" ./bin/elasticsearch`

要注意 Xms 和 Xmx 值要相同，避免发生 java heap resize 消耗性能。

以及官方教程中提到一个 `ES_HEAP_SIZE` 的变量，但是实测在 5.4.0 版本中已经被禁用，提示说最好是用 `ES_JAVA_OPTS` 进行设置。

### 如何配置最合适的内存大小
1. 分配的内存最大不要超过 50%，因为 Lucene 也需要使用内存（off-heap）
2. 不要超过 32G。因为 HotSpot JVM 的一个 compress object pointers 技术。这里看不懂，参考下面 Ref 内容。基本上就是大于 32G 内存的时候，Java 的对象指针本身会变大，导致占用更多的空间，因此内存要达到 40-50G 的时候才能使性能与 32G 时候持平。因此尽量让 Java 堆内存保持在 32G 以下比较好
3. 简单考虑的话可以设置为 31G。（假设机器内存为 64G ）



## 通过 Reroute 命令重新分配 unassigned 节点

Ref : [Elasticsearch Reference [5.5] » Cluster APIs » Cluster Reroute](https://www.elastic.co/guide/en/elasticsearch/reference/5.5/cluster-reroute.html)

reroute 命令允许在包含特定命令的情况下对集群进行重新路由分配命令。通过这个命令我们可以将一个未分配的分片分配到一个特定的节点上。

```
POST /_cluster/reroute
{
    "commands" : [
        {
            "move" : {
                "index" : "test", 
				"shard" : 0,
                "from_node" : "node1", 
				"to_node" : "node2"
            }
        },
        {
          "allocate_replica" : {
                "index" : "test", 
				"shard" : 1,
                "node" : "node3"
          }
        }
    ]
}
```

需要注意，一旦重新分配，集群将会自动将状态平衡到均匀状态。比如说，如果将一个分片从 node1 移动到 node2，那么 es 为了保持均匀状态，会自动的将另一个分片从 node2 移到 node1。

可以将集群设置为禁止分配（disable allocations），那么就只能执行显示分配。

当时使用 reroute 命令是为了解决这样的问题：某个 node 挂了之后，上面的 shard 变成 Unassigned 状态了。等该 node 重新启动之后，shard 不知道为什么一直无法从 Unassigned 重新移到该 node 上，因此需要使用 allocate_replica 命令进行 reroute，手动将其移过去。猜测可能是 node 挂了之后，产生了一些 lock 文件阻碍了自动的 allocate，正确方式大概是删掉那些 lock 文件然后就可以自动 allocate 了。不过看上去手动执行 reroute 也可以达到效果。


Updated@17.08.17 : 当 shard 过大的时候，重新分配之后往往会是 yellow 状态。通过 `GET _cat/shards` 命令可以查看当前各个 shard 的状态，如果能看到其状态是 INITIALIZING 且可预知其 size 很大的话，是需要等待很长时间的。我碰到这个问题的时候，是一个 300+G 的索引，每个 shard 上有 60G 上下的内容，启动了四个多小时。可想而知这个启动时间跟机器性能也有关系。