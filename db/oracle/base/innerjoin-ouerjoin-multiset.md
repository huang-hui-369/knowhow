# join,left join,right join，full out join 区别

![](img\2021-06-24-20-18-32.png)

# inner join

```sql
SELECT column_name(s)
FROM table1
INNER JOIN table2
ON table1.column_name = table2.column_name;
```
# LEFT (OUTER) JOIN

```
SELECT column_name(s)
FROM table1
LEFT JOIN table2
ON table1.column_name = table2.column_name;
```

# RIGHT (OUTER) JOIN

```
SELECT column_name(s)
FROM table1
RIGHT JOIN table2
ON table1.column_name = table2.column_name;
```

# FULL OUTER JOIN

```
SELECT column_name(s)
FROM table1
FULL OUTER JOIN table2
ON table1.column_name = table2.column_name
WHERE condition;
```

**table data**

![](img\2021-06-24-20-37-17.png)

**sql语句**
```
SELECT a.id , a.value , b.id AS Expr1, b.value AS Expr2
FROM   a
       FULL OUTER JOIN b
       ON     a.id = b.id
```

**结果**
![](img\2021-06-24-20-38-40.png)

就是将两个表中包含id的所有记录都显示出来。

# cross join

就是将两个table的记录的所有组合都select出来

```
select *
from tableA,tableB
```
或
```
select * from tableA
cross join tableB
```

![](img\2021-06-24-20-45-31.png)

# multiset

从字面就可以看出set就是一个集合，而且允许数据重复。

我们可以将表的一个子集转换为multiset使用。

MULTISET UNION:取得两个嵌套表的并集 结果集中会包含重复值，不去重；

MULTISET UNION DISTINCT: 取得两个嵌套表的并集 并取消重复结果,去重；
MULTISET INTERSECT:用于取得两个嵌套表的交集；
NULTISET EXCEPT:用于取得两个嵌套表的差集 在A集合中存在,但在B集合中不存在。

## oracle中multiset的用法
要使用multiset就要创建一个TYPE

### 1. 事前准备创建表和数据

1. 创建一个制造商表，就定义了制造商id和name
   ![](img\2021-06-25-12-20-40.png)
    ```sql
    CREATE TABLE manufacturer (
    manufacturer_id INT,
    manufacturer_name VARCHAR(100)
    );

    -- 插入数据
    INSERT INTO manufacturer (manufacturer_id, manufacturer_name) VALUES
    (
    1, 'micro soft'
    );

    INSERT INTO manufacturer (manufacturer_id, manufacturer_name) VALUES
    (
    2, 'apple'
    );

    INSERT INTO manufacturer (manufacturer_id, manufacturer_name) VALUES
    (
    3, 'google'
    );
    ```
2. 创建一个产品表、产品id，产品名，制造商
   ![](img\2021-06-25-12-22-14.png)
    ```
    CREATE TABLE product (
    product_id INT,
    product_name VARCHAR(100),
    manufacturer_id INT
    );

    INSERT INTO product (product_id, product_name, manufacturer_id) VALUES
    (1, 'Chair', 1);
    INSERT INTO product (product_id, product_name, manufacturer_id) VALUES
    (2, 'Table', 1);
    INSERT INTO product (product_id, product_name, manufacturer_id) VALUES
    (3, 'Couch', 2);
    INSERT INTO product (product_id, product_name, manufacturer_id) VALUES
    (4, 'Desk', 2);
    INSERT INTO product (product_id, product_name, manufacturer_id) VALUES
    (5, 'Dining Table', 1);

    ```

### 2. create tyep product_t 相当于product table

1. create TYPE_DEMO_PRODUCT用来定义多个列名
    ```
    CREATE OR REPLACE TYPE TYPE_DEMO_PRODUCT AS OBJECT 
    ( /* TODO enter attribute and method declarations here */ 
        product_id VARCHAR2(10)
        ,product_name VARCHAR2(255)
    ,manufacturer_id VARCHAR2(10)
    )
    ```

2. create tyep product_t 用来定义一个相当于多行多列的table
    ```
    create or replace TYPE product_t AS TABLE OF TYPE_DEMO_PRODUCT;
    ```

### 3. 将product表的一个子集当做multiset使用结合制造商表显示

![](img\2021-06-25-12-35-16.png)

multiset sql语句
```
SELECT
m.manufacturer_id,
m.manufacturer_name,
prod.*
FROM manufacturer m, 
TABLE(
  CAST(
    MULTISET(
      SELECT
      product_id,
      product_name,
      manufacturer_id
      FROM product
      where product.MANUFACTURER_ID = m.MANUFACTURER_ID
    ) as product_t
  )
) (+) prod;
```
显示结果
![](img\2021-06-25-12-47-11.png)

从结果来看这就像是一个left join,如果没有(+)就相当于inner join

left join sql语句
```
SELECT
*
FROM manufacturer m 
left join (SELECT
      product_id,
      product_name,
      manufacturer_id
      FROM product
      -- where product.MANUFACTURER_ID = m.MANUFACTURER_ID
    )  prod
on m.manufacturer_id = prod.manufacturer_id;
```
这和上面的结果是一样的。
![](img\2021-06-25-12-52-29.png)

那为什么要用cast mulitiset呢？
针对上面的例子来说确实没有必要用。
1. 是因为有MULTISET UNION，NULTISET EXCEPT等用法
2. cast mulitiset通常是定义一个函数返回一个table或者table的子集，这要调用函数时必须将返回值cast 成 mulitiset。

# cross apply
如果两个表A，B 有A.id = B.id这样的条件的话，就相当于inner join
如果没有条件的话就相当于cross join

```
SELECT
m.manufacturer_id,
m.manufacturer_name,
prod.*
FROM manufacturer m
CROSS APPLY 
(
      SELECT
      product_id,
      product_name,
      manufacturer_id
      FROM product
      where product.MANUFACTURER_ID = m.MANUFACTURER_ID
    )  
    prod; 
```

cross apply 主要在于不需要join的on结合条件也可以将两个表join

```
SELECT
m.manufacturer_id,
m.manufacturer_name,
prod.*
FROM manufacturer m
CROSS APPLY 
(
  select * from(
      SELECT
      product_id,
      product_name,
      manufacturer_id
      FROM product
      where product.MANUFACTURER_ID = m.MANUFACTURER_ID
    )  
  where ROWNUM<3
)  prod; 
```
<font style='color:orangered'>
综上所述 cross apply 主要有两种用途</br>
1. 根据top n的结果，将两个table join结合起来。</br>
2. 调用函数时想使用inner join的时候。
</font>
 

# outer apply
就相当于out join

```
SELECT
m.manufacturer_id,
m.manufacturer_name,
prod.*
FROM manufacturer m
OUTER APPLY 
(
      SELECT
      product_id,
      product_name,
      manufacturer_id
      FROM product
      where product.MANUFACTURER_ID = m.MANUFACTURER_ID
    )  
    prod; 
```

有了 cross join为什么还要cross apply，outer apply
个人猜测也是用来补充调用一个函数返回一个table子表时所进行的join 结合操作的需要，
因为现有的inner join是不能支持调用一个函数的。
