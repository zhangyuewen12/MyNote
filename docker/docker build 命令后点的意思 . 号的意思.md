### docker build 命令后"."

我们在使用 docker build 命令去构建镜像时，往往会看到命令最后会有一个 . 号。

```bash
docker build -t xxxxx .
```

那么这里的 . 号代表什么意思呢？

在我们学习对 . 号的理解有所偏差，以为是用来指定 Dockerfile 文件所在的位置的，但其实 -f 参数才是用来指定 Dockerfile 的路径的，那么 . 号究竟是用来做什么的呢？

Docker 在运行时分为 Docker引擎（服务端守护进程） 以及 客户端工具，我们日常使用各种 docker 命令，其实就是在使用客户端工具与 Docker 引擎 进行交互。

那么当我们使用 docker build 命令来构建镜像时，这个构建过程其实是在 Docker引擎 中完成的，而不是在本机环境。

那么如果在 Dockerfile 中使用了一些 COPY 等指令来操作文件，如何让 Docker引擎 获取到这些文件呢？

这里就有了一个 镜像构建上下文 的概念，当构建的时候，由用户指定构建镜像的上下文路径，而 docker build 会将这个路径下所有的文件都打包上传给 Docker 引擎，引擎内将这些内容展开后，就能获取到所有指定上下文中的文件了。

比如说 dockerfile 中的 COPY ./package.json /project，其实拷贝的并不是本机目录下的 package.json 文件，而是 docker引擎 中展开的构建上下文中的文件，所以如果拷贝的文件超出了构建上下文的范围，Docker引擎 是找不到那些文件的。

所以 docker build 最后的 . 号，其实是在指定镜像构建过程中的上下文环境的目录。