# [查看mysql连接数和状态](https://www.cnblogs.com/pegasus827/p/8692290.html)



1. show full processlist; 

若不加上full选项，则最多显示100条记录。



```
查看当前各用户连接数据库的数量
select USER , count(*) from information_schema.processlist group by USER;


查看连接到数据库的客户端ip及各连接数
SELECT substring_index(host, ':',1) AS host_name,state,count(*) FROM information_schema.processlist GROUP BY state,host;
```



