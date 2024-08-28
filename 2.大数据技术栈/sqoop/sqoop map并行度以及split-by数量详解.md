# sqoop map并行度以及split-by数量详解

## 1. sqoop -m

```
有一种情况可以不需要指定 sqoop -m
就是你要同步的表有主键,这样的话sqoop默认会按照主键进行split然后分成4个map并行执行
```

## 2.sqoop --split-by

--split-by 表示按照某个一字段去分割成多个map
–split-by 默认能跟 int类型 和 date类型的字段
一般 --split-by 要配合 -m使用
–split-by 要分割的字段的值不能有null值存在，不然会丢数据

## 3.sqoop --split-by 分割机制

### 3.1 分割int类型

比如要同步的表中id为int类型,id的值有 1,1,2,3,3,4,5,10,13
sqoop xxxxxx 
--split-by id \
-m 3
默认会选取 [MIN(id),MAX(id)+1) 左闭右开 区间去进行分割  分割成3个map
此时的split-size就是 MAX(id)-MIN(id)=(13-1)/3=4

```
map1------> [1,1+4)--->[1,5)
map2------> [5,5+4)--->[5,9)
map3------> [9,9+4+1)-->[9,14)
```

### 3.2 分割date类型

比如要同步的表中createtimee为date类型
sqoop xxxxxx
--split-by createtime \
-m 4
假设此时MIN(createtime)为'2016-10-08 16:54:44'--------->1475916884
       MAX(createtime)为'2019-03-02 21:53:40'--------->1551534820
此时sqoop内部会将date解析为timestamp类型去split,内部的时间戳会转化成13位
按照[Min(createtime),Max(createtime)+1) 左闭右开 区间进行分割,分割成4个map
此时的split-size就是 Max(createtime)-Min(createtime)=(1551534820000-1475916884000)/4=18904484000
上面的000只是sqoop进行分割时候默认认定13为的时间戳,实际每个map的区间如下

```
map1------> [1475916884,1475916884+18904484)--->[1475916884,1494821368)
map2------> [1494821368,1494821368+18904484)--->[1494821368,1513725852)
map3------> [1513725852,1513725852+18904484)--->[1513725852,1532630336)
map3------> [1532630336,1532630336+18904484]--->[1532630336,1551534820]
--------------------------------------------------
```

注意split-size 表示的是区间的大小,并不代表数据条数

### 3.3 分割varchar类型

```
默认--split-by 是不支持分割varchar类型的字段,如果表中缺少int或者date类型字段去分割，也可以考虑使用varchar类型的字段去分割，但是需要指定一个参数
sqoop import
-D org.apache.sqoop.splitter.allow_text_splitter=true; \
--connect xxx \
xxxxxxxxxxxx
-----------------------------------
对于varchar内部具体怎么分割 ascii还是其他方式，这个我测试不出来
```



## 4. sqoop --split-limit

- soop --split-limit 是来限制 --split-by 与 -m 配合使用之后生成的split-size的区间大小
- sqoop --split-limit 使用有一个约定，即–split-by 的字段必须为 int 或者 date类型

## 5. map的数量由什么决定

```
1.如果不指定-m , 并且原表中有主键, 默认4个map

2.如果原表中没有主键,则需要指定-m 1，此时1个map

3.如果指定–split-by xxx -m n,此时n个map

注意：如果n的数量小于集群结点数,就是n个map
     如果n的数量大于集群结点数，map的数量就是集群结点数量
1
2
4.如果指定–split-by xxx -m n --split-limit xxx,此时的map与–split-limit 有关

例如3.2测试中将split-limit设定为计算的split-size的一般，此时map数量翻倍
注意：此时map的数量与集群结点的限制无关
     集群结点的限制只是代表最大可并行执行的map的数量
```

