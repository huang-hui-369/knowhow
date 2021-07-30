# create user
```
--ユーザ削除
DROP USER "demo1" CASCADE;

-- ユーザ作成
CREATE USER "demo1" PROFILE "DEFAULT" IDENTIFIED BY "demo1" DEFAULT TABLESPACE "USERS" TEMPORARY TABLESPACE "TEMP" ACCOUNT UNLOCK;
GRANT "CONNECT" TO "demo1";
GRANT "RESOURCE" TO "demo1";
GRANT "SELECT_CATALOG_ROLE" TO "demo1";

-- ユーザに表領域USERSへの権限を与える
ALTER USER "demo1" QUOTA UNLIMITED ON USERS;
```

# create 12c user without c## prefix

```
connect system/manager as sysdba

# set without c## prefix
alter session set "_ORACLE_SCRIPT"=true;

# create user c##fred identified by flintstone;
create user fred identified by flintstone;

grant dba to pubs;

connect fred/flintstone
```

# 12c user

![](img\2021-07-20-18-56-17.png)

![](img\2021-07-20-18-56-52.png)


A common user can log in to any container (including CDB$ROOT) in which it has the CREATE SESSION privilege.

