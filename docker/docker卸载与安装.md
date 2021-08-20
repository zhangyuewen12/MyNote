# docker 镜像 **centos**系统

## 卸载

```
# 1. Uninstall the Docker Engine, CLI, and Containerd packages:
$ sudo yum remove docker-ce docker-ce-cli containerd.io

# 2.Images, containers, volumes, or customized configuration files on your host are not automatically removed. To delete all images, containers, and volumes:

$ sudo rm -rf /var/lib/docker
$ sudo rm -rf /var/lib/containerd
```



## 安装



```
#Older versions of Docker were called `docker` or `docker-engine`. If these are installed, uninstall them, #along with associated dependencies.

$ sudo yum remove docker \
                  docker-client \
                  docker-client-latest \
                  docker-common \
                  docker-latest \
                  docker-latest-logrotate \
                  docker-logrotate \
                  docker-engine
#Install the `yum-utils` package (which provides the `yum-config-manager` utility) and set up the **stable** #repository   

# 安装工具类
1. sudo yum install -y yum-utils

# 不推荐
$ sudo yum-config-manager \
    --add-repo \
    https://download.docker.com/linux/centos/docker-ce.repo
    
# 设置国内仓库地址
2. sudo yum-config-manager \
   --add-repo \
   http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
   
# 更新yum 软件包索引
3. yum makecache fast

# Install Docker Engine
# Install the latest version of Docker Engine and containerd, or go to the next step to install a specific # # version:

4. sudo yum install docker-ce docker-ce-cli containerd.io

# Start Docker.
5. sudo systemctl start docker

# 测试命令
6. docker version
```