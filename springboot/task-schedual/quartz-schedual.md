# Quartz任务调度

Quartz是一个定时任务框架，基础核心使用可以参考官网

Quartz源码：[github.com/quartz-sche…](https://link.juejin.cn?target=https%3A%2F%2Fgithub.com%2Fquartz-scheduler%2Fquartz)

Quartz官网地址：[www.quartz-scheduler.org/documentati…](https://link.juejin.cn?target=https%3A%2F%2Fwww.quartz-scheduler.org%2Fdocumentation%2F)

## 1. Scheduler

1. Scheduler为quartz中的调度器，Quartz通过调度器来注册、暂停、删除Trigger和JobDetail
2. Scheduler拥有SchedulerContext，通过SchedulerContext可以获取到触发器和任务的一些信息

## 2. Trigger

包括SimpleTrigger和CronTrigger

1. Trigger为触发器，通过cron表达式或日历指定任务执行的周期
2. 系统时间走到触发器指定的时间时，触发器就会触发任务的执行

## 3. JobDetail

1. 任务描述，JobDetail是任务的定义，而Job是任务的执行逻辑

## 4.Job

1. 完成任务的最小实现类，如果需要被定时调度的类都需要实现此接口

## 5.代码

```java
// 1、创建调度器Scheduler
        SchedulerFactory schedulerFactory = new StdSchedulerFactory();
        Scheduler scheduler = schedulerFactory.getScheduler();
        // 2、创建JobDetail实例，并与PrintWordsJob类绑定(Job执行内容)
        JobDetail jobDetail = JobBuilder.newJob(PrintWordsJob.class)
                                      .usingJobData("jobDetail1", "这个Job用来测试的")
                                      .withIdentity("job1", "group1").build();
        // 3、构建Trigger实例,每隔1s执行一次
        Date startDate = new Date();
        startDate.setTime(startDate.getTime() + 5000);

        Date endDate = new Date();
        endDate.setTime(startDate.getTime() + 5000);

        CronTrigger cronTrigger = TriggerBuilder.newTrigger()
                                          .withIdentity("trigger1", "triggerGroup1")
                                          .usingJobData("trigger1", "这是jobDetail1的trigger")
                                          .startNow()//立即生效
                                          .startAt(startDate)
                                          .endAt(endDate)
                                          .withSchedule(CronScheduleBuilder.cronSchedule("* 30 10 ? * 1/5 2018"))
                                          .build();
        //4、执行
        scheduler.scheduleJob(jobDetail, cronTrigger);
        System.out.println("--------scheduler start ! ------------");
        scheduler.start();
        System.out.println("--------scheduler shutdown ! ------------");
```



# springboot实现Quartz任务调度

## 1，pom文件引入依赖，只需要springboot 

```xml
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
			<artifactId>spring-boot-starter-quartz</artifactId>
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
public class QuartzTaskService {

	public void doJob() {
		log.info("run task1");
	}

}

```

## 3, 创建QuartzJob

继承于QuartzJobBean 或者实现 Job接口

```java
@Component
public class QuartzJob1 extends QuartzJobBean {
	@Autowired
	private QuartzTaskService scheduleService;

	@Override
	protected void executeInternal(JobExecutionContext context) throws JobExecutionException {
		scheduleService.doJob();
	}

}
```



## 4,创建QuartzJobConfig

spring系统自动创建了Scheduler，这里创建了JobDetail和Trigger

```java
@Configuration 
public class QuartzJobConfig {
	
	static {
		System.out.format("QuartzJobConfig init \r\n");
	}
	
	@Value("${quartz.task1}")
	private String testScheduleCron;
	
	
	@Bean
    public JobDetail teatQuartzDetail(){
        return JobBuilder.newJob(QuartzJob1.class).withIdentity("testQuartz").storeDurably().build();
    }

    @Bean
    public Trigger testQuartzTrigger(){
    	System.out.format("ScheduleCron:%s",testScheduleCron);
    	
        CronScheduleBuilder scheduleBuilder = CronScheduleBuilder.cronSchedule(testScheduleCron);
        return TriggerBuilder.newTrigger().forJob(teatQuartzDetail())
                .withIdentity("testQuartz")
                .withSchedule(scheduleBuilder)
                .build();
    }
}

	
```

## 5， 创建app

```java
@SpringBootApplication
public class SpringQuartzSchedualApplication {

	public static void main(String[] args) {
		SpringApplication.run(SpringQuartzSchedualApplication.class, args);
	}

}


```



## 6， application.properties文件中以cron方式设置任务的执行时间周期

```properties
# 每2秒执行一次
schedule.task1=0/2 * * * * ?

```



## 7，启动任务

```shell
  .   ____          _            __ _ _
 /\\ / ___'_ __ _ _(_)_ __  __ _ \ \ \ \
( ( )\___ | '_ | '_| | '_ \/ _` | \ \ \ \
 \\/  ___)| |_)| | | | | || (_| |  ) ) ) )
  '  |____| .__|_| |_|_| |_\__, | / / / /
 =========|_|==============|___/=/_/_/_/
 :: Spring Boot ::                (v2.6.3)

2022-02-04 18:15:46.480  INFO 19864 --- [           main] l.q.SpringQuartzSchedualApplication      : Starting SpringQuartzSchedualApplication using Java 1.8.0_65 on GUT-D2019-06 with PID 19864 (D:\github\java\spring-quartz-schedual\target\classes started by huanghui in D:\github\java\spring-quartz-schedual)
2022-02-04 18:15:46.482  INFO 19864 --- [           main] l.q.SpringQuartzSchedualApplication      : No active profile set, falling back to default profiles: default
QuartzJobConfig init 
ScheduleCron:0/2 * * * * ?2022-02-04 18:15:46.899  INFO 19864 --- [           main] org.quartz.impl.StdSchedulerFactory      : Using default implementation for ThreadExecutor
2022-02-04 18:15:46.906  INFO 19864 --- [           main] org.quartz.core.SchedulerSignalerImpl    : Initialized Scheduler Signaller of type: class org.quartz.core.SchedulerSignalerImpl
2022-02-04 18:15:46.906  INFO 19864 --- [           main] org.quartz.core.QuartzScheduler          : Quartz Scheduler v.2.3.2 created.
2022-02-04 18:15:46.907  INFO 19864 --- [           main] org.quartz.simpl.RAMJobStore             : RAMJobStore initialized.
2022-02-04 18:15:46.907  INFO 19864 --- [           main] org.quartz.core.QuartzScheduler          : Scheduler meta-data: Quartz Scheduler (v2.3.2) 'quartzScheduler' with instanceId 'NON_CLUSTERED'
  Scheduler class: 'org.quartz.core.QuartzScheduler' - running locally.
  NOT STARTED.
  Currently in standby mode.
  Number of jobs executed: 0
  Using thread pool 'org.quartz.simpl.SimpleThreadPool' - with 10 threads.
  Using job-store 'org.quartz.simpl.RAMJobStore' - which does not support persistence. and is not clustered.

2022-02-04 18:15:46.907  INFO 19864 --- [           main] org.quartz.impl.StdSchedulerFactory      : Quartz scheduler 'quartzScheduler' initialized from an externally provided properties instance.
2022-02-04 18:15:46.907  INFO 19864 --- [           main] org.quartz.impl.StdSchedulerFactory      : Quartz scheduler version: 2.3.2
2022-02-04 18:15:46.908  INFO 19864 --- [           main] org.quartz.core.QuartzScheduler          : JobFactory set to: org.springframework.scheduling.quartz.SpringBeanJobFactory@4cf8b2dc
2022-02-04 18:15:46.934  INFO 19864 --- [           main] o.s.s.quartz.SchedulerFactoryBean        : Starting Quartz Scheduler now
2022-02-04 18:15:46.934  INFO 19864 --- [           main] org.quartz.core.QuartzScheduler          : Scheduler quartzScheduler_$_NON_CLUSTERED started.
2022-02-04 18:15:46.939  INFO 19864 --- [           main] l.q.SpringQuartzSchedualApplication      : Started SpringQuartzSchedualApplication in 0.751 seconds (JVM running for 1.351)
2022-02-04 18:15:46.939  INFO 19864 --- [eduler_Worker-1] l.quartz.task.service.QuartzTaskService  : run task1
2022-02-04 18:15:48.010  INFO 19864 --- [eduler_Worker-2] l.quartz.task.service.QuartzTaskService  : run task1
2022-02-04 18:15:50.008  INFO 19864 --- [eduler_Worker-3] l.quartz.task.service.QuartzTaskService  : run task1
2022-02-04 18:15:52.013  INFO 19864 --- [eduler_Worker-4] l.quartz.task.service.QuartzTaskService  : run task1
2022-02-04 18:15:54.009  INFO 19864 --- [eduler_Worker-5] l.quartz.task.service.QuartzTaskService  : run task1
```



## 

