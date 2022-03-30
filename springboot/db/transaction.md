# @Transactional 事务不回滚排查

- @Transactional注解的方法是否为public

- @Transactional所注解的方法所在的类，是否已经使用了注解@Service或@Component等

- 需要调用该方法，且需要支持事务特性的调用方，是在 @Transactional所在的类的外面

  类内部的其他方法调用这个注解了@Transactional的方法，事务是不会起作用的。

- 事务的回滚仅仅对于RuntimeException,Error的异常有效。对于Exception异常无效。也就是说事务回滚仅仅发生在出现RuntimeException或Error的时候。

  

  

## 解决办法

1. 给注解@Transactional 加上参数如

   ```java
   @Transactional(rollbackFor=Exception.class)
   ```

   

2. 对Exception异常进行catch，然后抛出RuntimeException异常

3. 在service层方法的catch语句中增加：TransactionAspectSupport.currentTransactionStatus().setRollbackOnly();语句，手动回滚.

