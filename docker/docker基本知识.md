

## 参考学习链接

https://blog.csdn.net/dhaiuda/article/details/82782189

## 什么是容器？

容器是镜像的运行时实例。

## 什么是 Dockerfile？

Dockerfile 是一个用来构建镜像的文本文件，文本内容包含了一条条构建镜像所需的指令和说明。

## 什么是Docker Compose？

它能够在Docker节点上，以**单引擎模式(single-engine mode)进行多容器应用的部署和管理。**

docker compose本质上是一个**基于dokcer API开发的外部Python工具**。使用它时，通过编写多容器应用的YAML文件，然后将其交由docker-compose命令处理，Docker Compose就会基于Dokcer引擎AP完成应用的不是。

## 什么是Docker Swarm？

Docker Swarm 和 Docker Compose 一样，都是 Docker 官方容器编排项目，但不同的是，Docker Compose 是一个在单个服务器或主机上创建多个容器的工具，而 Docker Swarm 则可以在多个服务器或主机上创建容器集群服务，对于微服务的部署，显然 Docker Swarm 会更加适合。

Swarm deamon只是一个调度器(Scheduler)加路由器(router),Swarm自己不运行容器，它只是接受Docker客户端发来的请求，调度适合的节点来运行容器，这就意味着，即使Swarm由于某些原因挂掉了，集群中的节点也会照常运行，放Swarm重新恢复运行之后，他会收集重建集群信息。

<img src="/Users/zyw/Library/Application Support/typora-user-images/image-20210622153444013.png" alt="image-20210622153444013" style="zoom:50%;" />

## stacks, services, and tasks 之间的关系？

<img src="/Users/zyw/Library/Application Support/typora-user-images/image-20210622153920895.png" alt="image-20210622153920895" style="zoom:50%;" />

#### services

swarm service是一个抽象的概念，它只是一个对运行在swarm集群上的应用服务，所期望状态的描述。它就像一个描述了下面物品的清单列表一样：

- 服务名称
- 使用哪个镜像来创建容器
- 要运行多少个副本
- 服务的容器要连接到哪个网络上
- 应该映射哪些端口

#### task

在Docker Swarm中，task是一个部署的最小单元，task与容器是一对一的关系。

#### stack

stack是描述一系列相关services的集合。我们通过在一个YAML文件中来定义一个stack

## 什么是Docker Service?



