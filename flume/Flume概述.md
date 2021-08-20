# Flume概述
https://www.bilibili.com/video/BV184411B7kU?from=search&seid=9105331552740690431

# 1.1 Flume简介

Flume是一个分布式、可靠和高可用的海量日志聚合系统，支持在系统中定制各类数据发送方，用于收集数据;同时，Flume提供对数据进行简单处理、并写入各种数据接收方的能力。Flume有如下几个特点:  
1. 收集、聚合时间流数据的分布式架构  
2. 通常用于log数据  
3. 采用ad-hoc方案  
4. 声明式配置、可以动态更新配置
5. 提供上下文路由功能
6. 支持负载均衡和故障转移
7. 完全的可拓展

# 1.2 Architecture

A Flume event is defined as a unit of data flow having a byte payload and an optional set of string attributes. A Flume agent is a (**JVM**) process that hosts the components through which events flow from an external source to the next destination (hop).  
![](http://flume.apache.org/_images/UserGuide_image00.png)

A Flume source consumes events **delivered to it by an external source like a web server**. The external source sends events to Flume in a format **that is recognized by the target Flume source**. For example, an Avro Flume source can be used to receive Avro events from Avro clients or other Flume agents in the flow that send events from an Avro sink. A similar flow can be defined using a Thrift Flume Source to receive events from a Thrift Sink or a Flume Thrift Rpc Client or Thrift clients written in any language generated from the Flume thrift protocol.When a Flume source receives an event, it stores it into one or more channels. **The channel is a passive store that keeps the event until it’s consumed by a Flume sink.** The file channel is one example – it is backed by the local filesystem. The sink removes the event from the channel and puts it into an external repository like HDFS (via Flume HDFS sink) or forwards it to the Flume source of the next Flume agent (next hop) in the flow. The source and sink within the given agent run **asynchronously** with the events staged in the channel.