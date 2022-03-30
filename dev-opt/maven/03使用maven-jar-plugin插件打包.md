# 使用maven-jar-plugin打包可执行jar

## project目录

- project/src/main/java/*.java

  所有的source文件

- project/src/main/java/*.properties

  source目录下的配置文件

- project/src/main/java/*.xml

  source目录下的配置文件

- project/src/main/resources

  配置文件目录

## 打包后的目录

- project/target/classes

  编译后的class，配置文件

- project/target/lib

  需要依赖的jar文件

- project/target/test-classes

  编译后的test class文件

- project/target/project.jar

  生成的jar 文件

  在targer目录下可执行

  ```shell
  java -jar project.jar
  ```

# 1. 利用maven-jar-plugin将class打包成jar文件

```xml
<!-- 打包可执行jar -->
      <plugin>
        <groupId>org.apache.maven.plugins</groupId>
        <artifactId>maven-jar-plugin</artifactId>
        <version>3.2.2</version>
        <configuration>
          <!-- 将src目录下的配置文件打包到class目录下，如果不加<includes>节点，只会打包*.java文件-->
          <includes>
            <include>/*</include>
          </includes>
          <archive>
            <manifest>  
                 <!-- 会在MANIFEST.MF文件中追加 Class-Path: lib/commons-net-3.8.0.jar lib/slf4j-api-1.7.35.jar 。。。 -->
                 <addClasspath>true</addClasspath>  
                 <classpathPrefix>lib/</classpathPrefix>  
                 <!-- 可执行jar包 入口 main 文件-->
                 <mainClass>lhn.ftp.client.schedual.FtpDownloadSchedualApp</mainClass>  
             </manifest>  
          </archive>
        </configuration>
        
      </plugin>
```

# 2. 利用maven-dependency-plugin将所依赖jar文件打包到lib目录下

如果不用maven-dependency-plugin，target目录下不会有所依赖jar文件

```xml
<plugin>  
    <groupId>org.apache.maven.plugins</groupId>  
    <artifactId>maven-dependency-plugin</artifactId>  
    <version>3.2.0</version>  
    <executions>  
        <execution>  
            <id>copy-dependencies</id>  
            <phase>package</phase>  
            <goals>  
                <goal>copy-dependencies</goal>  
            </goals>  
            <configuration>  
                <outputDirectory>${project.build.directory}/lib</outputDirectory>  
            </configuration>  
        </execution>  
    </executions>  
</plugin>  
```



# 3. 可选打包配置文件

1. 在 maven-jar-plugin 中加入 <includes>节点

   ![image-20220128195728164](D:\github\knowhow\dev-opt\maven-jar-plugin.assets\image-20220128195728164.png)

2. 在 build中加入 <rusources>节点

   ![image-20220128195827709](D:\github\knowhow\dev-opt\maven-jar-plugin.assets\image-20220128195827709.png)

# 4. 缺点

没有将class文件和依存的jar文件打包成一个jar文件

```
project/target
          | --  lib
                 | -- 1.jar，2.jar
          | --  project.jar        
```

​             

