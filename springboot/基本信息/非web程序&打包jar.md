# springboot 非web server程序

有时程序没有规划好，混在一起时，可以在springboot程序中写多个app文件作为程序入口，针对每个APP文件制作一个pom文件，然后打包成jar 例如：

- WebServerApp   

  web server 程序 -- pom.xml

- ExecApp

  非web server 程序 -- exec-app-pom.xml

## 1. 设置WebApplicationType.NONE，实现CommandLineRunner interface

```java
@SpringBootApplication
@MapperScan("dbo.mapper")
@EnableTransactionManagement(proxyTargetClass = true)
@Slf4j
public class ExecApp implements CommandLineRunner {
	
	 public static void main(String[] args) {
		SpringApplication app = new SpringApplication(ExecApp.class);
		app.setBannerMode(Mode.OFF);
		app.setWebApplicationType(WebApplicationType.NONE);
		ConfigurableApplicationContext ctx = app.run(args);
		
		log.info("ExecApp start run");
		log.info("db host:{}", ctx.getEnvironment().getProperty("spring.datasource.url"));
		
//		ConfigurableApplicationContext ctx = new SpringApplicationBuilder(ImportShopExcelApp.class)
//			.bannerMode(Mode.OFF)
//			.web(WebApplicationType.NONE)
//			.run(args);
	}
	
	 
	@Override
	public void run(String... args) throws Exception{
		
        System.out.println("hello word");
		
	}

}
```

## 2.  修改pom文件exclude  embed tomcat  server 

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
    <exclusions>
        <exclusion>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-tomcat</artifactId>
        </exclusion>
    </exclusions>
</dependency>
```

或者 remove spring-boot-starter-web dependency

```xml
<dependency>
   <groupId>org.springframework.boot</groupId>
   <artifactId>spring-boot-starter-web</artifactId>
</dependency>
```

## 3. 利用spring-boot-maven-plugin打包成可执行jar文件

注意：**设置mainClass**

```xml
<build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
                <version>2.3.4.RELEASE</version>
                 <configuration>
	                <mainClass>org.ExecApp</mainClass>
	            </configuration>
                <executions>
                    <execution>
                        <goals>
                            <goal>repackage</goal>
                        </goals>
                    </execution>
                </executions>
            </plugin>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <version>3.1</version>
                <configuration>
                    <source>8</source>
                    <target>8</target>
                    <!-- 设置编码为 UTF-8 -->
					<encoding>UTF-8</encoding>	
                </configuration>
            </plugin>
        </plugins>
        <resources>
            <resource>
                <directory>src/main/resources</directory>
                <filtering>true</filtering>
                <includes>
                    <include>**/*.yml</include>
                </includes>
            </resource>
            <resource>
                <directory>src/main/resources</directory>
                <filtering>false</filtering>
                <excludes>
                    <exclude>**/*.yml</exclude>
                </excludes>
            </resource>
        </resources>
    </build>
```

## 4.执行mvn命令打包jar文件

```shell
mvn clean package -f ExecPom.xml
```

## 5. springboot実行可能 Jar文件构造

- ​	org.springframework.boot.loader 包 -- 加载可执行文件的loader JarLauncher
- BOOT-INF/classes 为所写execApp class文件目录
- BOOT-INF/lib 为所依赖的jar包

```
execApp.jar
 |
 +-META-INF
 |  +-MANIFEST.MF
 +-org
 |  +-springframework
 |     +-boot
 |        +-loader
 |           +-<spring boot loader classes>
 +-BOOT-INF
    +-classes
    |  +-org
    |     +-project
    |        +-ExecApp.class
    +-lib
       +-dependency1.jar
       +-dependency2.jar
```



### mainfest.mf

从mainfest.mf文件可以看出，实际上执行的是org.springframework.boot.loader.JarLauncher

JarLauncher再去加载app的**Start-Class**: org.ExecApp

```
Manifest-Version: 1.0
Spring-Boot-Classpath-Index: BOOT-INF/classpath.idx
Archiver-Version: Plexus Archiver
Built-By: huanghui
Start-Class: org.ExecApp
Spring-Boot-Classes: BOOT-INF/classes/
Spring-Boot-Lib: BOOT-INF/lib/
Spring-Boot-Version: 2.3.4.RELEASE
Created-By: Apache Maven 3.6.3
Build-Jdk: 1.8.0_202
Main-Class: org.springframework.boot.loader.JarLauncher
```



