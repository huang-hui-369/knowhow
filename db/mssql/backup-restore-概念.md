# 数据库文件构造

- 数据库数据文件
- 数据库操作log文件

当我们执行insert，delete，update时，会先将操作写入log文件，然后将数据反映到数据库文件中。

# 数据库备份主要类型

- full backup 

  备份现在时点的数据和log文件

- differential backup 

  备份上一次完全备份后的差分数据,差异备份本质上是累积备份。

- log backup

  它允许将数据库恢复到特定的时间点， 这意味着事务日志备份是增量备份，备份自上一次备份log文件时间点后的增量log文件。

- Tail log backups

  在上次进行了log backup后，如果数据库发生了故障，那意味着上次备份log的时间点到发生故障时间点之间的操作数据都没有进行备份，如果想要恢复这段时间的数据，必须进行Tail log backup，这个有可能失败，失败的话那么这段时间的数据就会丢失。

  

首先full backup是基础，你必须有了full backup后才可以有differential backup，log backup

![image-20220309180628861](D:\github\knowhow\db\mssql\backup-restore-概念.assets\image-20220309180628861.png)

# 一个数据库备份计划样例

如下图，

我们可以每周进行一次full backup

每天进行differential backup

每小时进行log backup

这样的话我们可以最大限度的恢复数据。

![image-20220309182605298](D:\github\knowhow\db\mssql\backup-restore-概念.assets\image-20220309182605298.png)



# 一个完整的数据库备份恢复例子 -- sql

- ## 备份数据库

```sql

-- 1 full back up to new MediaSet

BACKUP DATABASE [demo] TO  DISK = N'D:\tmp\demo_week_backup_set.bak' 
WITH FORMAT, -- 此选项用于指定是否覆盖媒体头信息，FORMAT覆盖任意现有备份并创建MediaSet，NOFORMAT将保留所有信息
INIT -- INIT用于创建新的备份集，NOINIT用于将备份附加到现有备份集
-- MEDIANAME = 'demo_week_backup_MediaSet';  

GO

-- 2 differential back up to exist MediaSet
BACKUP DATABASE [demo] TO  DISK = N'D:\tmp\demo_week_backup_set.bak' 
WITH  DIFFERENTIAL , -- 差分备份 
NOFORMAT,    -- 追加到现有 MediaSet
NOINIT       -- 将备份追加到现有backup set
-- MEDIANAME = 'demo_week_backup_MediaSet';  
-- SKIP,      -- skip参数用于跳过备份集上的到期检查
-- NOREWIND,  -- 此参数用于使磁带设备保持打开状态并准备使用 
-- NOUNLOAD,  -- 此参数用于指示SQL Server在备份操作完成后不要从驱动器中卸载磁带
-- STATS = 10 -- STATS选项可用于在备份进度的常规阶段获取备份操作的状态。

GO

-- 3 log back up to exist MediaSet
BACKUP LOG [demo] TO  DISK = N'D:\tmp\demo_week_backup_set.bak' 
WITH NOFORMAT, 
NOINIT
-- MEDIANAME = 'demo_week_backup_MediaSet';    
-- SKIP, 
-- NOREWIND, 
-- NOUNLOAD,  
-- STATS = 10
GO


-- 4 tail log back up to exist MediaSet
BACKUP LOG demo  
TO DISK = N'D:\tmp\demo_week_backup_set.bak' 
   WITH CONTINUE_AFTER_ERROR -- 当您要备份受损数据库的尾部时，必须使用 CONTINUE_AFTER_ERROR。
 
, NORECOVERY      每当您准备对数据库继续执行还原操作时，请使用 NORECOVERY。 NORECOVERY 使数据库进入还原状态。
-- , NO_TRUNCATE     除非数据库受损，否则不建议使用 NO_TRUNCATE，不需要NO_TRUNCATE

GO



```

- ## 还原数据库

```sql
-- 1. Restore the full database backup (from backup set 1).  
RESTORE DATABASE demo   
  FROM DISK = N'D:\tmp\demo_week_backup_set.bak' 
  WITH FILE=1,   
    NORECOVERY;  

GO

-- 2. Restore the diffrential database backup (from backup set 2).  
RESTORE DATABASE demo   
  FROM DISK = N'D:\tmp\demo_week_backup_set.bak' 
  WITH FILE=2,   
    NORECOVERY;  

GO

-- 3. Restore the regular log backup (from backup set 3).  
RESTORE LOG demo   
  FROM DISK = N'D:\tmp\demo_week_backup_set.bak' 
  WITH FILE=3,   
    NORECOVERY;  
  
--4. Restore the tail-log backup (from backup set 4).  
RESTORE LOG demo   
  FROM DISK = N'D:\tmp\demo_week_backup_set.bak'
  WITH FILE=4,   
    NORECOVERY;  
GO  
--5. recover the database:  
RESTORE DATABASE demo WITH RECOVERY;  
GO 


```

- ## 再备份数据库Log

  ```sql
  -- 3 log back up to exist MediaSet
  BACKUP LOG [demo] TO  DISK = N'D:\tmp\demo_week_backup_set.bak' 
  WITH NOFORMAT, 
  NOINIT
  -- MEDIANAME = 'demo_week_backup_MediaSet';    
  -- SKIP, 
  -- NOREWIND, 
  -- NOUNLOAD,  
  -- STATS = 10
  GO
  ```

- ## 确认备份文件

  发现 -- 针对损坏的数据库进行结尾日志备份，file4不见了， 取代的是file5

![image-20220309210539583](D:\github\knowhow\db\mssql\backup-restore-概念.assets\image-20220309210539583.png)

# 一个完整的数据库备份恢复例子 -- GUI

我们可以每周进行一次full backup，每天进行differential backup，每小时进行log backup然后将这些一周的备份数据全部保存到一个backup set文件中demo_week_backup_set.bak。

- ## 数据库备份

  1. ###  full backup

     step1 -- 选中demo数据库右键 --》task --》 backup

     ![image-20220309183430918](D:\github\knowhow\db\mssql\backup-restore-概念.assets\image-20220309183430918.png)

     step2 -- 全般界面中

     ![image-20220309183304088](D:\github\knowhow\db\mssql\backup-restore-概念.assets\image-20220309183304088.png)

     step3  --  new backup set

     ![image-20220309184652342](D:\github\knowhow\db\mssql\backup-restore-概念.assets\image-20220309184652342.png)

     step4  --  backup set中追加

     ![image-20220309183952375](D:\github\knowhow\db\mssql\backup-restore-概念.assets\image-20220309183952375.png)

  2.  differential backup

     ![image-20220309211335908](D:\github\knowhow\db\mssql\backup-restore-概念.assets\image-20220309211335908.png)

     ![image-20220309211428189](D:\github\knowhow\db\mssql\backup-restore-概念.assets\image-20220309211428189.png)

  3. log backup

     ![image-20220309211523514](D:\github\knowhow\db\mssql\backup-restore-概念.assets\image-20220309211523514.png)

     ![image-20220309211556121](D:\github\knowhow\db\mssql\backup-restore-概念.assets\image-20220309211556121.png)

     

- ## 模拟数据库出错

  1. stop sql server

     ![image-20220309212016780](D:\github\knowhow\db\mssql\backup-restore-概念.assets\image-20220309212016780.png)

  2. 删除demo数据库文件

     ![image-20220309212125349](D:\github\knowhow\db\mssql\backup-restore-概念.assets\image-20220309212125349.png)

  3. 启动 sql server

  

- ## ssms 数据库恢复

  1. 确认demo数据库不可用

     ![image-20220309193050162](D:\github\knowhow\db\mssql\backup-restore-概念.assets\image-20220309193050162.png)

  2. 末尾Log备份

     只能用sql 来备份

     ```sql
     BACKUP LOG demo  
     TO DISK = N'D:\tmp\demo_week_backup_set.bak' 
        WITH CONTINUE_AFTER_ERROR -- 当您要备份受损数据库的尾部时，必须使用 CONTINUE_AFTER_ERROR。
     , NORECOVERY      每当您准备对数据库继续执行还原操作时，请使用 NORECOVERY。 NORECOVERY 使数据库进入还原状态。
     -- , NO_TRUNCATE     除非数据库受损，否则不建议使用 NO_TRUNCATE，不需要NO_TRUNCATE
     ```

  3. ssms 恢复demo数据库

     ![image-20220309212919840](D:\github\knowhow\db\mssql\backup-restore-概念.assets\image-20220309212919840.png)

     ![image-20220309213249742](D:\github\knowhow\db\mssql\backup-restore-概念.assets\image-20220309213249742.png)



# 为磁盘文件定义逻辑备份设备

![image-20220309213718492](D:\github\knowhow\db\mssql\backup-restore-概念.assets\image-20220309213718492.png)

![image-20220309213941419](D:\github\knowhow\db\mssql\backup-restore-概念.assets\image-20220309213941419.png)

这样以后备份或恢复时只要只指定device名就可以了

![image-20220309214203604](D:\github\knowhow\db\mssql\backup-restore-概念.assets\image-20220309214203604.png)

# 完整备份+log备份

![image-20220309214454836](D:\github\knowhow\db\mssql\backup-restore-概念.assets\image-20220309214454836.png)

1. 备份

   ```sql
   USE demo;  
   ALTER DATABASE demo SET RECOVERY FULL;  
   GO  
   -- Back up the demo database to new media set (backup set 1). 
   -- 创建数据库的完整数据库备份
   BACKUP DATABASE demo  
     TO DISK = 'd:\SQLServerBackups\DemoFull_data.bak'   
     WITH FORMAT;  
   GO  
   --Create a routine log backup (backup set 2).  
   -- 创建日志备份
   BACKUP LOG demo TO DISK = 'Z:\SQLServerBackups\DemoFull_log.bak';  
   GO  
   ```

2. 恢复

   ```sql
   USE demo;  
   -- Step 1: Create a tail-log backup by using WITH NORECOVERY. 
   BACKUP LOG demo   
   TO DISK = 'Z:\SQLServerBackups\DemoFull_log.bak'    
      WITH NORECOVERY;   
   GO  
   -- Step 2: Restore the full database backup. 
   RESTORE DATABASE demo   
     FROM DISK = 'Z:\SQLServerBackups\DemoFull_data.bak'   
     WITH NORECOVERY;  
     
   -- Step 3: Restore the transaction log backup.
   RESTORE LOG demo   
     FROM DISK = 'Z:\SQLServerBackups\DemoFull_log.bak'   
     WITH NORECOVERY;  
     
   -- Step 4: Restore the tail-log backup. 
   RESTORE LOG demo   
     FROM DISK = 'Z:\SQLServerBackups\DemoFull_log.bak'  
     WITH NORECOVERY;  
   GO  
   -- Step 5: Recover the database. 
   RESTORE DATABASE demo WITH RECOVERY;  
   GO  
   ```

   

# 缩小日志文件

1. 首先备份fullback

2. 备份log文件的同时会truncate log

3. 查看log使用情况

   ```sql
   use saitama;
   dbcc sqlperf('logspace')
   ```

   查看到saitama log 使用了751M

   ![image-20220309175043151](D:\github\knowhow\db\mssql\backup-restore-概念.assets\image-20220309175043151.png)

4. 确认log文件名

```sql
use saitama;
select * from sysfiles;
dbcc shrinkfile(dbName_log,100)
```

确认到log文件名为 SAITAMA1_log

![image-20220309175422242](D:\github\knowhow\db\mssql\backup-restore-概念.assets\image-20220309175422242.png)

5. 缩小日志文件

   ```sql
   use saitama;
   dbcc shrinkfile(SAITAMA1_log,100); -- 缩小到100M
   ```

6. 查看log文件大小缩小到150M

   ![image-20220309175933963](D:\github\knowhow\db\mssql\backup-restore-概念.assets\image-20220309175933963.png)

   

# 媒体集，备份集

## 媒体集

*媒体集* 是 *备份媒体*（磁带或磁盘文件，或者是 Azure Blob）的有序集合

例如：与媒体集相关联的备份设备可能是三个名为 `\\.\TAPE0`、`\\.\TAPE1` 和 `\\.\TAPE2` 的磁带驱动器

或者 DISK = 'f:\dbbackup\dbdemo_1.BAK', DISK = 'f:\dbbackup\dbdemo_2.BAK', DISK = 'f:\dbbackup\dbdemo_3.BAK',



## 备份集

成功的备份操作将向介质集中添加一个“备份集” 。

例如，先备份full back，就是一个backup set1，

![image-20220309230513627](D:\github\knowhow\db\mssql\backup-restore-概念.assets\image-20220309230513627.png)

​          再备份差分back，就是一个backup set2，



![image-20220309230549114](D:\github\knowhow\db\mssql\backup-restore-概念.assets\image-20220309230549114.png)