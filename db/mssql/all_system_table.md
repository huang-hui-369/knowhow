# 获取schema的所有table

```sql
SELECT table_name
FROM INFORMATION_SCHEMA.TABLES
WHERE TABLE_TYPE='BASE TABLE'
and table_schema = 'SAI16' 
```

# 通过sys.sequences获取schema的所有sequence

```sql
SELECT * FROM sys.sequences 
join sys.schemas on sequences.schema_id = schemas.schema_id
where schemas.name = 'SAI16';

-- 如果sequence的current_value的值为200，last_used_value的值为null
-- 获取下一个sequence 会得到为200，同时last_used_value的值被设置为200
select NEXT VALUE FOR SEQ_DATA_RENKEI_LOG_ID;
-- 再调用获取下一个sequence 会得到正常的201
select NEXT VALUE FOR SEQ_DATA_RENKEI_LOG_ID;
```

# 通过sys.sequences获取具体sequence的信息

```sql
SELECT * FROM sys.sequences WHERE name = 'SEQ_SISYUTU_HUTAN_KAMOKU_ID' ;
select NEXT VALUE FOR SEQ_SISYUTU_HUTAN_KAMOKU_ID
```



# 更新sequence的last_used_value为current_value

```sql
UPDATE sys.sequences SET last_used_value = current_value
FROM sys.sequences
join sys.schemas on sequences.schema_id = schemas.schema_id
where schemas.name = 'SAI16';
```



# 系统的外键表sys.foreign_key_columns

```sql
-- 获取table T_PRODUCT 的所有外键
SELECT ROW_NUMBER() OVER (ORDER BY OBJECT_NAME(parent_object_id), clm1.name) as ID,
       OBJECT_NAME(constraint_object_id) as ConstraintName,
       OBJECT_NAME(parent_object_id) as TableName,
       clm1.name as ColumnName, 
       OBJECT_NAME(referenced_object_id) as ReferencedTableName,
       clm2.name as ReferencedColumnName
  FROM sys.foreign_key_columns fk
       JOIN sys.columns clm1 
         ON fk.parent_column_id = clm1.column_id 
            AND fk.parent_object_id = clm1.object_id
       JOIN sys.columns clm2
         ON fk.referenced_column_id = clm2.column_id 
            AND fk.referenced_object_id= clm2.object_id
 --WHERE OBJECT_NAME(parent_object_id) not in ('//tables that you do not wont to be truncated')
 WHERE OBJECT_NAME(referenced_object_id) = 'T_PRODUCT'
 ORDER BY OBJECT_NAME(parent_object_id)
```

