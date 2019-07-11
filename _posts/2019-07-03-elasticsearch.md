---
layout: post
title: 'elasticsearch实践与理论'
subtitle: 'elasticsearch实践与理论'
date: 2019-07-03 
categories: 搜索
author: yates
cover: 'www.baidu.com'
tags: 搜索
---

##前言 
elasticsearch是一个基于Lucene构建的开源，分布式，restful接口全文搜索引擎。他拥有一个分布式文档数据库，每个字段都是被索引的数据且可被搜索，能够处理数以百计服务器和PB级数据，能够很短时间存储，搜索，分析大量数据。

**倒排索引**
索引表中每一项都包括一个属性值和具有该属性值的各记录的地址。一般表示为一个关键词，然后是频度，位置，信息等。将其保存为词典文件，频率文件，位置文件。其中关键字按字符顺序排列，而不是B树，所以可以二元搜索快速定位关键词。

elasticsearch支持rest API的调用，通过索引关键字的搜索得到文档
**创建索引**
PUT http://127.0.0.1:9200/secisland
```java
{
    "settings": {
        "index": {
            "number_of_shards": 3,
            "number_of_replicas": 2
        }
    }
}
```

**删除索引**
DELETE http://127.0.0.1:9200/secisland

**获取索引**
GET http://127.0.0.1:9200/secisland

只返回需要的属性
GET http://127.0.0.1:9200/{index}/_{attr}

获取多个索引和多个文档类型
GET http://127.0.0.1:9200/{index}/_mapping/{type} // index和type可以接受,分隔符

打开/关闭索引
GET http://127.0.0.1:9200/{index}/_open
GET http://127.0.0.1:9200/{index}/_close

在同一个索引的不同类型中，相同名称的字段中必须有相同的映射，因为在内部它们是在同一个领域，如果在这种情况下更新映射参数，系统将会抛出异常

**判断类型是否存在**
HEAD http://127.0.0.1:9200/{index}/{type}

**索引别名**
POST http://127.0.0.1:9200/_aliases
增加别名
```java
{
    "actions": [
        {
            "add": {
                "index": "secisland",
                "alias": "alias1"
            }
        }
    ]
}
```
删除别名
```java
{
    "actions": [
        {
            "remove": {
                "index": "secisland",
                "alias": "alias1"
            }
        }
    ]
}
```

**索引分析**
索引分析过程是这样的：首先，把一个文本块分析成一个个单独的词，为了后面的倒排索引主备，；然后，标准化这些词，提高可搜索性。

分析器组成

- 字符过滤器：字符串经过字符串过滤器去除HTML标记。
- 分词器：把字符串标记拆分为标准化，独立的词
- 标记过滤器：每个词通过所有标记过滤处理，通过修改，增加，去除等操作

elasticsearch可以有多种组合的分析器

**索引模板**
索引模板是创建好的一个索引参数设置和映射的模板，在创建新索引的时候指定模板名称就可以使用模板定义好的参数设置和映射的模板

**索引统计**
GET http://127.0.0.1:9200/{index}/stats