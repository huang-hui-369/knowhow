# QueryWrapper条件查询
<font style='color:red'>QueryWrapper 默认为 and条件。</font>

针对于下面的select语句
```sql
SELECT * FROM user_info WHERE 1=1 AND age = 20
```
## QueryWrapper写法
```java
QueryWrapper<UserInfo> queryWrapper = new QueryWrapper<>();
queryWrapper.eq("age", 20);
List<UserInfo> list = userInfoMapper.selectList(queryWrapper );
```

## QueryWrapper lambda写法
```java
QueryWrapper<UserInfo> queryWrapper = new QueryWrapper<>();
queryWrapper.lambda().eq(UserInfo::getAge, 20);
List<UserInfo> list = userInfoMapper.selectList(queryWrapper );
```

## LambdaQueryWrapper
```java
LambdaQueryWrapper<UserInfo> queryWrapper = new LambdaQueryWrapper<>();
queryWrapper.eq(UserInfo::getAge, 20);
List<UserInfo> list = userInfoMapper.selectList(queryWrapper );
```

# 多表join查询并且想要利用QueryWrapper

```sql
SELECT
	u.*,
	d.* 
FROM
	user_info u
	INNER JOIN depart d ON ( u.depart_id = d.id ) 
WHERE
	1 = 1 
	AND u.age > 20
    AND d.group_id = 10
```

## 1. 新建 UserDepartVO.java

```java
import com.demo.entity.UserInfo;
import lombok.Data;

@Data
public class UserDepartVO extends UserInfo {

    // 
    private String departName;
    private String departAddr;
}
```
## 2. 新建 UserDepartMappper.java

关键是在于${ew.customSqlSegment},会将QueryWrapper中的where部分追加到执行的sql语句中
或者${ew.sqlSegment}等价于${ew.customSqlSegment}

```java
public interface StoreInfoMapper {
    @ResultType(com.demo.vo.UserDepartVO.class)
    @Select(
    		"<script>"
	            "SELECT\n"
                + "	u.*,\n"
                + "	d.* \n"
                + "FROM\n"
                + "	user_info u\n"
                + "	INNER JOIN depart d ON ( u.depart_id = d.id ) "
	            + "${ew.customSqlSegment} \n"
            + "</script>"
            
    )
    IPage<UserDepartVO> selectUserDepartList(Page<UserDepartVO> page,  @Param(Constants.WRAPPER) Wrapper<UserDepartVO> queryWrapper);
}
```

## 3. 新建 UserDepartServImpl.java
```java
// 条件查询
QueryWrapper<UserInfoVO> queryWrapper = new QueryWrapper<>();
// 这个会转成u.age吗？
queryWrapper.lambda().eq(UserInfo::getAge, 20);
// 
queryWrapper.eq("d.group_id", 10);
// 分页对象
Page<UserInfoVO> queryPage = new Page<>(page, limit);
// 分页查询
IPage<UserInfoVO> iPage = userInfoMapper.list(queryPage , queryWrapper);
// 数据总数
Long total = iPage.getTotal();
// 用户数据
List<UserInfoVO> list = iPage.getRecords();
```



# 分页查询

## 分页查询
```java
// 条件查询
LambdaQueryWrapper<UserInfo> queryWrapper = new LambdaQueryWrapper<>();
queryWrapper.eq(UserInfo::getAge, 20);
// 分页对象
Page<UserInfo> queryPage = new Page<>(page, limit);
// 分页查询
IPage<UserInfo> iPage = userInfoMapper.selectPage(queryPage , queryWrapper);
// 总件数
Long total = iPage.getTotal();
// 总页数
long pages = iPage.getPages();
// 当前页index
long curPageIdx = iPage.getCurrent();
// 每页件数
long size1page = iPage.getSize()();
// 查询结果List
List<UserInfo> list = iPage.getRecords();
```


# fulltext查询

```sql
SELECT 
    *,
    MATCH (user_name , user_addr , user_desc , tel ) AGAINST ('张三A' ) AS score
FROM
    user_info
WHERE
    (status = 1
        AND (MATCH (user_name , user_addr , user_desc , tel) AGAINST ('张三A' )))
ORDER BY score DESC , create_time DESC
```

如果全文查询执行上面的全文检索sql语句，因为是模糊查询所以会出来张三A，张三b，张三C，张三。。。

如果想要张三A出现在最前面，需要按MATCH (user_name , user_addr , user_desc , tel ) AGAINST ('张三A' ) 排序。

思考一下，其实只需要在select语句中，和where语句中追加MATCH (user_name , user_addr , user_desc , tel) AGAINST ('张三A' )。

可以考虑在QueryWrapper中做，也可以在mapper中写sql语句。

## 如果用 IN BOOLEAN MODE的话就会只出现 张三A

```sql
SELECT 
    *,
    MATCH (user_name , user_addr , user_desc , tel ) AGAINST ('张三A' IN BOOLEAN MODE) AS score
FROM
    user_info
WHERE
    (status = 1
        AND (MATCH (user_name , user_addr , user_desc , tel) AGAINST ('张三A' IN BOOLEAN MODE)))
ORDER BY score DESC , create_time DESC
```

## QueryWrapper写法

service中

```java
public IPage<UserINfo> selctUserInfoByFulltext(String fulltext, int userState) {
		
		QueryWrapper<UserINfo> wrapper = new QueryWrapper<UserINfo>();
		if(StringUtils.isNoneEmpty(fulltext)) {
			String fulltext = String.format(" MATCH (user_name , user_addr , user_desc , tel) AGAINST ('%s' IN BOOLEAN MODE) AS score "
					, fulltext); 
            // 重写 selct语句        
			wrapper.select("*", fulltext);
            // 追加 order by
			wrapper.orderByDesc("score").orderByDesc("create_time");
		} else {
			wrapper.orderByDesc("create_time");
		}
		

		if (userState>=0) {
			wrapper.lambda().eq(UserINfo::getStatus, userState);
		} 
		// 在where中追击 全文检索
		if(StringUtils.isNoneEmpty(storeText)) {
			String fulltext = String.format(" MATCH (user_name , user_addr , user_desc , tel) AGAINST ('%s' IN BOOLEAN MODE) "
					, fulltext); 
			wrapper.apply(fulltext);
		}
		
		System.out.println(wrapper.getSqlSelect());
		System.out.println(wrapper.getTargetSql());
		
		IPage<UserINfo> iPage = userInfoMapper.selectPage(page, wrapper);
	
		return iPage;
	}
```

# 查询时排除列 @TableField(select = false) 

## @TableField(select = false)

我们查询用户数据时不需要查询deleted字段，那么通过@TableField(select = false)标识该字段，mybatis-plus在生成select语句时就不会包含deleted。

```java
/**
 * 是否删除：0否1是，默认0
 */
private Integer deleted;
```

## QueryWrapper的select方法
```java
@Test
public void method1() {
    QueryWrapper<User> qw = new QueryWrapper<>();
    qw.select(User.class, p -> !p.getColumn().equals("create_time") && !p.getColumn().equals("manager_id"));
    List<User> userList = userMapper.selectList(qw);
    userList.forEach(System.out::println);
}
```
# where子句嵌套查询条件
查询最新录入的一条邮箱是“@baomidou.com”或者“@qq.com”结尾的艺人的姓名和邮件。
```java
@Test
public void method1() {
    QueryWrapper<User> qw = new QueryWrapper<>();
    qw.select("name, email");
    qw.eq("position_no", "P003");
    qw.and(w -> {
        w.likeLeft("email", "@baomidou.com").or().likeLeft("email", "@qq.com");
    });
    qw.orderByDesc("create_time");
    qw.last("LIMIT 1");
    List<User> userList = userMapper.selectList(qw);
    userList.forEach(System.out::println);
}
```

分析：
1.通过qw.select()选择只查询用户姓名和邮件；
2.qw.eq(“position_no”, “P003”);过滤艺人；
3.qw.and(w -> {xxx});实现邮箱“或者”关系的嵌套；
4. w.likeLefe()实现邮件后缀的匹配；
5. qw.orderByDesc()根据创建时间倒序结合LIMIT 1，实现或者最新一条；


对应的SQL语句：

```sql
SELECT u.name, u.email FROM `user` u 
WHERE u.position_no='P003' 
AND (u.email LIKE '%@baomidou.com' OR u.email LIKE '%@qq.com') 
ORDER BY u.create_time DESC 
LIMIT 1;
```

上面嵌套的另一种写法
 qw.nested(w -> {
        w.likeLeft("email", "@baomidou.com").or().likeLeft("email", "@qq.com");
    });

## qw.and(w -> {xxx});嵌套里面还可以通过for循环动态拼接where子句。

```java
@Test
public void method1() {
    List<Long> ids = Arrays.asList(new Long[]{1087982257332887553L, 1088248166370832385L, 1088250446457389058L});

    QueryWrapper<User> qw = new QueryWrapper<>();
    qw.eq("position_no", "P003");
    qw.and(w -> {
        for (Long id : ids) {
            w.or().eq("id", id);
        }
    });
    List<User> userList = userMapper.selectList(qw);
    userList.forEach(System.out::println);
}

```

对应的SQL语句：

```sql
SELECT id,name,age,email,manager_id,create_time FROM USER 
WHERE position_no ='P003' 
AND (id = '1087982257332887553' OR id = '1088248166370832385' OR id = '1088250446457389058')

```

# where子句IN子查询
假设场景：查询年龄在20至30岁的艺人。

```java
@Test
public void method1() {
    QueryWrapper<User> qw = new QueryWrapper<>();
    qw.between("age", 20, 30);
    qw.inSql("position_no", "SELECT p.position_no FROM `position` p WHERE p.position_name='艺人'");
    List<User> userList = userMapper.selectList(qw);
    userList.forEach(System.out::println);
}

```

对应的SQL语句：

```sql
SELECT id,name,age,email,manager_id,create_time FROM user
WHERE age BETWEEN 20 AND 40 AND position_no IN (SELECT p.position_no FROM position p WHERE p.position_name='艺人')
```

或者先获取职位编码后直接使用mybatis-plus的in()。
```
@Test
public void method1() {
    List<String> positionNos = Arrays.asList("P002", "P003");
    QueryWrapper<User> qw = new QueryWrapper<>();
    qw.between("age", 20, 30);
    qw.in("position_no", positionNos);
    List<User> userList = userMapper.selectList(qw);
    userList.forEach(System.out::println);
}
```

# where子句使用SQL函数 (apply())
数据create_time是包含年月日时分秒的，现在通过日期匹配。

```java
@Test
public void method1() {
    QueryWrapper<User> qw = new QueryWrapper<>();
    qw.apply("date_format(create_time,'%Y-%m-%d') = {0}", "2021-03-31");
    List<User> userList = userMapper.selectList(qw);
    userList.forEach(System.out::println);
}

```
对应的SQL语句：

```sql
SELECT id,`name`,age,email,manager_id,create_time FROM `user` 
WHERE DATE_FORMAT(create_time,'%Y-%m-%d') = '2021-03-31'

```

# 除了QueryWrapper还可以用Map作为参数
```
@Test
public void method1() {
    Map<String, Object> params = new HashMap<>();
    params.put("name", "张靓颖");
    List<User> userList = userMapper.selectByMap(params);
    userList.forEach(System.out::println);
}

```

对应的SQL语句：

```sql
SELECT id,name,age,email,manager_id,create_time FROM user
WHERE name = '张靓颖'
```

