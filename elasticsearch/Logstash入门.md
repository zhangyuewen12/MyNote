# [Logstash快速入门](https://www.cnblogs.com/xuwujing/p/13412108.html)

## ELK介绍

ELK是三个开源软件的缩写，分别表示：Elasticsearch , Logstash, Kibana , 它们都是开源软件。新增了一个FileBeat，它是一个轻量级的日志收集处理工具(Agent)，Filebeat占用资源少，适合于在各个服务器上搜集日志后传输给Logstash，官方也推荐此工具。

- Elasticsearch是个开源分布式搜索引擎，提供搜集、分析、存储数据三大功能。它的特点有：分布式，零配置，自动发现，索引自动分片，索引副本机制，restful风格接口，多数据源，自动搜索负载等。
- Logstash 主要是用来日志的搜集、分析、过滤日志的工具，支持大量的数据获取方式。一般工作方式为c/s架构，client端安装在需要收集日志的主机上，server端负责将收到的各节点日志进行过滤、修改等操作在一并发往elasticsearch上去。
- Kibana 也是一个开源和免费的工具，Kibana可以为 Logstash 和 ElasticSearch 提供的日志分析友好的 Web 界面，可以帮助汇总、分析和搜索重要数据日志。
- Filebeat是一个轻量型日志采集器，可以方便的同kibana集成，启动filebeat后，可以直接在kibana中观看对日志文件进行detail的过程。

## Logstash介绍

Logstash是一个数据流引擎：

> 它是用于数据物流的开源流式ETL引擎,在几分钟内建立数据流管道,具有水平可扩展及韧性且具有自适应缓冲,不可知的数据源,具有200多个集成和处理器的插件生态系统,使用Elastic Stack监视和管理部署

Logstash包含3个主要部分： 输入(inputs)，过滤器(filters)和输出(outputs)。
inputs主要用来提供接收数据的规则，比如使用采集文件内容；
filters主要是对传输的数据进行过滤，比如使用grok规则进行数据过滤；
outputs主要是将接收的数据根据定义的输出模式来进行输出数据，比如输出到ElasticSearch中.



<img src="/Users/zyw/Library/Application Support/typora-user-images/image-20210607145619075.png" alt="image-20210607145619075" style="zoom:50%;" />

