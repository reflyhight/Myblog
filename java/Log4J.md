title: Log4J简单应用
tags: 
	- logs
	- Java

categories:
	- Java
---------------------




log4j.properties 内容：

	log4j.rootLogger=DEBUG,stdout,
	log4j.logger.access =INFO,stdout,access
	
	### stdout ###
	log4j.appender.stdout=org.apache.log4j.ConsoleAppender
	log4j.appender.stdout.Target=System.out
	log4j.appender.stdout.layout=org.apache.log4j.PatternLayout
	log4j.appender.stdout.layout.ConversionPattern= %d{yyyy -MM-dd HH:mm:ss} %c{1} [%p] %m%n
	
	### access ###
	log4j.appender.access=org.apache.log4j.DailyRollingFileAppender
	log4j.appender.access.File=/opt/data/access.log
	log4j.appender.access.Append=true
	log4j.appender.access.DatePattern='.'yyyy-MM-dd
	log4j.appender.access.layout=org.apache.log4j.PatternLayout
	log4j.appender.access.layout.ConversionPattern= %m%n

log4j.properties 需要放在项目的classpath下面，一般放在名为resources 的source folder中



log4j中打印日志的基本单位是Logger
log4j.rootLogger 是最基础的Logger，是所有Logger都需要同时打印的。
定义Logger的方法是log4j.logger.logname=inforstandard,appendername ....
其中logname是定义logger的名字，inforstandard 是该Logger的日志级别，appendername 是该日志使用的appender

日志级别包括： TRACE,DEBUG,INFO,WARN,ERROR,FATAL。一个logger只有一个日志级别，可以有多个
TRACE,
DEBUG,
INFO,
WARN,
ERROR and
FATAL

appender 是输出时的基本工具，你可以把它看作是System.out.print()。 appender可以有多个

接下来的appender 是一个类。这个类都实现了 org.apache.log4j.Appender 接口
log4j中的配置文件 log4j.appender.stdout.layout配置的值，其实是将该值通过Appender中的setLayout()方法注入的





配置好log4J的配置文件后，就可以在JAVA代码中使用了。


public static void main(String[] args) {
            Logger loger= Logger.getLogger( "access");
             loger.info("I am a access log4j" );
            Logger logger= Logger.getLogger(Flume4Log4J. class);
             loger.info("ddd" );
      }


因为自定义logger “access ” 中定义了两个appender,一个使用控制台输出了，一个输出到文件了。
【注】 windows下面 /opt/data/access.log  会输出到项目所在盘的 /opt/data/access.log下。
java项目放在e盘的话，就会输出到E:\opt\data\access.log

2016-05-21 17:20:55 access [INFO] I am a access log4j
2016-05-21 17:20:55 access [INFO] ddd


用JAVA代码也可以实现配置文件相同功能。

```java
public static void main(String[] args) throws IOException {

	Logger logger = Logger.getLogger("hhh");
	logger.setLevel(Level.DEBUG);

	org.apache.log4j.ConsoleAppender appender = 
			new org.apache.log4j.ConsoleAppender();
	appender.setName("sta");
	appender.setTarget("System.out");
	appender.setWriter(new FileWriter(new File("e:/test.txt")));

	org.apache.log4j.PatternLayout layout = new PatternLayout();
	layout.setConversionPattern("%d{yyyy-MM-dd HH:mm:ss} %c{1} [%p] %m%n");
	appender.setLayout(layout);

	logger.addAppender(appender);

	logger.info("log4j of JAVA Code");

	Logger loggerRoot = Logger.getRootLogger();
	logger.warn("ddd");

	}

```

>需要引入的包

```java
	import java.io.File;
	import java.io.FileWriter;
	import java.io.IOException;
	
	import org.apache.log4j.Level;
	import org.apache.log4j.Logger;
	import org.apache.log4j.PatternLayout;

```