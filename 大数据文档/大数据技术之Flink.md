# 第一章 Flink基础介绍

## 1.1 简介

<img src="/Users/zyw/Library/Application Support/typora-user-images/image-20221118232828470.png" alt="image-20221118232828470" style="zoom:50%;" />

Flink 的具体定位是:Apache Flink 是一个框架和分布式处理引擎，如图所示，用于对无界和有界数据流进行有状态计算。Flink 被设计在所有常见的集群环境中运行，以内存执行速度和任意规模来执行计算。

## 1.2 分层API

Flink offers different levels of abstraction for developing streaming/batch applications.

<img src="/Users/zyw/Library/Application Support/typora-user-images/image-20221118235451340.png" alt="image-20221118235451340" style="zoom:50%;" />



- 最低级别的抽象只是提供有状态和及时的流处理。它通过Process Function嵌入到DataStream API中。它允许用户自由处理一个或多个流中的事件，并提供一致的容错状态。此外，用户可以注册事件时间和处理时间回调，从而允许程序实现复杂的计算。

- 许多应用程序不需要上面的低级抽象，而是可以使用 Core API进行编程：DataStream API（有界/无界流）和 DataSet API（有边界数据集）。这些流畅的API为数据处理提供了通用的实现，如各种形式的用户指定的转换、连接、聚合、窗口、状态等。

  低级Process Function与DataStream API集成，使得可以根据需要使用低级抽象。DataSet API在有界数据集上提供了其他原语，如循环/迭代。

- Table API是一个以表为中心的声明性DSL，它可能是动态变化的表（当表示流时）。Table API遵循（扩展的）关系模型：表附加了一个模式（类似于关系数据库中的表），API提供了类似的操作，如select、project、join、groupby、aggregate等。Table API程序声明性地定义了应该执行的逻辑操作，而不是确切地指定操作代码的外观。虽然TableAPI可以通过各种类型的用户定义函数进行扩展，但它的表达能力不如核心API，使用起来更简洁（编写代码更少）。此外，表API程序还经过优化器，该优化器在执行之前应用优化规则。

- Flink提供的最高级别抽象是SQL。这种抽象在语义和表达力上都与表API相似，但将程序表示为SQL查询表达式。SQL抽象与TableAPI密切交互，SQL查询可以在TableAPI中定义的表上执行。



## 1.3 批处理和流处理

批处理：对有界数据流的处理通常被称为批处理。批处理不需要有序地获取数据。在批处理模式下，首先将数据流持久化到存储系统（文件系统或对象存储）中，然后对整个数据集的数据进行读取、排序、统计或汇总计算，最后输出结果。

实时流处理：对于无界数据流，通常在数据生成时进行实时处理。因为无界数据流的数据输入是无限的，所以必须持续地处理。数据被获取后需要立刻处理，不可能等到所有数据都到达后再进行处理。处理无界数据流通常要求以特定顺序（如事件发生的顺序）获取事件，以便能够保证推断结果的完整性。



# 第二章 Flink环境部署

## 2.1 WordCount样例

WordCount: 单词频次统计的基本思路是:先逐行读入文件数据，然后将每一行文字拆分成单 词;接着按照单词分组，统计每组数据的个数，就是对应单词的频次。

场景介绍：



### 2.1.1 创建项目

![image-20221118232948787](/Users/zyw/Library/Application Support/typora-user-images/image-20221118232948787.png)

![image-20221118233003959](/Users/zyw/Library/Application Support/typora-user-images/image-20221118233003959.png)

### 2.1.2 添加项目依赖

```pom
    <properties>
        <maven.compiler.source>8</maven.compiler.source>
        <maven.compiler.target>8</maven.compiler.target>
        <flink.version>1.13.6</flink.version>
        <target.java.version>1.8</target.java.version>
        <scala.binary.version>2.12</scala.binary.version>
    </properties>
    <dependencies>
        <!-- 引入 Flink 相关依赖-->
        <dependency>
            <groupId>org.apache.flink</groupId>
            <artifactId>flink-scala_${scala.binary.version}</artifactId>
            <version>${flink.version}</version>
        </dependency>
        <dependency>
            <groupId>org.apache.flink</groupId>
            <artifactId>flink-streaming-scala_${scala.binary.version}</artifactId>
            <version>${flink.version}</version>
        </dependency>
        <dependency>
            <groupId>org.apache.flink</groupId>
            <artifactId>flink-clients_${scala.binary.version}</artifactId>
            <version>${flink.version}</version>
        </dependency>
    </dependencies>
```

### 2.1.3 编写代码

```java
package com.bocom.flink;

import org.apache.flink.api.common.typeinfo.Types;
import org.apache.flink.api.java.tuple.Tuple2;
import org.apache.flink.streaming.api.datastream.DataStreamSource;
import org.apache.flink.streaming.api.datastream.SingleOutputStreamOperator;
import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;
import org.apache.flink.util.Collector;

/**
 * QuickStartFlinkBatchDemo
 *
 * @author zhangyuewen
 * @since 2022/11/18
 **/
public class QuickStartFlinkBatchDemo {
    public static void main(String[] args) throws Exception {
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
        DataStreamSource<String> source = env.socketTextStream("localhost",9999);

        SingleOutputStreamOperator<Tuple2<String, Long>> wordCount = source.flatMap((String line, Collector<Tuple2<String, Long>> out) -> {
            String[] words = line.split(",");
            for (String word : words) {
                out.collect(new Tuple2<>(word, 1L));
            }
        }).returns(Types.TUPLE(Types.STRING, Types.LONG)).keyBy(word -> word.f0).sum(1);

        wordCount.print();
        env.execute();
    }
}

```

### 2.1.4 运行

- 启动端口服务发送数据。

![image-20221118234840593](/Users/zyw/Library/Application Support/typora-user-images/image-20221118234840593.png)

- 运行程序

![image-20221118234933863](/Users/zyw/Library/Application Support/typora-user-images/image-20221118234933863.png)

## 2.2 本地启动

最简单的启动方式，其实是不搭建集群，直接本地启动。本地部署非常简单，直接解压安 装包就可以使用，不用进行任何配置;一般用来做一些简单的测试。

具体安装步骤如下:

**1.下载，解压安装包**

 进入 Flink 官网，下载 1.13.6 版本安装包 flink-1.13.6-bin-scala_2.12.tgz，注意此处选用对应 scala 版本为 scala 2.12 的安装包。下载解压

```
tar -xzf flink-*.tgz
cd flink-1.13.6
```

2.**启动本地集群**

```sh
./bin/start-cluster.sh 
Starting cluster.
Starting standalonesession daemon on host 192.168.1.7.
Starting taskexecutor daemon on host 192.168.1.7.
```

**3.提交作业**

进入Flink安装目录，将WordCount项目编译打包后放入Flink安装目录。通过命令行提交作业到Flink集群

```
cp FlinkQuickStart-1.0-SNAPSHOT.jar  $FLINK_HOME/
./bin/flink run  -c com.bocom.flink.WordCount ./FlinkQuickStart-1.0-SNAPSHOT.jar
```

**4.访问WebUI**

访问http://localhost:8081/

<img src="/Users/zyw/Library/Application Support/typora-user-images/image-20221120124935725.png" alt="image-20221120124935725" style="zoom:50%;" />



**5.测试**

<img src="/Users/zyw/Library/Application Support/typora-user-images/image-20221120125124408.png" alt="image-20221120125124408" style="zoom:50%;" />

<img src="/Users/zyw/Library/Application Support/typora-user-images/image-20221120125324157.png" alt="image-20221120125324157" style="zoom:50%;" />



**6.关闭集群**

```
./bin/stop-cluster.sh 
```



## 2.3 作业运行模式

<img src="/Users/zyw/Library/Application Support/typora-user-images/image-20221119000543655.png" alt="image-20221119000543655" style="zoom:50%;" />

在一些应用场景中，对于集群资源分配和占用的方式，可能会有特定的需求。Flink 为各 种场景提供了不同的部署模式，主要有以下三种:

- 会话模式(Session Mode) 

- 单作业模式(Per-Job Mode)

- 应用模式(Application Mode)

它们的区别主要在于:

1.集群的生命周期以及资源的分配方式.

2.应用的 main 方法到底 在哪里执行——客户端(Client)还是 JobManager。

### 2.3.1 会话模式(Session Mode)

<img src="/Users/zyw/Library/Application Support/typora-user-images/image-20221119000923271.png" alt="image-20221119000923271" style="zoom:50%;" />

会话模式假定一个已经运行的集群，并使用该集群的资源来执行任何提交的应用程序。在同一（会话）集群中执行的应用程序使用并因此竞争相同的资源。这样做的好处是，您不必为每个提交的作业支付启动完整集群的资源开销。但是，如果其中一个作业行为错误或导致TaskManager崩溃，则该TaskManager上运行的所有作业都将受到故障的影响。这除了对导致故障的作业产生负面影响外，还意味着可能会有一个巨大的恢复过程，所有重新启动的作业都会同时访问文件系统，并使其不可用于其他服务。此外，让一个集群运行多个作业意味着JobManager的负载更大，JobManager负责集群中所有作业管理。

### 2.3.2 单作业模式(Per-Job Mode)

<img src="/Users/zyw/Library/Application Support/typora-user-images/image-20221119000721844.png" alt="image-20221119000721844" style="zoom:50%;" />

> Per-job mode is only supported by YARN and has been deprecated in Flink 1.15. It will be dropped in [FLINK-26000](https://issues.apache.org/jira/browse/FLINK-26000). Please consider application mode to launch a dedicated cluster per-job on YARN.

为了提供更好的资源隔离保证，*Per Job* 模式使用可用的资源提供程序框架（例如YARN）为每个提交的作业启动一个集群。此群集仅可用于该作业。当作业完成时，集群将被拆除，所有延迟的资源（文件等）将被清除。这提供了更好的资源隔离，因为错误的作业只能导致自己的TaskManagers崩溃。

### 2.3.3 应用模式(Application Mode)

<img src="/Users/zyw/Library/Application Support/typora-user-images/image-20221119001049745.png" alt="image-20221119001049745" style="zoom:50%;" />

在所有其他模式中，应用程序的main方法都在客户端执行。该过程包括本地下载应用程序的依赖项，执行main以提取Flink运行时可以理解的应用程序表示（即JobGraph），并将依赖项和JobGraph发送到集群。这使得客户端成为一个沉重的资源消耗者，因为它可能需要大量的网络带宽来下载依赖项并将二进制文件发送到集群，以及CPU周期来执行main。当客户端在用户之间共享时，此问题可能会更加突出。

基于这一观察，ApplicationMode为每个提交的应用程序创建一个集群，但这一次，应用程序的main方法由JobManager执行。为每个应用程序创建集群可以看作是创建仅在特定应用程序的作业之间共享的 Session Mode，并在应用程序完成时关闭。使用此体系结构，应用程序模式提供了与每作业模式相同的资源隔离和负载平衡保证，但其粒度是整个应用程序的粒度。

Appplication Mode 建立在一个假设之上，即用户jar已经在所有需要访问它的Flink组件（JobManager、TaskManager）的类路径（usrlib文件夹）上可用。换句话说，您的应用程序与Flink发行版捆绑在一起。这允许应用程序模式加快部署/恢复过程，而不必像其他部署模式那样通过RPC将用户jar分发到Flink组件。



> Appplication Mode 假设用户jar与Flink distribution 捆绑在一起。
> 在集群上执行main（）方法可能会对代码产生其他影响，例如，使用registerCachedFile（）在环境中注册的任何路径都必须可由应用程序的JobManager访问。

与Per-Job（已弃用）模式相比，Appplication Mode 允许提交由多个作业组成的应用程序。作业执行的顺序不受部署模式的影响，而是受用于启动作业的调用的影响。使用execute（），这是一个阻塞，它将建立一个顺序，并将导致“下一个”作业的执行被推迟到“这个”作业完成。使用executeAsync（）（非阻塞）将导致“下一个”作业在“此”作业完成之前开始。

> 应用程序模式 提供 multi-`execute()`应用程序，但在这些情况下不支持高可用性。应用程序模式下的高可用性仅支持单执行（）应用程序。
> 此外，当在应用程序模式下运行的多个作业（例如使用executesync提交）中的任何一个被取消时，所有作业都将停止，JobManager将关闭。支持定期完成作业（源关闭）。

### 2.3.4 部署方式

#### 2.3.4.1 独立模式

独立模式(Standalone)是部署 Flink 最基本也是最简单的方式:所需要的所有 Flink 组件， 都只是操作系统上运行的一个 JVM 进程。

独立模式是独立运行的，不依赖任何外部的资源管理平台;当然独立也是有代价的:如果资源不足,或者出现故障没有自动扩展或重分配资源的保证，必须手动处理。所以独立模式 一般只用在开发测试或作业非常少的场景下。

**会话模式部署** 

独立模式的特点是不依赖外部资源管理平台，而会话模式的特点是先启动集群、后提交作业。

**单作业模式部署**

Flink 本身无法直接以单作业方式启动集群，一般需要借助资源管理平台。所以 Flink 的独 立(Standalone)集群并不支持单作业模式部署。

**应用模式部署**

应用模式下不会提前创建集群，所以不能调用 start-cluster.sh 脚本。我们可以使用同样在 bin 目录下的 standalone-job.sh 来创建一个 JobManager。

#### 2.3.4.2 YARN 

整体来说，YARN 上部署的过程是:客户端把 Flink 应用提交给 Yarn 的 ResourceManager, Yarn 的 ResourceManager 会向 Yarn 的 NodeManager 申请容器。

**1.  Application Mode**

Application Mode将在YARN上启动Flink集群，在YARN中的JobManager上执行应用程序jar的main方法。应用程序完成后，集群将立即关闭。您可以使用“yarn application kill＜ApplicationId＞”或取消Flink作业来手动停止集群。

```bash
./bin/flink run-application -t yarn-application ./examples/streaming/TopSpeedWindowing.jar
```

部署Application Mode 集群后，您可以与它进行交互，以执行取消或获取保存点等操作。

```bash
# List running job on the cluster
./bin/flink list -t yarn-application -Dyarn.application.id=application_XXXX_YY
# Cancel running job
./bin/flink cancel -t yarn-application -Dyarn.application.id=application_XXXX_YY <jobId>
```

请注意，取消应用程序群集上的作业将停止群集。

可以考虑配置参数`yarn.provided.lib.dirs`配置选项，并将应用程序jar预上载到集群中所有节点都可以访问的位置。在这种情况下，命令可能如下所示：

```bash
./bin/flink run-application -t yarn-application \
	-Dyarn.provided.lib.dirs="hdfs://myhdfs/my-remote-flink-dist-dir" \
	hdfs://myhdfs/jars/my-application.jar
```

由于所需的Flink jar和应用程序jar将由指定的远程位置拾取，而不是由客户机运送到集群，因此上述内容将使作业提交变得更加轻量级。

**2. Session Mode**

会话模式有两种操作模式：
附加模式：`yarn-session.sh`客户端将Flink集群提交给YARN，但客户端继续运行，跟踪集群的状态。如果集群失败，客户端将显示错误。如果客户端被终止，它也会发出集群关闭的信号。
分离模式（“-d”或“--separated”）：`yarn-sessoin.sh`客户端将Flink集群提交给YARN，然后客户端返回。需要再次调用客户端或YARN工具来停止Flink集群。

Session Mode 将在`/tmp/.YARN properties-＜username＞`中创建一个隐藏的YARN属性文件，提交作业时，命令行界面将根据该文件进行集群发现。

您还可以**在提交Flink作业时，在命令行界面中手动指定目标YARN集群**。

```bash
./bin/flink run -t yarn-session \
  -Dyarn.application.id=application_XXXX_YY \
  ./examples/streaming/TopSpeedWindowing.jar
```

您可以使用以下命令重新连接到YARN会话：

```
./bin/yarn-session.sh -id application_XXXX_YY
```

除了通过`conf/frink-conf.yaml`文件，您还可以在提交时将任何配置传递给`./bin/yarn-session.sh`客户端使用`-Dkey=value`参数。

**3. Per-Job Mode (deprecated)** 

> Per-job mode is only supported by YARN and has been deprecated in Flink 1.15. It will be dropped in [FLINK-26000](https://issues.apache.org/jira/browse/FLINK-26000). Please consider application mode to launch a dedicated cluster per-job on YARN.

Per-Job 模式将在YARN上启动Flink集群，然后在本地运行提供的应用程序jar，最后将JobGraph提交给YARN上的JobManager。如果传递`--detached`参数，则客户端将在接受提交后停止。

作业停止后，YARN集群将停止。

```bash
./bin/flink run -t yarn-per-job --detached ./examples/streaming/TopSpeedWindowing.jar
```

一旦部署了Per-Job Mode，您就可以与它进行交互以执行取消或获取保存点等操作。

```bash
# List running job on the cluster
./bin/flink list -t yarn-per-job -Dyarn.application.id=application_XXXX_YY
# Cancel running job
./bin/flink cancel -t yarn-per-job -Dyarn.application.id=application_XXXX_YY <jobId>
```

请注意，取消每作业群集上的作业将停止群集。

#### 2.3.4.3 Kubernetes

待补充



# 第三章 Flink运行时架构

## 3.1 概述和体系结构

下图显示了每个Flink集群的组件。一个客户端在运行。它获取Flink应用程序的代码，将其转换为JobGraph并提交JobManager。JobManager将工作分配到TaskManager上。在部署Flink时，每个Component通常有多个可用选项。我们在下图的表格中列出了它们。

<img src="/Users/zyw/Library/Application Support/typora-user-images/image-20221119005602151.png" alt="image-20221119005602151" style="zoom:50%;" />



| 组件                                     | 作用                                                         | 实现                                                         |
| :--------------------------------------- | :----------------------------------------------------------- | :----------------------------------------------------------- |
| Flink Client                             | 将批处理或流式应用程序编译成 dataflow graph，然后提交给JobManager。 | Command Line Interface、REST Endpoint、SQL Client            |
| JobManager                               | JobManager是Flink的中心工作协调组件的名称。针对不同的资源管理器提供了实现，这些资源管理者在高可用性、资源分配行为和支持的作业提交模式方面有所不同。<br/>**应用程序模式**：仅为一个应用程序运行集群。作业的主方法在JobManager上执行。支持在应用程序中多次调用`execute/executesync` .<br/>**Per-Job模式**:仅为一个作业运行群集。作业的主方法（或客户端）仅在创建集群之前运行.<br/>**会话模式**：一个JobManager实例管理共享同一TaskManager集群的多个作业 | Standalone、Kubernetes、YARN                                 |
| TaskManager                              | TaskManagers是实际执行Flink作业的服务。                      |                                                              |



## 3.2 系统架构

<img src="/Users/zyw/Library/Application Support/typora-user-images/image-20221119010158616.png" alt="image-20221119010158616" style="zoom:50%;" />

The *Client* is not part of the runtime and program execution, but is used to prepare and send a dataflow to the JobManager. After that, the client can disconnect (*detached mode*), or stay connected to receive progress reports (*attached mode*). The client runs either as part of the Java/Scala program that triggers the execution, or in the command line process `./bin/flink run ...`.

The JobManager and TaskManagers can be started in various ways: directly on the machines as a [standalone cluster](https://nightlies.apache.org/flink/flink-docs-release-1.16/docs/deployment/resource-providers/standalone/overview/), in containers, or managed by resource frameworks like [YARN](https://nightlies.apache.org/flink/flink-docs-release-1.16/docs/deployment/resource-providers/yarn/). TaskManagers connect to JobManagers, announcing themselves as available, and are assigned work.

### JobManager

JobManager 是一个 Flink 集群中任务管理和调度的核心，是控制应用执行的主进程。也就 是说，每个应用都应该被唯一的 JobManager 所控制执行。

JobManger 又包含 3 个不同的组件。

**1. JobMaster**

JobMaster 是 JobManager 中最核心的组件，负责处理单独的作业(job)。JobMaster 和具体的 job 是一一对应的，多个 job 可以同时运行在一个 Flink 集群中, 每个 job 都有一个自己的 JobMaster。

在作业提交时，JobMaster 会先接收到要执行的应用。JobMaster 会把 JobGraph 转换成一 个物理层面的数据流图，这个图被叫作“执行图”(ExecutionGraph)，它包含了所有可以并发执行的任务。JobMaster 会向资源管理器(ResourceManager)发出请求，申请执行任务必要的资源。一旦它获取到了足够的资源，就会将执行图分发到真正运行它们的 TaskManager 上。

而在运行过程中，JobMaster 会负责所有需要中央协调的操作，比如说检查点(checkpoints) 的协调。

**2. 资源管理器(ResourceManager)**

ResourceManager 主要负责资源的分配和管理，在 Flink 集群中只有一个。所谓“资源”， 主要是指 TaskManager 的任务槽(task slots)。任务槽就是 Flink 集群中的资源调配单元，包含了机器用来执行计算的一组 CPU 和内存资源。每一个任务(task)都需要分配到一个 slot 上 执行。

Flink 的 ResourceManager，针对不同的环境和资源管理平台(比如 Standalone 部署，或者 YARN)，有不同的具体实现。在 Standalone 部署时，因为 TaskManager 是单独启动的(没有 Per-Job 模式)，所以 ResourceManager 只能分发可用 TaskManager 的任务槽，不能单独启动新 TaskManager。

而在有资源管理平台时，就不受此限制。当新的作业申请资源时，ResourceManager 会将有空闲槽位的 TaskManager 分配给 JobMaster。如果 ResourceManager 没有足够的任务槽，它还可以向资源提供平台发起会话，请求提供启动 TaskManager 进程的容器。另外， ResourceManager 还负责停掉空闲的 TaskManager，释放计算资源。

**3. 分发器(Dispatcher)**

Dispatcher 主要负责提供一个 REST 接口，用来提交作业，并且负责为每一个新提交的作 业启动一个新的 JobMaster 组件。Dispatcher 也会启动一个 Web UI，用来方便地展示和监控作 业执行的信息。Dispatcher 在架构中并不是必需的，在不同的部署模式下可能会被忽略掉。

### TaskManagers

TaskManager 是 Flink 中的工作进程，负责数据流的具体计算任务(task)。Flink 集群中必须至少有一个 TaskManager;当然由于分布式计算的考虑，通常会有多个 TaskManager 运行， 每一个 TaskManager 都包含了一定数量的任务槽(task slots)。Slot 是资源调度的最小单位，slots 的数量限制了 TaskManager 能够并行处理的任务数量。

启动之后，TaskManager 会向资源管理器注册它的 slots;收到资源管理器的指令后， TaskManager就会将一个或者多个槽位提供给JobMaster调用，JobMaster就可以分配任务来执行了。

在执行过程中，TaskManager 可以缓冲数据，还可以跟其他运行同一应用的 TaskManager 交换数据。

## 3.2 作业提交流程

### 3.2.1 高层级抽象视角

Flink 的提交流程，随着部署模式、资源管理平台的不同，会有不同的变化。首先我们从 一个高层级的视角，来做一下抽象提炼，看一看作业提交时宏观上各组件是怎样交互协作的。

<img src="/Users/zyw/Library/Application Support/typora-user-images/image-20221120125826628.png" alt="image-20221120125826628" style="zoom:50%;" />

具体步骤如下:
(1)一般情况下，由客户端(App)通过分发器提供的 REST 接口，将作业提交给 JobManager。

(2)由分发器启动 JobMaster，并将作业(包含 JobGraph)提交给 JobMaster。

(3)JobMaster 将 JobGraph 解析为可执行的 ExecutionGraph，计算所需的资源数量，然后向资源管理器请求任务槽资源(slots)。

(4)资源管理器判断当前是否由足够的可用资源;如果没有，启动新的 TaskManager。

(5)TaskManager 启动之后，向 ResourceManager 注册自己的可用任务槽(slots)。 

(6)资源管理器通知 TaskManager 为新的作业提供 slots。

(7)TaskManager 连接到对应的 JobMaster，提供 slots。

(8)JobMaster 将需要执行的任务分发给 TaskManager。 

(9)TaskManager 执行任务，互相之间可以交换数据。

如果部署模式不同，或者集群环境不同(例如 Standalone、YARN、K8S 等)，其中一些步骤可能会不同或被省略，也可能有些组件会运行在同一个 JVM 进程中。

### 3.2.2 YARN 集群

**1.会话模式**

在会话模式下，我们需要先启动一个 YARN session，这个会话会创建一个 Flink 集群。

<img src="/Users/zyw/Library/Application Support/typora-user-images/image-20221120130312041.png" alt="image-20221120130312041" style="zoom:50%;" />

这里只启动了 JobManager，而 TaskManager 可以根据需要动态地启动。在 JobManager 内部，由于还没有提交作业，所以只有 ResourceManager 和 Dispatcher 在运行。

<img src="/Users/zyw/Library/Application Support/typora-user-images/image-20221120130351053.png" alt="image-20221120130351053" style="zoom:50%;" />

下面就是真正提交作业的流程

1. 客户端通过 REST 接口，将作业提交给分发器。
2. 分发器启动 JobMaster，并将作业(包含 JobGraph)提交给 JobMaster。
3. JobMaster 向资源管理器请求资源(slots)。
4. 资源管理器向 YARN 的资源管理器请求 container 资源。
5. YARN 启动新的 TaskManager 容器。
6. TaskManager 启动之后，向 Flink 的资源管理器注册自己的可用任务槽。
7. 资源管理器通知 TaskManager 为新的作业提供 slots。
8. TaskManager 连接到对应的 JobMaster，提供 slots。
9. JobMaster 将需要执行的任务分发给 TaskManager，执行任务。



**2.单作业模式**

在单作业模式下，Flink 集群不会预先启动，而是在提交作业时，才启动新的 JobManager。

具体流程如图所示

<img src="/Users/zyw/Library/Application Support/typora-user-images/image-20221120130714376.png" alt="image-20221120130714376" style="zoom:50%;" />

1. 客户端将作业提交给 YARN 的资源管理器，这一步中会同时将 Flink 的 Jar 包和配置上传到 HDFS，以便后续启动 Flink 相关组件的容器。
2. YARN的资源管理器分配容器(container)资源，启动Flink JobManager，并将作业提交给 JobMaster。这里省略了 Dispatcher 组件。
3. JobMaster 向资源管理器请求资源(slots)。
4. 资源管理器向 YARN 的资源管理器请求容器(container)。
5. YARN 启动新的 TaskManager 容器。
6. TaskManager 启动之后，向 Flink 的资源管理器注册自己的可用任务槽。
7. 资源管理器通知 TaskManager 为新的作业提供 slots。
8. TaskManager 连接到对应的 JobMaster，提供 slots。
   JobMaster 将需要执行的任务分发给 TaskManager，执行任务。

**3. 应用(Application)模式**

应用模式与单作业模式的提交流程非常相似，只是初始提交给 YARN 资源管理器的不再是具体的作业，而是整个应用。

## 3.3 重要概念

### 3.3.1 数据流图(Dataflow Graph)

Flink 是流式计算框架。它的程序结构，其实就是定义了一连串的处理操作，每一个数据 输入之后都会依次调用每一步计算。在 Flink 代码中，我们定义的每一个处理转换操作都叫作“算子”(Operator)。所有的 Flink 程序都可以归纳为由三部分构成:Source、Transformation 和 Sink。
 ⚫ Source表示“源算子”，负责读取数据源。
 ⚫ Transformation表示“转换算子”，利用各种算子进行处理加工。
 ⚫ Sink表示“下沉算子”，负责数据的输出。
 在运行时，Flink 程序会被映射成所有算子按照逻辑顺序连接在一起的一张图，这被称为“逻辑数据流”(logical dataflow)，或者叫“数据流图”(dataflow graph)。在数据流图中，可以清楚地看到 Source、Transformation、Sink 三部分。



### 3.3.2 并行度(Parallelism)

#### 3.3.2.1并行子任务和并行度

把一个算子操作，“复制”多份到多个节点，数据来了之后就可以到其中任意一个执行。 这样一来，一个算子操作就被拆分成了多个并行的“子任务”(subtasks)，再将它们分发到不 同节点，就真正实现了并行计算。

在 Flink 执行过程中，每一个算子(operator)可以包含一个或多个子任务(operator subtask)， 这些子任务在不同的线程、不同的物理机或不同的容器中完全独立地执行。

<img src="/Users/zyw/Library/Application Support/typora-user-images/image-20221120171505270.png" alt="image-20221120171505270" style="zoom:50%;" />

一个特定算子的子任务(subtask)的个数被称之为其并行度(parallelism)。这样，包含并行子任务的数据流，就是并行数据流，它需要多个分区(stream partition)来分配并行任务。 一般情况下，一个流程序的并行度，可以认为就是其所有算子中最大的并行度。一个程序中， 不同的算子可能具有不同的并行度。

如图所示，当前数据流中有 Source、map()、keyBy()/window()/apply()、Sink 四个算子， 除最后 Sink，其他算子的并行度都为 2。整个程序包含了 7 个子任务，至少需要 2 个分区来并行执行。我们可以说，这段流处理程序的并行度就是 2。

#### 3.3.2.1并行度的设置

在 Flink 中，可以用不同的方法来设置并行度，它们的有效范围和优先级别也是不同的。

 **(1)代码中设置**

我们在代码中，可以很简单地在算子后跟着调用 setParallelism()方法，来设置当前算子的

并行度:

```java
final StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();

DataStream<String> text = [...];
DataStream<Tuple2<String, Integer>> wordCounts = text
    .flatMap(new LineSplitter())
    .keyBy(value -> value.f0)
    .window(TumblingEventTimeWindows.of(Time.seconds(5)))
    .sum(1).setParallelism(5);

wordCounts.print();

env.execute("Word Count Example");
```

另外，我们也可以直接调用执行环境的 setParallelism()方法，全局设定并行度。

```
final StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
env.setParallelism(3);

DataStream<String> text = [...];
DataStream<Tuple2<String, Integer>> wordCounts = [...];
wordCounts.print();

env.execute("Word Count Example");
```

这样代码中所有算子，默认的并行度就都为 3了。我们一般不会在程序中设置全局并行度， 因为如果在程序中对全局并行度进行硬编码，会导致无法动态扩容。

**(2)提交作业时设置**

在使用flink run命令提交作业时，可以增加-p参数来指定当前应用程序执行的并行度， 它的作用类似于执行环境的全局设置:

如果我们直接在 Web UI 上提交作业，也可以在对应输入框中直接添加并行度。

```
bin/flink run –p 2 ...
```

**(3)配置文件中设置**

我们还可以直接在集群的配置文件 flink-conf.yaml 中直接更改默认并行度:

```
parallelism.default: 2
```

这个设置对于整个集群上提交的所有作业有效，初始值为 1。无论在代码中设置、还是提交时的-p 参数，都不是必须的。所以，在没有指定并行度的时候，就会采用配置文件中的集群默认并行度。

### 3.3.3 算子链(Operator Chain)

对于分布式执行，Flink将每个子任务（operator subtasks）连在一起形成任务（tasks）。每个任务由一个线程执行。算子链(Operator Chain)是一种有用的优化：它减少了线程间切换和缓冲的开销，并在降低延迟的同时增加了总吞吐量。

仔细观察 Web UI 上给出的图，如图所示，上面的节点似乎跟代码中的算子又不是一一对应的。

![image-20221120174145292](/Users/zyw/Library/Application Support/typora-user-images/image-20221120174145292.png)



很明显，这里的一个节点，会把转换处理的很多个任务都连接在一起，合并成了一个“大任务”。

#### 3.3.3.1 算子任务间的数据传输 

我们先来考察一下算子任务之间数据传输的方式。

![image-20221120174224696](/Users/zyw/Library/Application Support/typora-user-images/image-20221120174224696.png)

如图所示，一个数据流在算子之间传输数据的形式可以是一对一(one-to-one)的直通 (forwarding)模式，也可以是打乱的重分区(redistributing)模式，具体是哪一种形式，取决于算子的种类。

(1)一对一(One-to-one，forwarding) 

这种模式下，数据流维护着分区以及元素的顺序。比如图中的 Source 和 map()算子，Source算子读取数据之后，可以直接发送给 map()算子做处理，它们之间不需要重新分区，也不需要调整数据的顺序。这就意味着 map() 算子的子任务，看到的元素个数和顺序跟 Source 算子的子任务产生的完全一样，保证着“一对一”的关系。map()、filter()、flatMap()等算子都是这种 one-to-one 的对应关系。

(2)重分区(Redistributing) 

在这种模式下，数据流的分区会发生改变。如图中的 map()和后面的keyBy()/window()/apply()算子之间(这里的 keyBy()是数据传输方法，后面的 window()、apply() 方法共同构成了窗口算子)，以及窗口算子和 Sink 算子之间，都是这样的关系。

每一个算子的子任务，会根据数据传输的策略，把数据发送到不同的下游目标任务。例如，keyBy()是分组操作，本质上基于键(key)的哈希值(hashCode)进行了重分区;而当并行度 改变时，比如从并行度为 2 的 window 算子，要传递到并行度为 1 的 Sink 算子，这时的数据 传输方式是再平衡(rebalance)，会把数据均匀地向下游子任务分发出去。这些传输方式都会 引起重分区(redistribute)的过程，这一过程类似于 Spark 中的 shuffle。

#### 3.3.3.2.合并算子链

在 Flink 中，并行度相同的一对一(one to one)算子操作，可以直接链接在一起形成一个 “大”的任务(task)，这样原来的算子就成为真正任务里的一部分，如图所示。每个 task会被一个线程执行。这样的技术被称为“算子链”(Operator Chain)。

<img src="/Users/zyw/Library/Application Support/typora-user-images/image-20221120174626111.png" alt="image-20221120174626111" style="zoom:50%;" />

Flink 默认会按照算子链的原则进行链接合并，如果我们想要禁止合并或者自行定义，也可以在代码中对算子做一些特定的设置:

```
.map((_,1)).disableChaining() 
// 从当前算子开始新链 
.map((_,1)).startNewChain()
```

### 3.3.4 作业图(JobGraph)与执行图(ExecutionGraph)

由 Flink 程序直接映射成的数据流图(dataflow graph)，也被称为逻辑流图(logical StreamGraph)。到具体执行环节时，Flink 需要进一步将逻辑流图进行解析，转换为物理执行 图。

在这个转换过程中，有几个不同的阶段，会生成不同层级的图，其中最重要的就是作业图 (JobGraph)和执行图(ExecutionGraph)。Flink 中任务调度执行的图，按照生成顺序可以分成四层:
 逻辑流图(StreamGraph)→ 作业图(JobGraph)→ 执行图(ExecutionGraph)→ 物理图(Physical Graph)。

#### 3.3.4.1编译阶段生成JobGraph

<img src="/Users/zyw/Library/Application Support/typora-user-images/image-20211124181347616.png" alt="image-20211124181347616" />



#### 3.3.4.2 运行阶段生成调度ExecutionGraph

![image-20211124181400951](/Users/zyw/Library/Application Support/typora-user-images/image-20211124181400951.png)

### 4.3.5 任务槽(Task Slots)和资源(Resources)

#### 4.3.5.1 Task Slots

每个TaskManager都是一个*JVM进程*，可以在单独的线程中执行一个或多个子任务。为了控制TaskManager接受多少任务，它有所谓的**任务槽**（至少一个）。

每个*任务槽*表示TaskManager的固定资源子集。例如，具有三个Task Slots的TaskManager将为每个插槽分配1/3的托管内存。分配资源意味着子任务不会与其他作业的子任务竞争托管内存，而是保留一定数量的托管内存。注意这里没有CPU隔离；当前，插槽仅将任务的托管内存分开。

通过调整任务槽的数量，用户可以定义子任务之间的隔离方式。每个TaskManager有一个插槽意味着每个任务组都在单独的JVM中运行（例如，可以在单独的容器中启动）。具有多个插槽意味着更多的子任务共享同一个JVM。同一JVM中的任务共享TCP连接（通过多路复用）和心跳消息。它们还可以共享数据集和数据结构，从而减少每个任务的开销。

<img src="/Users/zyw/Library/Application Support/typora-user-images/image-20221119010601075.png" alt="image-20221119010601075" style="zoom:50%;" />

#### 4.3.5.2 Task Slots的设置

可以通过集群的配置文件来设定 TaskManager 的 slots 数量:

```
taskmanager.numberOfTaskSlots: 8
```

####  4.3.5.3 任务对任务槽的共享

默认情况下，Flink允许substasks 共享插槽，即使它们是不同任务的子任务，只要它们来自同一作业。

一个task slots 可以容纳整个作业管道。允许这种“插槽共享”有两个主要好处：

- Flink集群需要的slots固定。数量与作业的最高并行度完全相同的任务槽。这样就无需计算一个程序总共包含多少任务（具有不同的并行度）。

- 更容易获得更好的资源利用率。如果没有插槽共享，非密集型*source/map（）*子任务和阻塞与资源密集型*window*子任务需要一样多的资源。通过时隙共享，将示例中的基本并行度从两个增加到六个，可以充分利用时隙资源，同时确保在TaskManager之间公平分配繁重的子任务。

<img src="/Users/zyw/Library/Application Support/typora-user-images/image-20221119010617192.png" alt="image-20221119010617192" style="zoom:50%;" />

如果希望某个算子对应的任务完全独占一个 slot，或者只有某一部分算子共享 slots，我们

也可以通过设置“slot 共享组”(SlotSharingGroup)手动指定: 

```
.map((_,1)).slotSharingGroup(“1”);
```

这样，只有属于同一个 slot 共享组的子任务，才会开启 slots 共享;不同组之间的任务是 完全隔离的，必须分配到不同的 slots 上。

**4.3.5.4 任务槽和并行度的关系**

slots 和并行度确实都跟程序的并行执行有关，但两者是完全不同的概念。简单来说，task slots 是静态的概念，是指 TaskManager 具有的并发执行能力，可以通过参数 taskmanager.numberOfTaskSlots 进行配置;而并行度(parallelism)是动态概念，也就是 TaskManager 运行程序时实际使用的并发能力，可以通过参数 parallelism.default 进行配置。

<img src="/Users/zyw/Library/Application Support/typora-user-images/image-20221120181709880.png" alt="image-20221120181709880" style="zoom:50%;" />



<img src="/Users/zyw/Library/Application Support/typora-user-images/image-20221120181738896.png" alt="image-20221120181738896" style="zoom:50%;" />

<img src="/Users/zyw/Library/Application Support/typora-user-images/image-20221120181749872.png" alt="image-20221120181749872" style="zoom:50%;" />

<img src="/Users/zyw/Library/Application Support/typora-user-images/image-20221120181804449.png" alt="image-20221120181804449" style="zoom:50%;" />

<img src="/Users/zyw/Library/Application Support/typora-user-images/image-20221120181815406.png" alt="image-20221120181815406" style="zoom:50%;" />



#### 4.3.5.4 Task之间的数据交互

![image-20211124181430903](/Users/zyw/Library/Application Support/typora-user-images/image-20211124181430903.png)

上图代表了一个简单的 map-reduce 类型的作业，有两个并行的任务。有两个 TaskManager，每个 TaskManager 都分别运行一个 map Task 和一个 reduce Task。我们重点观察 M1 和 R2 这两个 Task 之间的数据传输的发起过程。数据传输用粗箭头表示，消息用细箭头表示。首先，M1 产出了一个 ResultPartition(RP1)（箭头1）。当这个 RP 可以被消费是，会告知 JobManager（箭头2）。JobManager 会通知想要接收这个 RP 分区数据的接收者（tasks R1 and R2）当前分区数据已经准备好。如果接受放还没有被调度，这将会触发对应任务的部署（箭头 3a，3b）。接着，接受方会从 RP 中请求数据（箭头 4a，4b）。这将会初始化 Task 之间的数据传输（5a,5b）,数据传输可能是本地的(5a)，也可能是通过 TaskManager 的网络栈进行（5b）

对于一个 RP 什么时候告知 JobManager 当前已经出于可用状态，在这个过程中是有充分的自由度的：例如，如果在 RP1 在告知 JVM 之前已经完整地产出了所有的数据（甚至可能写入了本地文件），那么相应的数据传输更类似于 Batch 的批交换；如果 RP1 在第一条记录产出时就告知 JM，那么就是 Streaming 流交换。

![image-20211124181454080](/Users/zyw/Library/Application Support/typora-user-images/image-20211124181454080.png)



1. `ResultPartition as RP` 和 `ResultSubpartition as RS`
   ExecutionGraph 是 JobManager 中用于描述作业拓扑的一种逻辑上的数据结构，其中表示并行子任务的 `ExecutionVertex` 会被调度到 `TaskManager` 中执行，一个 Task 对应一个 ExecutionVertex。同 ExecutionVertex 的输出结果 IntermediateResultPartition 相对应的Task的输出结果则是 `ResultPartition`。IntermediateResultPartition 可能会有多个 ExecutionEdge 作为消费者，那么在 Task 这里，ResultPartition 就会被拆分为多个 `ResultSubpartition`，下游每一个需要从当前 ResultPartition 消费数据的 Task 都会有一个专属的 `ResultSubpartition`。
   `ResultPartitionType`指定了`ResultPartition` 的不同属性，这些属性包括是否流水线模式、是否会产生反压以及是否限制使用的 Network buffer 的数量。`enum ResultPartitionType` 有三个枚举值：
   BLOCKING：非流水线模式，无反压，不限制使用的网络缓冲的数量
   PIPELINED：流水线模式，有反压，不限制使用的网络缓冲的数量
   PIPELINED_BOUNDED：流水线模式，有反压，限制使用的网络缓冲的数量

2. `InputGate as IG` 和 `InputChannel as IC`
   在 `Task` 中，`InputGate`是对输入的封装，`InputGate` 是和 `JobGraph` 中 `JobEdge` 一一对应的。也就是说，`InputGate` 实际上对应的是该 `Task` 依赖的上游算子（包含多个并行子任务），每个 `InputGate` 消费了一个或多个 `ResultPartition`。`InputGate` 由 `InputChannel` 构成，`InputChannel` 和`ExecutionEdge` 一一对应；也就是说， `InputChannel` 和 `ResultSubpartition` 一一相连，一个 `InputChannel`接收一个`ResultSubpartition` 的输出。根据读取的`ResultSubpartition` 的位置，`InputChannel` 有 `LocalInputChannel` 和 `RemoteInputChannel` 两种不同的实现。

# 第四章 Flink的时间和窗口

## 4.1 时间语义

<img src="/Users/zyw/Library/Application Support/typora-user-images/image-20221120190905420.png" alt="image-20221120190905420" style="zoom:50%;" />

1. 处理时间(Processing Time)

处理时间的概念非常简单，就是指执行处理操作的机器的系统时间。

2. 事件时间(Event Time)

事件时间，是指每个事件在对应的设备上发生的时间，也就是数据生成的时间。

数据一旦产生，这个时间自然就确定了，所以它可以作为一个属性嵌入到数据中。这其实 就是这条数据记录的“时间戳”(Timestamp)。

## 4.2 水位线(WaterMark)

### 4.2.1 什么是水位线

在事件时间语义下，我们不依赖系统时间，而是基于数据自带的时间戳去定义了一个时钟， 用来表示当前时间的进展。于是每个并行子任务都会有一个自己的逻辑时钟，它的前进是靠数 据的时间戳来驱动的。

我们可以把时钟也以数据的形式传递出去，告诉下游任务当前时间的进展;而且这个时钟 的传递不会因为窗口聚合之类的运算而停滞。一种简单的想法是，在数据流中加入一个时钟标 记，记录当前的事件时间;这个标记可以直接广播到下游，当下游任务收到这个标记，就可以 更新自己的时钟了。由于类似于水流中用来做标志的记号，在 Flink 中，这种用来衡量事件时 间(Event Time)进展的标记，就被称作“水位线”(Watermark)。

具体实现上，水位线可以看作一条特殊的数据记录，它是插入到数据流中的一个标记点， 主要内容就是一个时间戳，用来指示当前的事件时间。而它插入流中的位置，就应该是在某个 数据到来之后;这样就可以从这个数据中提取时间戳，作为当前水位线的时间戳了。

<img src="/Users/zyw/Library/Application Support/typora-user-images/image-20221120191438245.png" alt="image-20221120191438245" style="zoom:50%;" />

1. 有序流中的水位线

   在理想状态下，数据应该按照它们生成的先后顺序、排好队进入流中;而在实际应用中， 如果当前数据量非常大，可能会有很多数据的时间戳是相同的，这时每来一条数据就提取时间 戳、插入水位线就做了大量的无用功。所以为了提高效率，一般会每隔一段时间生成一个水位线，这个水位线的时间戳，就是当前最新数据的时间戳，如图所示。所以这时的水位线， 其实就是有序流中的一个周期性出现的时间标记。

<img src="/Users/zyw/Library/Application Support/typora-user-images/image-20221120191542549.png" alt="image-20221120191542549" style="zoom:50%;" />

2. 乱序流中的水位线

   在分布式系统中，数据在节点间传输，会因为网络传输延迟的不确定性，导致顺序发生改变，这就是所谓的“乱序数据”。

<img src="/Users/zyw/Library/Application Support/typora-user-images/image-20221120191650988.png" alt="image-20221120191650988" style="zoom:50%;" />

对于连续数据流，我们插入新的水位线时，要先判断一下时间戳是否比之前的大，否则就 不再生成新的水位线，如图所示。也就是说，只有数据的时间戳比当前时钟大，才能推动时钟前进，这时才插入水位线。

<img src="/Users/zyw/Library/Application Support/typora-user-images/image-20221120191723419.png" alt="image-20221120191723419" style="zoom:50%;" />



如果考虑到大量数据同时到来的处理效率，我们同样可以周期性地生成水位线。这时只需要保存一下之前所有数据中的最大时间戳，需要插入水位线时，就直接以它作为时间戳生成新的水位线，如图所示。

<img src="/Users/zyw/Library/Application Support/typora-user-images/image-20221120191759869.png" alt="image-20221120191759869" style="zoom:50%;" />

为了让窗口能够正确收集到迟到的数据，我们可以等上 2 秒;也就是用当前已有数据的最大时间戳减去 2 秒，就是要插入的水位线的时间戳，如图所示

<img src="/Users/zyw/Library/Application Support/typora-user-images/image-20221120191843537.png" alt="image-20221120191843537" style="zoom:50%;" />

如果仔细观察就会看到，这种“等 2 秒”的策略其实并不能处理所有的乱序数据。所以我 们可以试着多等几秒，也就是把时钟调得更慢一些。最终的目的，就是要让窗口能够把所有迟 到数据都收进来，得到正确的计算结果。对应到水位线上，其实就是要保证，当前时间已经进展到了这个时间戳，在这之后不可能再有迟到数据来了。

总结一下水位线的特性:

⚫  水位线是插入到数据流中的一个标记，可以认为是一个特殊的数据

⚫  水位线主要的内容是一个时间戳，用来表示当前事件时间的进展

⚫  水位线是基于数据的时间戳生成的

⚫  水位线的时间戳必须单调递增，以确保任务的事件时间时钟一直向前推进

⚫  水位线可以通过设置延迟，来保证正确处理乱序数据

⚫  一个水位线 Watermark(t)，表示在当前流中事件时间已经达到了时间戳 t, 这代表 t 之前的所有数据都到齐了，之后流中不会出现时间戳 t’ ≤ t 的数据。

水位线是 Flink 流处理中保证结果正确性的核心机制，它往往会跟窗口一起配合，完成对乱序数据的正确处理。

## 4.3 窗口(Window)

### 4.3.1 窗口的概念

Flink 是一种流式计算引擎，主要是来处理无界数据流的，数据源源不断、无穷无尽。想 要更加方便高效地处理无界流，一种方式就是将无限数据切割成有限的“数据块”进行处理，这 就是所谓的“窗口”(Window)。在 Flink 中, 窗口就是用来处理无界流的核心。

<img src="/Users/zyw/Library/Application Support/typora-user-images/image-20221121132227818.png" alt="image-20221121132227818" style="zoom:50%;" />



这里注意为了明确数据划分到哪一个窗口，定义窗口都是包含起始时间、不包含结束时间 的，用数学符号表示就是一个左闭右开的区间。

<img src="/Users/zyw/Library/Application Support/typora-user-images/image-20221121132326357.png" alt="image-20221121132326357" style="zoom:50%;" />

对于处理时间下的窗口而言，这样理解似乎没什么问题。然而如果我们采用事件时间语义， 就会有些令人费解了。由于有乱序数据，我们需要设置一个延迟时间来等所有数据到齐。比如上面的例子中，我们可以设置延迟时间为 2 秒，如图所示，这样 0~10 秒的窗口会在时间戳为 12 的数据到来之后，才真正关闭计算输出结果，这样就可以正常包含迟到的 9 秒数据了。

<img src="/Users/zyw/Library/Application Support/typora-user-images/image-20221121132413405.png" alt="image-20221121132413405" style="zoom:50%;" />

但是这样一来，0~10 秒的窗口不光包含了迟到的 9 秒数据，连 11 秒和 12 秒的数据也包 含进去了。我们为了正确处理迟到数据，结果把早到的数据划分到了错误的窗口——最终结果都是错误的。

<img src="/Users/zyw/Library/Application Support/typora-user-images/image-20221121132449076.png" alt="image-20221121132449076" style="zoom:50%;" />

所以在 Flink 中，窗口其实并不是一个“框”，流进来的数据被框住了就只能进这一个窗 口。相比之下，我们应该把窗口理解成一个“桶”，如图所示。在 Flink 中，窗口可以把流切割成有限大小的多个“存储桶”(bucket);每个数据都会分发到对应的桶中，当到达窗口 结束时间时，就对每个桶中收集的数据进行计算处理。

这里需要注意的是，Flink 中窗口并不是静态准备好的，而是动态创建——当有落在这个 窗口区间范围的数据达到时，才创建对应的窗口。另外，这里我们认为到达窗口结束时间时， 窗口就触发计算并关闭，事实上“触发计算”和“窗口关闭”两个行为也可以分开。



### 4.3.2 窗口的分类

在 Flink 中，窗口的应用非常 灵活，我们可以使用各种不同类型的窗口来实现需求。接下来我们就从不同的角度，对 Flink 中内置的窗口做一个分类说明。

#### 4.3.2.1 按照驱动类型分类

窗口本身是截取有界数据的一种方式，所以窗口一个非常重要的信息其实就是“怎样截取数据”。换句话说，就是以什么标准来开始和结束数据的截取，我们把它叫作窗口的“驱动类型”。

我们最容易想到的就是按照时间段去截取数据，这种窗口就叫作“时间窗口”(Time Window)。这在实际应用中最常见，之前所举的例子也都是时间窗口。除了由时间驱动之外， 窗口其实也可以由数据驱动，也就是说按照固定的个数，来截取一段数据集，这种窗口叫作“计 数窗口”(Count Window)。





# 第五章 状态机制(State)与容错机制(checkpoint)

## 5.1 Flink中的状态

在流处理中，数据是连续不断到来和处理的。每个任务进行计算处理时，可以基于当前数据直接转换得到输出结果;也可以依赖一些其他数据。这些由一个任务维护，并且用来计算输 出结果的所有数据，就叫作这个任务的状态。

## 5.2 状态的分类

1. 托管状态(Managed State)和原始状态(Raw State)

Flink 的状态有两种:托管状态(Managed State)和原始状态(Raw State)。托管状态就是 由 Flink 统一管理的，状态的存储访问、故障恢复和重组等一系列问题都由 Flink 实现，我们 只要调接口就可以;而原始状态则是自定义的，相当于就是开辟了一块内存，需要我们自己管 理，实现状态的序列化和故障恢复。

具体来讲，托管状态是由 Flink 的运行时(Runtime)来托管的;在配置容错机制后，状 态会自动持久化保存，并在发生故障时自动恢复。当应用发生横向扩展时，状态也会自动地重 组分配到所有的子任务实例上。

而对比之下，原始状态就全部需要自定义了。Flink 不会对状态进行任何自动操作，也不 知道状态的具体数据类型，只会把它当作最原始的字节(Byte)数组来存储。我们需要花费大 量的精力来处理状态的管理和维护。

所以只有在遇到托管状态无法实现的特殊需求时，我们才会考虑使用原始状态;一般情况 下不推荐使用。绝大多数应用场景，我们都可以用 Flink 提供的算子或者自定义托管状态来实 现需求。

2. 算子状态(Operator State)和按键分区状态(Keyed State)

接下来我们的重点就是托管状态(Managed State)。
 我们知道在 Flink 中，一个算子任务会按照并行度分为多个并行子任务执行，而不同的子

### 5.2.1 Keyed State

待补充

### 5.2.2 Operator State

待补充

## 5.3 检查点(checkpoint)

在Flink中，有一套完整的容错机制(fault tolerance)来保证故障后的恢复，其中最重要 的就是检查点(checkpoint)。

发生故障之后怎么办?最简单的想法当然是重启机器、重启应用。由于流处理应用中的任 务都是有状态的，为了在重启后能够继续之前的处理计算，我们应该对内存中的状态做一个持 久化存盘——就像编写文档或是玩 RPG 游戏一样。

我们可以将之前某个时间点所有的状态保存下来，这份“存档”就是所谓的“检查点” (checkpoint)。

遇到故障重启的时候，我们可以从检查点中“读档”，恢复出之前的状态，这样就可以回 到当时保存的一刻接着处理数据了。

检查点是 Flink 容错机制的核心。这里所谓的“检查”，其实是针对故障恢复的结果而言 的:故障恢复之后继续处理的结果，应该与发生故障前完全一致，我们需要“检查”结果的正 确性。所以，有时又会把 checkpoint 叫作“一致性检查点”。

### 5.3.1 检查点的保存

什么时候进行检查点的保存呢?最理想的情况下，我们应该“随时”保存，也就是每处理完一个数据就保存一下当前的状态;这样如果在处理某条数据时出现故障，我们只要回到上一 个数据处理完之后的状态，然后重新处理一遍这条数据就可以。

**1.周期性的触发保存**

“随时存档”确实恢复起来方便，可是需要我们不停地做存档操作。如果每处理一条数据就进行检查点的保存，当大量数据同时到来时，就会耗费很多资源来频繁做检查点，数据处理的速度就会受到影响。所以更好的方式是，每隔一段时间去做一次存档，这样既不会影响数据 的正常处理，也不会有太大的延迟。在 Flink 中，检查点的保存是周期性触发的，间隔时间可以进行设置。

所以检查点作为应用状态的一份“存档”，其实就是所有任务状态在同一时间点的一个“快照”(snapshot)，它的触发是周期性的。具体来说，当每隔一段时间检查点保存操作被触发时， 就把每个任务当前的状态复制一份，按照一定的逻辑结构放在一起持久化保存起来，就构成了检查点。

**2.保存的时间点**

我们保存状态的策略是:当所有任务都恰好处理完一个相同的输入数据的时候，将它们的 状态保存下来。

首先，这样避免了除状态之外其他额外信息的存储，提高了检查点保存的效率。其次，一个数据要么就是被所有任务完整地处理完，状态得到了保存;要么就是没处理完，状态全部没 保存:这就相当于构建了一个“事务”(transaction)。如果出现故障，我们恢复到之前保存的状态，故障时正在处理的所有数据都需要重新处理;所以我们只需要让源(Source)任务向数据源重新提交偏移量、请求重放数据就可以了。这需要源任务可以把偏移量作为算子状态保存下来，而且外部数据源能够重置偏移量;Kafka 就是满足这些要求的一个最好的例子。

### 5.3.2 检查点的配置

1. 启用检查点

默认情况下，Flink 程序是禁用检查点的。如果想要为 Flink 应用开启自动保存快照的功能，需要在代码中显式地调用执行环境的 enableCheckpointing()方法:

```
val env =
StreamExecutionEnvironment.getExecutionEnvironment // 每隔 1 秒启动一次检查点保存 env.enableCheckpointing(1000)
```

这里需要传入一个长整型的毫秒数，表示周期性保存检查点的间隔时间。如果不传参数直接启用检查点，默认的间隔周期为 500 毫秒，这种方式已经被弃用。

检查点的间隔时间是对处理性能和故障恢复速度的一个权衡。如果我们希望对性能的影响更小，可以调大间隔时间;而如果希望故障重启后迅速赶上实时的数据处理，就需要将间隔时 间设小一些。

2. 检查点存储(Checkpoint Storage)

检查点具体的持久化存储位置，取决于“检查点存储”(CheckpointStorage)的设置。默 认情况下，检查点存储在 JobManager 的堆(heap)内存中。而对于大状态的持久化保存，Flink 也提供了在其他存储位置进行保存的接口，这就是 CheckpointStorage。

具体可以通过调用检查点配置的 setCheckpointStorage()来配置，需要传入一个 CheckpointStorage 的实现类。Flink 主要提供了两种 CheckpointStorage:分别是将检查点存储 至作业管理器的堆内存( JobManagerCheckpointStorage )和文件系统(FileSystemCheckpointStorage)。

```
// 配置存储检查点到 JobManager 堆内存 
env.getCheckpointConfig.setCheckpointStorage(new JobManagerCheckpointStorage) 
// 配置存储检查点到文件系统
env.getCheckpointConfig.setCheckpointStorage(new
FileSystemCheckpointStorage("hdfs://namenode:40010/flink/checkpoints"))
```

对于实际生产应用，我们一般会将 CheckpointStorage 配置为高可用的分布式文件系统 (HDFS，S3 等)。

## 5.4 保存点(savepoint)

除了检查点(checkpoint)外，Flink 还提供了另一个非常独特的镜像保存功能——保存点 (savepoint)。

从名称就可以看出，这也是一个存盘的备份，它的原理和算法与检查点完全相同，只是多了一些额外的元数据。事实上，保存点就是通过检查点的机制来创建流式作业状态的一致性镜像(consistent image)的。

保存点中的状态快照，是以算子 ID 和状态名称组织起来的，相当于一个键值对。从保存点启动应用程序时，Flink 会将保存点的状态数据重新分配给相应的算子任务。

### 5.4.1 保存点的用途

保存点与检查点最大的区别，就是触发的时机。检查点是由 Flink 自动管理的，定期创建， 发生故障之后自动读取进行恢复，这是一个“自动存盘”的功能;而保存点不会自动创建，必 须由用户明确地手动触发保存操作，所以就是“手动存盘”。因此两者尽管原理一致，但用途 就有所差别了:检查点主要用来做故障恢复，是容错机制的核心;保存点则更加灵活，可以用 来做有计划的手动备份和恢复。

保存点可以当作一个强大的运维工具来使用。我们可以在需要的时候创建一个保存点，然 后停止应用，做一些处理调整之后再从保存点重启。它适用的具体场景有:

⚫ 版本管理和归档存储 

对重要的节点进行手动备份，设置为某一版本，归档(archive)存储应用程序的状态。

⚫ 更新Flink版本
 目前 Flink 的底层架构已经非常稳定，所以当 Flink 版本升级时，程序本身一般是兼容的。

这时不需要重新执行所有的计算，只要创建一个保存点，停掉应用、升级 Flink 后，从保存点 重启就可以继续处理了。

⚫ 更新应用程序

我们不仅可以在应用程序不变的时候，更新 Flink 版本;还可以直接更新应用程序。前提是程序必须是兼容的，也就是说更改之后的程序，状态的拓扑结构和数据类型都是不变的，这样才能正常从之前的保存点去加载。

这个功能非常有用。我们可以及时修复应用程序中的逻辑 bug，更新之后接着继续处理; 也可以用于有不同业务逻辑的场景，比如 A/B 测试等等。

⚫ 调整并行度

如果应用运行的过程中，发现需要的资源不足或已经有了大量剩余，也可以通过从保存点 重启的方式，将应用程序的并行度增大或减小。

⚫ 暂停应用程序

有时候我们不需要调整集群或者更新程序，只是单纯地希望把应用暂停、释放一些资源来 处理更重要的应用程序。使用保存点就可以灵活实现应用的暂停和重启，可以对有限的集群资 源做最好的优化配置。

### 5.4.2 保存点的使用

保存点的使用非常简单，我们可以使用命令行工具来创建保存点，也可以从保存点重启应用。

(1)创建保存点，要在命令行中为运行的作业创建一个保存点镜像，只需要执行:

```
bin/flink savepoint :jobId [:targetDirectory]
```

这里 jobId 需要填充要做镜像保存的作业 ID，目标路径 targetDirectory 可选，表示保存点存储的路径。

对于保存点的默认路径，可以通过配置文件 flink-conf.yaml 中的 state.savepoints.dir 项来设

定:

```
state.savepoints.dir: hdfs:///flink/savepoints
```

当然对于单独的作业，我们也可以在程序代码中通过执行环境来设置:

```
env.setDefaultSavepointDir("hdfs:///flink/savepoints");
```

由于创建保存点一般都是希望更改环境之后重启，所以创建之后往往紧接着就是停掉作业的操作。除了对运行的作业创建保存点，我们也可以在停掉一个作业时直接创建保存点:

```
bin/flink stop --savepointPath [:targetDirectory] :jobId
```

(2)从保存点重启应用
 我们已经知道，提交启动一个Flink作业，使用的命令是flink run;现在要从保存点重启

```
一个应用，其实本质是一样的:
bin/flink run -s :savepointPath [:runArgs]
```

只要增加一个-s 参数，指定保存点的路径就可以了，其他启动时的参数还是完全一样的。



## 5.5 状态持久化和状态后端

检查点的保存离不开 JobManager 和 TaskManager，以及外部存储系统的协调。在应用进

行检查点保存时，首先会由 JobManager 向所有 TaskManager 发出触发检查点的命令,TaskManger 收到之后，将当前任务的所有状态进行快照保存，持久化到远程的存储介质中; 完成之后向 JobManager 返回确认信息。这个过程是分布式的，当 JobManger 收到所有 TaskManager 的返回信息后，就会确认当前检查点成功保存，如图所示。而这一切工作的协调，就需要一个“专职人员”来完成。

<img src="/Users/zyw/Library/Application Support/typora-user-images/image-20221121142722676.png" alt="image-20221121142722676" style="zoom:50%;" />

在 Flink 中，状态的存储、访问以及维护，都是由一个可插拔的组件决定的，这个组件就 叫作状态后端(state backend)。状态后端主要负责两件事:一是本地的状态管理，二是将检查点(checkpoint)写入远程的持久化存储。

### 5.5.1 状态后端的分类

状态后端是一个“开箱即用”的组件，可以在不改变应用程序逻辑的情况下独立配置。 Flink 中提供了两类不同的状态后端，一种是“哈希表状态后端”(HashMapStateBackend)，另 一种是“内嵌 RocksDB 状态后端”(EmbeddedRocksDBStateBackend)。如果没有特别配置， 系统默认的状态后端是 HashMapStateBackend。

(1)哈希表状态后端(HashMapStateBackend) 

这种方式就是我们之前所说的，把状态存放在内存里。具体实现上，哈希表状态后端在内部会直接把状态当作对象(objects)，保存在 Taskmanager 的 JVM 堆(heap)上。普通的状态， 以及窗口中收集的数据和触发器(triggers)，都会以键值对(key-value)的形式存储起来，所以底层是一个哈希表(HashMap)，这种状态后端也因此得名。

对于检查点的保存，一般是放在持久化的分布式文件系统(file system)中，也可以通过 配置“检查点存储”(CheckpointStorage)来另外指定。

HashMapStateBackend 是将本地状态全部放入内存的，这样可以获得最快的读写速度，使计算性能达到最佳;代价则是内存的占用。它适用于具有大状态、长窗口、大键值状态的作业， 对所有高可用性设置也是有效的。

(2)内嵌 RocksDB 状态后端(EmbeddedRocksDBStateBackend)

RocksDB 是一种内嵌的 key-value 存储介质，可以把数据持久化到本地硬盘。配置EmbeddedRocksDBStateBackend 后，会将处理中的数据全部放入 RocksDB 数据库中，RocksDB默认存储在 TaskManager 的本地数据目录里。
 与 HashMapStateBackend 直接在堆内存中存储对象不同，这种方式下状态主要是放在RocksDB 中的。数据被存储为序列化的字节数组(Byte Arrays)，读写操作需要序列化/反序列 化，因此状态的访问性能要差一些。另外，因为做了序列化，key 的比较也会按照字节进行， 而不是直接调用 hashCode()和 equals()方法。 

对于检查点，同样会写入到远程的持久化文件系统中。

EmbeddedRocksDBStateBackend 始终执行的是异步快照，也就是不会因为保存检查点而阻塞数据的处理;而且它还提供了增量式保存检查点的机制，这在很多情况下可以大大提升保存效率。

由于它会把状态数据落盘，而且支持增量化的检查点，所以在状态非常大、窗口非常长、 键/值状态很大的应用场景中是一个好选择，同样对所有高可用性设置有效。

### 5.5.2 如何选择状态后端

HashMap 和 RocksDB 两种状态后端最大的区别，就在于本地状态存放在哪里:前者是内 存，后者是 RocksDB。在实际应用中，选择那种状态后端，主要是需要根据业务需求在处理 性能和应用的扩展性上做一个选择。

HashMapStateBackend 是内存计算，读写速度非常快;但是，状态的大小会受到集群可用内存的限制，如果应用的状态随着时间不停地增长，就会耗尽内存资源。

而 RocksDB 是硬盘存储，所以可以根据可用的磁盘空间进行扩展，而且是唯一支持增量 检查点的状态后端，所以它非常适合于超级海量状态的存储。不过由于每个状态的读写都需要 做序列化/反序列化，而且可能需要直接从磁盘读取数据，这就会导致性能的降低，平均读写 性能要比 HashMapStateBackend 慢一个数量级。

### 5.5.3 状态后端的配置

在不做配置的时候，应用程序使用的默认状态后端是由集群配置文件 flink-conf.yaml 中指 定的，配置的键名称为 state.backend。这个默认配置对集群上运行的所有作业都有效，我们可 以通过更改配置值来改变默认的状态后端。另外，我们还可以在代码中为当前作业单独配置状 态后端，这个配置会覆盖掉集群配置文件的默认值。

(1)配置默认的状态后端
 在 flink-conf.yaml 中，可以使用 state.backend 来配置默认状态后端。
 配置项的可能值为 hashmap，这样配置的就是 HashMapStateBackend;也可以是 rocksdb，

这样配置的就是 EmbeddedRocksDBStateBackend。 下面是一个配置 HashMapStateBackend 的例子:

```
# 默认状态后端
state.backend: hashmap
# 存放检查点的文件路径
state.checkpoints.dir: hdfs://namenode:40010/flink/checkpoints
```

这里的 state.checkpoints.dir 配置项，定义了状态后端将检查点和元数据写入的目录。

(2)为每个作业(Per-job)单独配置状态后端 

每个作业独立的状态后端，可以在代码中，基于作业的执行环境直接设置。代码如下:

```
val env = StreamExecutionEnvironment.getExecutionEnvironment();
env.setStateBackend(new HashMapStateBackend())
```

上面代码设置的是 HashMapStateBackend，如果想要设置 EmbeddedRocksDBStateBackend， 可以用下面的配置方式:

```
val env = StreamExecutionEnvironment.getExecutionEnvironment();
env.setStateBackend(new EmbeddedRocksDBStateBackend())
```

需要注意，如果想在 IDE 中使用 EmbeddedRocksDBStateBackend，需要为 Flink 项目添加 依赖:

```
<dependency>
   <groupId>org.apache.flink</groupId>
   <artifactId>flink-statebackend-rocksdb_2.11</artifactId>
   <version>1.13.5</version>
</dependency>
```



# 第六章 生产运维

## 6.1 生产环境说明

1. 生产环境Flink版本目前为1.13.5。
2. 生产环境目前主要的Flink业务场景，包括用户功能访问量统计，人资大屏，薪酬实时报表。其中功能访问量统计是基于Flink DataStream开发，后面两个场景都是基于Flink SQL开发，后面的项目开发会优先考虑Flink SQL。
3. 生产环境目前主要运行的Flink作业采用Per-Job模式和Session模式，未来会将Per-Job模式逐步替换成Application模式。另外要注意Session模式的缺点，一旦Sesssion模式中的一个作业挂掉了，整个集群中的其他作业也都会挂掉。

4. 后续的Flink作业提交会通过实时作业平台(待完善)统一提交，实时作业平台提供实时作业的提交，监控等工作。

## 6.2 注意事项说明

1. 端到端的数据一致性。

要注意整个数据处理流程中的一致性，包括Flink内部数据的一致性和Flink与外部交互数据的一致性。Flink内部的数据一致性，可以基于Flink框架的checkpoint机制保值，Flink与外部交互的一致性，则依赖与外部数据介质是否支持数据一致性。

2. Sink端的写入瓶颈。

Flink写入数据到外部数据源，一定要注意数据写入的速率，否则可能会出现数据丢失，数据不一致的可能。

3. Flink SQL维表关联效率和对维表数据库影响

Flink SQL在关联外部数据维表的时候，要注意关联维表算子的速率，注意维表是否建立索引等。

4. 实时作业对Yarn资源的控制。

Flink任务同样依赖于Yarn的资源，要注意实时作业申请的资源过多是否影响Spark任务的运行。

## 6.3 生产环境Flink任务提交流程

### 6.3.1 Flink Jar任务提交流程

待补充

### 6.3.1 Flink SQL 任务提交流程

待补充



# 第七章 Flink 实时数仓搭建规划

## 基础架构

支持Flink Jar，Flink SQL类型的任务提交。

支持Session，Application模式的任务提交。

## 整库同步

支持MySQL cdc数据的单任务同步

## 实时作业的监控

支持Flink任务报错的邮件提醒

## 元数据管理

支持Flink Table 的元数据同步

## 表结构变更

支持MySQL表结构变更，自动向下游同步的功能

## Hive SQL平迁Flink SQL

支持Hive SQL迁移到Flink SQL中执行