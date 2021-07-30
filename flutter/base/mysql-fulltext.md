```sql
CREATE TABLE articles (
    id INT auto_increment PRIMARY KEY,
    title VARCHAR (200),
    body text,
    FULLTEXT (title, body) WITH PARSER ngram
);
```
```sql
insert into t_articles(title,body) values
('数据库管理','专业课'),
('数据库','专业课'),
('计算机操作系统','专业课'),
('MySQL','专业课'),
('MySQL数据库','专业课'),
('shuju','this is shuju'),
('shujuku','this is shujuku'),
('MySQL Tutorial','DBMS stands for DataBase...'),
('How To Use MySQL Well','After you went through a ...'),
('Optimizing MySQL','In this tutorial we will show...'),
('1001 MySQL Tricks','1.Never run mysqld as root. 2 . ...'),
('MySQL vs. YourSQL','In the following database comparision...'),
('Tuning DB2','For IBM database ...'),
('IBM History','DB2 history for IBM ...');
```

```sql
select * from articles where match(title,body) against('MySQL数据库' in natural language mode); 
```

```sql
select * from articles where match(title,body) against('MySQL数据库' in natural language mode); 
```

```sql
select * from articles where match(title,body) against('+数据 +管理' in boolean mode);
```

```sql
select * from shop where match(shop_name,body) against('+東京 +ヤマダ' in boolean mode);
-- select * from shop where match(shop_name,body) against('東京 ヤマダ' in natural language mode);
```