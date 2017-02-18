---
title: Java中的日志(一)：log4j
date: 2017-02-12 16:47:34
tags: Java
---
log4j是一个基于java的日志工具。它由牛人Ceki Gülcü在2001年发布，至今已经有了16年的历史，现在是Apache基金会的一个项目。经过这么多年的发展，在java生态圈中已经成了日志事实上的标准，很多早期的著名的应用都是依赖log4j来完成其日志记录的。查了一些资料，log4j始于1996年初的E.U. SEMPER （安全电子市场为欧洲）跟踪API的项目，经过了无数的改进最终发布成了现在的log4j。然而，任何事物都有英雄迟暮，美人白头的那一天，虽然第一代log4j现在可能已经不是记录日志的首选工具，但是看看后来的log4c,log4cpp,log4net等日志工具的命名，就应该能够感受到log4j如日中天时在业界的江湖地位。

###log4j配置的3个重要组件(以properties为例)
####日志信息的优先级
在log4j中日志的级别分为OFF、FATAL、ERROR、WARN、INFO、DEBUG、ALL，我们常用的主要是ERROR、WARN、INFO、DEBUG这四个级别。优先级从高到底以此为ERROR>WARN>INFO>DEBUG,在将日志级别配置为DEBUG时，ERROR,WARN,INFO日志也都打印；当把日志级别配置为INFO时，ERROR,WARN级别也都被打印，依次类推。每个级别的简要说明

* ERROR:程序内部发生的错误，打印在日志中，应用能够继续运行
* WARN:应用内部的告警，可能会引起应用工作不正常
* INFO:能够在粗粒度上打印应用程序正在运行的情况，尽量保持在最低限度，INFO日志打印过多会影响应用的性能
* DEBUG:详细的程序运行流程信息，将这些信息记录到文件中，能够通过DEBUG日志定位问题

这样简单明了的打印应用的日志无论对日常运维和问题的定位都是非常有帮助的。日志过少可能信息量不够，无法判断出问题的所在；日志过多也会有同样的问题。曾经优化过一个系统性能，发现里面日志打印量巨大，把这些日志去掉后性能提升了25%左右。询问当时的开发人员为什么打印这多日志，他说日志全部打印出来分析问题比较容易。但是在一些性能测试中，模拟几万个节点的并发调用，在这种情况下日志本身就能压垮你的系统，更不要提业务了。
####日志文件的输出目的地
在log4j的配置文件中，描述输出目的地信息的配置叫做Appender。在log4j中有很多种的Appender:

* org.apache.log4j.ConsoleAppender   将日志输出到控制台
* org.apache.log4j.FileAppender      将日志输出到文件
* org.apache.log4j.DailyRollingFileAppender 将每天的日志输出到一个文件
* org.apache.log4j.RollingFileAppender      当一个日志文件到达一定大小时，重新创建一个新的日志文件
* org.apache.log4j.WriterAppender           将日志以流的方式发送到其他地方

当然你也可以定义自己的Appender，在上面这些Appender中在实际的工程中运用的最多的我觉得是ConsoleAppender和RollingFileAppender。console在调试运行或者一些小程序的时候是比较方便的，而RollingFileAppender由于其大小可控制，绕接的日志文件个数也是可控制的，等于有了一个自己清理老化数据的能力，所以在实际的工程运用中还是比较多的。唯一的缺点在于定位问题时，时间过长的日志数据会被绕接掉，现在云化的开发环境中，一般应用都会有日志收集器，日志都会及时的存储到日志中心所以这个问题也就不存在了。

####日志信息的格式
日志信息的格式，用log4j专用的属于叫做layout，也就是布局。log4j主要提供了如下几种布局

* org.apache.log4j.HTMLLayout 
* org.apache.log4j.PatternLayout
* org.apache.log4j.SimpleLayout 
* org.apache.log4j.TTCCLayout

在实际的工程应用中，使用的最多的应该是PatternLayout，它能够灵活的配置日志的格式。往往一个大的应用的日志格式都是一致的，这样便于后期对日志进行分析。

了解了log4j这几个重要的概念就可以在实际的项目中使用了。

###在代码中使用log4j
####添加log4j的依赖
maven仍然是java应用最主流的项目工程构建和管理工具。我们使用log4j最后的发布版本

```	
<dependency>
    <groupId>log4j</groupId>
    <artifactId>log4j</artifactId>
    <version>1.2.17</version>
</dependency>
```
添加完依赖后写一个简单的日志的测试类

```
package com.berrybay.log;

import org.apache.log4j.Logger;

public class SimpleLog {
    public static final Logger logger = Logger.getLogger(SimpleLog.class);
    public static void main(String[] args)
    {
        logger.error("this is error log");
        logger.warn("this is warn log");
        logger.info("this is info log");
        logger.debug("this is debug log");
    }
}
```
我们在一个main函数中依次输出error，warn，info，debug四种日志。前面说过了要同时输出这四种日志对日志的级别是有要求的，如果我们选择日志级别为ERROR时，只会输出ERROR日志，因为它的级别在这四种日志中最高。所以我们选择debug级别。下面是log4j.properties配置文件的内容，我们把日志输出到console和file两个地方

```
log4j.rootLogger = debug,stdout,rollingfile

#输出信息到控制台
log4j.appender.stdout = org.apache.log4j.ConsoleAppender
log4j.appender.stdout.Target = System.out
log4j.appender.stdout.layout = org.apache.log4j.PatternLayout
log4j.appender.stdout.layout.ConversionPattern = [%-5p] %d{yyyy-MM-dd HH:mm:ss,SSS} method:%l%n%m%n

#輸出信息到rolling file
##定义日志输出到哪里
log4j.appender.rollingfile = org.apache.log4j.RollingFileAppender
log4j.appender.rollingfile.File = D:\\dm_code\\logFile\\simplelog.log
log4j.appender.rollingfile.Threshold = DEBUG
##定义日志的格式
log4j.appender.rollingfile.layout = org.apache.log4j.PatternLayout
log4j.appender.rollingfile.layout.ConversionPattern = %-d{yyyy-MM-dd HH:mm:ss}  [ %t:%r ] - [ %p ]  %m%n
```

在上面的配置文件中，我们定义了两个appender:ConsoleAppender,RollingFileAppender我们将打印的日志同时输出到控制台和simplelog.log文件中，可以看一下执行结果

这是IDEA控制台的输出

```
"C:\Program Files\Java\jdk1.8.0_111\bin\java" -Didea.launcher.port=7533 "-Didea.launcher.bin.path=C:\Program Files (x86)\JetBrains\IntelliJ IDEA 2016.2.5\bin" -Dfile.encoding=UTF-8 -classpath "C:\Program Files\Java\jdk1.8.0_111\jre\lib\charsets.jar;C:\Program Files\Java\jdk1.8.0_111\jre\lib\deploy.jar;C:\Program Files\Java\jdk1.8.0_111\jre\lib\ext\access-bridge-64.jar;C:\Program Files\Java\jdk1.8.0_111\jre\lib\ext\cldrdata.jar;C:\Program Files\Java\jdk1.8.0_111\jre\lib\ext\dnsns.jar;C:\Program Files\Java\jdk1.8.0_111\jre\lib\ext\jaccess.jar;C:\Program Files\Java\jdk1.8.0_111\jre\lib\ext\jfxrt.jar;C:\Program Files\Java\jdk1.8.0_111\jre\lib\ext\localedata.jar;C:\Program Files\Java\jdk1.8.0_111\jre\lib\ext\nashorn.jar;C:\Program Files\Java\jdk1.8.0_111\jre\lib\ext\sunec.jar;C:\Program Files\Java\jdk1.8.0_111\jre\lib\ext\sunjce_provider.jar;C:\Program Files\Java\jdk1.8.0_111\jre\lib\ext\sunmscapi.jar;C:\Program Files\Java\jdk1.8.0_111\jre\lib\ext\sunpkcs11.jar;C:\Program Files\Java\jdk1.8.0_111\jre\lib\ext\zipfs.jar;C:\Program Files\Java\jdk1.8.0_111\jre\lib\javaws.jar;C:\Program Files\Java\jdk1.8.0_111\jre\lib\jce.jar;C:\Program Files\Java\jdk1.8.0_111\jre\lib\jfr.jar;C:\Program Files\Java\jdk1.8.0_111\jre\lib\jfxswt.jar;C:\Program Files\Java\jdk1.8.0_111\jre\lib\jsse.jar;C:\Program Files\Java\jdk1.8.0_111\jre\lib\management-agent.jar;C:\Program Files\Java\jdk1.8.0_111\jre\lib\plugin.jar;C:\Program Files\Java\jdk1.8.0_111\jre\lib\resources.jar;C:\Program Files\Java\jdk1.8.0_111\jre\lib\rt.jar;D:\dm_code\learn-test\target\classes;D:\dev-tool\maven_localRepository\log4j\log4j\1.2.17\log4j-1.2.17.jar;C:\Program Files (x86)\JetBrains\IntelliJ IDEA 2016.2.5\lib\idea_rt.jar" com.intellij.rt.execution.application.AppMain com.berrybay.log2.SimpleLog2
2017-02-14 22:51:36  [ main:0 ] - [ ERROR ]  this is error log
2017-02-14 22:51:36  [ main:1 ] - [ WARN ]  this is warn log
2017-02-14 22:51:36  [ main:2 ] - [ INFO ]  this is info log
2017-02-14 22:51:36  [ main:2 ] - [ DEBUG ]  this is debug log

Process finished with exit code 0
```

下面是simplelog.log文件中的内容

```
2017-02-14 22:32:34  [ main:0 ] - [ ERROR ]  this is error log
2017-02-14 22:32:34  [ main:2 ] - [ WARN ]  this is warn log
2017-02-14 22:32:34  [ main:3 ] - [ INFO ]  this is info log
2017-02-14 22:32:34  [ main:5 ] - [ DEBUG ]  this is debug log
```

我们发现两个地方输出的日志是一样，这是因为这两个appender使用了相同的日志级别和相同的日志格式。我们也可以将文件的日志级别设置成其他级别，只需要修改log4j.appender.rollingfile.Threshold这个配置项就可以了。

在实际的日常使用中整个应用不仅仅包含我们自己的代码，还包括很多第三方jar包，比如mybatis，netty和其他apache的包。我们不想打印所有的日志，比如对于那些中间件我们只需要开ERROR日志，我们的应用开INFO日志，这样才能在掌握自己应用运行的情况下更好的提高整体的性能。

清空日志，创建一个新的包log2,我们让log的日志输出级别为WARN，让log2的日志输出级别为ERROR。下面是配置

```
log4j.logger.com.berrybay.log=WARN
log4j.logger.com.berrybay.log2=ERROR
```
只要增加如上的配置就能控制一些特定的类的日志输出级别，也就是可以控制某个模块的日志输出级别，这个是非常实用的一个地方。
关于log4j还有一个没有总结，就是日志的格式如何配置，这样总结的文章有很多，摘录如下：

* %p 输出优先级，即DEBUG，INFO，WARN，ERROR，FATAL  
* %r 输出自应用启动到输出该log信息耗费的毫秒数  
* %c 输出所属的类目，通常就是所在类的全名  
* %t 输出产生该日志事件的线程名  
* %n 输出一个回车换行符，Windows平台为“rn”，Unix平台为“n”  
* %d 输出日志时间点的日期或时间，默认格式为ISO8601，也可以在其后指定格式，比如：%d{yyy MMM dd HH:mm:ss,SSS}，输出类似：2002年10月18日 22：10：28，921  
* %l 输出日志事件的发生位置，包括类目名、发生的线程，以及在代码中的行数。举例：Testlog4.main(TestLog4.java:10)
* %m 输出与日志记录事件相关联的应用程序提供的消息

在看一下我们上面在log4j中配置的内容:

```
%-d{yyyy-MM-dd HH:mm:ss}  [ %t:%r ] - [ %p ]  %m%n
```
从我们上面的内容中我们得出

````
2017-02-14 22:32:34  [ main:0 ] - [ ERROR ]  this is error log
2017-02-14 22:32:34  [ main:2 ] - [ WARN ]  this is warn log
````
我们的日志中记录了:日志时间，日志线程名称，日志输出到应用启动的时间，日志的级别，日志的内容和回车符

小伙伴们根据上面的信息就可以随意配置出自己想要打印的日志格式了。
