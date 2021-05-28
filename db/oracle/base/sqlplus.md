# 连接oracle数据库

sqlplus [username]/[passowrd]@[IP Addr]:[Port Number]/[Service Name]

sqlplus [username]/[user_password]@ORCL

```
sqlplus saitama/Grandunit123@192.168.1.10:1521/orclpdb
sqlplus saitama/Grandunit123@orcl
```

### sqlplus log

```
spool c:\work\oraclelog.txt

spool off

set linesize 999

set pagesize 999
```

## linux console 文字化け

```
export NLS_LANG=Japanese_Japan.AL32UTF8
```

## 导入数据乱码 确认oracle 字符编码

### 1. 确认导出文件的字符字符编码

### 2. oracle 服务器端 字符编码

```
SQL> SELECT
  2    *
  3  FROM
  4    NLS_DATABASE_PARAMETERS
  5  WHERE
  6    PARAMETER='NLS_CHARACTERSET';

PARAMETER
--------------------------------------------------------------------------------
VALUE
--------------------------------------------------------------------------------
NLS_CHARACTERSET
JA16SJIS
```

### 3. oracle client端 字符编码

* linux 确认环境变量
```
echo $NLS_LANG
```

* windows  确认环境变量 和注册便

HKEY_LOCAL_MACHINE\SOFTWARE\ORACLE\KEY_Oraclient11g_home1の中の「NLS_LANG」を確認します

![](img\2021-05-25-15-28-16.png)


# 执行 SQL 文件

使用 SQL PLUS 命令

```
$ sqlplus [username]/password@[orcl] @path/file.name
```

# Sql Loader

```
sqlldr dbuser/dbpass@dbservice control=users.ctl
```

# 查看基本信息

### Oracleバージョンの確認方法

```
select * from v$version;


```

### 查看数据库名

```
SQL> select name from v$database;

NAME
---------
ORCL
```

### 查询当前数据库实例名

```
SQL> select instance_name from v$instance;

INSTANCE_NAME
----------------
orcl
```

### 查看当前用户

```
SQL> show user
ユーザーは"SAITAMA"です。
```

### 查看数据库用户

```
select * from dba_users;  
```

### 解锁用户

```
$ sqlplus / as sysdba
SQL> alter user [username] account unlock;

SQL> alter user [username] account lock;
```

### 设置密码过期

```
SQL> alter user [username] password expire;
```

### 修改密码

```
# 修改当前登录用户密码：

SQL> password

# 修改某个用户的密码：

SQL> alter user [username] identified by [password];
```

### 查看用户所拥有的表

```
# 查看用户所拥有的表：

SQL> SELECT TABLE_NAME FROM USER_TABLES; 

# 查看用户可存取的表：

SQL> SELECT TABLE_NAME FROM ALL_TABLES; 

# 数据库中所有表：

SQL> SELECT TABLE_NAME FROM DBA_TABLES;
```

### 查看表空间

```
SQL> SELECT FILE_NAME,TABLESPACE_NAME from DBA_DATA_FILES;
```

### 创建新用户

CREATE USER [用户名]  
IDENTIFIED BY [密码]  
DEFAULT TABLESPACE [表空间] (默认USERS)  
TEMPORARY TABLESPACE [临时表空间] (默认TEMP)  
例如：

```
SQL> create USER software
  2  identified by 123456
  3  default tablespace Softwaremanagement;
分配权限
SQL> GRANT CONNECT TO [username];  
SQL> GRANT RESOURCE TO [username];  
SQL> GRANT DBA TO [username];  -- DBA为最高级权限，可以创建数据库、表等。
```

### 删除用户，及所拥有的表。。。

```
drop user SAI15 cascade;
```


## oracle中的权限

Oracle 中的权限分为两类：

系统权限：系统规定用户使用数据库的权限，系统权限是对用户而言。
实体权限：某种权限的用户对其他用户的表或视图的存取权限，是针对表或者视图而言。如 select、update、insert、delete、alter、index、all，其中 all 包含所有的实体权限。

### 系统权限分类
DBA：拥有全部特权，是系统最高权限，只有DBA才可以创建数据库结构。
RESOURCE：拥有resource权限的用户只可以创建实体，不可以创建数据库结构。
CONNECT：拥有connect权限的用户只可以登录oracle，不可以创建实体，不可以创建数据库结构。
建议： 对于普通用户，授予 CONNECT、RESOURCE 权限； 对于 DBA管理用户，授予 CONNECT、RESOURCE、DBA 权限。

## 导入导出
数据库的导入导出也是一个很常见的需求。

### 导出

```
$ exp [username]/[password]@[orcl] file=./database.dmp  full=y

```

username 是数据库用户名
password 是数据库用户密码
orcl 是数据库实例名称
file 后面的参数是导出的数据库文件存放位置及文件名
full 其值为 y 表示全部导出，默认为 no。
如果只需导出某几张表，可以指定 tables 参数：tables='(tableName, tableName1)'。

### 导入
```
$ imp [username]/[password]@[orcl] file=./database.dmp

```
和导出数据库语法一样，只是关键字不一样。

