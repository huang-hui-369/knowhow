# create user error in ora12c

ora12cにsystem　or sysでcreate user実行してエラーが出て来ました。

## 現象
```
ORA-65096: invalid common user or role name
Cause: An attempt was made to create a common user or role with a name that wass not valid for common users or roles. In addition to the usual rules for user and role names, common user and role names must start with C## or c## and consist only of ASCII characters.
Action: Specify a valid common user or role name.
```

## 解決策
```
alter session set "_Oracle_SCRIPT"=true;
```

# impdp

```sh
# create directory
create or replace directory EXP_DIR as 'D:/project/demo/ORACLE_EXP';
# 権限を付与
grant read, write on directory TEST_DIR to SCOTT;

impdp SYSTEM/SYSTEM123@orcl schemas=SCOTT dumpfile=DDL:EXP_DB.DMP logfile=DDL:impdp.log
# OR
impdp SYSTEM/SYSTEM123@orcl schemas=SCOTT directory=SAITAMA_EXP_DIR dumpfile=EXP_DB.DMP logfile=impdp.log
```

# execute imp by sysdba

```
imp 'sys/sys as sysdba' file=/oracle/implog.dmp log=imp.log full=y
```