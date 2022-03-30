# spring boot实现定时任务

spring boot内置了定时任务，使用@scheduler注解

- scheduler注解方式，一旦定时任务产生异常，那么此定时任务就无法再次启动，意味该定时任务就失效了，而Quartz不会。

- scheduler注解方式，当前面的定时任务没有完成的时候，无法再次开启定时任务，这说明scheduler注解方式是单线程，而Quartz是多线程，同一定时任务可以并发处理。

简单的任务可以使用scheduler注解方式因为设置简单

## 1，pom文件中不需要引入任何依赖，只需要springboot 

```xml
<modelVersion>4.0.0</modelVersion>
	<parent>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-parent</artifactId>
		<version>2.6.3</version>
		<relativePath/> <!-- lookup parent from repository -->
	</parent>
	<properties>
		<java.version>1.8</java.version>
	</properties>
	<dependencies>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter</artifactId>
		</dependency>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-test</artifactId>
			<scope>test</scope>
		</dependency>
	</dependencies>
```

## 2，创建执行具体任务的service

创建两个具体干活的task ScheduleTaskService1，ScheduleTaskService2

```java
@Service
@Slf4j
public class ScheduleTaskService1 {

	public void doJob() {
		log.info("run task1");
	}

}

@Service
@Slf4j
public class ScheduleTaskService2 {

	public void doJob() {
		log.info("run task2");
	}

}


```

## 3. 创建schedual Config

配置两个task按照配置文件application.properties中schedule.task1的配置时间定期运行。

```java
@Configuration
@EnableScheduling
@Slf4j
public class ScheduleTaskConfig {
	@Autowired
	ScheduleTaskService1 scheduleService1;
	
	@Autowired
	ScheduleTaskService2 scheduleService2;
	
	static {
		log.info("ScheduleConfig init");
	}
	

	@Scheduled(cron = "${schedule.task1}")
	public void doTask1() {
		scheduleService1.doJob();
	}
	
	@Scheduled(cron = "${schedule.task2}")
	public void doTask2() {
		scheduleService2.doJob();
	}
}
	
```

## 4， 创建app

```java
@SpringBootApplication
public class SpringSchedualApplication {

	public static void main(String[] args) {
		SpringApplication.run(SpringSchedualApplication.class, args);
	}

}

```



## 5， application.properties文件中以cron方式设置任务的执行时间周期

```properties
# 每10秒执行一次
schedule.task1=0/10 * * * * ?

# 0分开始，每1分钟执行一次
schedule.task2=0 0/1 * * * ?
```



## 6，启动任务

```shell
  .   ____          _            __ _ _
 /\\ / ___'_ __ _ _(_)_ __  __ _ \ \ \ \
( ( )\___ | '_ | '_| | '_ \/ _` | \ \ \ \
 \\/  ___)| |_)| | | | | || (_| |  ) ) ) )
  '  |____| .__|_| |_|_| |_\__, | / / / /
 =========|_|==============|___/=/_/_/_/
 :: Spring Boot ::                (v2.6.3)

2022-02-04 17:58:40.544  INFO 20296 --- [           main] lhn.schedual.SpringSchedualApplication   : Starting SpringSchedualApplication using Java 1.8.0_65 on GUT-D2019-06 with PID 20296 (D:\github\java\spring-schedual\target\classes started by huanghui in D:\github\java\spring-schedual)
2022-02-04 17:58:40.547  INFO 20296 --- [           main] lhn.schedual.SpringSchedualApplication   : No active profile set, falling back to default profiles: default
2022-02-04 17:58:40.838  INFO 20296 --- [           main] lhn.schedual.config.ScheduleTaskConfig   : ScheduleConfig init
2022-02-04 17:58:40.964  INFO 20296 --- [           main] lhn.schedual.SpringSchedualApplication   : Started SpringSchedualApplication in 0.718 seconds (JVM running for 1.344)
2022-02-04 17:58:50.011  INFO 20296 --- [   scheduling-1] l.schedual.service.ScheduleTaskService1  : run task1
2022-02-04 17:59:00.018  INFO 20296 --- [   scheduling-1] l.schedual.service.ScheduleTaskService2  : run task2
2022-02-04 17:59:00.019  INFO 20296 --- [   scheduling-1] l.schedual.service.ScheduleTaskService1  : run task1
2022-02-04 17:59:10.011  INFO 20296 --- [   scheduling-1] l.schedual.service.ScheduleTaskService1  : run task1
2022-02-04 17:59:20.015  INFO 20296 --- [   scheduling-1] l.schedual.service.ScheduleTaskService1  : run task1
2022-02-04 17:59:30.007  INFO 20296 --- [   scheduling-1] l.schedual.service.ScheduleTaskService1  : run task1
2022-02-04 17:59:40.009  INFO 20296 --- [   scheduling-1] l.schedual.service.ScheduleTaskService1  : run task1
2022-02-04 17:59:50.003  INFO 20296 --- [   scheduling-1] l.schedual.service.ScheduleTaskService1  : run task1
2022-02-04 18:00:00.005  INFO 20296 --- [   scheduling-1] l.schedual.service.ScheduleTaskService2  : run task2
2022-02-04 18:00:00.005  INFO 20296 --- [   scheduling-1] l.schedual.service.ScheduleTaskService1  : run task1
```



## 7，cron表达式参照

https://www.bejson.com/othertools/cron/

```shell
（1）0/2 * * * * ?   表示每2秒 执行任务

  （1）0 0/2 * * * ?    表示每2分钟 执行任务

  （1）0 0 2 1 * ?   表示在每月的1日的凌晨2点调整任务

  （2）0 15 10 ? * MON-FRI   表示周一到周五每天上午10:15执行作业

  （3）0 15 10 ? 6L 2002-2006   表示2002-2006年的每个月的最后一个星期五上午10:15执行作

  （4）0 0 10,14,16 * * ?   每天上午10点，下午2点，4点 

  （5）0 0/30 9-17 * * ?   朝九晚五工作时间内每半小时 

  （6）0 0 12 ? * WED    表示每个星期三中午12点 

  （7）0 0 12 * * ?   每天中午12点触发 

  （8）0 15 10 ? * *    每天上午10:15触发 

  （9）0 15 10 * * ?     每天上午10:15触发 

  （10）0 15 10 * * ?    每天上午10:15触发 

  （11）0 15 10 * * ? 2005    2005年的每天上午10:15触发 

  （12）0 * 14 * * ?     在每天下午2点到下午2:59期间的每1分钟触发 

  （13）0 0/5 14 * * ?    在每天下午2点到下午2:55期间的每5分钟触发 

  （14）0 0/5 14,18 * * ?     在每天下午2点到2:55期间和下午6点到6:55期间的每5分钟触发 

  （15）0 0-5 14 * * ?    在每天下午2点到下午2:05期间的每1分钟触发 

  （16）0 10,44 14 ? 3 WED    每年三月的星期三的下午2:10和2:44触发 

  （17）0 15 10 ? * MON-FRI    周一至周五的上午10:15触发 

  （18）0 15 10 15 * ?    每月15日上午10:15触发 

  （19）0 15 10 L * ?    每月最后一日的上午10:15触发 

  （20）0 15 10 ? * 6L    每月的最后一个星期五上午10:15触发 

  （21）0 15 10 ? * 6L 2002-2005   2002年至2005年的每月的最后一个星期五上午10:15触发 

  （22）0 15 10 ? * 6#3   每月的第三个星期五上午10:15触发
```

 

