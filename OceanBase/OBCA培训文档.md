# 一、OceanBase数据库产品家族



<img src="/Users/zyw/Library/Application Support/typora-user-images/image-20230225175733765.png" alt="image-20230225175733765" style="zoom:50%;" />



## 1.OCP产品架构和功能，一站式的管理运维工具

![image-20230225180221643](/Users/zyw/Library/Application Support/typora-user-images/image-20230225180221643.png)

## 2.OceanBase开发者中心（ODC）产品架构

![image-20230225180302904](/Users/zyw/Library/Application Support/typora-user-images/image-20230225180302904.png)

<img src="/Users/zyw/Library/Application Support/typora-user-images/image-20230225180402392.png" alt="image-20230225180402392" style="zoom:50%;" />

## 3.OMS核心功能简介

![image-20230225180447425](/Users/zyw/Library/Application Support/typora-user-images/image-20230225180447425.png)

# 二、OceanBase支持多种部署形式

<img src="/Users/zyw/Library/Application Support/typora-user-images/image-20230225180535751.png" alt="image-20230225180535751" style="zoom:50%;" />

# 三、OceanBase基本概念介绍

![image-20230225181222629](/Users/zyw/Library/Application Support/typora-user-images/image-20230225181222629.png)

## 1.集群、Zone和OB Server

<img src="/Users/zyw/Library/Application Support/typora-user-images/image-20230225181533277.png" alt="image-20230225181533277" style="zoom:50%;" />

## 2.RootService总控服务（RS）

![image-20230225182004594](/Users/zyw/Library/Application Support/typora-user-images/image-20230225182004594.png)

## 3.多租户机制，资源隔离，数据隔离

![image-20230225182206241](/Users/zyw/Library/Application Support/typora-user-images/image-20230225182206241.png)

 每个租户拥有若干资源池（Resource Pool）

![image-20230225182604987](/Users/zyw/Library/Application Support/typora-user-images/image-20230225182604987.png)

OceanBase可以为不同类型的应用分配不同类型和不同数量的Unit，满足业务不同的需求。资源并不是静态 的，可以随着业务的发展不断调整（调高或者调低）





# 四、创建租户

OCP也可以更方便的创建租户，为了更详细的讲解创建过程，以命令行方式讲解：
 • 步骤一、创建“资源单元规格”，create resource unit命令，指定资源单元的规格；

 • 步骤二、创建“资源池”，create resource pool命令，根据资源单元规格的定义创建资源单元，并赋给一个新的资源池；
 • 步骤三、创建租户，create tenant命令，将资源池赋给一个新的租户；



## 1.创建资源单元（仅仅是规格定义，不实际分配资源）

```
CREATE RESOURCE UNIT unit1
max_cpu = 4,
max_memory = 10737418240, -- 10GB
min_memory = 10737418240, -- 10GB
max_iops = 1000,
min_iops = 128,
max_session_num = 300,
max_disk_size = 21474836480 -- 20GB
;
```



## 2.创建资源池（会实际创建unit，按规格定义分配资源）

```
CREATE RESOURCE POOL pool1
UNIT = 'unit1',
UNIT_NUM = 1,
ZONE_LIST = ('zone1', 'zone2', 'zone3')
;

• 每个resource pool在每个OB Server上只能有一个resource unit；如果unit_num大于1，每个zone内都必须有和unit_num对应数目的机器
• zone List一般与zone个数保持一致
• 如果在某个zone内找不到有足够剩余资源的机器来创建resource unit，资源池会创建失败
```



## 3.检查集群状态

查看集群中的整体资源分配情况：__all_virtual_server_stat;

<img src="/Users/zyw/Library/Application Support/typora-user-images/image-20230226204144591.png" alt="image-20230226204144591" style="zoom:50%;" />



## 4.查看unit

查看系统中定义的resource unit规格：select * from __all_unit_config;

<img src="/Users/zyw/Library/Application Support/typora-user-images/image-20230226204250756.png" alt="image-20230226204250756" style="zoom:50%;" />

查看系统中已经分配的resource unit: select * from __all_unit;

![image-20230226204305481](/Users/zyw/Library/Application Support/typora-user-images/image-20230226204305481.png)

## 5.系统日志

OB Server 日志（/home/admin/oceanbase/log目录）

Ø observer.log：observer运行时的日志文件 

Ø rootservice.log：observer上RootServer的日志文件 

Ø election.log：observer上选举模块的日志文件 

控制OB Server日志文件个数

 Ø 为了避免硬盘被日志填满，可以开启日志循环 

Ø enable_syslog_recycle = True; max_syslog_file_count =<count>

  日志级别

 Ø syslog_level = [DEBUG,TRACE,INFO,WARN,USER_ERR,ERROR]



# 五、数据分区与分区副本

![image-20230226204722620](/Users/zyw/Library/Application Support/typora-user-images/image-20230226204722620.png)



# 六、支持三种副本，满足不同业务类型的需求

<img src="/Users/zyw/Library/Application Support/typora-user-images/image-20230309141856022.png" alt="image-20230309141856022" style="zoom:50%;" />

# 七、多副本一致性协议

![image-20230309142424201](/Users/zyw/Library/Application Support/typora-user-images/image-20230309142424201.png)



# 八、自动负载均衡与智能路由

![image-20230309142747535](/Users/zyw/Library/Application Support/typora-user-images/image-20230309142747535.png)

# 十、通过多副本同步Redo Log确保数据持久化

![image-20230309142828475](/Users/zyw/Library/Application Support/typora-user-images/image-20230309142828475.png)



# 十一、OB Proxy为应用提供智能路由服务，应用透明访问

![image-20230309143539540](/Users/zyw/Library/Application Support/typora-user-images/image-20230309143539540.png)



## 十二、通过设置Primary Zone，将业务汇聚到特定Zone

![image-20230309144048865](/Users/zyw/Library/Application Support/typora-user-images/image-20230309144048865.png)



# 十三、Table Group，将多个表的相同分区ID的主副本聚集在一个 OB Server中，减少分布式事务引入的开销

![image-20230309145038158](/Users/zyw/Library/Application Support/typora-user-images/image-20230309145038158.png)

![image-20230309145119658](/Users/zyw/Library/Application Support/typora-user-images/image-20230309145119658.png)



