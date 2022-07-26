# install epel
Extra Packages for Enterprise Linux (EPEL)
add repo

yum install https://dl.fedoraproject.org/pub/epel/epel-release-latest-7.noarch.rpm

yum list | grep postgresql

subscription-manager repos --enable "rhel-*-optional-rpms" --enable "rhel-*-extras-rpms"  --enable "rhel-ha-for-rhel-*-server-rpms"

# postgre


## install
```shell
# pgdg95リポジトリを追加する
yum install -y https://download.postgresql.org/pub/repos/yum/9.5/redhat/rhel-7-x86_64/pgdg-redhat-repo-42.0-11.noarch.rpm
# 情報の確認
yum info postgresql95-server
# install
yum install -y postgresql95-server postgresql95-contrib
```

## install location
/usr/pgsql-9.5/bin

## 初期化

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

## サービスの起動
```
systemctl start postgresql-9.5.service
```

## サービスの自動起動の設定
```sh
systemctl enable postgresql-9.6.service

# show version
psql --version
```
## 设置PostgreSQL监听地址和允许访问IP地址
1、PostgreSQL监听地址也就是你用什么host地址去访问它，修改配置文件postgresql.conf
listen_addresses=’*’
2、允许访问PostgreSQL服务器的客户端IP地址，修改配置文件pg_hba.conf，添加：
host all all 192.168.0.0/16 password
其中：192.168.0.0/16表示允许192.168.0.1-192.168.255.255网段访问。
password 是通过密码认证
修改
host all all 127.0.0.1/32 trust
trust 是不需要认证


## ユーザーの作成
PostgreSQL会在安装阶段默认创建一个超级用户角色和一个database，均是postgres，
修改PostgreSQL数据库默认用户postgres的密码
```sh
passwd postgres

su - postgres

# psql -h 192.168.56.26 -U bob -d postgres -p 5432

-bash-4.2$ psql


psql (9.5.25)
"help" でヘルプを表示します.

postgres-# \l
                                         データベース一覧
   名前    |  所有者  | エンコーディング |  照合順序   | Ctype(変換演算子) |      アクセス権
-----------+----------+------------------+-------------+-------------------+-----------------------
 postgres  | postgres | UTF8             | ja_JP.UTF-8 | ja_JP.UTF-8       |
 template0 | postgres | UTF8             | ja_JP.UTF-8 | ja_JP.UTF-8       | =c/postgres          +
           |          |                  |             |                   | postgres=CTc/postgres
 template1 | postgres | UTF8             | ja_JP.UTF-8 | ja_JP.UTF-8       | =c/postgres          +
           |          |                  |             |                   | postgres=CTc/postgres
(3 行)

postgres-# \q
```

## 角色
postgresql中的role就是一般的用户
CREATE ROLE 角色术语来表示用户账户的概念， CREATE USER （可登录角色）和 CREATE GROUP（组角色）不建议使用。

CREATE ROLE envadmin login password 'grandunit';


## install odbc driver

まずsubscription-managerを登録して、登録しないとunixODBCのインストールができません。
30日無料使えます。
subscription-manager register

```
yum install postgresql95-odbc.x86_64 unixODBC
```

## create link

ln -s /usr/pgsql-9.5/lib/psqlodbcw.so /usr/lib/psqlodbcw.so
ln -s /usr/pgsql-9.5/lib/psqlodbcw.so /usr/lib64/psqlodbcw.so
ln -s /usr/pgsql-9.5/lib/psqlodbc.so /usr/lib/psqlodbc.so
ln -s /usr/pgsql-9.5/lib/psqlodbc.so /usr/lib64/psqlodbc.so


https://www.postgresql.org/ftp/odbc/versions/src/

https://www.postgresql.org/download/linux/redhat/

## 确认odbcinst.ini
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

### Configure Our ODBC Connections in /etc/odbc.ini or $HOME/.odbc.ini

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

## access pssql by odbc

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

## 基本用法

### 1. show version
```
# psql --version
psql (PostgreSQL) 9.5.25
```

### 2. 连接Db
```
psql -h 127.0.0.1 -p 5021 -U db2inst1 -d sample
```

### 3. 执行sql文件
```
postgres=# \i /home/env/src/testdata/masterdata.sql
```

### 4. log file

/var/lib/pgsql/9.5/data/pg_log

### 5. サービスの起動

```sh
systemctl start postgresql-9.5.service
```

# apache install


```sh
# add httpd repos
yum -y install "https://repo.ius.io/ius-release-el7.rpm"

# install httpd
yum -y install httpd24u.x86_64

# show version
httpd -v
```

## 設定ファイル
vim /etc/httpd/conf/httpd.conf

/etc/httpd/conf/httpd.conf

## DocumentRoot
DocumentRoot "/var/www/html"

## start service
```
systemctl restart httpd
```


# php install

```sh
# add repos
yum install http://rpms.famillecollet.com/enterprise/remi-release-7.rpm
yum-config-manager --enable remi-php56

# 确认有效的repos
yum repolist all

# install php
yum -y install --enablerepo=remi,remi-php56 php php-mbstring php-odbc php-pear php-xml php-snmp

# show version
php -v
php -i | head
```

以下参数需要吗？
php-devel php-fpm
不需要这是另外一种apache结合php执行的方式，现在是用的Apache 2.0 Handler

## install pear

```
yum install php-pear
pear channel-update pear.php.net
```

## pear 安装目录
/usr/share/pear

## install pear pager

```
pear install -a pager
```

## install pear mail

```
pear install -a Mail
install ok: channel://pear.php.net/Mail-1.4.1
install ok: channel://pear.php.net/Net_Socket-1.2.2
install ok: channel://pear.php.net/Auth_SASL-1.1.0
install ok: channel://pear.php.net/Net_SMTP-1.10.0

pear install -a Mail_mimeDecode
install ok: channel://pear.php.net/Mail_Mime-1.10.10
install ok: channel://pear.php.net/Mail_mimeDecode-1.5.6
```

## mail setting

![](img\2021-07-13-19-11-50.png)

## send mail error
**現象**
SMTP: Failed to connect socket: Permission denied (code: -1, response: )

**解決方法1**
SELinuxの無効化

状態一覧
- enforcing ･･･ SELinuxは有効で、アクセス制限も有効。
- permissive ･･･ SELinuxは有効だが、アクセス制限は行わず警告を出力。
- disabled ･･･ SELinux機能は無効。

```sh
# 一時的に無効にする
setenforce 0
getenforce
permissive

# 永続的に無効にする
vi /etc/selinux/config
SELINUX=disabled # enforcing --> disabled
```

**解決方法2**
```sh
# ソケット通信を許可する
setsebool -P httpd_can_network_connect on

# httpdメール送信可否
getsebool httpd_can_sendmail
httpd_can_sendmail --> off
# httpdメール送信可
setsebool -P httpd_can_sendmail on
```


Apache跟PHP是怎么关联起来的呢？

实际上我们安装php的时候，系统已经自动添加了php的模块文件到Apache的安装目录下，即/etc/httpd/conf.d，在这个目录下我们可以看到有一个php.conf的文件，这个就是Apache关联php模块的配置。
在Apache的配置文件最底下一行我们也可以看到IncludeOptional conf.d/*.conf，这句配置就是加载/conf.d下面的所有.conf文件

## 設定ファイル
/etc/php.ini



# php ["failed to open stream: Permission denied" error]

**現象**
file_get_contents('/xxx/yyy.pdf'); failed to open stream: Permission denied

**错误调查**

1.  确认 apache error log

   ```sh
   # /var/log/apache2或/var/log/httpd
   tail -f /var/log/httpd/error_log
   # 发现 file_get_contents('/xxx/yyy.pdf'); failed to open stream: Permission denied
   ```

   

2. 确认 php 运行用户

   ```php
   <?php echo exec('whoami'); ?>
   ```

   

3. 确认 apache httpd 运行用户

   ```sh
   ps aux | grep httpd
   # 确认第一行
   root      1217  0.0  1.6 424736 17040 ?        Ss    2月08   1:15 /usr/sbin/httpd -DFOREGROUND
   root      4150  0.0  0.1 117040  1016 pts/1    S+   12:45   0:00 grep --color=auto httpd
   apache   16400  0.0  1.7 483124 18068 ?        Sl    2月15   1:00 /usr/sbin/httpd -DFOREGROUND
   apache   16412  0.0  1.7 483188 17976 ?        Sl    2月15   1:00 /usr/sbin/httpd -DFOREGROUND
   ```

   

4. 确认 /xxx/yyy.pdf的所有者，访问权限

   如果httpd的运行用户为apache，/xxx/yyy.pdf的所有者为demo1：demo1将apache用户加入到demo1组，更改/xxx/下的所有子文件夹及文件的权限为644（只读）或664（读写），更狠点777（用来排除错误）

   ```sh
   将apache用户加入到demo1 group
   usermod -a -G demo1 apache
   # 确认apache 用户的组
   id apache
   chmod 777 /xxx/ -R
   ```

   

5. 确认SELinux的设置

   ```sh
   sestatus
   
   SELinux status:                 enabled
   SELinuxfs mount:                /sys/fs/selinux
   SELinux root directory:         /etc/selinux
   Loaded policy name:             targeted
   Current mode:                   enforcing
   Mode from config file:          enforcing
   Policy MLS status:              enabled
   Policy deny_unknown status:     allowed
   Max kernel policy version:      31
   ```

   

**解決方法1**
SELinuxの無効化

**解決方法2**

设置SELinux Policies for Apache

### Apache Context Types

```sh
man httpd_selinux
```

| httpd_sys_content_t    | Read-only directories and files used by Apache               |
| :--------------------- | ------------------------------------------------------------ |
| httpd_sys_rw_content_t | Readable and writable directories and files used by Apache. Assign this to directories where files can be created or modified by your application, or assign it to files directory to allow your application to modify them. |
| httpd_log_t            | Used by Apache to generate and append to web application log files. |
| httpd_cache_t          | Assign to a directory used by Apache for caching, if you are using mod_cache. |

```sh
# 确认现有设置
semanage fcontext -l | grep httpd
# Create a policy to assign the httpd_sys_content_t context to the /xxx directory, and all child directories and files.
semanage fcontext -a -t httpd_sys_content_t "/xxx(/.*)?"
# Create a policy to assign the httpd_log_t context to the logging directories(暂时不需要)
semanage fcontext -a -t httpd_log_t "/xxx/logs(/.*)?"
# Create a policy to assign the httpd_cache_t context to our cache directories(暂时不需要)
semanage fcontext -a -t httpd_cache_t "/xxx/cache(/.*)?"
# 如果需要读写的话Create a policy to assign the httpd_sys_rw_content_t context to the upload directory, and all child files.
semanage fcontext -a httpd_sys_rw_content_t "/xxx/app1/public_html/uploads(/.*)?"
# Apply the SELinux policies
restorecon -Rv /xxx
# 确认设置
ls -lZ /xxx
drwxr-xr-x. root root unconfined_u:object_r:httpd_sys_content_t:s0 apps
drwxr-xr-x. root root unconfined_u:object_r:httpd_log_t:s0 logs
drwxr-xr-x. root root unconfined_u:object_r:httpd_cache_t:s0 cache
```



参考文档 

https://www.serverlab.ca/tutorials/linux/web-servers-linux/configuring-selinux-policies-for-apache-web-servers/

https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/7/html/selinux_users_and_administrators_guide/sect-managing_confined_services-the_apache_http_server-configuration_examples

 # redhat  subscription-manager

```
subscription-manager register
登録中: subscription.rhsm.redhat.com:443/subscription
ユーザー名: huanghui369
パスワード:
このシステムは、次の ID で登録されました: 24ca72e5-d78f-441c-90d3-60be0d8f0318
登録したシステム名: localhost.env
```

# php56

```php
<?php
$dsn='pg';
$user='postgres';
$password='grandunit';
$server='192.168.1.146';
$database='postgres';
 
$conn=odbc_connect($dsn,$user,$password);//odbc连接数据库

/*
try{
	$conn=odbc_connect("Driver={/usr/lib/psqlodbcw.so};Server=$server;Database=$database;", $user, $password);
}
catch (Exception $e){
            echo $e->getMessage();
}
*/
echo odbc_errormsg();

if (!$conn)
{
    exit("接続失敗です2。" . $conn);
}
 
 
$sql="SELECT id,text FROM test";//sql语句
 
$rs=odbc_exec($conn,$sql);//执行sql
 
//遍历记录集
while (odbc_fetch_row($rs))
{
//输出当前记录
$id=odbc_result($rs,"id");
echo $id;
$text=odbc_result($rs,"text");
echo $text;

}
 
odbc_close($conn);//关闭odbc
?>

```

# php 接続pssql error

php [unixODBC]could not connect to server: Permission denied Is the server running on host

原因是Linux阻止httpd进程--httpd_can_network_connect_db 连接数据库--不管是哪种类型的数据库

所以需要确保设置正确的布尔值以允许Web应用程序与数据库通信，使用setsebool命令改变该布尔变量的状态，从而使得httpd进程能够访问数据库服务器：

setsebool -P httpd_can_network_connect_db 1

# php.ini 反映されてない

执行如下命令，会提示php.ini文件中的错误
```
php -i | grep php.ini
HP:  syntax error, unexpected '&' in /etc/php.ini on line 109
Configuration File (php.ini) Path => /etc
```

# 参考資料
https://tech.hitsug.net/2019/05/centos-7--php72-install.html

https://qiita.com/qiitamatumoto/items/efffc0ef6e6249533201