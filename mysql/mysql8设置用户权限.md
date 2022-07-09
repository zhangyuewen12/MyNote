mysql8设置用户权限报错You have an error in your SQL syntax；right syntax to use near ‘IDENTIFIED BY

sherryaxx

于 2022-03-03 16:03:57 发布

355
 收藏
分类专栏： 云平台 文章标签： mysql
版权

云平台
专栏收录该内容
2 篇文章0 订阅
订阅专栏
mysql 8 设置用户权限命令和之前不一样
之前：
grant all privileges on *.* to 'myuser'@'%' identified by 'mypassword' with grant option;

报错：

You have an error in your SQL syntax; check the manual that corresponds to your MySQL server version for the right syntax to use near ‘IDENTIFIED BY ‘123456’ WITH GRANT OPTION’ at line 1

现在：
CREATE USER 'myuser'@'%' IDENTIFIED BY '123456'; #创建用户
grant all privileges on *.* to 'myuser'@'%' ; #分配权限
FLUSH PRIVILEGES; #刷新权限
————————————————
版权声明：本文为CSDN博主「sherryaxx」的原创文章，遵循CC 4.0 BY-SA版权协议，转载请附上原文出处链接及本声明。
原文链接：https://blog.csdn.net/sherryaxx/article/details/123256702