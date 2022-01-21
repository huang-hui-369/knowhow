# sql_variant Data Type

sql_variant 声明的变量可以存放任何类型的变量。可以通过SQL_VARIANT_PROPERTY来判断变量类型。

```sql
declare @now datetime
set @now = getdate() 
SELECT  SQL_VARIANT_PROPERTY(@now, 'BaseType') 

-- datetime
```

