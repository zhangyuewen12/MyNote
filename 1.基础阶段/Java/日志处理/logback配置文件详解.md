logback介绍和配置详解
logback是Java的开源框架，性能比log4j要好。是springboot自带的日志框架。该框架主要有3个模块：

logback-core：核心代码块（不介绍）

log back-classic：实现了slf4j的api，加入该依赖可以实现log4j的api。

log back-access：访问模块与servlet容器集成提供通过http来访问日志的功能（也就是说不需要访问服务器，直接在网页上就可以访问日志文件）。



## 引入依赖

```
<dependency>
    <groupId>ch.qos.logback</groupId>
    <artifactId>logback-core</artifactId>
</dependency>
<dependency>
    <groupId>ch.qos.logback</groupId>
    <artifactId>logback-classic</artifactId>
</dependency>
<!--<dependency>-->
<!--<groupId>ch.qos.logback</groupId>-->
<!--<artifactId>logback-access</artifactId>-->
<!--</dependency>-->

```

## logback.xml格式详解

### 1 根节点configuration包含的属性

> 根节点，包含下面三个属性：
>
> scan: 当此属性设置为true时，配置文件如果发生改变，将会被重新加载，默认值为true。
> 　　　　scanPeriod: 设置监测配置文件是否有修改的时间间隔，如果没有给出时间单位，默认单位是毫秒。当scan为true时，此属性生效。默认的时间间隔为1分钟。
> 　　　　debug: 当此属性设置为true时，将打印出logback内部日志信息，实时查看logback运行状态。默认值为false。

```
<configuration scan="true" scanPeriod="60 seconds" debug="false"> 

　　  <!--其他配置省略--> 

</configuration>

```

### 2 根节点configuration的子节点

#### 2.1 设置上下文名称

**contextName**

> 每个logger都关联到logger上下文，默认上下文名称为“default”。但可以使用设置成其他名字，用于区分不同应用程序的记录。一旦设置，不能修改。

```
<configuration scan="true" scanPeriod="60 seconds" debug="false">  
      <contextName>myAppName</contextName>  
      <!-- 其他配置省略-->  
</configuration>  
```

#### 2.2 设置变量

**property**

> 用来定义变量值的标签， 有两个属性，name和value；其中name的值是变量的名称，value的值时变量定义的值。通过定义的值会被插入到logger上下文中。定义变量后，可以使“${}”来使用变量。

```
<configuration scan="true" scanPeriod="60 seconds" debug="false">  
      <property name="APP_Name" value="myAppName" />   
      <contextName>${APP_Name}</contextName>  
      <!-- 其他配置省略-->  
</configuration>   

```





#### 2.4 子节点appender

> 子节点：负责写日志的组件，它有两个必要属性name和class。name指定appender名称，class指定appender的全限定名
>
> appender class 类型主要有三种：ConsoleAppender、FileAppender、RollingFileAppender
>
> ConsoleAppender
> 把日志输出到控制台，有以下子节点：
>
> ：对日志进行格式化。
>
> ：字符串System.out(默认)或者System.err（区别不多说了）
>
> FileAppender
> 把日志添加到文件，有以下子节点：
>
> ：被写入的文件名，可以是相对目录，也可以是绝对目录，如果上级目录不存在会自动创建，没有默认值。
>
> ：如果是 true，日志被追加到文件结尾，如果是 false，清空现存文件，默认是true。
>
> ：对记录事件进行格式化。
>
> ：如果是 true，日志会被安全的写入文件，即使其他的FileAppender也在向此文件做写入操作，效率低，默认是 false。
>
> RollingFileAppender
> 滚动记录文件，先将日志记录到指定文件，当符合某个条件时，将日志记录到其他文件。有以下子节点：
>
> ：被写入的文件名，可以是相对目录，也可以是绝对目录，如果上级目录不存在会自动创建，没有默认值。
>
> ：如果是 true，日志被追加到文件结尾，如果是 false，清空现存文件，默认是true。
>
> :当发生滚动时，决定RollingFileAppender的行为，涉及文件移动和重命名。属性class定义具体的滚动策略类
>



#### 2.5 设置logger



<logger>



> 用来设置某一个包或者具体的某一个类的日志打印级别、以及指定。仅有一个name属性，一个可选的level和一个可选的addtivity属性。
>
> name:
>
> 用来指定受此loger约束的某一个包或者具体的某一个类。
>
> level:
>
> 用来设置打印级别，大小写无关：TRACE, DEBUG, INFO, WARN, ERROR, ALL 和 OFF，还有一个特俗值INHERITED或者同义词NULL，代表强制执行上级的级别。
>
> 如果未设置此属性，那么当前loger将会继承上级的级别。
>
> addtivity:
>
> 是否向上级loger传递打印信息。默认是true。
>
> 可以包含零个或多个元素，标识这个appender将会添加到这个loger。



<root>  （特殊的 <logger>）

> 也是<logger>元素，但是它是根logger。只有一个level属性，因为已经被命名为"root".
>
> level:
>
> 用来设置打印级别，大小写无关：TRACE, DEBUG, INFO, WARN, ERROR, ALL 和 OFF，不能设置为INHERITED或者同义词NULL。
>
> 默认是DEBUG。
>
> <root>可以包含零个或多个<appender-ref>元素，标识这个appender将会添加到这个logger。



样例

```xml
<?xml version="1.0" encoding="UTF-8"?>
<configuration>
	<!--用于区分不同应用程序的记录-->
	<contextName>jtcn-wx-service</contextName>
	<!--定义日志文件的存储地址 勿在 LogBack 的配置中使用相对路径 -->
	<property name="LOG_HOME" value="../logs" />
	<!--定义日志文件名称，一般一个项目对应一个名-->
	<property name="LOG_FILE_NAME" value="jtcn-wx" />

	<!-- 控制台输出 -->
	<appender name="console" class="ch.qos.logback.core.ConsoleAppender">
		<encoder class="ch.qos.logback.classic.encoder.PatternLayoutEncoder">
			<!--格式化输出：%d表示日期，%thread表示线程名，%-5level：级别从左显示5个字符宽度%msg：日志消息，%n是换行符 -->
			<!-- <pattern>%d{yyyy-MM-dd HH:mm:ss.SSS} [%thread] %-5level %logger{50} -%msg%n</pattern> -->
			<pattern>%d{yyyy-MM-dd HH:mm:ss.SSS} %-5level %logger{50} %line - %msg%n</pattern>
		</encoder>
	</appender>

	<!-- 按照每天生成日志文件 -->
	<appender name="file"
		class="ch.qos.logback.core.rolling.RollingFileAppender">
		<rollingPolicy class="ch.qos.logback.core.rolling.SizeAndTimeBasedRollingPolicy">
			<!--日志文件输出的文件名 -->
			<FileNamePattern>${LOG_HOME}/${LOG_FILE_NAME}.%d{yyyy-MM-dd}.%i.log</FileNamePattern>
			<!--日志文件保留天数 -->
			<MaxHistory>20</MaxHistory>
			<MaxFileSize>25MB</MaxFileSize>
			<totalSizeCap>5000MB</totalSizeCap> 
		</rollingPolicy>
		<encoder class="ch.qos.logback.classic.encoder.PatternLayoutEncoder">
			<!--格式化输出：%d表示日期，%thread表示线程名，%-5level：级别从左显示5个字符宽度%msg：日志消息，%n是换行符 -->
			<!-- <pattern>%d{yyyy-MM-dd HH:mm:ss.SSS} [%thread] %-5level %logger{50} - %msg%n</pattern> -->
			<pattern>%d{yyyy-MM-dd HH:mm:ss.SSS} %-5level %logger{50} %line - %msg%n</pattern>
		</encoder>
	</appender>

	<!-- 配置日志平台，将日志收集到flylog平台中-->
	<appender name="flylog" class="com.iflytek.fsp.flylog.sdk.java.plugin.logback.FlylogAppender">
	</appender>
	<!-- 修改为自己项目的包名 -->
	<logger name="com.iflytek.jtcn" level="INFO" additivity="false">
		<appender-ref ref="flylog"/>
	</logger>
	<!-- 配置日志平台，将日志收集到flylog平台中-->

	<logger name="com.iflytek.jtcn" level="debug" >
		<appender-ref ref="console" />
		<appender-ref ref="file" />
	</logger>

	<!-- 日志输出级别 -->
	<root level="INFO">
		<appender-ref ref="console" />
		<appender-ref ref="file" />
	</root>
</configuration> 

```

