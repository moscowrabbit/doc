#  Elasticsearch introduction

***You know, for search (and analysis)***

elasticsearch是位于弹性堆栈核心的分布式搜索和分析引擎。Logstash和Beats有助于收集、聚合和丰富数据，并将其存储在Elasticsearch中。Kibana使您能够交互式地探索、可视化和共享对您的数据的见解，并管理和监控堆栈。Elasticsearch是索引、搜索和分析魔术发生的地方。

Elasticsearch为所有类型的数据提供实时搜索和分析。无论您使用的是结构化或非结构化文本、数值数据还是地理空间数据，Elasticsearch都可以以支持快速搜索的方式有效地存储和建立索引。您不仅可以进行简单的数据检索和信息聚合，还可以发现数据中的趋势和模式。随着您的数据和查询量的增长，Elasticsearch的分布式特性使您的部署可以随着它无缝地增长。

虽然并不是每个问题都是一个搜索问题，但Elasticsearch提供了在各种用例中处理数据的速度和灵活性:

- 在应用程序或网站上添加搜索框

- 存储和分析日志、指标和安全事件数据

- 使用机器学习来实时自动建模数据的行为

- 使用Elasticsearch作为存储引擎来自动化业务工作流

- 使用Elasticsearch作为地理信息系统(GIS)管理、集成和分析空间信息

- 使用Elasticsearch作为生物信息学研究工具存储和处理遗传数据


我们不断地被人们使用搜索的新奇方式所震惊。但是，无论您的用例与其中一个类似，还是使用Elasticsearch来处理新问题，在Elasticsearch中处理数据、文档和索引的方式都是相同的。

##  Data in: documents and indices

Elasticsearch是一个分布式文档存储。Elasticsearch存储的是序列化为JSON文档的复杂数据结构，而不是以列数据行的形式存储信息。当集群中有多个Elasticsearch节点时，存储的文档分布在整个集群中，可以立即从任何节点访问。

当存储一个文档时，它几乎是实时的，在1秒内就可以被索引和完全搜索。Elasticsearch使用了一种名为倒排索引的数据结构，它支持非常快速的全文搜索。反向索引列出任何文档中出现的每个惟一单词，并标识每个单词出现的所有文档。

可以将索引看作是文档的优化集合，每个文档是字段的集合，这些字段是包含数据的键值对。默认情况下，Elasticsearch对每个字段中的所有数据进行索引，每个索引字段都有一个专用的、优化的数据结构。例如，文本字段存储在倒排索引中，数字和地理字段存储在BKD树中。使用每个字段的数据结构来组装和返回搜索结果的能力是Elasticsearch如此快速的原因。

Elasticsearch还具有无模式的能力，这意味着可以对文档进行索引，而不必显式地指定如何处理文档中可能出现的每个不同字段。当动态映射被启用时，Elasticsearch会自动检测并向索引添加新的字段。这种默认行为使得创建索引和浏览数据变得很容易——只要开始创建索引文档，Elasticsearch就会检测布尔值、浮点值和整数值、日期和字符串，并将它们映射到合适的Elasticsearch数据类型。

但是，最终，您比Elasticsearch更了解您的数据以及您想如何使用它。您可以定义规则来控制动态映射，并显式地定义映射来完全控制字段的存储和索引方式。

定义您自己的映射使您能够:

- 区分全文字符串字段和精确值字符串字段

- 执行特定于语言的文本分析

- 为部分匹配优化字段

- 使用自定义日期格式

- 使用geo_point和geo_shape等不能自动检测的数据类型


为不同的目的以不同的方式为同一个字段建立索引通常是很有用的。例如，您可能希望将字符串字段索引为全文搜索的文本字段和用于排序或聚合数据的关键字字段。或者，您可以选择使用多个语言分析器来处理包含用户输入的字符串字段的内容。

在索引期间应用于全文字段的分析链也在搜索时使用。当查询全文字段时，在索引中查找术语之前，查询文本会进行相同的分析。

##  Information out: search and analyze

虽然您可以使用Elasticsearch作为文档存储和检索文档及其元数据，但它的真正强大之处在于能够轻松访问构建在Apache Lucene搜索引擎库上的全套搜索功能。

Elasticsearch提供了一个简单、一致的REST API，用于管理集群、索引和搜索数据。出于测试目的，您可以直接从命令行或通过Kibana中的Developer Console轻松地提交请求。在应用程序中，您可以使用Elasticsearch客户端来选择语言:Java、JavaScript、Go、. net、PHP、Perl、Python或Ruby。

###  Searching your data

Elasticsearch REST api支持结构化查询、全文查询和结合这两种查询的复杂查询。结构化查询类似于您可以在SQL中构造的查询类型。例如，您可以在雇员索引中搜索性别和年龄字段，并根据hire_date字段对匹配项进行排序。全文查询查找与查询字符串匹配的所有文档，并根据相关度(它们与搜索词的匹配程度)返回它们。

除了搜索单个术语外，您还可以执行短语搜索、相似度搜索和前缀搜索，并获得自动补全建议。

是否有想要搜索的地理空间数据或其他数字数据?Elasticsearch在优化的数据结构中对非文本数据进行索引，支持高性能的地理和数字查询。

您可以使用Elasticsearch的全面的json风格的查询语言(query DSL)访问所有这些搜索功能。您还可以构造SQL风格的查询来在Elasticsearch内部本地搜索和聚合数据，JDBC和ODBC驱动程序允许许多第三方应用程序通过SQL与Elasticsearch交互。

####  But wait, there’s more

想要自动化时间序列数据的分析?您可以使用机器学习功能在您的数据中创建正常行为的精确基线，并识别异常模式。通过机器学习，你可以检测到:

- 与值、计数或频率的时间偏差有关的异常

- 统计罕见

- 种群成员的异常行为


最好的部分是什么?您可以在不指定算法、模型或其他数据科学相关配置的情况下完成此工作。

##  Scalability and resilience: clusters, nodes, and shards

Elasticsearch始终可用，并可根据您的需求进行扩展。它通过自然分配来做到这一点。您可以将服务器(节点)添加到集群中以增加容量，Elasticsearch会自动将您的数据和查询负载分布到所有可用的节点上。Elasticsearch不需要彻底检查应用程序，它知道如何平衡多节点集群以提供规模化和高可用性。节点越多越快乐。

这是如何工作的呢?实际上，Elasticsearch索引只是一个或多个物理碎片的逻辑分组，其中每个碎片实际上是一个自包含的索引。通过将索引中的文档分布到多个分片上，并将这些分片分布到多个节点上，Elasticsearch可以确保冗余，既可以防止硬件故障，又可以随着节点添加到集群中而增加查询容量。随着集群的增长(或收缩)，Elasticsearch会自动迁移碎片来重新平衡集群。

有两种类型的分片:基本分片和复制分片。索引中的每个文档都属于一个主分片。复制分片是主分片的副本。副本提供数据的冗余副本，以防止硬件故障，并增加服务读取请求(如搜索或检索文档)的容量。

索引中主分片的数量在索引创建时是固定的，但是复制分片的数量可以在不中断索引或查询操作的情况下随时改变。

###  It depends…

关于分片大小和为索引配置的主分片数量，有许多性能方面的考虑和权衡。碎片越多，维护这些索引的开销就越大。碎片的大小越大，当Elasticsearch需要重新平衡集群时，移动碎片所需的时间就越长。

查询大量的小分片可以使每个分片的处理速度更快，但是查询越多意味着开销越大，因此查询少量的大分片可能会更快。总之，这要看情况。

作为起点:

- 目标是将平均碎片大小保持在几GB到几十GB之间。对于基于时间的数据的用例，通常会看到20GB到40GB范围的分片。

- 避免大量碎片的问题。一个节点可以容纳的分片数量与可用堆空间成正比。作为一般规则，每GB堆空间的碎片数量应该小于20个。


为您的用例确定最佳配置的最佳方法是通过使用您自己的数据和查询进行测试。

###  In case of disaster

出于性能原因，集群内的节点需要位于同一个网络上。在不同数据中心的节点之间平衡集群中的分片花费的时间太长了。但是高可用性架构要求您避免把所有鸡蛋放在一个篮子里。当一个位置发生重大故障时，另一个位置的服务器需要能够无缝接管。The answer? Cross-cluster 复制 (CCR)。

CCR提供了一种从主集群自动同步索引到可作为热备份的辅助远程集群的方法。如果主集群故障，辅助集群可以接管。您还可以使用CCR创建辅助集群，以服务于与用户地理位置相近的读请求。

跨集群复制是主备复制。主集群上的索引是活动领导索引，并处理所有写请求。复制到次要集群的索引是只读的追随者。

###  Care and feeding

与任何企业系统一样，您需要工具来保护、管理和监视您的Elasticsearch集群。安全、监控和管理特性集成到Elasticsearch中，使您能够使用Kibana作为管理集群的控制中心。数据汇总和索引生命周期管理等特性可以帮助您随着时间的推移智能地管理数据。

#  Getting started with Elasticsearch

准备好试用Elasticsearch并亲自看看如何使用REST api存储、搜索和分析数据了吗?

步骤通过这个入门教程:

- 启动并运行Elasticsearch集群

- 对一些样本文档进行索引

- 使用Elasticsearch查询语言搜索文档

- 使用桶和指标聚合分析结果

需要更多的背景吗?

请查看Elasticsearch简介，学习术语并了解Elasticsearch如何工作的基础知识。如果您已经熟悉Elasticsearch，并希望了解它如何与堆栈的其他部分一起工作，您可能想要跳到Elastic stack教程，了解如何使用Elasticsearch、Kibana、Beats和Logstash设置系统监控解决方案。

##  Get Elasticsearch up and running

为了在Elasticsearch中寻找一个测试驱动器，您可以在Elasticsearch Service中创建一个托管部署，或者在您自己的Linux、macOS或Windows机器上建立一个多节点Elasticsearch集群。

###  Run Elasticsearch on Elastic Cloud

当您在Elasticsearch Service上创建部署时，该服务将与Kibana和APM一起提供三节点的Elasticsearch集群。

要创建部署:

- 注册免费试用并验证你的电子邮件地址。

- 为您的帐户设置密码。

- 单击创建部署。


一旦创建了部署，就可以为一些文档建立索引了。

###  Run Elasticsearch locally on Linux, macOS, or Windows

当您在Elasticsearch Service上创建一个部署时，将自动提供一个主节点和两个数据节点。通过从tar或zip归档文件中安装，您可以在本地启动多个Elasticsearch实例，以查看多节点集群的行为。

在本地运行三节点的Elasticsearch集群:

1. Download the Elasticsearch archive for your OS:

     Linux: [elasticsearch-7.4.2-linux-x86_64.tar.gz](https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-7.4.2-linux-x86_64.tar.gz)

   ```
   curl -L -O https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-7.4.2-linux-x86_64.tar.gz
   ```

2. Extract the archive:

   Linux:

   ```
   tar -xvf elasticsearch-7.4.2-linux-x86_64.tar.gz
   ```

3. Start Elasticsearch from the `bin` directory:

   Linux and macOS:

   ```
   cd elasticsearch-7.4.2/bin
   ./elasticsearch
   ```

4. Start two more instances of Elasticsearch so you can see how a typical multi-node cluster behaves. You need to specify unique data and log paths for each node.

   Linux and macOS:

   ```
   ./elasticsearch -Epath.data=data2 -Epath.logs=log2
   ./elasticsearch -Epath.data=data3 -Epath.logs=log3
   ```

5. 使用cat健康API来验证您的三节点集群是否正在运行。cat api以一种比原始JSON更易于阅读的格式返回关于集群和索引的信息。

   通过向Elasticsearch REST API提交HTTP请求，您可以直接与集群交互。本指南中的大多数示例都允许您复制适当的cURL命令，并从命令行将请求提交到本地Elasticsearch实例。如果您已经安装并运行Kibana，您也可以打开Kibana并通过开发控制台提交请求。

   The response should indicate that the status of the `elasticsearch` cluster is `green` and it has three         nodes: 

   ```shell
   GET /_cat/health?v
   
   epoch      timestamp cluster           status node.total node.data shards pri relo init unassign pending_tasks max_task_wait_time active_shards_percent
   1625403419 12:56:59  css-elasticsearch green           3         3      4   2    0    0        0             0                  -                100.0%  
   
   cluster ，集群名称
   status，集群状态 green代表健康；yellow代表分配了所有主分片，但至少缺少一个副本，此时集群数据仍旧完整；red代表部分主分片不可用，可能已经丢失数据。
   node.total，代表在线的节点总数量
   node.data，代表在线的数据节点的数量
   shards， active_shards 存活的分片数量
   pri，active_primary_shards 存活的主分片数量 正常情况下 shards的数量是pri的两倍。
   relo， relocating_shards 迁移中的分片数量，正常情况为 0
   init， initializing_shards 初始化中的分片数量 正常情况为 0
   unassign， unassigned_shards 未分配的分片 正常情况为 0
   pending_tasks，准备中的任务，任务指迁移分片等 正常情况为 0
   max_task_wait_time，任务最长等待时间
   active_shards_percent，正常分片百分比 正常情况为 100%
   ```

###  Other installation options

从归档文件安装Elasticsearch使您能够轻松地在本地安装和运行多个实例，以便进行尝试。要运行单个实例，您可以在Docker容器中运行Elasticsearch，在Linux上使用DEB或RPM包安装Elasticsearch，在macOS上使用Homebrew安装Elasticsearch，在Windows上使用MSI包安装程序安装Elasticsearch。有关更多信息，请参见安装Elasticsearch。

## Index some documents 

一旦集群启动并运行，就可以为一些数据建立索引了。Elasticsearch有多种吸收选项，但最终它们都做了同样的事情:将JSON文档放入Elasticsearch索引中。

你可以直接通过一个简单的PUT请求来实现这一点，该请求指定你想要添加的文档的索引，一个唯一的文档ID，以及请求体中的一个或多个“field”:“value”对:

```
PUT /customer/_doc/1
{
  "name": "John Doe"
}
```

如果客户索引不存在，该请求将自动创建该索引，添加一个ID为1的新文档，并存储和索引名称字段。

因为这是一个新文档，响应显示操作的结果是创建了文档的版本:

```json
{
  "_index" : "customer",
  "_type" : "_doc",
  "_id" : "1",
  "_version" : 1,
  "result" : "created",
  "_shards" : {
    "total" : 2,
    "successful" : 2,
    "failed" : 0
  },
  "_seq_no" : 26,
  "_primary_term" : 4
}
```

可以立即从集群中的任何节点获得新文档。你可以通过一个指定文档ID的GET请求来检索它:

```
GET /customer/_doc/1
```

响应指示找到了具有指定ID的文档，并显示了索引的原始源字段。

```json
{
  "_index" : "customer",
  "_type" : "_doc",
  "_id" : "1",
  "_version" : 1,
  "_seq_no" : 26,
  "_primary_term" : 4,
  "found" : true,
  "_source" : {
    "name": "John Doe"
  }
}
```

###  Indexing documents in bulk

如果您有很多文档要索引，您可以将它们与批量API一起分批提交。使用bulk对文档操作进行批处理比单独提交请求要快得多，因为它最小化了网络往返。

最佳批处理大小取决于许多因素:文档大小和复杂性、索引和搜索负载，以及集群可用的资源。一个很好的起点是一批1,000到5,000个文档，总有效负载在5MB到15MB之间。在那里，你可以通过实验找到最佳位置。

为了Elasticsearch中获取一些数据，你可以开始搜索和分析:

1. Download the [`accounts.json`](https://github.com/elastic/elasticsearch/blob/master/docs/src/test/resources/accounts.json?raw=true) sample data set. The documents in this randomly-generated data set represent user accounts with the following information:

   ```json
   {
       "account_number": 0,
       "balance": 16623,
       "firstname": "Bradshaw",
       "lastname": "Mckenzie",
       "age": 29,
       "gender": "F",
       "address": "244 Columbus Place",
       "employer": "Euron",
       "email": "bradshawmckenzie@euron.com",
       "city": "Hobucken",
       "state": "CO"
   }
   ```

2. Index the account data into the `bank` index with the following `_bulk` request:

   ```
   curl -H "Content-Type: application/json" -XPOST "localhost:9200/bank/_bulk?pretty&refresh" --data-binary "@accounts.json"
   curl "localhost:9200/_cat/indices?v"
   ```

   The response indicates that 1,000 documents were indexed successfully.

   ```
   health status index uuid                   pri rep docs.count docs.deleted store.size pri.store.size
   yellow open   bank  l7sSYV2cQXmu6_4rJWVIww   5   1       1000            0    128.6kb
   ```

   

