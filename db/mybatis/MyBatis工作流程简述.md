# MyBatis作用简述

## jdbc select 流程
我们想想没有用MyBatis的时候，直接用jdbc的话是怎么写的呢
1. 通过jdbc driver获取connection
2. 通过connection获取statement。
3. 通过调用statement的select方法（sql语句）获取ResultSet对象
   在这里需要将sql语句中的参数进行转换（java类型 --> jdbc类型）
4. 遍历ResultSet获取每一条数据库记录
   在这里需要将每一条数据库记录进行转换（jdbc类型 --> java类型）

## 作用1 数据类型转换 java类型 <--> jdbc类型 
MyBatis定义了一系列的类型转换，基本上可以覆盖99%的场景，
1. 接口org.apache.ibatis.type.TypeHandler
2. 基类org.apache.ibatis.type.BaseTypeHandler
3. 若干类型转换类 如：BooleanTypeHandler，LongTypeHandler，
4. TypeHandlerRegistry 已经注册好了的类型对应关系
    register(Boolean.class, new BooleanTypeHandler());
    register(Boolean.TYPE, new BooleanTypeHandler());
    register(JdbcType.BOOLEAN, new BooleanTypeHandler());
    register(JdbcType.BIT, new BooleanTypeHandler());
    register(Byte.class, new ByteTypeHandler());
    register(Byte.TYPE, new ByteTypeHandler());
    register(JdbcType.TINYINT, new ByteTypeHandler());
    register(Short.class, new ShortTypeHandler());
    register(Short.TYPE, new ShortTypeHandler());
    register(JdbcType.SMALLINT, new ShortTypeHandler());
    register(Integer.class, new IntegerTypeHandler());
    register(Integer.TYPE, new IntegerTypeHandler());
    。。。

## 作用2 不需要遍历ResultSet直接返回List
你只要定义一个接口，入力出力参数，以及sql语句，然后mybatis会自动设置sql语句中需要的参数，将返回的数据集转换为List<E>,E为一个pojo对象，mybatis会自动将ResultSet中的记录映射到E的pojo对象中。

# MyBatis工作流程简述

```java
public static void main(String[] args) {
        // 1. 读取配置文件
		InputStream inputStream = Resources.getResourceAsStream("mybatis-config.xml");
        // 2. 创建SqlSessionFactoryBuilder对象，调用build(inputstream)方法读取并解析配置文件，返回SqlSessionFactory对象
		SqlSessionFactory factory = new SqlSessionFactoryBuilder().build(inputStream);
        // 3. 由SqlSessionFactory创建SqlSession 对象，没有手动设置的话事务默认开启
		SqlSession sqlSession = factory.openSession();
        // 4  调用SqlSession中的api，传入Statement Id和参数，内部进行复杂的处理，最后调用jdbc执行SQL语句，封装结果返回。
		List<User> list = sqlSession.selectList("com.demo.mapper.UserMapper.getUserByName",params);
        // 4' 或者调用SqlSession 的getMapper方法，获得了Mapper，调用Mapper中的方法。
		UserMapper mapper = sqlSession.getMapper(UserMapper.class);
		List<User> list = mapper.getUserByName(params);
}
```

## Configuration
默认是通过读取xml配置文件，并将配置内容赋值到Configuration对象中。
我们同样可以在一个类中创建Configuration对象，手动设置其中属性的值来达到配置的效果。

## SqlSessionFactoryBuilder
这个类可以被实例化、使用和丢弃，一旦创建了 SqlSessionFactory，就不再需要它了。 因此 SqlSessionFactoryBuilder 实例的最佳作用域是方法作用域（也就是局部方法变量）。 

## SqlSessionFactory
SqlSessionFactory 一旦被创建就应该在应用的运行期间一直存在，没有任何理由丢弃它或重新创建另一个实例。 使用 SqlSessionFactory 的最佳实践是在应用运行期间不要重复创建多次，多次重建 SqlSessionFactory 被视为一种代码“坏习惯”。因此 SqlSessionFactory 的最佳作用域是应用作用域。 有很多方法可以做到，最简单的就是使用单例模式或者静态单例模式。

SqlSessionFactory用来生成SqlSession，生成SqlSession同时就拥有了以下对象。
- Transaction 用来管理事物的，是否自动commit
- Connection 根据DataSource获取一个数据库连接。
- Execution 是一个接口他有三个常用的实现类BatchExecutor（重用语句并执行批量更新），ReuseExecutor（重用预处理语句prepared statements），SimpleExecutor（普通的执行器，默认）。

## SqlSession接口，实现类：DefaultSqlSession（默认）
SqlSession是MyBatis中用于和数据库交互的顶层类，通常将它与ThreadLocal绑定（spring这么做的？），一个会话使用一个SqlSession，并且在使用完毕后需要close。
<font style='color:red'>注意：SqlSession中应该持有connection，SqlSession是于与ThreadLocal绑定的，如果多线程来执行更新，怎么保证在同一个事务中。</font>

每个线程都应该有它自己的 SqlSession 实例。SqlSession 的实例不是线程安全的，因此是不能被共享的，所以它的最佳的作用域是请求或方法作用域。 绝对不能将 SqlSession 实例的引用放在一个类的静态域，甚至一个类的实例变量也不行。 也绝不能将 SqlSession 实例的引用放在任何类型的托管作用域中，比如 Servlet 框架中的 HttpSession。 如果你现在正在使用一种 Web 框架，考虑将 SqlSession 放在一个和 HTTP 请求相似的作用域中。 换句话说，每次收到 HTTP 请求，就可以打开一个 SqlSession，返回一个响应后，就关闭它。 这个关闭操作很重要，为了确保每次都能执行关闭操作，你应该把这个关闭操作放到 finally 块中。


## MappedStatement 
MappedStatement与Mapper配置文件中的一个select/update/insert/delete节点相对应。mapper中配置的标签都被封装到了此对象中，主要用途是描述一条SQL语句。
mappedStatements 是一个HashMap，存储时key = 全限定类名 + 方法名，value = 对应的MappedStatement对象。

## transaction manager


## mybatis-spring

### SqlSessionTemplate
SqlSessionTemplate 是 MyBatis-Spring 的核心。作为 SqlSession 的一个实现，这意味着可以使用它无缝代替你代码中已经在使用的 SqlSession。 SqlSessionTemplate 是线程安全的，可以被多个 DAO 或映射器所共享使用。

## thread-safe
SqlSession 本身不是thread safe的，不建议在各thread中共享SqlSession。
依赖注入框架可以创建线程安全的、基于事务的 SqlSession 和映射器，并将它们直接注入到你的 bean 中，因此可以直接忽略它们的生命周期。 如果对如何通过依赖注入框架使用 MyBatis 感兴趣，可以研究一下 MyBatis-Spring 或 MyBatis-Guice 两个子项目。

## 插件（plugins）

## 官网
https://mybatis.org/mybatis-3/zh/sqlmap-xml.html


# 使用MyBatis

重点是围绕着Mapper中定义的sql语句，需要传递什么参数，返回什么结果

1. Mapper Interface
   确定Mapper中的sql语句和接口 List<User> selectUserListByName(String name)
   将Mapper中的接口的入力参数，和出力结果定义为pojo对象(User)。
   这个interface不用自己实现，MyBatis会通过代理的方式来注入实现。

2. 定义Mapper中的sql语句
   可以定义在xml中
    ```xml
    <?xml version="." encoding="UTF-"?>
    <!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper.//EN" "http://mybatis.org/dtd/mybatis--mapper.dtd">
    
    <!-- （namespace 就是定义的inerface名 ） -->
    <mapper namespace="com.demo.mapper.User"> ()
        
        <!-- （select的id为方法名 ） -->
        <select id="selectList" parameterType="java.lang.String" resultType="java.util.HashMap" useCache="false">
            select * from user
            where name = #{name,jdbcType=VARCHAR} <!-- （jdbcType可省略） -->
        </select>
    </mapper>
    </xml>

    ```
   也可以通过定义interface的方法，然后通过annotation来定义sql

3. 通过sqlSession调用mapper的方法获取结果
   
## 参数传递
- #{}预编译 会创建 PreparedStatement 参数占位符？（可防止sql注入）
  ```java
    //  会创建 PreparedStatement, #{userName} 会使用 ? 预处理。
    @Select("select * from user where user_id = #{userId}")
    User selectById(@Param("userId") String userId);
    ```
- ${}非预编译（直接的替换，不能防止sql注入）
  
  `ORDER BY ${columnName}`

```java
// 其中 ${column} 会被直接替换，而 #{value} 会使用 ? 预处理。
@Select("select * from user where ${column} = #{value}")
User findByColumn(@Param("column") String column, @Param("value") String value);
```


## 参数类型有三种
1、 基本数据类型
    `parameterType="java.lang.String"`
2、 HashMap（使用方式和pojo类似 ）
    `parameterType="java.util.map"`
3、 Pojo自定义包装类型
    `parameterType="com.demo.pojo.user"`

## 返回类型 resultMap
当bean的属性名和table的列名不一致的时候可以定义映射关系
```xml java
  @Results(id = "userMap",
            value = {
                    @Result(property = "id", column = "id"),
                    @Result(property = "username", column = "email"),
                    @Result(property = "password", column = "password"),
                    @Result(property = "role", column = "role")
            })
    @Select("select id" +
            "       ,email" +
            "       ,password      " +
            "       ,#{role}   as role      " +
            "  from t_store_manager" +
            " where id = #{id}" +
            "   and email = #{email}"
    )
     List<UserToken> selectStore(@Param(value = "id") Integer id,@Param(value = "email") String email, @Param(value = "role")Integer role);
```
## 返回类型 resultType
```xml
<select id="selectList" parameterType="java.lang.String" resultType="java.util.HashMap" useCache="false">
            select * from user
            where name = #{name,jdbcType=VARCHAR} <!-- （jdbcType可省略） -->
</select>
```


