# Log4j

## 定义

​		Log4j是Apache的一个开源项目，通过使用Log4j，我们可以控制日志信息输送的目的地是控制台、文件、GUI组件，甚至是套接口服务器、NT的事件记录器、UNIX Syslog守护进程等。

## 关键结构

### 整体关系

​		Log4j由三个重要的组成构成：日志记录器(Loggers)，输出端(Appenders)和日志格式化器(Layout)。

* 1.Logger：控制要启用或禁用哪些日志记录语句，并对日志信息进行级别限制



* 2.Appenders : 指定了日志将打印到控制台还是文件中



* 3.Layout : 控制日志信息的显示格式



​		使用时结构：Logger-->Appender-->Layout     //指向相当于引用

```java
			Logger logger=Logger.getLogger(MybatisTest.class);

        
      ConsoleAppender consoleAppender=new ConsoleAppender(new SimpleLayout(),ConsoleAppender.SYSTEM_OUT);

       
        logger.addAppender(consoleAppender);
```

注意：

​		一个logger可以有多个appender，每个appender表示一个目的地，多个appender表示多个目的地。但一个appender只有一个layout表示输出格式。有些appender还需要设置具体地址，比如FileAppender。

​		appender可以通过Threshold设置日志级别，从而对logger传送过来的日志信息再过滤。当然logger本身也可以设置level过滤日志。



整体关系见下图：

![Log4j_Constructure](http://typora-imges.oss-cn-beijing.aliyuncs.com/img/Log4j_Constructure.png)

​		



### Logger

Logger被指定为实体，获取方法：

​		Logger.getLogger(Class clazz)

这里的clazz可以任意指定一个class，但最好是当前的class，日志输出时可以输出该clazz信息，从而容易定位问题出现在那个类



### Appender

​		Appender用来指定日志信息输出到哪个地方，可以同时指定多个输出目的地。Log4j允许将信息输出到许多不同的输出设备中，一个log信息输出目的地就叫做一个Appender。

​		每个Logger都可以拥有一个或多个Appender，每个Appender表示一个日志的输出目的地。可以使用	Logger.addAppender(Appender app)为Logger增加一个Appender，也可以使用Logger.removeAppender(Appender app)为Logger删除一个Appender。



以下为Log4j几种常用的输出目的地：

​		a：org.apache.log4j.ConsoleAppender：将日志信息输出到控制台。
​		b：org.apache.log4j.FileAppender：将日志信息输出到一个文件。
​		c：org.apache.log4j.DailyRollingFileAppender：将日志信息输出到一个日志文件，并且每天输出到一个新的日志文件。
​		d：org.apache.log4j.RollingFileAppender：将日志信息输出到一个日志文件，并且指定文件的尺寸，当文件大小达到指定尺寸时，会自动把文件改名，同时产生一个新的文件。
​		e：org.apache.log4j.WriteAppender：将日志信息以流格式发送到任意指定地方。
​		f：org.apache.log4j.jdbc.JDBCAppender：通过JDBC把日志信息输出到数据库中。



### Layout

用于控制日志输出格式，有如下四种：

* 1.HTMLLayout : 格式化日志输出为HTML表格形式
* 2.SimpleLayout : 以一种非常简单的方式格式化日志输出，它打印三项内容：级别-信息
* 3.PatternLayout : 根据指定的转换模式格式化日志输出，或者如果没有指定任何转换模式，就使用默认的转化模式格式。
* 4.TTCCLayout : 包含日志产生的时间、线程、类别等等信息



### 日志级别

每个Logger都被了一个日志级别（log level），用来控制日志信息的输出。日志级别从高到低分为：

A：off         最高等级，用于关闭所有日志记录。
B：fatal       指出每个严重的错误事件将会导致应用程序的退出。
C：error      指出虽然发生错误事件，但仍然不影响系统的继续运行。
D：warn     表明会出现潜在的错误情形。
E：info         一般和在粗粒度级别上，强调应用程序的运行全程。
F：debug     一般用于细粒度级别上，对调试应用程序非常有帮助。
G：all           最低等级，用于打开所有日志记录。



上面这些级别是定义在org.apache.log4j.Level类中,Log4j只建议使用4个级别，优先级从高到低分别是:

​	C.error

​	D.warn

​	E.info

​	F.debug



​		通过使用日志级别，可以控制应用程序中相应级别日志信息的输出。例如，如果使用了info级别，则应用程序中所有低于info级别的日志信息(如debug)将不会被打印出来。



### 配置

​		配置Log4j环境就是指配置root Logger，包括把Logger为哪个级别，为它增加哪些Appender,以及为这些Appender设置Layout，等等。因为所有其他的Logger都是root Logger的后代，所以它们都继承了root Logger的性质。有以下几种方式来配置Log4j：

​		A：配置放在文件里，通过环境变量传递文件名等信息，利用Log4j默认的初始化过程解析并配置。
​		B：配置放在文件里，通过应用服务器配置传递文件甸等信息，利用一个特定的Servlet来完成配置。
​		C：在程序中调用BasicConfigor.configure()方法。
​		D：配置放在文件里，通过命令行PropertyConfigurator.configure(args[])解析log4j.properties文件并配置Log4j。



下面对BasicConfigurator.configure()方法和PropertyConfigurator.config()方法分别进行介绍。

**这两个方法配置后，配置信息都是保存在类上面，因此之后无论在那里使用logger，都按照这个配置来，不需要创建一个logger重新配置一次。**



#### BasicConfigurator

BasicConfigurator.configure()方法：

​		1：用默认的方式创建PatternLayout对象p:
 			 PatternLayout p = new PatternLayout("%-4r[%t]%-5p%c%x-%m%n");

​		2：用p创建ConsoleAppender对象a，目标是System.out，标准输出设备：
 			ConsoleAppender a = new CpnsoleAppender(p,ConsoleAppender.SYSTEM_OUT);

​		3：为root Logger增加一个ConsoleAppender p;
 			rootLogger.addAppender(p);

​		4：把rootLogger的log level设置为DUBUG级别;
​			 rootLogger.setLevel(Level.DEBUG);



#### PropertyConfigurator

PropertyConfigurator.configure()方法:

​		生成logger时，如果没有调用BasicConfigurator.configure()，PropertyConfigurator.configure()或DOMConfigurator.configure()方法，Log4j会自动加载CLASSPATH下名为log4j.properties的配置文件。如果把此配置文件改为其他名字，例如my.properties,程序虽然仍能运行，但会报出不能正确初始化Log4j系统的提示。这时可以在程序中加上：

```java
PropertyConfigurator.configure("classes/my.properties");
```

注意：如果提示my.properties找不到可以改为绝对路径，如：

```java
PropertyConfigurator.configure("/Users/shiwen/Documents/shiwenCode/myBatisLearn/target/classes/my.properties");
```





配置文件：

虽然可以不用配置文件，而在程序中实现配置，但这种方法在如今的系统开发中显然是不可取的，能采用配置文件的地方一定一定要用配置文件。Log4j支持两 种格式的配置文件：XML格式和Java的property格式。

```properties
################################################################################ 
#①配置根Logger，其语法为： 
# 
#log4j.rootLogger = [level],appenderName,appenderName2,... 
#level是日志记录的优先级，分为OFF,TRACE,DEBUG,INFO,WARN,ERROR,FATAL,ALL 
##Log4j建议只使用四个级别，优先级从低到高分别是DEBUG,INFO,WARN,ERROR 
#通过在这里定义的级别，您可以控制到应用程序中相应级别的日志信息的开关 
#比如在这里定义了INFO级别，则应用程序中所有DEBUG级别的日志信息将不被打印出来 
#appenderName就是指定日志信息输出到哪个地方。可同时指定多个输出目的 
################################################################################ 
################################################################################ 
#②配置日志信息输出目的地Appender，其语法为： 
# 
#log4j.appender.appenderName = fully.qualified.name.of.appender.class 
#log4j.appender.appenderName.optionN = valueN 
# 
#Log4j提供的appender有以下几种： 
#1)org.apache.log4j.ConsoleAppender(输出到控制台) 
#2)org.apache.log4j.FileAppender(输出到文件) 
#3)org.apache.log4j.DailyRollingFileAppender(每天产生一个日志文件) 
#4)org.apache.log4j.RollingFileAppender(文件大小到达指定尺寸的时候产生一个新的文件) 
#5)org.apache.log4j.WriterAppender(将日志信息以流格式发送到任意指定的地方) 
# 
#1)ConsoleAppender选项属性 
# -Threshold = DEBUG:指定日志消息的输出最低层次 
# -ImmediateFlush = TRUE:默认值是true,所有的消息都会被立即输出 
# -Target = System.err:默认值System.out,输出到控制台(err为红色,out为黑色) 
# 
#2)FileAppender选项属性 
# -Threshold = INFO:指定日志消息的输出最低层次 
# -ImmediateFlush = TRUE:默认值是true,所有的消息都会被立即输出 
# -File = C:\log4j.log:指定消息输出到C:\log4j.log文件 
# -Append = FALSE:默认值true,将消息追加到指定文件中，false指将消息覆盖指定的文件内容 
# -Encoding = UTF-8:可以指定文件编码格式 
# 
#3)DailyRollingFileAppender选项属性 
# -Threshold = WARN:指定日志消息的输出最低层次 
# -ImmediateFlush = TRUE:默认值是true,所有的消息都会被立即输出 
# -File = C:\log4j.log:指定消息输出到C:\log4j.log文件 
# -Append = FALSE:默认值true,将消息追加到指定文件中，false指将消息覆盖指定的文件内容 
# -DatePattern='.'yyyy-ww:每周滚动一次文件,即每周产生一个新的文件。还可以按用以下参数: 
#              '.'yyyy-MM:每月 
#              '.'yyyy-ww:每周 
#              '.'yyyy-MM-dd:每天 
#              '.'yyyy-MM-dd-a:每天两次 
#              '.'yyyy-MM-dd-HH:每小时 
#              '.'yyyy-MM-dd-HH-mm:每分钟 
# -Encoding = UTF-8:可以指定文件编码格式 
# 
#4)RollingFileAppender选项属性 
# -Threshold = ERROR:指定日志消息的输出最低层次 
# -ImmediateFlush = TRUE:默认值是true,所有的消息都会被立即输出 
# -File = C:/log4j.log:指定消息输出到C:/log4j.log文件 
# -Append = FALSE:默认值true,将消息追加到指定文件中，false指将消息覆盖指定的文件内容 
# -MaxFileSize = 100KB:后缀可以是KB,MB,GB.在日志文件到达该大小时,将会自动滚动.如:log4j.log.1 
# -MaxBackupIndex = 2:指定可以产生的滚动文件的最大数 
# -Encoding = UTF-8:可以指定文件编码格式 
################################################################################ 
################################################################################ 
#③配置日志信息的格式(布局)，其语法为： 
# 
#log4j.appender.appenderName.layout = fully.qualified.name.of.layout.class 
#log4j.appender.appenderName.layout.optionN = valueN 
# 
#Log4j提供的layout有以下几种： 
#5)org.apache.log4j.HTMLLayout(以HTML表格形式布局) 
#6)org.apache.log4j.PatternLayout(可以灵活地指定布局模式) 
#7)org.apache.log4j.SimpleLayout(包含日志信息的级别和信息字符串) 
#8)org.apache.log4j.TTCCLayout(包含日志产生的时间、线程、类别等等信息) 
#9)org.apache.log4j.xml.XMLLayout(以XML形式布局) 
# 
#5)HTMLLayout选项属性 
# -LocationInfo = TRUE:默认值false,输出java文件名称和行号 
# -Title=Struts Log Message:默认值 Log4J Log Messages 
# 
#6)PatternLayout选项属性 
# -ConversionPattern = %m%n:格式化指定的消息(参数意思下面有) 
# 
#9)XMLLayout选项属性 
# -LocationInfo = TRUE:默认值false,输出java文件名称和行号 
# 
#Log4J采用类似C语言中的printf函数的打印格式格式化日志信息，打印参数如下： 
# %m 输出代码中指定的消息 
# %p 输出优先级，即DEBUG,INFO,WARN,ERROR,FATAL 
# %r 输出自应用启动到输出该log信息耗费的毫秒数 
# %c 输出所属的类目,通常就是所在类的全名 
# %t 输出产生该日志事件的线程名 
# %n 输出一个回车换行符，Windows平台为“\r\n”，Unix平台为“\n” 
# %d 输出日志时间点的日期或时间，默认格式为ISO8601，也可以在其后指定格式 
#    如：%d{yyyy年MM月dd日 HH:mm:ss,SSS}，输出类似：2012年01月05日 22:10:28,921 
# %l 输出日志事件的发生位置，包括类目名、发生的线程，以及在代码中的行数 
#    如：Testlog.main(TestLog.java:10) 
# %F 输出日志消息产生时所在的文件名称 
# %L 输出代码中的行号 
# %x 输出和当前线程相关联的NDC(嵌套诊断环境),像java servlets多客户多线程的应用中 
# %% 输出一个"%"字符 
# 
# 可以在%与模式字符之间加上修饰符来控制其最小宽度、最大宽度、和文本的对齐方式。如： 
#  %5c: 输出category名称，最小宽度是5，category<5，默认的情况下右对齐 
#  %-5c:输出category名称，最小宽度是5，category<5，"-"号指定左对齐,会有空格 
#  %.5c:输出category名称，最大宽度是5，category>5，就会将左边多出的字符截掉，<5不会有空格 
#  %20.30c:category名称<20补空格，并且右对齐，>30字符，就从左边交远销出的字符截掉 
################################################################################ 
################################################################################ 
#④指定特定包的输出特定的级别 
#log4j.logger.org.springframework=DEBUG 
################################################################################ 
 
#OFF,systemOut,logFile,logDailyFile,logRollingFile,logMail,logDB,ALL 
log4j.rootLogger =ALL,systemOut,logFile,logDailyFile,logRollingFile,logMail,logDB 
 
#输出到控制台 
log4j.appender.systemOut = org.apache.log4j.ConsoleAppender 
log4j.appender.systemOut.layout = org.apache.log4j.PatternLayout 
log4j.appender.systemOut.layout.ConversionPattern = [%-5p][%-22d{yyyy/MM/dd HH:mm:ssS}][%l]%n%m%n 
log4j.appender.systemOut.Threshold = DEBUG 
log4j.appender.systemOut.ImmediateFlush = TRUE 
log4j.appender.systemOut.Target = System.out 
 
#输出到文件 
log4j.appender.logFile = org.apache.log4j.FileAppender 
log4j.appender.logFile.layout = org.apache.log4j.PatternLayout 
log4j.appender.logFile.layout.ConversionPattern = [%-5p][%-22d{yyyy/MM/dd HH:mm:ssS}][%l]%n%m%n 
log4j.appender.logFile.Threshold = DEBUG 
log4j.appender.logFile.ImmediateFlush = TRUE 
log4j.appender.logFile.Append = TRUE 
log4j.appender.logFile.File = ../Struts2/WebRoot/log/File/log4j_Struts.log 
log4j.appender.logFile.Encoding = UTF-8 
 
#按DatePattern输出到文件 
log4j.appender.logDailyFile = org.apache.log4j.DailyRollingFileAppender 
log4j.appender.logDailyFile.layout = org.apache.log4j.PatternLayout 
log4j.appender.logDailyFile.layout.ConversionPattern = [%-5p][%-22d{yyyy/MM/dd HH:mm:ssS}][%l]%n%m%n 
log4j.appender.logDailyFile.Threshold = DEBUG 
log4j.appender.logDailyFile.ImmediateFlush = TRUE 
log4j.appender.logDailyFile.Append = TRUE 
log4j.appender.logDailyFile.File = ../Struts2/WebRoot/log/DailyFile/log4j_Struts 
log4j.appender.logDailyFile.DatePattern = '.'yyyy-MM-dd-HH-mm'.log' 
log4j.appender.logDailyFile.Encoding = UTF-8 
 
#设定文件大小输出到文件 
log4j.appender.logRollingFile = org.apache.log4j.RollingFileAppender 
log4j.appender.logRollingFile.layout = org.apache.log4j.PatternLayout 
log4j.appender.logRollingFile.layout.ConversionPattern = [%-5p][%-22d{yyyy/MM/dd HH:mm:ssS}][%l]%n%m%n 
log4j.appender.logRollingFile.Threshold = DEBUG 
log4j.appender.logRollingFile.ImmediateFlush = TRUE 
log4j.appender.logRollingFile.Append = TRUE 
log4j.appender.logRollingFile.File = ../Struts2/WebRoot/log/RollingFile/log4j_Struts.log 
log4j.appender.logRollingFile.MaxFileSize = 1MB 
log4j.appender.logRollingFile.MaxBackupIndex = 10 
log4j.appender.logRollingFile.Encoding = UTF-8 
 
#用Email发送日志 
log4j.appender.logMail = org.apache.log4j.net.SMTPAppender 
log4j.appender.logMail.layout = org.apache.log4j.HTMLLayout 
log4j.appender.logMail.layout.LocationInfo = TRUE 
log4j.appender.logMail.layout.Title = Struts2 Mail LogFile 
log4j.appender.logMail.Threshold = DEBUG 
log4j.appender.logMail.SMTPDebug = FALSE 
log4j.appender.logMail.SMTPHost = SMTP.163.com 
log4j.appender.logMail.From = xly3000@163.com 
log4j.appender.logMail.To = xly3000@gmail.com 
#log4j.appender.logMail.Cc = xly3000@gmail.com 
#log4j.appender.logMail.Bcc = xly3000@gmail.com 
log4j.appender.logMail.SMTPUsername = xly3000 
log4j.appender.logMail.SMTPPassword = 1234567 
log4j.appender.logMail.Subject = Log4j Log Messages 
#log4j.appender.logMail.BufferSize = 1024 
#log4j.appender.logMail.SMTPAuth = TRUE 
 
#将日志登录到MySQL数据库 
log4j.appender.logDB = org.apache.log4j.jdbc.JDBCAppender 
log4j.appender.logDB.layout = org.apache.log4j.PatternLayout 
log4j.appender.logDB.Driver = com.mysql.jdbc.Driver 
log4j.appender.logDB.URL = jdbc:mysql://127.0.0.1:3306/xly 
log4j.appender.logDB.User = root 
log4j.appender.logDB.Password = 123456 
log4j.appender.logDB.Sql = INSERT INTOT_log4j(project_name,create_date,level,category,file_name,thread_name,line,all_category,message)values('Struts2','%d{yyyy-MM-ddHH:mm:ss}','%p','%c','%F','%t','%L','%l','%m')
```



## 用法

### 不用配置文件

```java
    public static void main(String[] args) throws IOException {
        //获取 logger，并设置过滤级别
        Logger logger=Logger.getLogger(MybatisTest.class);
        logger.setLevel(Level.WARN);

        //获取 appender（需要传入 Layer、目的地址 等信息）
        FileAppender fileAppender=new FileAppender(new SimpleLayout(),"/Users/shiwen/Documents/shiwenCode/myBatisLearn/log/out.txt",false);
        ConsoleAppender consoleAppender=new ConsoleAppender(new SimpleLayout(),ConsoleAppender.SYSTEM_OUT);

        //添加 appender 到 logger
        logger.addAppender(fileAppender);
        logger.addAppender(consoleAppender);


        //使用，输出日志信息
        logger.error("this is an error message");
        logger.warn("this is a warn message");
        logger.info("this is an info message");
        logger.debug("this is a degub message");

        System.out.println("hello world");
    }
```



### 使用配置文件

步骤1:创建配置文件。例如在CLASSPATH下创建名为 log4j.properties 的配置文件，log4j会自动加载配置文件。否则需要手动加载配置文件（/关键结构/配置/PropertyConfigurator）。



步骤2:java里直接使用即可，因为log4j通过配置文件已经配置好了

```java
    public static void main(String[] args) throws IOException {
        //获取 logger，并设置过滤级别
        Logger logger=Logger.getLogger(MybatisTest.class);

        //使用，输出日志信息
        logger.error("this is an error message");
        logger.warn("this is a warn message");
        logger.info("this is an info message");
        logger.debug("this is a degub message");

        System.out.println("hello world");
    }
```



## Mybatis的日志

Mybatis有自己的日志组件，但与Log4j类似，需要在classpath 下创建 log4j.properties 文件，用于加载配置信息。否则出错。



# Mybatis

## 定义

​		MyBatis 本是apache的一个开源项目iBatis, 2010年这个项目由apache software foundation 迁移到了google code，并且改名为MyBatis 。

​		2013年11月迁移到Github。MyBatis是一个优秀的持久层框架，它对jdbc的操作数据库的过程进行封装，使开发者只需要关注 SQL 本身，而不需要花费精力去处理例如注册驱动、创建connection、创建statement、手动设置参数、结果集检索等jdbc繁杂的过程代码。Mybatis通过xml或注解的方式将要执行的各种statement（statement、preparedStatemnt、CallableStatement）配置起来，并通过java对象和statement中的sql进行映射生成最终执行的sql语句，最后由mybatis框架执行sql并将结果映射成java对象并返回。



## 整体结构

![Mybatis_Constructure](http://typora-imges.oss-cn-beijing.aliyuncs.com/img/Mybatis_Constructure.png)



## 原始使用

### 1、加载相应的jar包

```xml
<packaging>jar</packaging>
<dependencies>
    <dependency>
        <groupId>org.mybatis</groupId>
        <artifactId>mybatis</artifactId>
        <version>3.4.5</version>
    </dependency>
    <dependency>
        <groupId>mysql</groupId>
        <artifactId>mysql-connector-java</artifactId>
        <version>5.1.6</version>
    </dependency>
</dependencies>
```

这里关键需要两个jar包，`mybits` 和 `mysql-connector-java`，因为 mybatis 需要连接数据库的驱动。



### 2、创建实体类

```java
package com.ben.domain;

import lombok.Data;

import java.io.Serializable;
import java.util.Date;

@Data
public class User implements Serializable {
    private int id;
    private String user_name;// 用户姓名
    private String sex;// 性别
    private Date birthday;// 生日
    private String address;// 地址
}
```

注意：这里的 getter、setter 都通过 lombock 自动生成，见 @Data 注解



### 3、创建配置文件（关键）

#### 主配置文件

​		名字随意，用于如 SqlMapConfig.xml 。用于配置相应的环境

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE configuration
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">

<configuration>
    <!--   配置环境-->
    <environments default="mysql">
        <!--配置mysql的环境-->
        <environment id="mysql">
            <!--配置事务的类型-->
            <transactionManager type="JDBC"></transactionManager>
            <!--配置连接池-->
            <dataSource type="POOLED">
                <!--配置连接数据库的4个基本信息-->
                <property name="driver" value="com.mysql.jdbc.Driver"/>
                <property name="url" value="jdbc:mysql://localhost:3306/mybatis"/>
                <property name="username" value="root"/>
                <property name="password" value="199402160579shch"/>
            </dataSource>
        </environment>
    </environments>

    <!--指定映射配置文件的位置，映射配置文件指的是每个dao独立的配置文件-->
    <mappers>
        <mapper resource="sqlMap/User.xml"/>
    </mappers>
</configuration>
```



#### 从配置文件

​		用于配置主配置文件的mapper标签。最终可以通过相应的定位拿到相应的sql语句来执行，不需要再手动写sql语句

一个住配置文件可以有多个从配置文件

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">

<!-- namespace：命名空间，用于隔离sql，还有一个很重要的作用，后面会讲 -->
<mapper namespace="test">
    <!-- 通过Id查询一个用户   -->
    <select id="findUserById" parameterType="Integer" resultType="com.ben.domain.User">
        select * from user where id = #{v}
    </select>

    <!-- 通过用户名查询用户   -->
    <select id="findUserByUsername" parameterType="String" resultType="com.ben.domain.User">
        select * from user where user_name like #{username}
    </select>
  
    <!--  添加用户，这里没有resultType，默认放回int（影响行数）  -->
    <insert id="insertUser" parameterType="com.ben.domain.User" >
        <!--用于返回数据到传入的entity的属性，不建议使用，影响因素太多，结果很不确定-->
        <selectKey keyProperty="id" resultType="Integer" order="AFTER">
            select LAST_INSERT_ID()
        </selectKey>
        insert into User (id,user_name,birthday,address,sex) values(#{id},#{user_name},#{birthday},#{address},#{sex})
    </insert>

</mapper>
```



#### 注意

*  其实可以用一个主配置文件就好，但这样 mapper 标签必须具体写，造成主配置文件内容过多。通过从配置文件配置mapper使得主配置文件更加简洁清晰，主配置文件的mapper标签只需要引用从配置即可。

​		一般来说，一个从配置文件写一个dao的数据交互sql。

* 包装为普通类的bean时，如果没有指明resultMap，那查询结果只能把那些在bean中含有同名字段的列的数据保存在bean，不同名的不会保存。如果指明resultMap，那查询结果到bean就可以根据resultMap的匹配来保存。

* 包装为基本类型时，每行结果只会把第一个数据放到基本类型里面去，如果类型匹配或者转换成功，就返回，否则抛出异常。

* 解决列与bean属性不同命的方法还可以通过 `select  xx as yy` 来实现。

* mapper.xml文件里元素的id不能带有`.` 号，否则运行时报错。

* 从配置文件的sql元素如果需要多个参数，可以在接口方法的参数上加上 `@param  ` 来给参数带上名字，好对应sql中占位符的名字填入参数。参数过多的话推荐放在bean里传过来，sql语句的占位符通过自身名称匹配bean中属性名获取bean中参数以替换自身。

  



### 4、增删改查

#### 查

```java
    public  void testSearchById() throws IOException {
        //1.读取配置文件
        InputStream in= Resources.getResourceAsStream("SqlMapConfig.xml");

        //2.创建SqlSessionFactory工厂
        SqlSessionFactory sqlSessionFactory=new SqlSessionFactoryBuilder().build(in);

        //3.使用工厂生产SqlSession对象
        SqlSession sqlSession=sqlSessionFactory.openSession();

        //4.执行Sql语句
        User user=sqlSession.selectOne("test.findUserById", 5);

        //5. 打印结果
        System.out.println(user);

        //6.释放资源
        sqlSession.close();
        in.close();
    }	


```

其中，第4步的 "test.findUserById" 就是为了定位到相应的sql语句，5是sql语句占位符中对应的值



#### 增

```java
public void testInsertUser() throws IOException {
        //1.读取配置文件
        InputStream in=Resources.getResourceAsStream("SqlMapConfig.xml");

        //2.创建SqlSessionFactory工厂
        SqlSessionFactory sqlSessionFactory=new SqlSessionFactoryBuilder().build(in);

        //3.使用工厂生产SqlSession对象
        SqlSession sqlSession=sqlSessionFactory.openSession();

        //4.执行Sql语句
        User user=new User(8,"gg1","Female",new Date(),"beijing");
        int insert = sqlSession.insert("user.insertUser",user);
        sqlSession.commit();

        //5. 打印结果
        System.out.println("insert: "+insert);

        //6.释放资源
        sqlSession.close();
        in.close();
    }
```



#### 删

```java
public void testDeleteUser() throws IOException {
        //1.读取配置文件
        InputStream in=Resources.getResourceAsStream("SqlMapConfig.xml");

        //2.创建SqlSessionFactory工厂
        SqlSessionFactory sqlSessionFactory=new SqlSessionFactoryBuilder().build(in);

        //3.使用工厂生产SqlSession对象
        SqlSession sqlSession=sqlSessionFactory.openSession();

        //4.执行Sql语句
        int delete = sqlSession.delete("user.deleteUserById", 4);
        sqlSession.commit();

        //5. 打印结果
        System.out.println("delete: "+delete);

        //6.释放资源
        sqlSession.close();
        in.close();
    }
```



#### 改

```java
public void testUpdateUser() throws IOException {
        //1.读取配置文件
        InputStream in=Resources.getResourceAsStream("SqlMapConfig.xml");

        //2.创建SqlSessionFactory工厂
        SqlSessionFactory sqlSessionFactory=new SqlSessionFactoryBuilder().build(in);

        //3.使用工厂生产SqlSession对象
        SqlSession sqlSession=sqlSessionFactory.openSession();

        //4.执行Sql语句
        User user=new User(8,"gg1","Female",new Date(),"beijing");
        int update = sqlSession.update("user.updateUserById", user);
        sqlSession.commit();

        //5. 打印结果
        System.out.println("update: "+update);

        //6.释放资源
        sqlSession.close();
        in.close();
    }
```



#### 注意

增删改在执行sql语句后需要执行commit方法提交事物，否则不会更改到数据库



### 日志

​		mybatis有自己的日志组建，需要在 classPath 下创建 log4j.properties 文件用于日志配置，配置方法与log4j 一样。

​		如果需要输入日志信息就需要创建这个配置文件，如果不需要输出日志，就不用管



## Mapper接口使用

### 为什么用mapper接口

​		上一节的使用，只是说了xml映射器的一种使用方法（通过sqlSession使用）。但这种方式是不推荐的，因为可能有类型问题，比如sql元素需要Integer的入参，但是实际传入的是String，这在编译时发现不了但运行时会抛出异常。其次其表达力也比较弱。

​		为了更好的进行数据库交互，尝尝通过一个接口mapper来执行sql元素。一个接口mapper对应一个sql元素池（由mapper.xml文件或者接口得信息来生成），接口mapper的方法对应sql元素池的一个sql元素。执行时调用接口方法，即可获取对应sql元素来执行然后返回结果了。



### 静态mapper接口与动态mapper接口

​		mapper分为静态mapper与动态mapper。但结构上是一样的，即mapper与sql元素池的对应关系不变，不同的地方就是静态mapper的sql写死了，只能传入参数替换占位符，而动态mapper是传入整个sql语句，灵活性更大。

* **构建静态mapper有两种方式**：

​			1、普通mapper接口 与 mapper.xml文件。加载xml即可，会自动匹配上接口的。

​			2、带有sql注解的mapper接口。加载接口即可，会生成对应sql元素池的。

* **构建动态mapper只有一种种方式**：

​			1、mapper接口。加载接口即可，会生成对应sql元素池的。

**注意**：

​		动态mapper除了自定义的mapper外，还可以用一些公共的mapper如CommonSelectMapper，与自定义mapper一样，需要在主配置文件引入才能注册到运行时的mybatis中。

​		这些公共mapper对应的sql元素池里的sql元素出参都是Map，也就是结果的每行元素是封装在Map里面，对应的mapper接口方法出参是Map或者List< Map>。



公共的mapper对数据的java封装是依据原数据库数据类型来的，如：			

| 数据库类型                                 | java类型           |
| ------------------------------------------ | ------------------ |
| char、varchar                              | String             |
| int                                        | Integer            |
| bigint                                     | Long               |
| decimal、大部分聚合计算如 avg(xx)，sum(xx) | BigDecimal         |
| long、聚合计算count(*)                     | Long               |
| date                                       | java.sql.Date      |
| timestamp、datetime                        | java.sql.Timestamp |
|                                            |                    |

sql里，date保存日期，如 2021-07-10。 datetime、timestamp保存日期时间，如 2021-07-10 12:42:53。





### mapper接口与sql元素池

​		<u>sql元素的输入是占位符的值，用于补全sql；sql元素的输出是封装了执行结果的bean列表。</u>

​		<u>接口mapper的方法是对sql元素的简单封装：</u>

​			<u>入参对应sql元素占位符</u>

​			<u>出参要么原样返回 List< Bean>，要么取出元素再返回Bean（这要求sql元素执行结果的List长度为1，否则报错，因为List长度不为1，说明执行结果有多行元素，不符合要求）。</u>



​       接口mapper方法的入参类型不需要和sql元素的入参类型对应（sql元素入参类型不会产生任何影响，mybatis会获取实参类型），只要实入参能匹配好占位符就行。

​		**出参关键点**：**接口mapper方法的出参类型是`Bean`或者`List< Bean> `  ，sql元素的出参类型是 `Bean`**

​	   因为mapper方法的出参表示最终返回结果，而sql元素的出参表示查询结果每一行的包装类型（sql语句执行后把每一行元素包装后放在List里面返回，再根据需要决定返回一个元素还是整个List）。这是mybatis指定sql语句执行的结果包装类型与返回类型的途径。



**静态sql入参注意：**

   sql元素的语句占位符只有一个，那么入参无论有无名字都可以，反正肯定放在这个占位符上。

   sql元素的语句占位符有多个时：

​		如果占位符没有名称（即名称是param1，param2之类），那么传过来的参数有无名称都可以，按占位符指定顺序替换，如param2表示第二个参数。

​		如果占位符有名称，那么传过来的要么是一个普通类的bean（属性名对应占位符名），要么是带有名称的实参（接口方法的参数使用`@Param`注解）。通过名称对应来替换占位符。



**动态sql：**

​		动态sql只有接口，是根据接口生成不完备的sql元素池的。执行时传入sql语句，结合sql元素来执行的，其sql元素不需要入参设定了（因为执行时会传入最终的sql语句），只需要关注其出参类型即可。sql元素出参与接口mapper出参的关系前面已经说了，sql元素是通过接口mapper生成的，mybatis会给我们做这个工作。



![MyBatis_Mapper](http://typora-imges.oss-cn-beijing.aliyuncs.com/img/MyBatis_Mapper.png)

详见：https://www.cnblogs.com/nuccch/p/9056482.html#%E9%85%8D%E7%BD%AExml%E6%98%A0%E5%B0%84%E5%99%A8



## Dynamic Sql（动态Mapper）

以下是使用dynamic sql的用法：

```java
package com.example.demo;

import com.example.demo.entity.Apple;
import com.example.demo.mapper.AppleMapper;
import org.apache.ibatis.session.SqlSession;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.mybatis.dynamic.sql.SqlBuilder;
import org.mybatis.dynamic.sql.SqlBuilder.*;
import org.mybatis.dynamic.sql.SqlBuilder.*;
import org.mybatis.dynamic.sql.delete.render.DeleteStatementProvider;
import org.mybatis.dynamic.sql.insert.render.InsertStatementProvider;
import org.mybatis.dynamic.sql.render.RenderingStrategies;
import org.mybatis.dynamic.sql.render.RenderingStrategy;
import org.mybatis.dynamic.sql.select.render.SelectStatementProvider;
import org.mybatis.dynamic.sql.update.render.UpdateStatementProvider;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.test.context.junit4.SpringRunner;

import java.math.BigDecimal;
import java.util.List;
import java.util.Optional;

import static com.example.demo.mapper.AppleDynamicSqlSupport.price;
import static com.example.demo.mapper.AppleDynamicSqlSupport.name;
import static com.example.demo.mapper.AppleDynamicSqlSupport.id;
import static com.example.demo.mapper.AppleDynamicSqlSupport.apple;
import static org.mybatis.dynamic.sql.SqlBuilder.*;

@RunWith(SpringRunner.class)
@SpringBootTest
public class MyBatisTest {

    @Autowired
    private SqlSession sqlSession;

    @Test
    public void insertTest(){
        Apple apple=new Apple();
        apple.setId(4);
        apple.setName("d");
        apple.setPrice(new BigDecimal(13.56));

        AppleMapper appleMapper = sqlSession.getMapper(AppleMapper.class);
        int insert = appleMapper.insert(apple);
        sqlSession.commit();
        System.out.println(insert);
    }

    @Test
    public void deleteTest(){
//        DeleteStatementProvider deleteStatementProvider=deleteFrom(apple)
//                .where(id,isGreaterThan(3)).build().render(RenderingStrategies.MYBATIS3);
//
//        AppleMapper appleMapper = sqlSession.getMapper(AppleMapper.class);
//        int delete = appleMapper.delete(deleteStatementProvider);
//        System.out.println("delete:"+delete);
//        sqlSession.commit();

        AppleMapper appleMapper = sqlSession.getMapper(AppleMapper.class);
        int delete=appleMapper.delete(c->c.where(id,isGreaterThan(3)));
        sqlSession.commit();
        System.out.println("delete:"+delete);
    }

    @Test
    public void updateTest(){
//        UpdateStatementProvider updateStatementProvider=update(apple)
//                .set(name).equalTo("hello")
//                .set(price).equalTo(new BigDecimal(16.66))
//                .where(id,isEqualTo(4))
//                .build().render(RenderingStrategies.MYBATIS3);
//        AppleMapper appleMapper = sqlSession.getMapper(AppleMapper.class);
//        int update = appleMapper.update(updateStatementProvider);
//        sqlSession.commit();
//        System.out.println(update);


        AppleMapper appleMapper = sqlSession.getMapper(AppleMapper.class);
        int update = appleMapper.update(c -> c.set(name).equalTo("world").set(price).equalTo(new BigDecimal(18.88)).where(id, isEqualTo(4)));
        sqlSession.commit();
        System.out.println(update);
    }

    @Test
    public void selectOneTest(){
//        SelectStatementProvider selectStatementProvider=select(id,name,price)
//                .from(apple)
//                .where(id,isEqualTo(4))
//                .build().render(RenderingStrategies.MYBATIS3);
//        AppleMapper appleMapper = sqlSession.getMapper(AppleMapper.class);
//        Optional<Apple> apple1 = appleMapper.selectOne(selectStatementProvider);
//        apple1.ifPresent(c-> System.out.println(c));

        AppleMapper appleMapper = sqlSession.getMapper(AppleMapper.class);
        Optional<Apple> apple = appleMapper.selectOne(c -> c.where(id, isEqualTo(4)));
        apple.ifPresent(c-> System.out.println(c));
    }

    @Test
    public void selectManyTest(){
//        SelectStatementProvider selectStatementProvider=select(id,name,price)
//                .from(apple)
//                .where(id,isGreaterThan(0))
//                .build().render(RenderingStrategies.MYBATIS3);
//        AppleMapper appleMapper = sqlSession.getMapper(AppleMapper.class);
//        List<Apple> appleList = appleMapper.selectMany(selectStatementProvider);
//        for (Apple apple1 : appleList) {
//            System.out.println(apple1);
//        }

        AppleMapper appleMapper = sqlSession.getMapper(AppleMapper.class);
        List<Apple> appleList = appleMapper.select(c -> c.where(id, isGreaterThan(0)));
        appleList.forEach(c-> System.out.println(c));
    }

    @Test
    public void countTest(){
        //这里可以看出SqlBuilder可以构建新的列
        SelectStatementProvider selectStatementProvider=select(SqlBuilder.count(id))
                .from(apple)
                .where(id,isGreaterThan(2))
                .build().render(RenderingStrategy.MYBATIS3);
        AppleMapper appleMapper = sqlSession.getMapper(AppleMapper.class);
        long count = appleMapper.count(selectStatementProvider);
        System.out.println("count:"+count);

//        AppleMapper appleMapper = sqlSession.getMapper(AppleMapper.class);
//        long count = appleMapper.count(c -> c.where(id, isGreaterThan(2)));
//        System.out.println("count:"+count);
    }

}

```

​		由上面可以看出，用 `DSLCompleter`  比用 `StatementProvider`  更简洁方便，但如果涉及到复杂查询，还是需要`StatementProvider` 的，另外，SqlBuilder可以构建新的列，通常用于构建带有别名的列，或者聚合计算列。



## mybatis关键因素总结

1、数据库链接信息

2、mapper（接口mapper，和sql元素池）

3、模拟表格（如 AppleDynamicSqlSupport ，里面有表明，字段名等信息，用于构建dynamicSql）



通过一个xml配置文件把这些信息集中起来，构建sqlsession，mapper等



## generator

参考：https://juejin.cn/post/6844903982582743048

### 方式一：maven插件生成

注意：在pom.xml里面把 generator.xml文件绑定在生成器插件上（即在生成器上面指明generator.xml文件地址），再在generator.xml文件里面配置生成规则，最后运行插件生成即可

### 方式二：java生成



### 注意

1、generator里面的相对路径，是从当下模块出发的



## 结合springBoot

在springBoot中，引入如下依赖：

```xml
<dependency>
			<groupId>org.mybatis.spring.boot</groupId>
			<artifactId>mybatis-spring-boot-starter</artifactId>
			<version>1.3.2</version>
</dependency>
```

主要是为了通过AutoConfiguredMapperScannerRegistrar这个类来扫描mapper

​		这样不用手动生成 sqlsession 、mapper了。但需要我们在mapper接口上添加@Mapper注解。如果有需要配合mapper.xml的，在application.peoperties中，配置下面语句就可以了，否则mapper接口找不到匹配得mapper.xml文件。这种做法不用手动生成sqlsession，就没必要配主配置文件了

```properties
mybatis.mapper-locations=你的mapper.xml文件路径
```





## 注意

### Optional

​		版本 3.5.0以上才支持Optional对象，也就是查询返回Entity变成返回Optional< Entity>

关于optional的介绍看这里：https://juejin.cn/post/6844903960050925581





# Mockito

用于模拟对象，我们可以设置这个模拟对象的方法的输入，输出，当调用的时候就按照这个输入输出来。

## 手动创建mock对象

如：

```java
import static org.mockito.Mockito.*;

@Test
	public void when_thenReturn(){
		//mock一个Iterator类
		Iterator iterator = mock(Iterator.class);
		//预设当iterator调用next()时第一次返回hello，第n次都返回world
		when(iterator.next()).thenReturn("hello").thenReturn("world");
		//使用mock的对象
		String result = iterator.next() + " " + iterator.next() + " " + iterator.next();
		//验证结果
		assertEquals("hello world world",result);
	}
```

上面设置了两次返回结果，所以，第一次调用会返回hello，后面所有调用返回world



当用参数匹配器设置参数时，需要所用的参数都用参数匹配值设置，否则出错

```java
		import static org.mockito.Mockito.*;

		@Test
    public void when_thenThrow1() throws IOException {
        //anfInt()来自参数匹配值，1也来自参数匹配器
        when(mockCat.map(anyInt(),eq(1))).thenReturn("h");

        String map1=mockCat.map(1,1);
        String map2=mockCat.map(2,1);
        String map3=mockCat.map(3,1);

        System.out.println(map1);
        System.out.println(map2);
        System.out.println(map3);
				//验证调用次数
        verify(mockCat,times(1)).map(1,1);
    }
```



## 注解创建mock对象

```java
import static org.mockito.Mockito.*;

@ExtendWith(MockitoExtension.class)   //junit5用这个，创建mock对象
//@RunWith(MockitoJUnitRunner.class)  //junit4用这个，创建mock对象
public class NoticeServiceTest {

    @Mock
    private INoticeManager noticeManager;
    @Mock
    private MissionDomainService missionDomainService;
    @Mock
    private NoticeRepository noticeRepository;
    @InjectMocks
    private NoticeService noticeService;

    @Test
    public void testSendNotice() throws NoticeConfigurableInvalidException, NoticeSendUnexpectedException {
        UserMission userMission=getUserMission();
        NoticePackage noticePackage=getNoticePackage();
        BaseEvent event=new ProductEvent();
        event.setType(9308);

        Mockito.when(noticeRepository.getNoticePackageOfCampaign(anyLong())).thenReturn(noticePackage);
        noticeService.sendNotice(userMission,event);

        verify(noticeManager,times(1)).notifyUser(anyObject(),anyObject(),anyObject());
    }

    private UserMission getUserMission(){
        UserMission userMission=new UserMission();
        return userMission;
    }

    private NoticePackage getNoticePackage(){
        NoticePackage noticePackage=new NoticePackage();
        return noticePackage;
    }
}

```





# FastJson

具体可以参考：https://blog.csdn.net/xuforeverlove/article/details/80842148

json字符串可以转换成对象（JSONObject）或者数组（JSONArray）：

​       转成对象： JSON.parseObject(str)  

​       转成数组：JSON.parseArray(str)

​	   如果要转成特定对象，如User：JSON.parseObject(str, User.class);

​		如果要转成特定链表，如List< User>： JSON.parseObject(str, new TypeReference<List< User>>() {});



# thrift

## 核心原理

​		thrift是跨语言的，就是不同的语言之间可以进行远程调用，比如java客户端调用c服务器。但要实现这种跨语言的调用，必须定义一种中间语言。让二者的沟通是通过这种中间语言来通信的。

​		我们定义thrift接口的时候是用thrift语言定义的。接口的实现有客户端实现和服务器实现两种，服务器实现为业务处理，客户端实现为远程调用。**这种实现逻辑上是用thrift语言实现，实际上根据不同语言有不同的实现，但逻辑功能都与thrift语言实现等效而已。**

​		一般来说客户端实现的逻辑是固定的（即远程调用），而服务器实现的逻辑是可以随业务不同而不同的。因此，一般来说，thrift接口生成相应语言的时候，客户端的实现已经自动生成了，服务器的实现则还需要我们编写。



客户端：远程调用

服务器：处理调用信息、调用真正的实现逻辑、返回结果

客户端与服务器的连接是通过网络连接的，需要ip和端口号建立连接



总体如下图所示：

![ThriftPrinciple](http://typora-imges.oss-cn-beijing.aliyuncs.com/img/ThriftPrinciple.png)



## 注意：

* 加载的依赖版本需要和生成代码的thrift版本一致，否则会出问题，如@Override报错：https://blog.csdn.net/antony1776/article/details/78920888
* @Override报错的另一个原因就是，模块使用的java语言版本不对，参考：https://blog.csdn.net/u013985664/article/details/79170886



## 使用步骤

0、引入依赖（注意依赖版本需要和生成代码的thrift版本一致）

```xml
<dependency>
    <groupId>org.apache.thrift</groupId>
    <artifactId>libthrift</artifactId>
    <version>0.9.3</version>
</dependency>
```

1、编写thrift文件

2、把thrift文件生成实际语言的代码，命令：thrift -gen java Hello.thrift

3、编写接口实现类

4、编写服务端、客户端代码，并运行即可



## 代码结构、服务器、客户端

![Thrift](http://typora-imges.oss-cn-beijing.aliyuncs.com/img/Thrift.png)







