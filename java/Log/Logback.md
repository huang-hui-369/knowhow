# logback查找顺序

`logback`在启动时，根据以下步骤寻找配置文件：

①在`classpath`中寻找`logback-test.xml`文件→

②如果找不到`logback-test.xml`，则在 `classpath`中寻找`logback.groovy`文件→

③如果找不到 `logback.groovy`，则在`classpath`中寻找`logback.xml`文件

 ④如果上述的文件都找不到，则`logback`会使用`JDK`的`SPI`机制查找 `META-INF/services/ch.qos.logback.classic.spi.Configurator`中的 `logback` 配置实现类，这个实现类必须实现`Configuration`接口，使用它的实现来进行配置

⑤如果上述操作都不成功，`logback` 就会使用它自带的 `BasicConfigurator` 来配置，并将日志输出到`console`。

# logback配置文件

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!--scan:
            当此属性设置为true时，配置文件如果发生改变，将会被重新加载，默认值为true。
scanPeriod:
            设置监测配置文件是否有修改的时间间隔，如果没有给出时间单位，默认单位是毫秒。当scan为true时，此属性生效。默认的时间间隔为1分钟。
debug:
            当此属性设置为true时，将打印出logback内部日志信息，实时查看logback运行状态。默认值为false。

configuration 子节点为 appender、logger、root

            -->
<configuration scan="true" scanPeriod="60 seconds" debug="false">
    <!--用于区分不同应用程序的记录-->
    <contextName>edu-cloud</contextName>
    <!--日志文件所在目录，如果是tomcat，如下写法日志文件会在则为${TOMCAT_HOME}/bin/logs/目录下-->
    <property name="LOG_HOME" value="logs"/>

    <!--控制台-->
    <appender name="stdout" class="ch.qos.logback.core.ConsoleAppender">
        <encoder>
            <!--格式化输出：%d表示日期，%thread表示线程名，%-5level：级别从左显示5个字符宽度 %logger输出日志的logger名 %msg：日志消息，%n是换行符 -->
            <pattern>[%d{yyyy-MM-dd HH:mm:ss.SSS}] [%thread] %-5level %logger{36} : %msg%n</pattern>
            <!--解决乱码问题-->
            <charset>UTF-8</charset>
        </encoder>
    </appender>

    <!--滚动文件-->
    <appender name="infoFile" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <!-- ThresholdFilter:临界值过滤器，过滤掉 TRACE 和 DEBUG 级别的日志 -->
        <filter class="ch.qos.logback.classic.filter.ThresholdFilter">
            <level>INFO</level>
        </filter>
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <fileNamePattern>${LOG_HOME}/log.%d{yyyy-MM-dd}.log</fileNamePattern>
            <maxHistory>30</maxHistory><!--保存最近30天的日志-->
        </rollingPolicy>
        <encoder>
            <charset>UTF-8</charset>
            <pattern>[%d{yyyy-MM-dd HH:mm:ss.SSS}] [%thread] %-5level %logger{36} : %msg%n</pattern>
        </encoder>
    </appender>

    <!--滚动文件-->
    <appender name="errorFile" class="ch.qos.logback.core.rolling.RollingFileAppender">
             <!-- ThresholdFilter:临界值过滤器，过滤掉 TRACE 和 DEBUG 级别的日志 -->
        <filter class="ch.qos.logback.classic.filter.ThresholdFilter">
            <level>error</level>
        </filter>
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <fileNamePattern>${LOG_HOME}/error.%d{yyyy-MM-dd}.log</fileNamePattern>
            <maxHistory>30</maxHistory><!--保存最近30天的日志-->
        </rollingPolicy>
        <encoder>
            <charset>UTF-8</charset>
            <pattern>[%d{yyyy-MM-dd HH:mm:ss.SSS}] [%thread] %-5level %logger{36} : %msg%n</pattern>
        </encoder>
    </appender>

    <!--将日志输出到logstack-->
    <!--<appender name="logstash" class="net.logstash.logback.appender.LogstashTcpSocketAppender">
        <destination>47.93.173.81:7002</destination>
        <encoder class="net.logstash.logback.encoder.LogstashEncoder">
            <charset>UTF-8</charset>
        </encoder>
        <keepAliveDuration>5 minutes</keepAliveDuration>
    </appender>-->

    <!--这里如果是info，spring、mybatis等框架则不会输出：TRACE < DEBUG < INFO <  WARN < ERROR-->
    <!--root是所有logger的祖先，均继承root，如果某一个自定义的logger没有指定level，就会寻找
    父logger看有没有指定级别，直到找到root。-->
    <root level="debug">
        <appender-ref ref="stdout"/>
        <appender-ref ref="infoFile"/>
        <appender-ref ref="errorFile"/>
        <appender-ref ref="logstash"/>
    </root>

    <!--为某个包单独配置logger

    比如定时任务，写代码的包名为：com.seentao.task
    步骤如下：
    1、定义一个appender，取名为task（随意，只要下面logger引用就行了）
    appender的配置按照需要即可


    2、定义一个logger:
    <logger name="com.seentao.task" level="DEBUG" additivity="false">
      <appender-ref ref="task" />
    </logger>
    注意：additivity必须设置为false，这样只会交给task这个appender，否则其他appender也会打印com.seentao.task里的log信息。

    3、这样，在com.seentao.task的logger就会是上面定义的logger了。
    private static Logger logger = LoggerFactory.getLogger(Class1.class);
    -->

</configuration>
```

# 主要概念

- `appender`，负责定义日志的输出目的地（控制台、日志文件、滚动日志文件，其他如logstash等）。

- `encoder`负责定义日志的输出样式和字符编码，如果在控制台出现`?????`或乱码，则指定编码（一般是`UTF-8`）就好了。
- `filter`负责过滤日志，即使logger传来了`dubug`级别以上的日志，如果`filter`中设定了级别为`info`，则该`appender`只会将`info`级别及以上的日志输出到目的地。
- `rollingPolicy`负责制定日志文件的滚动规则，是根据日志文件大小还是根据日期进行滚动。

- `logger`，负责定义我们实际代码中使用的`logger`。

## logger

`logger`中有一个非常重要的属性`name`，`name`必须指定。在`logback`中，`logger`有继承关系，而所有的`logger`的祖先是`root`。
 举个例子，如果我们有个类叫`UserService`，所在的包为`com.maxwell.service`，在`UserService`中要打印日志，我们一般会这么写：

```java
①private  Logger logger = LoggerFactory.getLogger(UserService.class);
或者
②private  Logger logger = LoggerFactory.getLogger("com.maxwell.service.UserService");
```

这两种写法是一样的，第①中写法实际会转化为②中的方式来获取一个`logger`实例。
 当我们写下这行代码时，`logback`会依次检查以下各个`logger`实例的是否存在，如果不存在则依次创建：

```css
com
com.maxwell
com.maxwell.service
com.maxwell.service.UserService
```

而创建logger实例的时候，就会去配置文件中查找名字为`com`、`com.maxwell`、`com.maxwell.service`、`com.maxwell.service.UserService`的`logge`标签，并根据其中定义的规则创建。所以，假如你在配置文件中没有定义`name`为上述字符串的`logger`时，就会找到`root`这个祖先，根据`root`标签定义的规则创建`logger`实例。

理解了上面的这一点，想要实现某个包或者某个类单独输出到某日志文件的需求就很好实现了：①定义一个`appender`，指明日志文件的输出目的地；②定义一个`logger`，`name`设为那个包或类的全路径名。如果只想将这个类或包的日志输出到刚才定义的`appender`中，就将`additivity`设置为`false`。

一条日志想要顺利到达输出目的地，除了`logger`的级别要低于该级别，`appender`中的`filter`中设置的级别也要低于该级别。`（TRACE < DEBUG < INFO < WARN < ERROR）`

# logback模块

- logback-core：其它两个模块的基础模块
- logback-classic：它是log4j的一个改良版本，同时它完整实现了slf4j API使你可以很方便地更换成其它日志系统如log4j或JDK14 Logging
- logback-access：访问模块与Servlet容器集成提供通过Http来访问日志的功能



