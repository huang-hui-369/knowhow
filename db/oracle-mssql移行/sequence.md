# sequence

利用ssma for oracle 将oracle 的sequence转换后，会有个小问题。sys.sequences table的last_used_value为null.

```sql
SELECT s.name,s.start_value,s.increment,s.current_value,s.last_used_value FROM sys.sequences s
join sys.schemas on s.schema_id = schemas.schema_id
where schemas.name = 'SAI16';
```

![image-20220207191658395](D:\github\knowhow\db\oracle-mssql移行\sequence.assets\image-20220207191658395.png)



sql server中执行 select NEXT VALUE FOR SEQ_XXX

 当last_used_value为null   会返回 current_value 

 当last_used_value >0      会返回 current_value + 1



