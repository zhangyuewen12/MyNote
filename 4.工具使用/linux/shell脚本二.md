#shell脚本二
本节主要内容

* shell命令行中的基本运算
* 文件描述符与文件重定向


1. shell命令行中的基本运算

Linux Bash shell 命令行的变量都被解析成字符串

```
//变量被解析为字符串
root@sparkmaster:~/ShellLearning/chapter09# first=1
root@sparkmaster:~/ShellLearning/chapter09# second=2

//并不会进行加法操作，而是两个字符串拼接
root@sparkmaster:~/ShellLearning/chapter09# $first+$second
1+2: command not found

zywdeMacBook-Pro:~ zyw$ a=1
zywdeMacBook-Pro:~ zyw$ b=2
zywdeMacBook-Pro:~ zyw$ a+b
-bash: a+b: command not found
zywdeMacBook-Pro:~ zyw$ $a+$b
-bash: 1+2: command not found
```
那如何要进行基本的加减乘除等运算，怎么办呢？有三种命令可以做到，它们是let, (( )), and []。先来看一下let命令  

```
//使用let命令，会进行加法运算
root@sparkmaster:~/ShellLearning/chapter09# let result=first+second

root@sparkmaster:~/ShellLearning/chapter09# echo $result
3
```
let命令还提供c、c++风格的自增、自减等操作，如

```
root@sparkmaster:~/ShellLearning/chapter09# first=2
root@sparkmaster:~/ShellLearning/chapter09# $first
2: command not found
//自增操作
root@sparkmaster:~/ShellLearning/chapter09# let first++
root@sparkmaster:~/ShellLearning/chapter09# echo ${first}
3
//自减操作
root@sparkmaster:~/ShellLearning/chapter09# let first--
root@sparkmaster:~/ShellLearning/chapter09# echo ${first}
2
//相当于let first=first+10
root@sparkmaster:~/ShellLearning/chapter09# let first+=10
root@sparkmaster:~/ShellLearning/chapter09# echo ${first}
12
//相当于let first=first-10，其它操作如乘、除类似
root@sparkmaster:~/ShellLearning/chapter09# let first-=10
root@sparkmaster:~/ShellLearning/chapter09# echo ${first}
2
```
[]命令的功能与let命令类似，如  

```
root@sparkmaster:~/ShellLearning/chapter09# first=5
root@sparkmaster:~/ShellLearning/chapter09# second=6
root@sparkmaster:~/ShellLearning/chapter09# result=$[first+second]
root@sparkmaster:~/ShellLearning/chapter09# echo $result 
11
//result=$[$first+$second]与result=$[first+second]等同
root@sparkmaster:~/ShellLearning/chapter09# result=$[$first+$second]
root@sparkmaster:~/ShellLearning/chapter09# echo $result 
11
```

也可以使用(( )) 命令进行，如：  

```
root@sparkmaster:~/ShellLearning/chapter09# reslut=$((first+second))
root@sparkmaster:~/ShellLearning/chapter09# echo $result 
11
```
需要注意的是上述命令只对整型数值有效，不适用于浮点数

```
oot@sparkmaster:~/ShellLearning/chapter09# result=$[first+second]
bash: 5.5: syntax error: invalid arithmetic operator (error token is ".5")

root@sparkmaster:~/ShellLearning/chapter09# let resul=first+second
bash: let: 5.5: syntax error: invalid arithmetic operator (error token is ".5")
```

如果有浮点数参与运算，可以将echo与bc命令结合起来使用，代码如下：

```
root@sparkmaster:~/ShellLearning/chapter09# echo "$first+$second" | bc
12.0
root@sparkmaster:~/ShellLearning/chapter09# echo "$first*$second" | bc
35.7
```

2. 文件描述符与文件重定向

文件描述符（File descriptors ）与文件的输入输出相关，用整数表示，最常用的三种文件描述符号为stdin、stdout及stderr。stdin表示标准输入（standard input），文件描述符为0；stdout表示标准输出（standard output），文件描述符为1；stderr表示标准错误（standard error），文件描述为2。 

重定向时可以根据将重定向命令结合起来使用，如

```
//cmd命令无效，会产生标准错误，此时错误信息会重定向到文件stderr.txt文件当中
root@sparkmaster:~/ShellLearning/chapter10# cmd 2>stderr.txt 1>stdout.txt

root@sparkmaster:~/ShellLearning/chapter10# cat stderr.txt 
No command 'cmd' found, did you mean:
 Command 'dcmd' from package 'devscripts' (main)
 Command 'wmd' from package 'wml' (universe)
 Command 'tcmd' from package 'tcm' (universe)
 Command 'cmp' from package 'diffutils' (main)
 Command 'qcmd' from package 'renameutils' (universe)
 Command 'mmd' from package 'mtools' (main)
 Command 'cm' from package 'config-manager' (universe)
 Command 'mcd' from package 'mtools' (main)
 Command 'icmd' from package 'renameutils' (universe)
cmd: command not found
//stdout.txt中无内容

root@sparkmaster:~/ShellLearning/chapter10# cat stdout.txt 
//ls命令合法，会产生标准输出，此时会被重定向到stdout.txt文件当中
root@sparkmaster:~/ShellLearning/chapter10# ls 2>stderr.txt 1>stdout.txt 
root@sparkmaster:~/ShellLearning/chapter10# cat stdout.txt 
shell2.txt
shellError.txt
shell.txt
stderr.txt
stdout.txt
```

将标准输出与标准错误输出都重定向到一个文件，此时可以使用下列命令
```
//&>将标准错误输出转换为标准输出，相当于2>&1
root@sparkmaster:~/ShellLearning/chapter10# cmd &> output.txt
root@sparkmaster:~/ShellLearning/chapter10# cat output.txt 
No command 'cmd' found, did you mean:
 Command 'dcmd' from package 'devscripts' (main)
 Command 'wmd' from package 'wml' (universe)
 Command 'tcmd' from package 'tcm' (universe)
 Command 'cmp' from package 'diffutils' (main)
 Command 'qcmd' from package 'renameutils' (universe)
 Command 'mmd' from package 'mtools' (main)
 Command 'cm' from package 'config-manager' (universe)
 Command 'mcd' from package 'mtools' (main)
 Command 'icmd' from package 'renameutils' (universe)
cmd: command not found
root@sparkmaster:~/ShellLearning/chapter10# ls &>output.txt 
root@sparkmaster:~/ShellLearning/chapter10# cat output.txt 
output.txt
shell2.txt
shellError.txt
shell.txt
stderr.txt
stdout.txt
```

将标准错误输出重定向到/dev/null文件当中，/dev/null就像一个垃圾黑洞

```
//将错误信息丢弃
root@sparkmaster:~/ShellLearning/chapter10# cmd 2> /dev/null
```

标准错误输出或标准输出还可以作为管道命令的标准输入，例如

```
//标准输出作为另外一个命令的标准输入
root@sparkmaster:~/ShellLearning/chapter10# cat stdout.txt | more
shell2.txt
shellError.txt
shell.txt
stderr.txt
stdout.txt
//标准错误输出作为另一个命令的标准输入
root@sparkmaster:~/ShellLearning/chapter10# ls + | more
ls: cannot access +: No such file or directory
```

