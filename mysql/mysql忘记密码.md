**自己电脑数据库密码 12345678 2020/12/14**


1、关闭mysql服务 sc stop mysql
 
2、以安全模式启动mysql
mysqld -- skip-grant-tables
建议通过修改配置文件实现安全启动：
vi /etc/my.cnf
在[mysqld]段中加上一句: skip-grant-tables
保存并退出
 
3、重新启动mysqld
service mysqld restart
 
4、登陆mysql
直接输入mysql -u root -p 回车, （忽略密码）再回车；
 
5、更改root密码
UPDATE mysql.user SET Password=PASSWORD('xxx') where USER='root';
更改密码时出现错误：
ERROR 1054 (42S22): Unknown column 'Password' in 'field list'
原因：数据库下没有Password字段
解决办法：
查找数据库mysql的user表
      show databases;
      use mysql;
      show tables;
      select * from user;   
发现没有Password字段，Password字段改成了authentication_string
使用该字段重新修改root密码即可：
update mysql.user set authentication_string=password('xxx') where user='root' ;
注意： mysql新版本用于存用户密码的字段名为authentication_string而不是 password，且新密码必须使用password函数进行加密
 
6、修改完密码以后再打开配置文件，将skip-grant-tables注释掉。
vi /etc/my.cnf
#skip-grant-tables
 
7、重新运行mysql，即可使用新密码进行登陆。
service mysqld restart


## 异常内容：Error Code: 1175. You are using safe update mode and you tried to update a table without a WHERE that uses a KEY column To disable safe mode, toggle the option in Preferences -> SQL Queries and reconnect

这是因为MySql运行在safe-updates模式下，该模式会导致非主键条件下无法执行update或者delete命令，执行命令SET SQL_SAFE_UPDATES = 0;修改下数据库模式