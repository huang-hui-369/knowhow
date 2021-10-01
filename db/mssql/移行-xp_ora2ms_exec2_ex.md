# oracle --> mssql 移行

## O2SS0452警告
<font style='color:red'>O2SS0452: "xp_ora2ms_exec2_ex" when called from within UDF cannot bind to outer transaction. It can lead to dead locks and losing transaction atomicity. Consider calling $impl procedure directly.</font>

###　原因
1. oracle中允许function里 执行DML
   sql server 不允許 function里 更改table的值，例如 insert，update（DML 执行）但是 Stored Procedures 可以

2. sql server 不允許用戶做的fuction（UDF）調用procedure
3. sql server 不允許 function里使用try catch
4. 针对上面1，2，3的限制，sql server 为了解决用戶做的fuction（UDF）調用procedure 的问题，提供了一个模拟器（master.dbo.p_ora2ms_exec2_ex调用接口用来调用procedure）	个人感觉这个应该是另一个语言（c，c#）做的，所以用的时候会有一些限制。	
5. 使用master.dbo.p_ora2ms_exec2_ex调用接口时，会有一些副作用，O2SS0452警告就是其中之一,使用这个接口会它不能绑定在外部也就是调用它的transaction上，而是产生一个新的transaction，这样的话就会造成死锁。例如下面的例子

### SQL Server Migration Assistant for Oracle (OracleToSQL)
例如：Oracle 中以下的流程

1. procedure executeUpdateUser中代码
    ```
    select * from user for update
    where uid = paramUid

    call updateUserFunction(paramUid)
    call updateUserInfoFunction(paramUid)
    call updateUserDeptFunction(paramUid)
    ```   

2.  updateUserFunction中代码
    ```
    update user 
    set xxx = yyyy
    where uid = paramUid

    ```
3. 使用这个移行工具时，移行出的代码
   procedure executeUpdateUser
    ```
    create procedure executeUpdateUser
        call updateUserFunction(paramUid)
        call updateUserInfoFunction(paramUid)
        call updateUserDeptFunction(paramUid)
    ```
    FUNCTION updateUserFunction
    ```
    create FUNCTION updateUserFunction
    BEGIN 
        EXECUTE master.dbo.xp_ora2ms_exec2_ex 
            @active_spid, 
            @login_time, 
            @db_name, 
            N'SAI16', 
            N'updateUserFunction$IMPL', 
            N'true', 
            @ARG_UID, 
            @return_value_argument  OUTPUT

        RETURN @return_value_argument

    END 

    ```
    procedure updateUserFunction$impl
    ```
    create procedure updateUserFunction$impl
    BEGIN 
        update user 
        set xxx = yyyy
        where uid = paramUid

    END 

    ```
4. 分析死锁
   当java 程序调用 1 procedure executeUpdateUser时会有一个transaction 1 
   获取了user table的lock，然后调用updateUserFunction，因为通过master.dbo.xp_ora2ms_exec2_ex调用，所以产生了另一个transaction 2，在transaction 2中为了执行update user 也需要获取user table的lock，但是已经被transaction 1 获取了，所以等待中，而transaction 1 因为一直在等待updateUserFunction执行完成，才能结束transaction 1 而释放user table的lock，所以造成了死锁。

### 影响
最主要的影响是使用master.dbo.p_ora2ms_exec2_ex接口调用UDF，会产生一个新的transaction，从而容易造成死锁。
- upd lock
    一般是在transaction结束，才会释放。
    同一个transaction中是可以使用同一把锁的。
    即，先调用
    selec * from user with(updlock)
    然后再调用 update user
    是不会有获取锁的问题的。

### procedure
一般在procedure中是不明示transaction的，而是在调用方来管理。

