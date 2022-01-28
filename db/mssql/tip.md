# sql_variant Data Type

sql_variant 声明的变量可以存放任何类型的变量。可以通过SQL_VARIANT_PROPERTY来判断变量类型。

```sql
declare @now datetime
set @now = getdate() 
SELECT  SQL_VARIANT_PROPERTY(@now, 'BaseType') 

-- datetime
```



# EXEC sp_msforeachtable

? 相當於 table name 帶 shema name

EXEC sp_msforeachtable "ALTER TABLE ? NOCHECK CONSTRAINT ALL";
EXEC sp_MSForEachTable 'ALTER TABLE ? DISABLE TRIGGER ALL'