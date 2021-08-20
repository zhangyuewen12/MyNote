# Shell编程入门（四)

1. shell脚本调试

* 使用bash -x 命令调试
创建debug.sh文件，内容如下： 

```
#!/bin/bash
#Filename: debug.sh
echo "scripting"
echo "debuging"
ls +



root@sparkslave02:~/ShellLearning/Chapter12# bash -x debug.sh 
+ echo scripting
scripting
+ echo debuging
debuging
+ ls +
ls: cannot access +: No such file or directory
```

* 使用set -x; set +x；命令进行局部调试


```
#!/bin/bash
#Filename: debug2.sh
for i in {1..6}
do
set -x
//set -x表示跟在该命令后的脚本输出调试信息
echo $i
//set +x表示从此处禁用调试
set +x
done
echo "Script executed"


上面的代码意味着，只会打印输出echo $i，具体调试信息输出如下：

root@sparkslave02:~/ShellLearning/Chapter12# ./debug2.sh 
+ echo 1
1
+ set +x
+ echo 2
2
+ set +x
+ echo 3
3
+ set +x
+ echo 4
4
+ set +x
+ echo 5
5
+ set +x
+ echo 6
6
+ set +x
```

2. shell函数

```
function fname()
{
   shell脚本语句；
}
```

3. shell控制结构初步

* for i in $(seq 10)

```
root@sparkslave02:~/ShellLearning/Chapter12# vim forloop.sh
for i in $(seq 10)
do
echo $i
done

root@sparkslave02:~/ShellLearning/Chapter12# chmod a+x forloop.sh 
root@sparkslave02:~/ShellLearning/Chapter12# ./forloop.sh 
1
2
3
4
5
6
7
8
9
10
```

* for((i=1;i<=10;i++))

```
for((i=2;i<=10;i++))
do
echo $i
done
```

* for i in ls

```
root@sparkslave02:~/ShellLearning/Chapter12# vim forloop3.sh
for i in `ls`
do
echo $i
done

root@sparkslave02:~/ShellLearning/Chapter12# chmod a+x forloop3.sh
root@sparkslave02:~/ShellLearning/Chapter12# ./forloop3.sh 
debug2.sh
debug.sh
forloop2.sh
forloop3.sh
forloop.sh
functionDemo.sh
```


* for i in ${arr[*]}

```
root@sparkslave02:~/ShellLearning/Chapter12# vim forloop4.sh
arr=(0 1 2 3)
for i in ${arr[*]}
do
echo ${arr[i]}
done

root@sparkslave02:~/ShellLearning/Chapter12# ./forloop4.sh 
0
1
2
3
```