# 字段为空update修改不生效的问题

Mybatis-plus默认的字段策略是：

```yaml
mybatis-plus:
  mapper-locations: classpath*:mapper/**/*.xml
  global-config:
    db-config:
      #字段策略，默认非NULL判断
      field-strategy: not_null

```

此时，调用Mybatis-plus的update()或者updateById()时，如果对象的某个字段为NULL，就不会更新这个字段。
解决方式：
1、修改全局配置，如上面的yaml配置；
2、修改该字段的配置策略，如：

```java
@TableField(updateStrategy = FieldStrategy.IGNORED)
private LocalDateTime operateTime;

@TableField(updateStrategy = FieldStrategy.IGNORED)
private String operateUserId;

```
这样就能忽略了为NULL的判断，此时operateTime、operateUserId为NULL，调用修改操作后对应数据库的列就是null了。
