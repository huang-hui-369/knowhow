# CREATE DATABASE No CDB PDB
这样创建的DATABASE 就和orc11创建的是一样的一个instance一个db

![](img\2021-07-21-18-33-40.png)


# CREATE DATABASE CDB PDB基本概念

CDB与PDB是Oracle 12C引入的新特性，在ORACLE 12C数据库引入的多租用户环境（Multitenant Environment）中，允许一个数据库容器（CDB）承载多个可插拔数据库（PDB）。CDB全称为ContainerDatabase，中文翻译为数据库容器，PDB全称为Pluggable Database，即可插拔数据库。在ORACLE 12C之前，实例与数据库是一对一或多对一关系（RAC）：即一个实例只能与一个数据库相关联，数据库可以被多个实例所加载。而实例与数据库不可能是一对多的关系。当进入ORACLE 12C后，实例与数据库可以是一对多的关系。
关于CDB与PDB的关系图

12c中 oracle引入了容器数据库 CDB（container database），和可插拔数据库 PDB（pluggable database）。oracle 将CDB看成一个容器，用来存放数据库。
在CDB中可以有多个PDB，其中存在一个root根容器（PDB$ROOT）、一个种子容器（PDB$SEED）和多个PDBS。所有的PDB共用一个硬件系统资源、sga和pga、redo、临时段、控制文件、参数文件、还原段（还可对每个PDB单独指定）。

- PDB$ROOT：
  根容器用来做所有容器的跟，用来对每个PDB进行统一管理，sqlplus / as  sysdba连接进来默认是连接的根容器，需要切换到其他的PDB容器才可以对单独的PDB操作。其中有 system数据文件、sysaux数据文件、（undo数据文件、temp数据文件、redo、控制文件）。一般不存放生产数据文件

- PDB$SEED：
  种子容器作为插入PDB的模板而存在，每个CDB都有一个种子容器，且不可对其中对象进行修改。其中有 system数据文件、sysaux数据文件、其他数据文件。

- PDB：
  新插入容器，该容器用来存放数据库。其中有 system数据文件、sysaux数据文件、其他数据文件。12c中可以插入多个容器进行统一管理，来减少BDA的工作量。其中的数据库可以插入或拔出。

用户：12c中PDB$ROOT中的普通目录可以通过权限分配来访问一个或多个指定的PDB容器，最大权限用户是sysdba。其中PDB也可单独创建普通用户来管理该容器的数据库。默认创建的sys，system是cdb的用户，登录后默认是登录的cdb$root。

PDB资源管理：12c中将多个数据库运行在一个硬件资源上，CDB性能上得到优化。在CDB中为每个PDB确定使用CPU最低份额，CDB会按照一个PDB份额/分配的总份额数*100%，来保证PDB最低份额数。


CDB与PDB关系图

     COMMON USERS(普通用户)：经常建立在CDB层，用户名以C##或c##开头；

     LOCAL USERS(本地用户)：仅建立在PDB层，建立的时候得指定CONTAINE

![](img\2021-07-20-18-56-17.png)

![](img\2021-07-20-18-56-52.png)

CDB默认是只读的。

https://docs.oracle.com/cd/E96517_01/multi/introduction-to-the-multitenant-architecture.html#GUID-267F7D12-D33F-4AC9-AA45-E9CD671B6F22

https://docs.oracle.com/database/121/CNCPT/cdblogic.htm#CNCPT89257

## create database for cdb pdb

![](img\2021-07-21-18-31-30.png)

创建一个 orcl database时（我认为就是 cdb）会让指定 gloabal database名（默认orcl.docker.internal）
同时也会可选创建一个pdb （默认名字为orclpdb），
同时默认会有一个sid（orcl）
如下图
![](img\2021-07-21-11-01-54.png)

这样创建完database后，

会默认创建一个listener 监听1521
会默认创建一个cdb service orcl.docker.internal
会默认创建一个cdb sid orcl
会默认创建一个pdb service orclpdb.docker.internal
会默认创建一个sysdba 用户 sys
会默认创建一个system 用户 system
sys，system默认都是管理用户，可以连接访问cdb和pdb
会默认会创建一个连接cdb的service名为 orcl.docker.internal的tns。

![](img\2021-07-21-11-14-34.png)

## 起動

默认会起動cdb，但是pdb的状态为mount，必须改为open才可以写入。
须通过执行一下命令

```
sqlplus system/passowrd
alter pluggable database orclpdb open;
```

## 创建orclpdb的tns

还需要创建一个连接orclpdb的service为名为 orclpdb.docker.internal的tns。

![](img\2021-07-21-16-58-08.png)

- listener.ora
     ```
     SID_LIST_LISTENER =
     (SID_LIST =
     (SID_DESC =
          (SID_NAME = CLRExtProc)
          (ORACLE_HOME = D:\app\oracle\product\12.2.0\dbhome_1)
          (PROGRAM = extproc)
          (ENVS = "EXTPROC_DLLS=ONLY:D:\app\oracle\product\12.2.0\dbhome_1\bin\oraclr12.dll")
     )
     )

     LISTENER =
     (DESCRIPTION_LIST =
     (DESCRIPTION =
          (ADDRESS = (PROTOCOL = TCP)(HOST = host.docker.internal)(PORT = 1521))
          (ADDRESS = (PROTOCOL = IPC)(KEY = EXTPROC1521))
     )
     )

     ```

- tnsnames.ora
     ```
     ORCL =
     (DESCRIPTION =
          (ADDRESS_LIST =
               (ADDRESS = (PROTOCOL = TCP)(HOST = host.docker.internal)(PORT = 1521))
          )
          (CONNECT_DATA =
               (SERVER = DEDICATED)
               (SERVICE_NAME = orcl.docker.internal)
          )
     )  

     ORCLPDB =
     (DESCRIPTION =
          (ADDRESS_LIST =
               (ADDRESS = (PROTOCOL = TCP)(HOST = host.docker.internal)(PORT = 1521))
          )
          (CONNECT_DATA =
               (SERVER = DEDICATED)
               (SERVICE_NAME = orclpdb.docker.internal)
          )
     )        
     ```

## 连接db
是连接到cdb还是pdb，取决于连接的service name或者tns。

## 通过jdbc 直接连接

### sid 连接
sid 只能连接到cdb。
```
jdbc:oracle:thin:@192.168.0.66:1521:orcl
```

### service 连接
**注意端口号后的：换成了/**
```
jdbc:oracle:thin:@192.168.0.66:1521/orclpdb.docker.internal
```

## 通过sql developer 直接连接

### sid 连接
![](img\2021-07-21-11-27-17.png)

### service 连接
![](img\2021-07-21-11-26-18.png)


## pdb sql cmd

1. 通过sql plus 连接Oracle
     sqlplus 可以通过tns连接，也可以通过service name连接

     ```sql
     -- system 通过tns连接cdb 一般就用这个连接cdb拥有足够的权限操作通常的命令
     sqlplus system/password@orcl
     -- sysdba 通过tns连接cdb
     sqlplus sys/password as sysdba@orcl

     -- system 通过service name连接pdb 一般就用这个连接cdb拥有足够的权限操作通常的命令
     sqlplus system/password@192.168.0.8:1521/orclpdb.docker.internal
     ```

2. 接続先のコンテナ名を確認します。
     ``` sql
     -- 连接到pdb
     show con_name;
     con_name SAITAMAPDB 
     -- 连接到cdb
     show con_name;
     con_name CDB$ROOT
     ```

3. PDBの名前とopen_modeがREAD WRITEとなっているかを確認
     ```sql
     select name, open_mode from v$pdbs;
     -- もしくは --
     show pdbs;
     ```

4. open pdb

     ```sql
     alter pluggable database orclpdb open;
     -- もしくは --
     alter pluggable database all open;
     ```

5. 切换到指定容器 

     ```
     alter session set container = orclpdb;    
     alter session set container=CDB$ROOT;
     ```
6. 启动，停止pdb
     ```
     alter session set container = orclpdb; 
     startup; 
     shtudown;
     ```

7. 创建通用用户

     ```sql
     create user C##test identified by 123456;     

     --给新用户授权
     grant create session to C##test;  
     grant create table to   C##test;  
     grant create tablespace to   C##test;  
     grant create view to   C##test;

     ```

https://docs.oracle.com/cd/E57425_01/121/ADMIN/cdb_mon.htm

## create sql cmd

```
--创建临时表空间
create temporary tablespace TRUEDATA_TEMP
tempfile 'D:\Tools\Oracle_12C\oradata\mhyorcl\orclpdb\TRUEDATA_TEMP.DBF'
size 32m 
autoextend on 
next 32m maxsize 2048m 
extent management local; 
--创建表空间
create tablespace TRUEDATA
logging 
datafile 'D:\Tools\Oracle_12C\oradata\mhyorcl\orclpdb\TRUEDATA.DBF'
size 32m 
autoextend on 
next 32m maxsize 2048m 
extent management local; 

--CDB中给用户授权表空间 CDB创建用户需加上c##（c大小写无所谓）
create user C##truedata identified by 1234;
alter user C##truedata quota unlimited on TRUEDATA; 

--PDB创建用户并指定表空间     
create user truedata identified by 1234
default tablespace TRUEDATA
temporary tablespace TRUEDATA_TEMP;

--给用户授予权限 
grant connect,resource,dba to truedata; 
--删除用户
drop user truedata cascade;
--删除表空间包括物理
drop tablespace truedata_temp including contents and datafiles;
drop tablespace truedata including contents and datafiles;
```