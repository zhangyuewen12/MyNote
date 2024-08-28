
#Shell编程入门（三)
本节主要内容

* shell数组
* shell命令别名
* 时间操作

1. Shell数组

shell中的数组有两种，一种为普通数组，另外的一种称为关联数组。普通数据的存取通过整数进行，关联数组的存取通过字符串进行。具体如下:

```
//用()定义一个数组，注意数组元素间不能用,否则达不到预期目的
root@sparkmaster:~/ShellLearning/chapter11# arr=(1 2 3 4 5 6)
root@sparkmaster:~/ShellLearning/chapter11# echo ${arr[0]}
1

//用,号的话，数组只有一个元素
root@sparkmaster:~/ShellLearning/chapter11# arr=(1,2,3,4,5,6)
root@sparkmaster:~/ShellLearning/chapter11# echo ${arr[0]}
1,2,3,4,5,6

除了（）定义数组外，还可以采用逐个赋值的方法，例如

root@sparkmaster:~/ShellLearning/chapter11# strArr[0]="hello"
root@sparkmaster:~/ShellLearning/chapter11# strArr[1]="hello1"
root@sparkmaster:~/ShellLearning/chapter11# echo ${strArr[0]}
hello
```

输出数组中的所有内容及数组的长度，代码如下：

```
//用*号将输出数组中的所有内容
root@sparkmaster:~/ShellLearning/chapter11# echo ${strArr[*]}
hello hello1
//${#strArr[*]}取得数组的长度
root@sparkmaster:~/ShellLearning/chapter11# echo ${#strArr[*]}
2
```

关联数组的定义与普通数组不一样，关联数组需要使用declare命令进行声明，具体如下：

```
//declare -A associative_array声明一个关联数组
root@sparkmaster:~/ShellLearning/chapter11# declare -A associative_array
//填充内容
root@sparkmaster:~/ShellLearning/chapter11# associative_array=([key1]=value1 [key2]=value2 [key3]=value3)
//获取关联数组内容
root@sparkmaster:~/ShellLearning/chapter11# echo ${associative_array[key1]}
value1
```

在实际使用时，常常需要确定数组的索引值，具体使用代码如下：

```
//获取关联数组的索引
root@sparkmaster:~/ShellLearning/chapter11# echo ${!associative_array[*]}
key3 key2 key1

//获取普通数组的索引
root@sparkmaster:~/ShellLearning/chapter11# echo ${!strArr[*]}
0 1
```

2. shell命令别名

> alias表示的是给命令取别名，例如alias ll=’ls -alF’，表示ll是命令’ls -alF’的别名

```
//给sudo apt-get install取个别名install
root@sparkmaster:~/ShellLearning/chapter11# alias install='sudo apt-get install'

//直接使用install命令代替sudo apt-get install命令
root@sparkmaster:~/ShellLearning/chapter11# install opencv

//但是需要注意的是在终端取别名，一旦终端关闭，别名命令不会保存
//如果想永久使用即开机后该别名命令就生效的话，则需要将别名命令重定向
//保存到~/.bashrc文件中
root@sparkmaster:~/ShellLearning/chapter11# echo 'alias install="sudo apt-get install"' >> ~/.bashrc
```

3.时间操作

shell命令有许多时间操作命令，具体使用如下


```
zywdeMacBook-Pro:~ zyw$ date "+%Y-%b-%d"
2020-12-01
```

```
//查看当前时间
root@sparkmaster:~/ShellLearning/chapter11# date
Mon Oct  5 00:06:11 PDT 2015

//查看当前是星期几
root@sparkmaster:~/ShellLearning/chapter11# date +%A
Monday
//查看当前月份
root@sparkmaster:~/ShellLearning/chapter11# date +%B
October
//查看当前是当月的第几天
root@sparkmaster:~/ShellLearning/chapter11# date +%d
05


date命令参数列表如下：

参数作用	参数
Weekday	%a 简写法，例如Monday，简写为Mon;%A，全拼法Monday
Month	%b 简写法，例如October，简写为Otc;%B，全拼法October
Day	%d
格式化日期 (mm/dd/yy)	%D
Year	%y，简写法，例如2010，简写为10；%Y，全写法2010
Hour	%I 或%H
Minute	%M
Second	%S
Nano second(毫秒）	%N
Unix系统时间	%s
```



