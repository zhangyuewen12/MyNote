# Mac-brew install mysql
```
1.brew search mysql

==> Formulae
automysqlbackup            mysql-connector-c          mysql@5.5
mysql                                mysql-connector-c++        mysql@5.6
mysql++                           mysql-sandbox              mysql@5.7 ✔
mysql-client                    mysql-search-replace       mysqltuner
mysql-cluster                  mysql-utilities

==> Casks
mysql-connector-python     mysql-utilities            navicat-for-mysql
mysql-shell                mysqlworkbench             sqlpro-for-mysql


2. brew install mysql@5.7
3. #配置环境变量
echo 'export PATH="/usr/local/opt/mysql@5.7/bin:$PATH"' >> ~/.bash_profile
source ~/.bash_profile

#启动服务
mysql.server start
brew services start mysql@5.7

#初始化，设置密码
mysql_secure_installation

[root@zhangMySQL5711 bin]# mysql_secure_installation            
Enter password: 

Securing the MySQL server deployment.


VALIDATE PASSWORD PLUGIN can be used to test passwords                        //密码验证插件，为了提高安全性，需要验证密码
and improve security. It checks the strength of password                      // 它会检查密码的强度
and allows the users to set only those passwords which are                    //只允许用户设置足够安全的密码
secure enough. Would you like to setup VALIDATE PASSWORD plugin?              //提示安装密码验证插件

Press y|Y for Yes, any other key for No: y

There are three levels of password validation policy:                      //三个等级的验证策略

LOW Length >= 8                                                             //最小长度大于等于8个字符
MEDIUM Length >= 8, numeric, mixed case, and special characters             //数字，字母，特殊字符 混合，具体的应该是至少1个数字，1个字母，1个特殊字符，长度不超过32个字符
STRONG Length >= 8, numeric, mixed case, special characters and dictionary file   //  最严格，加上了，字典文件

Please enter 0 = LOW, 1 = MEDIUM and 2 = STRONG: 2                      //这里我选择2 MEDIUM
Using existing password for root.

Estimated strength of the password: 50                                   //这里也是密码强度的评级
Change the password for root ? ((Press y|Y for Yes, any other key for No) : y

New password:                                                        //密码 

Re-enter new password: 

Estimated strength of the password: 50 
Do you wish to continue with the password provided?(Press y|Y for Yes, any other key for No) : y     //提示要使用刚刚输入的密码吗？
 ... Failed! Error: Your password does not satisfy the current policy requirements                   //插件验证不通过，不符合当前安全要求级别

New password:                                                        //密码

Re-enter new password: 

Estimated strength of the password: 100 
Do you wish to continue with the password provided?(Press y|Y for Yes, any other key for No) : y
By default, a MySQL installation has an anonymous user,               //默认情况下，MySQL有一个匿名用户，
allowing anyone to log into MySQL without having to have              //这个匿名用户，不必有一个用户为他们创建，匿名用户允许任何人登录到MySQL，
a user account created for them. This is intended only for            //这只是为了方便测试使用
testing, and to make the installation go a bit smoother.
You should remove them before moving into a production                //在正式环境使用的时候，建议你移除它
environment.

Remove anonymous users? (Press y|Y for Yes, any other key for No) : y                //提示移除匿名用户
Success.

Normally, root should only be allowed to connect from                        //一般情况下，root用户只允许使用"localhost"方式登录，
'localhost'. This ensures that someone cannot guess at                       // 以此确保，不能被某些人通过网络的方式访问
the root password from the network.

Disallow root login remotely? (Press y|Y for Yes, any other key for No) : n          //

 ... skipping.
By default, MySQL comes with a database named 'test' that                      //默认情况下，MySQL数据库中有一个任何用户都可以访问的test库，
anyone can access. This is also intended only for testing,                     //这也仅仅是为了测试
and should be removed before moving into a production                          // 在正式环境下，应该移除掉
environment.


Remove test database and access to it? (Press y|Y for Yes, any other key for No) : y
 - Dropping test database...
Success.

 - Removing privileges on test database...
Success.

Reloading the privilege tables will ensure that all changes                           //刷新权限表，以确保所有的修改可以立刻生效
made so far will take effect immediately.

Reload privilege tables now? (Press y|Y for Yes, any other key for No) : y
Success.

All done! 

```