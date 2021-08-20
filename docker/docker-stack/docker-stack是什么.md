**大规模场景下的多服务部署和管理**是一件很难的事情。

幸运的是，[Docker](http://c.biancheng.net/docker/) Stack 为解决该问题而生，Docker Stack 通过提供期望状态、滚动升级、简单易用、扩缩容、健康检查等特性简化了应用的管理，这些功能都封装在一个完美的声明式模型当中。

在笔记本上测试和部署简单应用很容易。但这只能算业余选手。在真实的生产环境进行多服务的应用部署和管理，这才是专业选手的水平。

幸运的是，Stack 正为此而生！Stack 能够在单个声明文件中定义复杂的多服务应用。Stack 还提供了简单的方式来部署应用并管理其完整的生命周期：初始化部署 -> 健康检查 -> 扩容 -> 更新 -> 回滚，以及其他功能！

步骤很简单。在 Compose 文件中定义应用，然后通过 docker stack deploy 命令完成部署和管理。

Compose 文件中包含了构成应用所需的完整服务栈。此外还包括了卷、网络、安全以及应用所需的其他基础架构。然后基于该文件使用 docker stack deploy 命令来部署应用。

**Stack 是基于 Docker Swarm 之上来完成应用的部署。**因此诸如安全等高级特性，其实都是来自 Swarm。

简而言之，Docker 适用于开发和测试。Docker Stack 则适用于大规模场景和生产环境。

如果了解 Docker Compose，就会发现 Docker Stack 非常简单。事实上在许多方面，Stack 一直是期望的 Compose——完全集成到 Docker 中，并能够管理应用的整个生命周期。

从体系结构上来讲，Stack 位于 Docker 应用层级的最顶端。Stack 基于服务进行构建，而服务又基于容器，如下图所示

<img src="/Users/zyw/Library/Application Support/typora-user-images/image-20210820094359501.png" alt="image-20210820094359501" style="zoom:50%;" />