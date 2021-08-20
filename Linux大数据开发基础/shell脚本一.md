#Linux大数据开发基础：第九节：Shell编程入门(一)
本节主要内容

* shell编程简介
* 变量定义
* 常用特殊变量

1. 执行shell脚本的方式两种方式。一种是通过sh命令，另外一种是自执行方式。下面给出了具体演示

```
//完成后直接利用sh命令执行
root@sparkmaster:~/ShellLearning/chapter09# sh HelloWorld.sh 
Hello Shell
//自执行(self execute）方式，由于没有给文件加执行权限，所以执行失败
root@sparkmaster:~/ShellLearning/chapter09# ./HelloWorld.sh
bash: ./HelloWorld.sh: Permission denied
root@sparkmaster:~/ShellLearning/chapter09# ls -l
total 4
-rw-r--r-- 1 root root 31 2015-09-30 06:29 HelloWorld.sh
//chmod给文件加执行权限
root@sparkmaster:~/ShellLearning/chapter09# chmod a+x HelloWorld.sh 
//再通过自执行方式
root@sparkmaster:~/ShellLearning/chapter09# ./HelloWorld.sh
Hello Shell
root@sparkmaster:~/ShellLearning/chapter09# 
```
前面提到，脚本第一行是#!/bin/bash，它的作用是提示该脚本的执行路径是/bin/bash，对自执行方式有用，自执行方式最终是通过/bin/bash HelloWorld.sh 执行脚本，而利用sh HelloWorld.sh命令执行脚本时，#!/bin/bash 不起作用。

* 如果需要在一行执行多条语句，则需要使用";"分割  
echo "Hello Shell";echo "Hello World"

* echo命令用于输出一行内容（包括行符），后面的输出内容除可以用”“双引号之外，也可以不加，也可以用单引号”例如：

```
//不带引号，输出的是变量内容
root@sparkmaster:~/ShellLearning/chapter09# echo $JAVA_HOME
/hadoopLearning/jdk1.7.0_67
//双引号，输出的也是变量内容
root@sparkmaster:~/ShellLearning/chapter09# echo "$JAVA_HOME"
/hadoopLearning/jdk1.7.0_67
//单引号的话，内容原样输出，不会解析变量值
root@sparkmaster:~/ShellLearning/chapter09# echo '$JAVA_HOME'
$JAVA_HOME
```

2. 变量定义

```
自定义变量具有只能在当前进程中使用，当开启子进程时，变量在子进程中不起作用，如果需要父进程中定义的变量在子进程中也能够使用，则需要将其设置为环境变量，环境变量使用export命令进行定义，代码如下：

root@sparkmaster:~/ShellLearning/chapter09# export t1
root@sparkmaster:~/ShellLearning/chapter09# bash
//子进程中现在可能使用
root@sparkmaster:~/ShellLearning/chapter09# $t1
123: command not found
root@sparkmaster:~/ShellLearning/chapter09# echo $t1
123
```

不过，这样定义的环境变量，在命令行窗口关闭或系统重新启动时会丢失，如果需要在机器启动时环境变量就自动生效的话，可以将环境变量定义在~/.bashrc或/etc/profile文件中，其中~/.bashrc只对当前用户（例如当前用户是zhouzhihu，则只对本用户有效)，如果想对所有用户都有效，则将其放置在/etc/profile文件中。 

3. 常用特殊变量
```
$# 是传给脚本的参数个数
$0 是脚本本身的名字
$1 是传递给该shell脚本的第一个参数
$2 是传递给该shell脚本的第二个参数
$@ 是传给脚本的所有参数的列表
$* 是以一个单字符串显示所有向脚本传递的参数，与位置变量不同，参数可超过9个
$$ 是脚本运行的当前进程ID号
$? 是显示最后命令的退出状态，0表示没有错误，其他表示有错误
```
