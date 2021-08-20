# Dockerfile 

Docker can build images automatically by reading the instructions from a `Dockerfile`. 

A `Dockerfile` is a text document that contains all the commands a user could call on the command line to assemble an image. 

Using `docker build` users can create an automated build that executes several command-line instructions in succession.



## Usage

The [docker build](https://docs.docker.com/engine/reference/commandline/build/) command builds an image from a `Dockerfile` and a *context*. The build’s context is the set of files at a specified location `PATH` or `URL`. The `PATH` is a directory on your local filesystem. The `URL` is a Git repository location.

A context is processed recursively. So, a `PATH` includes any subdirectories and the `URL` includes the repository and its submodules. This example shows a build command that uses the current directory as context:

```
$ docker build .

Sending build context to Docker daemon  6.51 MB
...
```

The build is run by the Docker daemon, not by the CLI. The first thing a build process does is send the entire context (recursively) to the daemon. In most cases, it’s best to start with an empty directory as context and keep your Dockerfile in that directory. Add only the files needed for building the Dockerfile.

docker build 从一个dockerfile文件和一个构建过程的上下文构建镜像。构建过程是运行在 Docker引擎中（服务端守护进程）,并不是客户端进程。构建过程的第一步是递归传递镜像构建的上下文给服务端守护线程。



1. You can specify a repository and tag at which to save the new image if the build succeeds:可以通过-t参数指定构建镜像的标志

   ```
   $ docker build -t shykes/myapp .
   ```

2. Before the Docker daemon runs the instructions in the `Dockerfile`, it performs a preliminary validation of the `Dockerfile` and returns an error if the syntax is incorrect:在Docker服务端守护进程运行dockerfile中的指令之前，会首先验证dockerfile中的指令的正确性和语义。

   ```
   $ docker build -t test/myapp .
   
   Sending build context to Docker daemon 2.048 kB
   Error response from daemon: Unknown instruction: RUNCMD
   ```



**FROM**：定制的镜像都是基于 FROM 的镜像，这里的 nginx 就是定制需要的基础镜像。后续的操作都是基于 nginx。

**RUN**：用于执行后面跟着的命令行命令.

### COPY

复制指令，从上下文目录中复制文件或者目录到容器里指定路径。

### WORKDIR

指定工作目录。用 WORKDIR 指定的工作目录，会在构建镜像的每一层中都存在。

### VOLUME

定义匿名数据卷。在启动容器时忘记挂载数据卷，会自动挂载到匿名卷。

### EXPOSE

仅仅只是声明端口。

作用：

- 帮助镜像使用者理解这个镜像服务的守护端口，以方便配置映射。
- 在运行时使用随机端口映射时，也就是 docker run -P 时，会自动随机映射 EXPOSE 的端口。

### CMD

类似于 RUN 指令，用于运行程序，但二者运行的时间点不同:

- CMD 在docker run 时运行。
- RUN 是在 docker build。

**作用**：**为启动的容器指定默认要运行的程序，程序运行结束，容器也就结束。****CMD 指令指定的程序可被 docker run 命令行参数中指定要运行的程序所覆盖。**

**注意**：如果 Dockerfile 中如果存在多个 CMD 指令，仅最后一个生效。

格式:

```
CMD <shell 命令> 
CMD ["<可执行文件或命令>","<param1>","<param2>",...] 
CMD ["<param1>","<param2>",...]  # 该写法是为 ENTRYPOINT 指令指定的程序提供默认参数
推荐使用第二种格式，执行过程比较明确。第一种格式实际上在运行的过程中也会自动转换成第二种格式运行，并且默认可执行文件是 sh。
```

### ENTRYPOINT

类似于 CMD 指令，但其不会被 docker run 的命令行参数指定的指令所覆盖，而且**这些命令行参数(包括CMD参数)**会被当作参数送给 ENTRYPOINT 指令指定的程序。

但是, 如果运行 docker run 时使用了 --entrypoint 选项，将覆盖 CMD 指令指定的程序。

**优点**：在执行 docker run 的时候可以指定 ENTRYPOINT 运行所需的参数。

**注意**：如果 Dockerfile 中如果存在多个 ENTRYPOINT 指令，仅最后一个生效。

格式：

```
ENTRYPOINT ["<executeable>","<param1>","<param2>",...]
```

可以搭配 CMD 命令使用：一般是变参才会使用 CMD ，**这里的 CMD 等于是在给 ENTRYPOINT 传参**，以下示例会提到。

示例：

假设已通过 Dockerfile 构建了 nginx:test 镜像：

```
FROM nginx

ENTRYPOINT ["nginx", "-c"] # 定参
CMD ["/etc/nginx/nginx.conf"] # 变参 
```

1、不传参运行

```
$ docker run  nginx:test
```

容器内会默认运行以下命令，启动主进程。

```
nginx -c /etc/nginx/nginx.conf
```

2、传参运行

```
$ docker run  nginx:test -c /etc/nginx/new.conf
```

容器内会默认运行以下命令，启动主进程(/etc/nginx/new.conf:假设容器内已有此文件)

```
nginx -c /etc/nginx/new.conf
```

### EXPOSE

仅仅只是声明端口。

作用：

- 帮助镜像使用者理解这个镜像服务的守护端口，以方便配置映射。
- 在运行时使用随机端口映射时，也就是 docker run -P 时，会自动随机映射 EXPOSE 的端口。

格式：

```
EXPOSE <端口1> [<端口2>...]
```

### VOLUME

**定义匿名数据卷**。在启动容器时忘记挂载数据卷，会自动挂载到匿名卷。

作用：

- 避免重要的数据，因容器重启而丢失，这是非常致命的。
- 避免容器不断变大。

格式：

```
VOLUME ["<路径1>", "<路径2>"...]
VOLUME <路径>
```

### WORKDIR

指定工作目录。用 WORKDIR 指定的工作目录，会在构建镜像的每一层中都存在。（WORKDIR 指定的工作目录，必须是提前创建好的）。

docker build 构建镜像过程中的，每一个 RUN 命令都是新建的一层。只有通过 WORKDIR 创建的目录才会一直存在。

格式：

```
WORKDIR <工作目录路径>
```

