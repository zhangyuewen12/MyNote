docker run 命令



# [docker run -it centos /bin/bash 后面的 bin/bash的作用](https://www.cnblogs.com/Guhongying/p/10894434.html)

首先，docker run -it centos 的意思是，为centos这个镜像创建一个容器， -i和-t这两个参数的作用是，为该docker创建一个伪终端，这样就可以进入到容器的交互模式？（也就是直接进入到容器里面）后面的/bin/bash的作用是表示载入容器后运行bash ,docker中必须要保持一个进程的运行，要不然整个容器启动后就会马上kill itself，这样当你使用docker ps 查看启动的容器时，就会发现你刚刚创建的那个容器并不在已启动的容器队列中。*这个/bin/bash就表示启动容器后启动bash。*



```
docker run [OPTIONS] IMAGE [COMMAND] [ARG...]

Options

--detach , -d		Run container in background and print container ID
--name		Assign a name to the container
--publish , -p		Publish a container's port(s) to the host
--publish-all , -P		Publish all exposed ports to random ports
--rm		Automatically remove the container when it exits


$ docker run -itd --network=my-net busybox

eg1: docker run -d --name nginx01 -p 8080:80 nginx

eg2: docker run -p 127.0.0.1:80:8080/tcp ubuntu bash
This binds port 8080 of the container to TCP port 80 on 127.0.0.1 of the host machine. You can also specify udp and sctp ports. The Docker User Guide explains in detail how to manipulate ports in Docker.

```

设置docker的网络

```shell
--network		Connect a container to a network
When you start a container use the --network flag to connect it to a network. This adds the busybox container to the my-net network.

docker run -itd --network=my-net --ip=10.10.9.75 busybox
```

设置docker容器的环境变量

```shell
--env , -e		Set environment variables

Use the -e, --env, and --env-file flags to set simple (non-array) environment variables in the container you’re running, or overwrite variables that are defined in the Dockerfile of the image you’re running.

You can define the variable and its value when running the container:

$ docker run --env VAR1=value1 --env VAR2=value2 ubuntu env | grep VAR
VAR1=value1
VAR2=value2

You can also load the environment variables from a file. This file should use the syntax <variable>=value (which sets the variable to the given value) or <variable> (which takes the value from the local environment), and # for comments.

$ cat env.list

VAR1=value1
VAR2=value2
USER

$ docker run --env-file env.list ubuntu env | grep VAR
VAR1=value1
VAR2=value2
USER=denis
```

