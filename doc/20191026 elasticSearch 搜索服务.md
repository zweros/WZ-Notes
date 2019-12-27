---
title: 20191026 elasticSearch
date: 2019-10-26
---

[TOC]

## Lucene 

### 数据分类

- 结构化（关系型数据库），全文检索
  表：字段数量，字段类型

- 非结构化：

  文本文档，图片，视频，音乐...

- 半结构化：

  json，html，xml

### Lucene 简介

Lucene 是 Apache 软件基金会的一个项目，是一个开发源码的全文检索引擎工具包，是一个全文检索引擎的一个架构。提供了完成的查询引擎和检索引擎，部分文本分析引擎。

官方解释：

Lucene is a Java full-text search engine. Lucene is not a complete application, but rather a code library and API that can easily be used to add search capabilities to applications.

### 倒排索引*

通俗解释，我们通常都是通过查找文件位置及文件名，再查找文件的内容。倒排索引可以理解为通过文件内容来查找文件位置及文件名的。

倒排索引是一种索引方法，被用来存储在全文搜索下某个单词在一个文档或者一组文档中的存储位置的映射。它是文档检索系统中最常用的数据结构。通过倒排索引，可以根据单词快速获取包含这个单词的文档列表。

倒排索引也是 lucence 的索引核心。

举例：有两个文档 doc，id 分别为 1,2 。doc-id-1中的 content 是“我是中国人”，doc-id-2 中的 content 是 “中国是全球人口最多的国家，中国人也最多”。紧接着为这两个文档建立倒排索引。先读取 doc 文档，使用分词器对 content 中的内容进行分词，产生的结果放入索引 index 表中。以“中国”这个词为例，中国（1:1）{2}，（2:2）{0,15}。（1:1）{2}表示的意思是中国这个词出现在 id 为 1 的 doc 文档中，且出现的次数为 1，偏移量为 2 。（2:2）{0,15}  表示的意思是中国这个词出现在 id 为 2 的doc 文档猴子那个，且出现的次数为 2，偏移量有0,15 。

<img src="http://img.zwer.xyz/blog/20191027103435.png" style="zoom: 67%;" />



每个域可以设置三个类型：是否保存，是否索引，是否分词 

![](http://img.zwer.xyz/blog/20191027104508.png)

从 Hbase 中按行取出数据，建立 doc 文档，经过分析（分词，过滤）后，建立倒排索引。

参考： https://www.cnblogs.com/one--way/p/5708456.html

### Lucene 架构

- 横向扩展： 加机器

  ​	Lucene 集群之间的单机的计算速度，彼此应该相差不大 。

- 纵向扩展：主从模式

  ​	主节点-负责计算

  ​	从节点-1. 备机  2. 分担主节点读请求的压力

下面采用 3 个机器，作为  Lucene 的计算集群。

​		根据 doc 的 id 做 hash 取模，分配到 Lucene 计算集群中，doc  处理完毕后，会在本地生成 index 索引文件，各个机器上 index 索引文件最终汇聚到 master 节点。

![](http://img.zwer.xyz/blog/20191027104742.png)

## ElasticSearch 分布式安装

> 零配置，开箱即用
>没有繁琐的安装配置
> Java 版本要求：最低 1.7
> 下载地址：https://www.elastic.co/downloads/

启动

```shell
cd /usr/local/elasticsearch-2.2.0
./bin/elasticsearch
bin/elasticsearch -d(后台运行)
```

### 基础环境

安装 JDK ，并配置环境变量

### 安装工作

```shell
共享模式下：
useradd sxt
echo sxt | passwd --stdin sxt

su sxt
root 用户创建 /opt/sxt/es(普通用户无法创建)
mkdir -p /opt/sxt/es (注意：此时的目录权限属于root)
在父级目录 sxt 下执行： chown sxt:sxt es


单节点模式下root用户：
安装解压程序
ftp拷贝至根目录下，或者software目录下（备用，可选）

切换回sxt用户（共享模式，可以看到software内容）

单节点模式：

使用sxt用户解压es2.2.1zip包至es目录，保证es文件属于用户sxt：

unzip elasticsearch-2.2.1.zip -d /opt/sxt/es

进入es/conf, 修改elastic配置文件：
修改：
---------------cluster-------------------------

cluster.name: bjsxt-es
----------------node-------------------------------
node.name: node06 (分发后各节点修改)

----------------network--------------------------------
network.host: 192.168.133.6 (分发后修改)

http.port:9200 (放开)

末尾增加防脑裂：
discovery.zen.ping.multicast.enabled: false 
discovery.zen.ping.unicast.hosts: ["192.168.133.6","192.168.133.7", "192.168.133.8"]
discovery.zen.ping_timeout: 120s
client.transport.ping_timeout: 60s


增加图形可视化插件：

[root@node06 software]# cp -abr plugins/ /opt/sxt/es/elasticsearch-2.2.1/

分发其他节点：
scp -r ./elasticsearch-2.2.1/ sxt@node07:`pwd`
scp -r ./elasticsearch-2.2.1/ sxt@node08:`pwd`

修改配置文件


进入bin目录，启动脚本：
[root@node06 bin]# ./elasticsearch


--- 访问
http://node04:9200/_plugin/head/
```

![](http://img.zwer.xyz/blog/20191027123338.png)



### 注意

ElasticSearch 是不可以以 root 身份启动的，若以 root 身份启动后，启动会报错。然后删除 elasticSearch 安装目录下的 logs 文件夹及其子文件，切换到 elasticSearch 用户，重新启动。

```java
[root@node04 bin]# ./elasticsearch
Exception in thread "main" java.lang.RuntimeException: don't run elasticsearch as root.
	at org.elasticsearch.bootstrap.Bootstrap.initializeNatives(Bootstrap.java:93)
	at org.elasticsearch.bootstrap.Bootstrap.setup(Bootstrap.java:144)
	at org.elasticsearch.bootstrap.Bootstrap.init(Bootstrap.java:285)
	at org.elasticsearch.bootstrap.Elasticsearch.main(Elasticsearch.java:35)
Refer to the log for complete error details.

```

### 中文分词器

```shell
# 下载 elasticsearch 对应版本的中文分词器

# 上传到 Linux 服务器上，并修改  plugin-descriptor.properties  文件
# 将 es 的版本修改为当前安装 es  版本
elasticsearch.version=2.2.1

# 重新启动 es 
```



## REST

> REST 英文解释为: Representational State Transfer,  中文解释为： 表面层状态转移
> 一种软件架构风格，而不是标准，只是提供了一组设计原则和约束条件。

- 它主要用于客户端和服务器交互类的软件。

- 基于这个风格设计的软件可以更简洁，更有层次，更易于实现缓存等机制。

### REST 简介

![](http://img.zwer.xyz/blog/20191027201056.png)

### REST 操作

REST的操作分为以下几种:

- GET：获取对象的当前状态；
- PUT：改变对象的状态；
- POST：创建对象；
- DELETE：删除对象；
- HEAD：获取头信息。

### ES 内置的 REST 接口

![](http://img.zwer.xyz/blog/20191027201401.png)

## curl 命令

简单认为是可以在命令行下访问url的一个工具
curl是利用URL语法在命令行方式下工作的开源文件传输工具，使用curl可以简单实现常见的get/post请求。

### curl 使用

```
curl  
-X  指定http请求的方法   HEAD  GET POST  PUT DELETE
-d   指定要传输的数据
```

### ES 操作

```shell

file
segment（段，多个document组成）
document（一条记录，一个对象实例）
field（对象的属性）
term（项，分词之后的词条）



# yes
curl -XPUT http://192.168.133.6:9200/bjsxt/
# yes 
curl -XDELETE http://192.168.133.6:9200/test2/
curl -XDELETE http://192.168.133.6:9200/test3/

#document：yes 
curl -XPOST http://192.168.133.6:9200/bjsxt/employee -d '
{
 "first_name" : "bin",
 "age" : 33,
 "about" : "I love to go rock climbing",
 "interests": [ "sports", "music" ]
}'

curl -XPOST http://node02:9200/szxy/employee -d '
{
 "first_name" : "bin",
 "age" : 33,
 "about" : "I love to go rock climbing",
 "interests": [ "sports", "music" ]
}'

curl -XPOST http://node02:9200/szxy/employee -d '
{
 "first_name" : "gob bin",
 "age" : 43,
 "about" : "I love to go rock climbing",
 "interests": [ "sports", "music" ]
}'

curl -XPOST http://192.168.133.6:9200/bjsxt/employee/2 -d '
{
 "first_name" : "bin",
 "age" : 45,
 "about" : "I love to go rock climbing",
 "interests": [ "sports", "music" ]
}'


#add field yes

curl -XPOST http://192.168.133.6:9200/bjsxt/employee -d '
{
 "first_name" : "pablo2",
 "age" : 33,
 "about" : "I love to go rock climbing",
 "interests": [ "sports", "music" ],
 "sex": "man"
}'


curl -XPOST http://node02:9200/szxy/employee/1 -d '
{
 "first_name" : "pablo2",
 "age" : 35,
 "about" : "I love to go rock climbing",
 "interests": [ "sports", "music" ],
 "sex": "man"
}'


----------------------------------------


#put：yes


curl -XPUT http://node02:9200/szxy/employee/1 -d '
{
 "first_name" : "pablo2",
 "last_name" : "pang",
 "age" : 42,
 "about" : "I love to go rock climbing",
 "interests": [ "sports", "music" ]
}'

curl -XPUT http://node02:9200/szxy/employee -d '
{
 "first_name" : "god bin",
 "last_name" : "bin",
 "age" : 45,
 "about" : "I love to go rock climbing",
 "interests": [ "sports", "music" ]
}'


curl -XPUT http://node02:9200/szxy/employee/2 -d '
{
 "first_name" : "god bin",
 "last_name" : "bin",
 "age" : 45,
 "about" : "I love to go rock climbing",
 "interests": [ "sports", "music" ]
}'

curl -XPUT http://192.168.133.6:9200/bjsxt/employee/1 -d '
{
 "first_name" : "god bin",
 "last_name" : "pang",
 "age" : 40,
 "about" : "I love to go rock climbing",
 "interests": [ "sports", "music" ]
}'



#根据document的id来获取数据：(without pretty)
curl -XGET http://node02:9200/bjsxt/employee/1?pretty

#根据field来查询数据：
curl -XGET http://node02:9200/szxy/employee/_search?q=first_name="bin"

#根据field来查询数据：match
curl -XGET http://node02:9200/szxy/employee/_search?pretty -d '
{
 "query":
  {"match":
   {"first_name":"bin"}
  }
}'



#对多个field发起查询：multi_match
curl -XGET http://node02:9200/szxy/employee/_search?pretty -d '
{
 "query":
  {"multi_match":
   {
    "query":"bin",
    "fields":["last_name","first_name"],
    "operator":"and"
   }
  }
}'


#多个term对多个field发起查询:bool（boolean） 
# 组合查询，must，must_not,should 
#  must + must : 交集
#  must +must_not ：差集
#  should+should  : 并集

curl -XGET http://node02:9200/szxy/employee/_search?pretty -d '
{
 "query":
  {"bool" :
   {
    "must" : 
     {"match":
      {"first_name":"bin"}
     },
    "must" : 
     {"match":
      {"age":33}
     }
   }
  }
}'

curl -XGET http://192.168.133.6:9200/bjsxt/employee/_search?pretty -d '
{
 "query":
  {"bool" :
   {
    "must" : 
     {"match":
      {"first_name":"bin"}
     },
    "must_not" : 
     {"match":
      {"age":33}
     }
   }
  }
}'





curl -XGET http://192.168.133.6:9200/bjsxt/employee/_search?pretty -d '
{
 "query":
  {"bool" :
   {
    "must_not" : 
     {"match":
      {"first_name":"bin"}
     },
    "must_not" : 
     {"match":
      {"age":33}
     }
   }
  }
}'

##查询first_name=bin的，或者年龄在20岁到33岁之间的

curl -XGET http://192.168.133.6:9200/bjsxt/employee/_search -d '
{
 "query":
  {"bool" :
   {
   "must" :
    {"term" : 
     { "first_name" : "bin" }
    }
   ,
   "must_not" : 
    {"range":
     {"age" : { "from" : 20, "to" : 33 }
    }
   }
   }
  }
}'


#修改配置
curl -XPUT 'http://192.168.133.6:9200/test2/' -d'{"settings":{"number_of_replicas":2}}'

curl -XPUT 'http://192.168.133.6:9200/test3/' -d'{"settings":{"number_of_shards":3,"number_of_replicas":3}}'

curl -XPUT 'http://192.168.133.6:9200/test4/' -d'{"settings":{"number_of_shards":6,"number_of_replicas":4}}'


curl -XPOST http://192.168.9.11:9200/bjsxt/person/_mapping -d'
{
    "person": {
        "properties": {
            "content": {
                "type": "string",
                "store": "no",
                "term_vector": "with_positions_offsets",
                "analyzer": "ik_max_word",
                "search_analyzer": "ik_max_word",
                "include_in_all": "true",
                "boost": 8
            }
        }
    }
}'






```

## ES API操作

```java
public class TestES {
	
	Client client;
	
	@Before
	public void conn() throws Exception{

		Settings settings = Settings.settingsBuilder()
		        .put("cluster.name", "es-cluster").build();
		client = TransportClient.builder().settings(settings).build()
		        .addTransportAddress(new InetSocketTransportAddress(InetAddress.getByName("node02"), 9300))
		        .addTransportAddress(new InetSocketTransportAddress(InetAddress.getByName("node03"), 9300))
		        .addTransportAddress(new InetSocketTransportAddress(InetAddress.getByName("node04"), 9300));
	}
	
	
	@After
	public void close(){
		client.close();
	}
	
	@Test
	public void test01(){
		
		IndicesExistsResponse resp = client.admin().indices().prepareExists("javatest").execute().actionGet();
		if(resp.isExists()){
			System.err.println("index is exist...");
			client.admin().indices().prepareDelete("javatest");
		}
		Map<String, Object> sets = new HashMap<String,Object>();
		sets.put("number_of_replicas", 2);
		client.admin().indices().prepareCreate("javatest").setSettings(sets).execute().actionGet();
	}
	
	
	@Test
	public void test02(){
		
		Map<String, Object> data = new HashMap<String,Object>();
		data.put("name", "bjsxt");
		data.put("content", "hadoop");
		data.put("size", 10086);
		
		IndexResponse resp = client.prepareIndex("javatest","testfiled").setSource(data).execute().actionGet();
		System.out.println(resp.getId());
		
	}
	
	@Test
	public void test03(){
		
		QueryBuilder q =  new MatchQueryBuilder("content", "hadoop");
		SearchResponse resp = client.prepareSearch("javatest")
									.setTypes("testfiled")
									.addHighlightedField("content")
									.setHighlighterPreTags("<font color=red>")
									.setHighlighterPostTags("</font>")
									.setQuery(q)
									.setFrom(1)
									.setSize(2)
									.execute()
									.actionGet();
		
		
		SearchHits hits = resp.getHits();
		System.out.println(hits.getTotalHits());
		for (SearchHit hit : hits) {
			System.out.println(hit.getSourceAsString());
			System.out.println(hit.getSource().get("name"));
			System.out.println(hit.getHighlightFields().get("content").getFragments()[0]);
			
		}
		
	}

}

```

