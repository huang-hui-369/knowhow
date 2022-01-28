# 使用maven-shade-plugin插件打包

会见所有需要的依赖的jar一起打包

```xml
<build>  
    <plugins>  
  
        <plugin>  
            <groupId>org.apache.maven.plugins</groupId>  
            <artifactId>maven-shade-plugin</artifactId>  
            <version>2.4.1</version>  
            <executions>  
                <execution>  
                    <phase>package</phase>  
                    <goals>  
                        <goal>shade</goal>  
                    </goals>  
                    <configuration>  
                        <transformers>  
                            <transformer implementation="org.apache.maven.plugins.shade.resource.ManifestResourceTransformer">  
                                <mainClass>lhn.ftp.client.schedual.FtpDownloadSchedualApp</mainClass>  
                            </transformer>  
                        </transformers>  
                    </configuration>  
                </execution>  
            </executions>  
        </plugin>  
  
    </plugins>  
</build>  
```



https://maven.apache.org/plugins/index.html



## spring 打包问题

如果项目中用到了Spring Framework，将依赖打到一个jar包中，运行时会出现读取XML schema文件出错。原因是Spring Framework的多个jar包中包含相同的文件**spring.handlers**和**spring.schemas**，如果生成一个jar包会互相覆盖。为了避免互相影响，可以使用**AppendingTransformer**来对文件内容追加合并

```xml
<build>  
    <plugins>  
  
        <plugin>  
            <groupId>org.apache.maven.plugins</groupId>  
            <artifactId>maven-shade-plugin</artifactId>  
            <version>2.4.1</version>  
            <executions>  
                <execution>  
                    <phase>package</phase>  
                    <goals>  
                        <goal>shade</goal>  
                    </goals>  
                    <configuration>  
                        <transformers>  
                            <transformer implementation="org.apache.maven.plugins.shade.resource.ManifestResourceTransformer">  
                                <mainClass>com.xxg.Main</mainClass>  
                            </transformer>  
                            <transformer implementation="org.apache.maven.plugins.shade.resource.AppendingTransformer">  
                                <resource>META-INF/spring.handlers</resource>  
                            </transformer>  
                            <transformer implementation="org.apache.maven.plugins.shade.resource.AppendingTransformer">  
                                <resource>META-INF/spring.schemas</resource>  
                            </transformer>  
                        </transformers>  
                    </configuration>  
                </execution>  
            </executions>  
        </plugin>  
  
    </plugins>  
</build>  
```

