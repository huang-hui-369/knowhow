# 使用maven-assembly-plugin插件打包

```xml
<plugin>  
    <groupId>org.apache.maven.plugins</groupId>  
    <artifactId>maven-assembly-plugin</artifactId>  
    <version>3.3.0</version>  
    <configuration>  
        <archive>  
            <manifest>  
                <mainClass>lhn.ftp.client.schedual.FtpDownloadSchedualApp</mainClass>  
            </manifest>  
        </archive>  
        <descriptorRefs>  
            <descriptorRef>jar-with-dependencies</descriptorRef>  
        </descriptorRefs>  
    </configuration>  
    <executions>  
        <execution>  
            <id>make-assembly</id>  
            <phase>package</phase>  
            <goals>  
                <goal>single</goal>  
            </goals>  
        </execution>  
    </executions>  
</plugin>  
```

会生成project-jar-with-dependencies.jar，会在打包时，从网上download所需要的所有class文件

![image-20220128201506241](D:\github\knowhow\dev-opt\使用maven-assembly-plugin插件打包.assets\image-20220128201506241.png)