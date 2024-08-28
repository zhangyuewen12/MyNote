<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->
**Table of Contents**  *generated with [DocToc](https://github.com/thlorenz/doctoc)*

- [一、介绍](#%E4%B8%80%E4%BB%8B%E7%BB%8D)
- [二、HDFS 设计原理](#%E4%BA%8Chdfs-%E8%AE%BE%E8%AE%A1%E5%8E%9F%E7%90%86)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->

# 一、介绍
HDFS （Hadoop Distributed File System）是 Hadoop 下的分布式文件系统，具有高容错、高吞吐量等特性，可以部署在低成本的硬件上。
# 二、HDFS 设计原理

![avatar](https://camo.githubusercontent.com/9004fba74875e356ebe692fa949cf5ea888b13ac/68747470733a2f2f67697465652e636f6d2f68656962616979696e672f426967446174612d4e6f7465732f7261772f6d61737465722f70696374757265732f686466736172636869746563747572652e706e67)


## 2.1 HDFS 架构
HDFS 遵循主/从架构，由单个 NameNode(NN) 和多个 DataNode(DN) 组成：

* NameNode : 负责执行有关 文件系统命名空间 的操作，例如打开，关闭、重命名文件和目录等。它同时还负责集群元数据的存储，记录着文件中各个数据块的位置信息。
* DataNode：负责提供来自文件系统客户端的读写请求，执行块的创建，删除等操作