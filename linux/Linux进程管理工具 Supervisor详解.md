原文地址[https://zhuanlan.zhihu.com/p/147305277]

Supervisor安装与配置(linux/unix进程管理工具) Supervisor（[Supervisor: A Process Control System](https://link.zhihu.com/?target=http%3A//supervisord.org)）是用Python开发的一个client/server服务，是Linux/Unix系统下的一个进程管理工具，不支持Windows系统。它可以很方便的监听、启动、停止、重启一个或多个进程。用Supervisor管理的进程，当一个进程意外被杀死，supervisort监听到进程死后，会自动将它重新拉起，很方便的做到进程自动恢复的功能，不再需要自己写shell脚本来控制。



因为Supervisor是Python开发的,安装前先检查一下系统否安装了Python2.4以上版本。下面以CentOS7.6,Python2.7.5版本环境下,介绍Supervisor的安装与配置步聚：

## 一、实验环境

**1、系统平台**

```text
cat /etc/redhat-release
CentOS Linux release 7.6.1810 (Core)
```

**2、Python版本**

```text
python -V
Python 2.7.5
```

如果python版本低于2.6请升级，下面贴出一个安装python3.6.8的安装示例

```text
yum install zlib-devel bzip2-devel openssl-devel ncurses-devel sqlite-devel readline-devel tk-devel gdbm-devel db4-devel libpcap-devel xz-devel -y libffi-devel
wget https://www.python.org/ftp/python/3.6.8/Python-3.6.8.tar.xz
tar xf Python-3.6.8.tar.xz
cd Python-3.6.8
./configure --prefix=/usr/local/python368
make && make install
echo 'export PATH=/usr/local/python368/bin:$PATH' >> /etc/profile
source /etc/profile
python3 -V
```

## **二、安装 Supervisor**

安装 Supervisor的方法很多，以下介绍三种，我这里所采用的为第三种

**1、easy_install 安装 supervisor**

安装Python包管理工具(easy_install) easy_install是setuptools包里带的一个命令,使用easy_install实际上是在调用setuptools来完成安装模块的工作,所以安装setuptools即可:

```text
wget https://pypi.io/packages/source/s/setuptools/setuptools-33.1.1.zip
unzip setuptools-33.1.1.zip
cd setuptools-33.1.1
python setup.py install

easy_install supervisor
```

**2、pip 安装 supervisor**

使用 pip 来安装，前提要保证pip版本大于2.6

```text
pip install supervisor
```

**3、yum epel-release 安装 supervisor**

```text
yum install -y epel-release && yum install -y supervisor
```

[点击这里，获取更多免费资料](https://link.zhihu.com/?target=http%3A//image.qbangmang.com/counselor.html)

## **三、superviso命令**

supervisor安装完成后会生成三个执行程序:supervisortd、supervisorctl、echo_supervisord_conf:

•supervisortd：用于管理supervisor本身服务•supervisorctl：用于管理我们需要委托给superviso工具的服务•echo_supervisord_conf：用于生成superviso的配置文件•supervisor的守护进程服务(用于接收进程管理命令)•客户端(用于和守护进程通信,发送管理进程的指令)

```text
[root@Jumpserver /]# which supervisord
/bin/supervisord
[root@Jumpserver /]# which supervisorctl
/bin/supervisorctl
[root@Jumpserver /]# which echo_supervisord_conf
/bin/echo_supervisord_conf
```

## **四、配置Supervisor**

**1、通过运行echo_supervisord_conf程序生成supervisor的初始化配置文件**

如果使用yum安装则此步骤省略，直接进行修改配置文件步骤

```text
mkdir /etc/supervisord.d
echo_supervisord_conf > /etc/supervisord.conf
```

**2、修改配置文件**

supervisor的配置文件内容有很多，不过好多都不需要修改就行使用，我这里只修改了以下两项

```text
#修改socket文件的mode，默认是0700
sed -i 's/;chmod=0700/chmod=0766/g' /etc/supervisord.conf   

#在配置文件最后添加以下两行内容来包含/etc/supervisord目录
sed -i '$a [include] \
files = /etc/supervisord.d/*.conf' /etc/supervisord.conf
```

## **五、编写需要被Supervisor管理的进程**

Supervisor只能管理非dameon进程，像默认的redis为前台运行、Tomcat其实是 startup.sh shutdown.sh来调用catalina.sh进行后台运行的，默认catalina.sh为前台运行的程序，不能管理像Nginx一样的非dameon进程

### **1、Tomcat被Supervisor管理**

**Tomcat安装如下：**

```text
wget http://us.mirrors.quenda.co/apache/tomcat/tomcat-8/v8.5.47/bin/apache-tomcat-8.5.47.tar.gz
yum install java-1.8.0-openjdk.x86_64 java-1.8.0-openjdk-devel.x86_64 -y
tar xf apache-tomcat-8.5.47.tar.gz  -C /usr/local/
mv /usr/local/apache-tomcat-8.5.47 /usr/local/tomcat
```

**想要我们的应用被Supervisor管理，就需要在/etc/supervisord目录下编写配置文件，Tomcat案例如下：**

```text
vim /etc/supervisord.d/tomcat.conf
[program:tomcat]                                        #程序唯一名称
directory=/usr/local/tomcat                             #程序路径
command=/usr/local/tomcat/bin/catalina.sh run           #运行程序的命令
autostart=true                                          #是否在supervisord启动后tomcat也启动
startsecs=10                                            #启动10秒后没有异常退出，就表示进程正常启动了，默认为1秒
autorestart=true                                        #程序退出后自动重启,可选值：[unexpected,true,false]，默认为unexpected，表示进程意外杀死后才重启；意思为如果不是supervisord来关闭的该进程则认为不正当关闭，supervisord会再次把该进程给启动起来，只能使用该supervisorctl来进行关闭、启动、重启操作 
startretries=3                                          #启动失败自动重试次数，默认是3
user=root                                               #用哪个用户启动进程，默认是root
priority=999                                            #进程启动优先级，默认999，假如Supervisord需要管理多个进程，那么值小的优先启动
stopsignal=INT
redirect_stderr=true                                    #把stderr重定向到stdout标准输出，默认false
stdout_logfile_maxbytes=200MB                           #stdout标准输出日志文件大小，日志文件大小到200M后则进行切割，切割后的日志文件会标示为catalina.out1,catalina.out2,catalina.out3...，默认50MB
stdout_logfile_backups = 100                            #stdout标准输出日志文件备份数，保存100个200MB的日志文件，超过100个后老的将被删除，默认为10保存10个
stdout_logfile=/usr/local/tomcat/logs/catalina.out      #标准日志输出位置，如果输出位置不存在则会启动失败
stopasgroup=false                                       #默认为false,进程被杀死时，是否向这个进程组发送stop信号，包括子进程
killasgroup=false                                       #默认为false，向进程组发送kill信号，包括子进程
```

**启动进程** 使用supervisord管理启动后，当你使用/usr/local/tomcat/shutdown.sh或者kill $PID的时候，supervisord都会认为是意外关闭，会自动再次把进程拉起，除非是使用supervisord命令关闭。

```text
#supervisord启动
supervisord -c /etc/supervisord.conf                    #启动supervisord进程，我们在配置文件中设置了 autostart=true 参数，在supervisord启动的时候 tomcat也随之启动
ps -ef|grep java   
```

**程序管理**

```text
supervisorctl status tomcat                             #tomcat状态
supervisorctl stop tomcat                               #停止tomcat
supervisorctl start tomcat                              #启动tomcat
supervisorctl restart tomcat                            #重启tomcat
supervisorctl reoload tomcat 
```

2、**Redis被Supervisor管理**

redis默认不在配置文件中添加 `daemonize yes` 参数则是前台启动的，所以也可以被我们的的Supervisor所管理 redis配置文件如下：

```text
cat redis6001.conf
port 6001
bind 192.168.31.230
protected-mode yes
pidfile "/usr/local/redis/run/redis6001.pid"
loglevel notice
logfile "/usr/local/redis/logs/redis6001.log"
save 900 1
stop-writes-on-bgsave-error yes
rdbcompression yes
rdbchecksum  yes
dbfilename dump.rdb
dir "/usr/local/redis/data/rdb/"
timeout 0
tcp-keepalive 300
```

**编写redis被Supervisor管理的案例**

```text
vim /etc/supervisord.d/redis.conf
[program:redis]
directory=/usr/local/redis
command=/usr/local/redis/bin/redis-server /usr/local/redis/etc/redis6001.conf
autostart=true
startsecs=10
autorestart=true
startretries=3
user=root
priority=999
stopsignal=INT
redirect_stderr=true
stdout_logfile_maxbytes=200MB
stdout_logfile_backups = 100
stdout_logfile=/usr/local/redis/logs/redis6001.log
stopasgroup=false
killasgroup=false
```

**使用super启动redis**

```text
#关闭tomcat
supervisorctl stop tomcat
tomcat: stopped

#杀掉supervisord
ps -ef|grep supervisord
root     26927     1  0 10:47 ?        00:00:00 /usr/bin/python /bin/supervisord -c /etc/supervisord.conf
root     27549 27402  0 11:07 pts/2    00:00:00 grep --color=auto super
kill -9 26927

#重新启动supervisord使其重新加载配置文件，supervisord默认会把redis和tomcat都拉起来
supervisord -c /etc/supervisord.conf
```

**程序管理**

```text
supervisorctl status redis                              #redis状态
supervisorctl stop redis                                #停止redis
supervisorctl start redis                               #启动redis
supervisorctl restart reids                             #重启redis
supervisorctl reoload redis                             #重载redis
```

## **六、程序管理**

**程序管理**

```text
supervisorctl status all                            #查看所有进程状态
supervisorctl stop   all                            #停止所有进程
supervisorctl start  all                            #启动所有进程
supervisorctl restart all                           #重启所有进程
supervisorctl reoload all                           #重载所有进程
```

## **七、Supervisord开启启动配置**

```text
vim /usr/lib/systemd/system/supervisord.service
[Unit]
Description=Process Monitoring and Control Daemon
After=rc-local.service nss-user-lookup.target

[Service]
Type=forking
ExecStart=/usr/bin/supervisord -c /etc/supervisord.conf

[Install]
WantedBy=multi-user.target
```



```text
systemctl enable supervisord
systemctl is-enabled supervisord
```