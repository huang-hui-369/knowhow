# java log 门面接口

只提供接口定义，没有具体实现。类似于`JDBC` 的`api` 接口，具体的的`JDBC driver` 实现由各数据库提供商实现

## - commons-logging

​	apache最早提供的日志的门面接口。避免和具体的日志方案直接耦合。通过统一接口解耦，不过其内部也实现了一些简单日志方案。通过动态查找的机制，在程序运行时自动找出真正使用的日志库

## - SLF4J(Simple logging Facade for Java)

​	SLF4J，即简单日志门面（Simple Logging Facade for Java），不是具体的日志解决方案,是为了替代Commons-Logging，slf4j在编译时静态绑定真正的Log库

​	slf4j-api.jar SLF4J门面接口

​	以下是SLF4J的实现和SLF4J适配器

​    

| SLF4J门面     | log provider                          | SLF4J适配器          | 说明                          |
| ------------- | ------------------------------------- | -------------------- | ----------------------------- |
| slf4j-api.jar | slf4j-simple.jar                      | 不需要               | 一个简单的SLF4J门面实现       |
| slf4j-api.jar | logback-classic.jar，logback-core.jar | 不需要               | SLF4J门面实现                 |
| slf4j-api.jar | log4j-api.jar，log4j-core.jar         | slf4j-log4j-impl.jar | log4j，log4j2实现             |
| slf4j-api.jar | java.util.logging.Logger              | slf4j-jdk.jar        | jkd log实现                   |
| slf4j-api.jar | SLF4J-NOP.jar                         |                      | 一个空的实现，及不打印任何log |



## jar依赖构造图

![image-20220128122715737](D:\github\knowhow\java\Log\Java日志框架SLF4J-log4j-logback区别.assets\image-20220128122715737.png)



## log组成

- Appenders：输出源 有控制台Console、文件File、DB，NET
- PatternLayout： 指定输出log的格式
- Loggers
- level： TRACE, DEBUG, INFO, WARN, ERROR, ALL 
- 