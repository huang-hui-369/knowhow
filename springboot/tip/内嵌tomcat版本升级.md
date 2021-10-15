# pom文件修改
- 追加 tomcat.version
- 追加 dependencyManagement
- 追加 repository

![](img\2021-10-12-17-25-47.png)


## 追加 tomcat.version
在 pom文件中的properties节点下追加以下
`<tomcat.version>9.0.54</tomcat.version>`
![](img\2021-10-12-17-15-45.png)

## 追加 dependencyManagement

```
<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>org.apache.tomcat.embed</groupId>
            <artifactId>tomcat-embed-core</artifactId>
            <version>${tomcat.version}</version>
        </dependency>
        <dependency>
            <groupId>org.apache.tomcat.embed</groupId>
            <artifactId>tomcat-embed-el</artifactId>
            <version>${tomcat.version}</version>
        </dependency>
        <dependency>
            <groupId>org.apache.tomcat.embed</groupId>
            <artifactId>tomcat-embed-websocket</artifactId>
            <version>${tomcat.version}</version>
        </dependency>
    </dependencies>
</dependencyManagement>
```

![](img\2021-10-12-17-20-22.png)

## 追加 repository

```
<repositories>
    <repository>
        <id>spring-io</id>
        <name>Spring io</name>
        <url>http://repo.spring.io/plugins-release</url>
    </repository>
</repositories>
```

## 结果
tomcat 从9.0.30 升级到 9.0.54

![](img\2021-10-12-17-21-40.png)