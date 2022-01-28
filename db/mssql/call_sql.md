# create_all_porcedure.sql

```sql
:r D:\project\db\scripts\USE_DATABASE.sql
:r D:\project\db\scripts\PROCEDURE_DDL\PROCEDURE1.sql
:r D:\project\db\scripts\PROCEDURE_DDL\PROCEDURE2.sql
:r D:\project\db\scripts\PROCEDURE_DDL\PROCEDURE3.sql
:r D:\project\db\scripts\PROCEDURE_DDL\PROCEDURE4.sql
```



# create_all_procedure.bat

```shell
rem @echo off

set DB_HOST=localhost
set DB_DB=SAITAMA
set DB_USR=SAI16
set DB_PWD=SAI16


set SQL_FILE=D:\project\db\scripts\create_all_porcedure.sql
set LOG_FILE=D:\project\output\log\create_all_porcedure.log


echo import procedure from %SQL_FILE%
sqlcmd -S %DB_HOST% -d %DB_DB% -U %DB_USR% -P %DB_PWD% -i %SQL_FILE% -o %LOG_FILE%
 
```

