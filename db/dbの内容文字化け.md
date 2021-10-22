# DBの内容文字化け

以下三处的charset必须一致

以postgre为例

## DB serverのcharset

```
psql -h db2inst1.cxndjrqxsnhs.ap-northeast-1.rds.amazonaws.com -U db2inst1 -d sample
Password for user db2inst1:
psql (9.2.24, server 12.8)
WARNING: psql version 9.2, server version 12.0.
         Some psql features might not work.
SSL connection (cipher: ECDHE-RSA-AES256-GCM-SHA384, bits: 256)
Type "help" for help.

sample=> \l

```

![](img\2021-10-20-20-30-24.png)

## DB clientのcharset
```
sample=> show client_ecoding;
ERROR:  unrecognized configuration parameter "client_ecoding"
sample=> show client_encoding;
 client_encoding
-----------------
 UTF8
(1 row)
```
![](img\2021-10-20-20-31-07.png)
<font style='color:red'>因为不是EUC_JP，所以乱码</font>

## PHPのcharset
Phpinfo
![](img\2021-10-20-20-34-26.png)

### 结果画面

![](img\2021-10-20-20-36-41.png)

因为client_ecoding 是UTF-8，把页面charset 改为UTF-8，就会显示正常

![](img\2021-10-20-20-38-00.png)

![](img\2021-10-20-20-39-30.png)

###　ostgresql.confのclient_encodingの値の変更
~~~~
dateフォルダにあるpostgresql.confのclient_encodingの値をEUC_JPに変更したら、
画面OKとなりました。