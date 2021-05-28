1. backup 到host目录下
docker exec -it futari-mysql mysqldump -B -R futari | gzip > ${MYSQL_BACKUP_FOLDER}/mysql_futari.sql.gz

2. restore
gunzip -c mysql_futari.sql.gz > mysql_futari.sql
cp mysql_futari.sql 容器内
docker exec futari-mysql sh -c 'exec mysql -uuser -ppassword --database futari < /docker-entrypoint-initdb.d/mysql_futari.sql'

导出数据库
1
mysqldump --defaults-extra-file=/etc/my.cnf database > database.sql
导入数据库

1
mysql --defaults-extra-file=/etc/my.cnf database < database.sql
