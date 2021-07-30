## 1. install pgdg95
```shell
# pgdg95リポジトリを追加する
yum install -y https://download.postgresql.org/pub/repos/yum/9.5/redhat/rhel-7-x86_64/pgdg-redhat-repo-42.0-11.noarch.rpm
# 情報の確認
yum info postgresql95-server
# install
yum install -y postgresql95-server postgresql95-contrib
```

## 2. 初期化

```
# /usr/pgsql-9.5/bin/postgresql95-setup initdb
Initializing database ... OK
```

ファイル・ディレクトリ | 説明
------------|---
/var/lib/pgsql/9.5/ | PostgreSQL の設定やデータ、バックアップファイルが保存されるディレクトリ。
/var/lib/pgsql/9.5/data/postgresql.conf | PostgreSQLのメインとなる設定ファイル。
/var/lib/pgsql/9.5/data/pg_hba.conf | PostgreSQLのユーザ認証についての設定ファイル。
/usr/pgsql-9.5/bin/createdb | PostgreSQLのデータベースを作成するためのコマンド。
/usr/pgsql-9.5/bin/psql | PostgreSQLを操作するためのクライアント。
/usr/pgsql-9.5/bin/postgres | PostgreSQLサーバー。

## 3. サービスの起動

```sh
# サービス起動
systemctl start postgresql-9.5.service
# サービスの自動起動の設定
systemctl enable postgresql-9.6.service
# show version
psql --version
```

## 4. 设置PostgreSQL监听地址和允许访问IP地址

1. 设置PostgreSQL监听地址
   
    ```sh
    vi /var/lib/pgsql/9.5/data/postgresql.conf
    ```
    listen_addresses = 'local' 修改为
    listen_addresses = '*'

2. 设置允许访问PostgreSQL服务器的客户端IP地址
    ```sh
    vi /var/lib/pgsql/9.5/data/pg_hba.conf
    ``` 
    **添加：**
      host all all 192.168.0.0/16 password
      其中：192.168.0.0/16表示允许192.168.0.1-192.168.255.255网段访问。
    password 是通过密码认证(还可以为md5, sha1)
    **修改:**
      host all all 127.0.0.1/32 trust
      trust 是不需要认证

## 5. 登录db
PostgreSQL会在安装阶段默认
创建一个linux用户postgres
创建一个超级用户角色postgres
创建一个database：postgres

1. 修改linux用户postgres的密码
    ```sh
    passwd postgres
    ```
2. 切换到用户postgres登录db
    ```sh
    su - postgres
    # psql -h 192.168.56.26 -U bob -d postgres -p 5432
    -bash-4.2$ psql
    psql (9.5.25)
    "help" でヘルプを表示します.
    ```
## 6. 修改默认超级用户postgres的密码
    ```
    postgres=#  ALTER USER postgres PASSWORD 'grandunit'
    -bash-4.2$ psql -h 127.0.0.1 -U postgres -d postgres -p 5432
    ユーザ postgres のパスワード:
    grandunit
    psql (9.5.25)
    "help" でヘルプを表示します.
    ```    

## 7. install odbc driver

まずsubscription-managerを登録して、登録しないとunixODBCのインストールができません。
30日無料使えます。
subscription-manager register

```
yum install postgresql95-odbc.x86_64 unixODBC
```

### 7.1 create link

ln -s /usr/pgsql-9.5/lib/psqlodbcw.so /usr/lib/psqlodbcw.so
ln -s /usr/pgsql-9.5/lib/psqlodbcw.so /usr/lib64/psqlodbcw.so
ln -s /usr/pgsql-9.5/lib/psqlodbc.so /usr/lib/psqlodbc.so
ln -s /usr/pgsql-9.5/lib/psqlodbc.so /usr/lib64/psqlodbc.so


https://www.postgresql.org/ftp/odbc/versions/src/

https://www.postgresql.org/download/linux/redhat/

### 7.2 确认odbcinst.ini
vi /etc/odbcinst.ini

```
[PostgreSQL]
Description     = ODBC for PostgreSQL
Driver          = /usr/lib/psqlodbcw.so
Setup           = /usr/lib/libodbcpsqlS.so
Driver64        = /usr/lib64/psqlodbcw.so
Setup64         = /usr/lib64/libodbcpsqlS.so
FileUsage       = 1
```

### 7.3 Configure Our ODBC Connections in /etc/odbc.ini or $HOME/.odbc.ini

```
[postgres]
Description         = PostgreSQL connection to postgres
Driver              = PostgreSQL
Database            = postgres
Servername          = localhost
UserName            = postgres
Password            = grandunit
Port                = 5432
Protocol            = 9.5
ReadOnly            = No
RowVersioning       = No
ShowSystemTables    = No
ConnSettings        =
```

### 7.4 access pssql by odbc

```sh
# isql dns user password
isql pg postgres grandunit
+---------------------------------------+
| Connected!                            |
|                                       |
| sql-statement                         |
| help [tablename]                      |
| quit                                  |
|                                       |
+---------------------------------------+
SQL>
```