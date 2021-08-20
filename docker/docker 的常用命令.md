docker 的常用命令

```shell
docker version #显示docker的版本信息
docker info    #显示docker的信息信息，包括镜像、容器信息
docker 命令 --help  帮助命令
```



# 镜像命令

```shell
docker images #显示镜像
docker search #搜索镜像
docker pull   #拉取镜像
docker rmi 镜像id #删除镜像  eg:docker rmi $(docker images -qa)
```





# 容器命令

```shell
docker run 
docker ps
exit #停止容器并退出容器
ctrl+p+q #容器不停止退出
docker rm #删除容器
docker rm -f $(docker ps -aq) #删除所有容器
docker start #启动容器ID
docker restart #重启容器ID
docker stop #容器ID
docker kill #容器ID

docker inspect 容器Id  #查看容器的详细信息

docker exec 
docker attach

```

