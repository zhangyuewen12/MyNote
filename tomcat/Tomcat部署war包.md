1、xftp上传Tomcat

2、解压Tomcat：  tar -zvxf 文件                                unzip 文件  等解压命令

3、将war包放在Tomcat中的webapps目录下（如果webapps中有任何其他war包或解压后的文件都需要删除）

4、修改以上穿过去的文件的拥有者（非root用户下时）

5、查看需要使用的端口的占用情况（root用户下使用此命令，监控状态为LISTEN表示已经被占用）：netstat -anp | grep 8080

6、端口：进入tomcat目录下的conf文件夹，修改server.xml参数（此时我们需要使用8000端口）             将<Connector port=”8080″ protocol=”…” …>             改为<Connector port=”8000″ protocol=”…” …>

7、启动：进入Tomcat的bin目录，执行 sh startup.sh （看是否需要给catalina.sh赋予执行权限：chmod -R 764 catalina.sh）

8、日志：进入Tomcat的logs目录，执行 tail -300f catalina.out

9、停止：进入Tomcat的bin目录，执行 sh shutdown.sh

10、测试： http://localhost:8000/服务名



