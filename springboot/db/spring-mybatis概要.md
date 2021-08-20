# transction 管理原理
其实最根本就是用jdbc的connection的commit，rollback方法来管理。

```java
public static void main(String[] args) throws SQLException {
    HikariDataSource ds = new HikariDataSource();
    ds.setUsername("dev");
    ds.setDriverClassName("org.postgresql.Driver");
    ds.setJdbcUrl("jdbc:postgresql://192.168.11.116:5432/dev");
    ds.setPassword("secret");

    // トランザクションの設定
    Connection conn = ds.getConnection();
    conn.setAutoCommit(false);
    conn.setTransactionIsolation(Connection.TRANSACTION_READ_COMMITTED);

    // 同一コネクションを利用し、様々なSQLを実行する
    // これで同一トランザクションで処理が実行される
    PreparedStatement pstmt1 = conn.prepareStatement("INSERT INTO sample VALUES ('apple')");
    pstmt1.executeUpdate();
    PreparedStatement pstmt2 = conn.prepareStatement("INSERT INTO sample VALUES ('banana')");
    pstmt2.executeUpdate();
    
    conn.commit();
}
```

# 一个简单的例子ソースコード


## main class
```java

@RequiredArgsConstructor
@SpringBootApplication
public class Main {

  public static void main(String[] args) {
    new SpringApplicationBuilder(Main.class)
        .web(WebApplicationType.NONE)
        .run();
  }

  private final SampleService service;

  @Bean
  public CommandLineRunner run() {
    return i -> service.insert();
  }
}
```

## service

```java
@Service
@RequiredArgsConstructor
public class SampleServiceImpl implements SampleService {
  private final SampleRepository sampleRepository;

  @Override
  @Transactional
  public void insert() {
    sampleRepository.save("apple");
    sampleRepository.save("banana");
  }
}
```

## mapper

```java
@Mapper
public interface SampleRepository {
  @Insert("INSERT INTO sample VALUES (#{name})")
  int save(@Param("name") String name);
}
```

## application.properties

```properties
application.properties

spring.datasource.driver-class-name=org.postgresql.Driver
spring.datasource.url=jdbc:postgresql://192.168.11.116:5432/dev
spring.datasource.username=dev
spring.datasource.password=secret
```

# Spring-mybatis処理の概要
- Spring は、コネクションをスレッドローカルな値として管理する
- MyBatis は、スレッドローカルに管理されたコネクションを取得し、同一のコネクションを利用してSQLを発行する
- 上記２点により、 MyBatis で実行した SQL が Spring のトランザクションで実行できるようになる

# Spring のトランザクション処理
Spring は、@Transactionalがついたメソッドの実行前に、 TransactionInterceptor#invoke を実行する。 この処理は、 DataSource から Connection を取得し、@Transactionalアノテーションに設定された Isolation や propagation に基づいて Connection の設定を行う。また、TransactionSynchronizationManager にスレッドローカルな値として Connection を格納する。

![](img\2021-08-20-18-24-23.png)

# MyBatis のメソッド実行の処理

MyBatis のMapperインタフェースを実行すると、以下の処理を実行する。

SqlSession を継承した SqlSessionTemplate というクラスは spring-mybatis が提供しており、このクラスが MyBatis の SqlSession を管理している。これによって、同一トランザクションでは同一の SqlSession インスタンスが取得できる。

![](img\2021-08-20-18-31-45.png)

# SqlSession の取得処理

SqlSession を取得する処理。Transaction クラスを継承した SpringManagedTransaction を返す点がポイント。

![](img\2021-08-20-18-32-52.png)

# クエリの実行

Executor がクエリを実行する処理。TransactionSynchronizationManager から Connection を取得する。Spring のコネクションと同様の Connection を利用して SQL を実行することで、 Spring のトランザクション上で MyBatis の処理が実行される。

![](img\2021-08-20-18-34-05.png)