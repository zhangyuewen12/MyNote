一 测试数据构建

二 基本使用(单独使用）

三 聚合函数中的DISTINCT



一、建表

```
DROP TABLE IF EXISTS `test_distinct`;
CREATE TABLE `test_distinct` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `province` varchar(255) DEFAULT NULL,
  `city` varchar(255) DEFAULT NULL,
  `username` varchar(255) DEFAULT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=8 DEFAULT CHARSET=gbk;


INSERT INTO `test_distinct` VALUES ('1', 'BJ', 'BJ', 'houchenggong');
INSERT INTO `test_distinct` VALUES ('2', 'LN', 'DL', 'zhenhuasun');
INSERT INTO `test_distinct` VALUES ('3', 'LN', 'DL', 'yueweihua');
INSERT INTO `test_distinct` VALUES ('4', 'BJ', 'BJ', 'sunzhenhua');
INSERT INTO `test_distinct` VALUES ('5', 'LN', 'TL', 'fengwenquan');
INSERT INTO `test_distinct` VALUES ('6', 'LN', 'DL', 'renquan');
INSERT INTO `test_distinct` VALUES ('7', 'LN', 'DL', 'wuxin');
```



二、样例数据

<img src="/Users/zyw/Library/Application Support/typora-user-images/image-20220808102312661.png" alt="image-20220808102312661" style="zoom:50%;" />

三、基础用法

基本使用(单独使用）

> distinct一般是用来去除查询结果中的重复记录的，而且这个语句在select、insert、delete和update中只可以在select中使用，
>
> 具体的语法如下：
>
> select distinct expression[,expression...] from tables [where conditions];
> 这里的expressions可以是多个字段。





## 1.对一列操作，表示选取该列不重复的数据项，这是用的比较多的一种用法

<img src="/Users/zyw/Library/Application Support/typora-user-images/image-20220808102349783.png" alt="image-20220808102349783" style="zoom:50%;" />



## 2.对多列操作，表示选取 多列都不重复的数据，相当于 多列拼接的记录 的整个一条记录 , 不重复的记录。

<img src="/Users/zyw/Library/Application Support/typora-user-images/image-20220808102453140.png" alt="image-20220808102453140" style="zoom:50%;" />



# 3.DISTINCT 必须放在第一个参数。



<img src="/Users/zyw/Library/Application Support/typora-user-images/image-20220808102738441.png" alt="image-20220808102738441" style="zoom:50%;" />