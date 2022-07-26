# datetime的差值

```sql
-- datetime型相减得到的不是预期的差值7小时，而是datetime型
select CAST ('1970/01/01 7:00:00' AS datetime)- CAST('1970/01/01 0:00:00' AS datetime)
-- 结果：1900-01-01 07:00:00.000
select CAST ('1970/01/01 0:00:00' AS datetime)- CAST('1970/01/01 0:00:00' AS datetime)
-- 结果：1900-01-01 00:00:00.000

-- datetime型相减要用DATEDIFF函数得到差值
-- 小时
select DATEDIFF(HH, CAST('1970/01/01 0:00:00' AS datetime), CAST ('1970/01/01 7:00:00' AS datetime) )
-- 结果： 7 
-- 秒
select DATEDIFF(SS, CAST('1970/01/01 0:00:00' AS datetime), CAST ('1970/01/01 7:00:00' AS datetime) )
-- 毫秒
select DATEDIFF(MS, CAST('1970/01/01 0:00:00' AS datetime), CAST ('1970/01/01 0:00:00' AS datetime) )
-- 结果： 0
```

