最近再开发中现场环境使用了kerberos，在读取hive库的数据时需要先进行认证。具体的认证步骤如下。

1.需要如下东西 krb5.conf（配置文件） ， user.keytab（秘钥文件）、username（登录名）

2.做如下设置

```java
org.apache.hadoop.conf.Configuration configuration = new org.apache.hadoop.conf.Configuration();
configuration.set("hadoop.security.authentication", "Kerberos");
configuration.set("keytab.file", "D:\\MyJavaProject\\javakit\\conf\\user.keytab");
configuration.set("kerberos.principal", "hive/hadoop.hadoop.com@HADOOP.COM");
System.setProperty("java.security.krb5.conf", "D:\\MyJavaProject\\javakit\\conf\\krb5.conf")；
UserGroupInformation.setConfiguration(conf);
UserGroupInformation.loginUserFromKeytab("xxxx(登录名)", "D:\\MyJavaProject\\javakit\\conf\\user.keytab");
```




3.使用jdbc读取hive数据

// 创建hive连接

```java
Class.forName("org.apache.hive.jdbc.HiveDriver");
Class.forName("org.apache.hive.jdbc.HiveDriver");
Connection connection = DriverManager.getConnection("jdbc:hive2://ip:端口/库名;principal=hive/hadoop.hadoop.com@HADOOP.COM;sasl.qop=auth-conf");
String sql2 = " select  id from  表名";
PreparedStatement show_tables = connection.prepareStatement(sql2);
ResultSet resultSet = show_tables.executeQuery();
 while (resultSet.next()){
 resultSet.getObject(1);
}
```


使用了kerberos后数据库的用户名密码就没有意义了，只需要秘钥就好

4.kerberos认证24小时过期问题，可以开启一个线程周期执行认证就好

```
ScheduledThreadPoolExecutor scheduled = new ScheduledThreadPoolExecutor(1);
scheduled.scheduleAtFixedRate(()->{
            LOG.info("kerberos认证:"+ new SimpleDateFormat("yyyy-MM-dd HH:mm:ss").format(new Date()));
            org.apache.hadoop.conf.Configuration configuration = new org.apache.hadoop.conf.Configuration();
configuration.set("hadoop.security.authentication", "Kerberos");
configuration.set("keytab.file", "D:\\MyJavaProject\\javakit\\conf\\user.keytab");
configuration.set("kerberos.principal", "hive/hadoop.hadoop.com@HADOOP.COM");
System.setProperty("java.security.krb5.conf", "D:\\MyJavaProject\\javakit\\conf\\krb5.conf")；
UserGroupInformation.setConfiguration(conf);
UserGroupInformation.loginUserFromKeytab("xxxx(登录名)", "D:\\MyJavaProject\\javakit\\conf\\user.keytab");
```