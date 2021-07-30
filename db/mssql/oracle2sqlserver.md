## 基础类型

### Date

Date类型在Oracle中精确到秒，在Sqlserver中只能精确到天

## rowid， rownum

Oracle中所有的表都有一个共同的字段，rowid这是在物理上存在的，记录了每一条记录的行位置，rownum 是逻辑上的，根据排序方式的不同会出现不同的rownum,

s: ROW_NUMBER() OVER(partition by ttid ORDER BY ttid ASC) as rownum

o: 
```
select * from (select rownum as r,t.* from   
    (select emp.* from emp order by hiredate desc) t where rownum<=10)  
        where r>5;
```
s:
```
select * from (select rownum as r,t.* from
    (select top 10 emp.* from emp order by hiredate desc) t )
        where r>5;
```

## 函数

### 获取系统时间

O: sysdate
S: getdate()

### 日期加减

O: sysdate-1，sysdate+365
S: DATEADD

### to_date,to_char

Sqlserver中没有这样的方法，但是有conver,cast方法这两个方法可以实现Oacle两个方法的所有功能

oracle语句
select * from ZX_ZHONGGRMYXFCGSZX_GRXXJCB where SFZH = '452122199106224513' and XM = 'MMM' 
and ZTM = 9 and (to_char(JZSJ,'yyyy-MM-dd') >= '2015-03-27' or JZSJ IS NULL)

sqlserver语句
select * from ZX_ZHONGGRMYXFCGSZX_GRXXJCB where SFZH = '452122199106224513' and XM = 'MMM' 
and ZTM = 9 and (CONVERT(VARCHAR(10),JZSJ,120) >= '2015-03-27' or JZSJ IS NULL)

### NVL

S:select F1,IsNull(F2,10) value from Tbl

O:select F1,nvl(F2,10) value from Tbl

### 取整(大)

S:select ceiling(-1.001) value

O:select ceil(-1.001) value from dual

### 取整(截取)

S:select cast(-1.002 as int) value

O:select trunc(-1.002) value from dual

### 取e为底的对数
S:select log(2.7182818284590451) value 1

O:select ln(2.7182818284590451) value from dual; 1

### 取平方
S:select SQUARE(4) value 16

O:select power(4,2) value from dual 16

### 取随机数

S:select rand() value

O:select sys.dbms_random.value(0,1) value from dual;

### 求集合最大值

S:
```
select max(value) value from

(select 1 value

union

select -2 value

union

select 4 value

union

select 3 value)a

```
O:select greatest(1,-2,4,3) value from dual

### 求集合最小值

S:
```
select min(value) value from

(select 1 value

union

select -2 value

union

select 4 value

union

select 3 value)a

```
O:select least(1,-2,4,3) value from dual

### 从序号求字符

S:select char(97) value

O:select chr(97) value from dual

### 连接字符

S:select '11 '+ '22 '+ '33 ' value

O:select CONCAT( '11 ', '22 ')||33 value from dual

### 连接

Oracle用|| 符号作为连接符，而SQL Server的连接符是加号：+ 。
　　
Oracle查询如下所示：
Select ‘Name’ || ‘Last Name’ From tableName
　　
对应的SQL Server查询如下所示：
Select ‘Name’ + ‘Last Name’

### LENGTH和LEN

以下是Oracle的查询：
SELECT LENGTH('SQLMAG') \"Length in characters\" FROM DUAL;
　　
以上查询在SQL Server下是这样写的：
SELECT LEN('SQLMAG') \"Length in characters\" 

### 数据减月

以下代码在Oracle中直接对数据进行减法操作：
　SELECT sysdate - add_months(sysdate,12) FROM dual
　　
SQL Server则是这样做的：
　SELECT　datediff(dd, GetDate(),dateadd(mm,12,getdate()))

### 子串位置 --返回3

S:select CHARINDEX( 's ', 'sdsq ',2) value

O:select INSTR( 'sdsq ', 's ',2) value from dual

### 求子串

S:select substring( 'abcd ',2,2) value

O:select substr( 'abcd ',2,2) value from dual

### 子串代替 返回aijklmnef

S:SELECT STUFF( 'abcdef ', 2, 3, 'ijklmn ') value

O:SELECT Replace( 'abcdef ', 'bcd ', 'ijklmn ') value from dual

## order by

在sqlserver需要添加top()函数

## from dual

s:
```
select abs(-1) value
```
o:
```
select abs(-1) value from dual
```

### 单表更新别名

O:update work_bill wb set wb.getworkgroup='1' where wb.billid='1'

S:update work_bill  set getworkgroup='1' where billid='1'

sql执行单表更新时 不需要加别名直接写列名字就可以

### a.id=b.id(+)
O: select a.*,b.* from a,b where a.id=b.id(+),
S: select a.*,b.* from a left join b on a.id=b.id

### for update

O: SELECT * FROM ANKEN where ANKEN_ID = ? FOR UPDATE NOWAIT
S: SELECT * FROM ANKEN WITH (UPDLOCK) where ANKEN_ID = ?


# sql translate tool

SwisSQL
https://www.jooq.org/translate/