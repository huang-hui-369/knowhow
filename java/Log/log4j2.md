# Slf4j+log4j2的使用

## 1. 引入maven依赖

```xml
<!-- slf4j的依赖 -->
<dependency>
    <groupId>org.slf4j</groupId>
    <artifactId>slf4j-api</artifactId>
    <version>1.7.30</version>
</dependency>
<!-- log4j-slf4j 适配器的依赖 -->
<dependency>
    <groupId>org.apache.logging.log4j</groupId>
    <artifactId>log4j-slf4j-impl</artifactId>
    <version>2.13.3</version>
</dependency>

<!-- log4j2的依赖 -->
<dependency>
    <groupId>org.apache.logging.log4j</groupId>
    <artifactId>log4j-core</artifactId>
    <version>2.13.3</version>
</dependency>

<!-- log4j2的依赖 -->
<dependency>
    <groupId>org.apache.logging.log4j</groupId>
    <artifactId>log4j-api</artifactId>
    <version>2.13.3</version>
</dependency>


```

## 2. 配置文件log4j2.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<Configuration>
    <Properties>
        <property name="LOG_HOME">D:/logs</property>
        <!-- 定义日志格式 -->
        <Property name="log.pattern">%d{yyyy-MM-dd HH:mm:ss.SSS} [%t] %-5level %logger{36}%n%msg%n</Property>
        <!-- 定义文件名变量 -->
        <Property name="file.err.filename">${LOG_HOME}/log_error.log</Property>
        <Property name="file.err.pattern">${LOG_HOME}/error_$${date:yyyy-MM}/log_error-%d{yyyy-MM-dd HH-mm}-%i.log.gz</Property>
 
        <!-- 定义文件名变量 -->
        <Property name="file.info.filename">${LOG_HOME}/log_info.log</Property>
        <Property name="file.info.pattern">${LOG_HOME}/info_$${date:yyyy-MM}/log_info-%d{yyyy-MM-dd HH-mm}-%i.log.gz</Property>
 
    </Properties>
    <!-- 定义Appender，即目的地 -->
    <Appenders>
        <!-- 定义输出到屏幕 -->
        <Console name="console" target="SYSTEM_OUT">
            <!-- 日志格式引用上面定义的log.pattern -->
            <PatternLayout pattern="${log.pattern}" />
        </Console>
 
        <!-- 定义输出到文件,文件名引用上面定义的file.err.filename -->
        <RollingFile name="info" bufferedIO="true" fileName="${file.info.filename}" filePattern="${file.info.pattern}">
            <PatternLayout pattern="${log.pattern}" />
            <Policies>
                <!-- 根据文件大小自动切割日志 -->
                <SizeBasedTriggeringPolicy size="10 MB" />
            </Policies>
            <!-- 保留最近10份 -->
            <DefaultRolloverStrategy max="20" />
        </RollingFile>
 
        <!-- 定义输出到文件,文件名引用上面定义的file.err.filename -->
        <RollingFile name="err" bufferedIO="true" fileName="${file.err.filename}" filePattern="${file.err.pattern}">
            <PatternLayout pattern="${log.pattern}" />
            <Policies>
                <!-- 根据文件大小自动切割日志 -->
                <SizeBasedTriggeringPolicy size="10 MB" />
            </Policies>
            <!-- 保留最近10份 -->
            <DefaultRolloverStrategy max="20" />
        </RollingFile>
    </Appenders>
    <Loggers>
        <Root level="info">
            <!-- 对info级别的日志，输出到console -->
            <AppenderRef ref="console" level="info" />
            <!-- 对error级别的日志，输出到err，即上面定义的RollingFile -->
            <AppenderRef ref="err" level="error" />
            <!-- 对error级别的日志，输出到err，即上面定义的RollingFile -->
            <AppenderRef ref="info" level="info" />
        </Root>
    </Loggers>
</Configuration>
```

## 3. 测试类

```
import org.slf4j.Logger;
import org.slf4j.LoggerFactory; 

public class Test {
 
    public static void main(String[] args)  {
 
        Logger logger = LoggerFactory.getLogger(Test.class);
 
        for (int i =0 ;i<32000;i++){
            logger.trace("traceaaaaaaaaaaaaaaaa");
            logger.debug("debugaaaaaaaaaaaa");
            logger.info("infoaaaaaaaaaaaa");
            logger.warn("warnaaaaaaaaaa");
            logger.error("erroraaaaaaaaaaaaa");
        }
}
}
```





# springboot整合log4j2

## 1. pom.xml文件

```xml
<dependency>
           <groupId>org.springframework.boot</groupId>
           <artifactId>spring-boot-starter-web</artifactId>
           <exclusions><br>　　　　　　<!-- 去掉springboot默认的日志配置 -->
               <exclusion>
                   <groupId>org.springframework.boot</groupId>
                   <artifactId>spring-boot-starter-logging</artifactId>
               </exclusion>
           </exclusions>
</dependency>
 
<dependency> <!-- 引入log4j2依赖 -->
           <groupId>org.springframework.boot</groupId>
           <artifactId>spring-boot-starter-log4j2</artifactId>
</dependency>
```

## 2. 引入配置文件log4j2.xml

参照前面

## 3. 修改application.yml文件

```yaml
logging:
  config: classpath:log4j2.xml
```



# **输出到数据库** 

## 1.配置文件

```properties
\# LOG4J配置
log4j.rootCategory=INFO,stdout,jdbc
 
\# 数据库输出
log4j.appender.jdbc=org.apache.log4j.jdbc.JDBCAppender
log4j.appender.jdbc.driver=com.mysql.jdbc.Driver
log4j.appender.jdbc.URL=jdbc:mysql://127.0.0.1:3306/test?characterEncoding=utf8&useSSL=true
log4j.appender.jdbc.user=root
log4j.appender.jdbc.password=root
log4j.appender.jdbc.sql=insert into log_icecoldmonitor(level,category,thread,time,location,note) values('%p','%c','%t','%d{yyyy-MM-dd HH:mm:ss:SSS}','%l','%m')
```

## 2.引入数据库驱动

```
<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
</dependency>
```

## 3. 创建表

```sql
CREATE TABLE `log_icecoldmonitor` (
  `Id` int(11) NOT NULL AUTO_INCREMENT,
  `level` varchar(255) NOT NULL DEFAULT '' COMMENT '优先级',
  `category` varchar(255) NOT NULL DEFAULT '' COMMENT '类目',
  `thread` varchar(255) NOT NULL DEFAULT '' COMMENT '进程',
  `time` varchar(30) NOT NULL DEFAULT '' COMMENT '时间',
  `location` varchar(255) NOT NULL DEFAULT '' COMMENT '位置',
  `note` text COMMENT '日志信息',
  PRIMARY KEY (`Id`)
)
```



# Log4j2优点

1. 插件式结构。Log4j 2支持插件式结构。我们可以根据自己的需要自行扩展Log4j 2. 我们可以实现自己的appender、logger、filter。

2. 配置文件优化。在配置文件中可以引用属性，还可以直接替代或传递到组件。而且支持json格式的配置文件。不像其他的日志框架，它在重新配置的时候不会丢失之前的日志文件。

3. Java 5的并发性。Log4j 2利用Java 5中的并发特性支持，尽可能地执行最低层次的加锁。解决了在log4j 1.x中存留的死锁的问题。

4. 异步logger。Log4j 2是基于LMAX Disruptor库的。在多线程的场景下，和已有的日志框架相比，异步的logger拥有10倍左右的效率提升。

   ###### 

 官方建议一般程序员查看的日志改成异步方式，一些运营日志改成同步。日志异步输出的好处在于，使用单独的进程来执行日志打印的功能，可以提高日志执行效率，减少日志功能对正常业务的影响。异步日志在程序的classpath需要加载disruptor-3.0.0.jar或者更高的版本。

# 异步日志方式

## 依赖lib

```xml
 <dependency>
            <groupId>com.lmax</groupId>
            <artifactId>disruptor</artifactId>
            <version>3.3.6</version>
        </dependency>
```



## 全异步模式 

只需要在主程序代码开头，加一句系统属性的代码：

```java
System.setProperty("Log4jContextSelector", "org.apache.logging.log4j.core.async.AsyncLoggerContextSelector");  
```

## 异步和非异步混合输出模式 

在配置文件中Logger使用`<asyncRoot>` or `<asyncLogger>`

```xml
<loggers>  
     <AsyncLogger name="AsyncLogger" level="trace" includeLocation="true">  
        <appender-ref ref="Console" />  
        <appender-ref ref="debugLog" />  
        <appender-ref ref="errorLog" />  
    </AsyncLogger>  
 
    <asyncRoot level="trace" includeLocation="true">  
        <appender-ref ref="Console" />  
    </asyncRoot>   
</loggers>  
```



# 手动配置log4j2

## java source 配置log4j2

```java
public static void main(String[] args) throws IOException {  
    File file = new File("D:/log4j2.xml");  
    BufferedInputStream in = new BufferedInputStream(new FileInputStream(file));  
    final ConfigurationSource source = new ConfigurationSource(in);  
    Configurator.initialize(null, source);  
      
    Logger logger = LogManager.getLogger("myLogger");  
} 
```

## Web方式配置log4j2

```xml
<context-param>  
    <param-name>log4jConfiguration</param-name>  
    <param-value>/WEB-INF/conf/log4j2.xml</param-value>  
</context-param>  
  
<listener>  
    <listener-class>org.apache.logging.log4j.web.Log4jServletContextListener</listener-class>  
</listener>
```



# 使用 java source sample

```java
LogManager.getLogger(LogManager.ROOT_LOGGER_NAME) // root logger
Logger logger = LogManager.getLogger("AsyncFileLogger"); // Logger的名称

logger.trace("trace level");
logger.debug("debug level");
logger.info("info level");
logger.warn("warn level");
logger.error("error level");
logger.fatal("fatal level");
```

