# Flink的类加载问题

这个问题的由来是在人资项目中的日志动态分析项目中遇到的，在通过flink读取kafka的应用开发中，程序在本地运行可以的，在集群中运行报错。报错信息如下图所示，

这个问题很有迷惑性，所以在这里记录一下。先说一下最终的解决方案：

在${FLINK_HOME}/onf/flink-conf.yaml 添加如下内容并重启 flink.

```
classloader.resolve-order: parent-first
```

网上搜索到了问题的解决方案，但是对于问题出现的原因却解释的很少，为了深入探究问题的产生的原因，在官方文档找到了相关解释:

```
大概解释的内容如下:
一般来说，当运行一个Flink应用时，JVM会随着应用时间的推荐加载不同的类。这些class通常可以分为三类：
1.The Java Classpath
2.The Flink Plugin Components
3.The Dynamic User Code

与之对应的，Flink存在两种动态类加载层次结构：
1. Java’s application classloader
2. The dynamic plugin/user code classloader
application classloader主要加载存在于classpath中的所有的类。Dynamic classlaoder 主要加载plugin或者user-code jar。
另外，Dynamic classLoader以application classLoader作为其父类。默认情况下，flink将类加载顺序反转，意味着Flink首先使用dynamic classloader加载类，仅当dynamic classLoader无法加载类时的时候，才使用application classLoader，这就是Flink的类加载机制。

```

这个问题产生的原因主要是由于通过不断的类加载器加载了不同版本的class，不同版本之间的class相互转换产生的问题。