# DBファイルのサイズの設定

1. 选中database右键属性

   ![image-20220324151230791](D:\github\knowhow\db\mssql\databaseの物理設計.assets\image-20220324151230791.png)

2. 点击file，Data File 设定初期size为20G，自动扩张为4G，最大size为50G

   log file 设定初期size为5G，自动扩张为1G，最大size为10G

   ![image-20220324151424020](D:\github\knowhow\db\mssql\databaseの物理設計.assets\image-20220324151424020.png)

   - 要尽量根据硬盘大小和使用量来设置Data File 设定初期size，因为Data File 扩张时，数据库会被锁住？，不能使用

# 統計情報非同期自動更新设定为true

![image-20220324151903997](D:\github\knowhow\db\mssql\databaseの物理設計.assets\image-20220324151903997.png)

因为默认統計情報非同期自動更新为false，当执行sql语句时，sql server觉得统计情报需要更新时，会先更新统计情报，再执行sql

統計情報非同期自動更新设置为true，当执行sql语句时，sql server觉得统计情报需要更新时，会先并行更新统计情报，执行sql

# 文件和文件组的设计建议

- 如果使用多个数据文件，请为附加文件创建第二个文件组，并将其设置为默认文件组。 这样，主文件将只包含系统表和对象。
- 若要使性能最大化，请在尽可能多的不同可用磁盘上创建文件或文件组。 将争夺空间最激烈的对象置于不同的文件组中。
- 使用文件组将对象放置在特定的物理磁盘上。
- 将在同一联接查询中使用的不同表置于不同的文件组中。 由于采用并行磁盘 I/O 对联接数据进行搜索，所以此步骤可改善性能。
- 将最常访问的表和属于这些表的非聚集索引置于不同的文件组中。 如果文件位于不同的物理磁盘上，由于采用并行 I/O，所以使用不同的文件组可改善性能。
- 请勿将事务日志文件置于已有其他文件和文件组的同一物理磁盘上。
- 如果需要使用类似于[Diskpart](https://docs.microsoft.com/zh-cn/windows-server/administration/windows-commands/diskpart)的工具来扩展数据库文件所在的卷或分区，则应备份所有系统数据库和用户数据库，并首先停止 SQL Server 的服务。 此外，磁盘卷成功扩展后，应考虑运行 [`DBCC CHECKDB`](https://docs.microsoft.com/zh-cn/sql/t-sql/database-console-commands/dbcc-checkdb-transact-sql?view=sql-server-2016) 命令，确保驻留在该卷上的所有数据库的物理完整性。

## 例如：

 可以分别在三个磁盘驱动器上创建 `Data1.ndf`、`Data2.ndf` 和 `Data3.ndf`，然后将它们分配给文件组 `fgroup1`。 然后，可以明确地在文件组 `fgroup1` 上创建一个表。 对表中数据的查询将分散到三个磁盘上，从而提高了性能。 通过使用在 RAID（独立磁盘冗余阵列）条带集上创建的单个文件也能获得同样的性能提高。 但是，文件和文件组使您能够轻松地在新磁盘上添加新文件。

# メモリー的设定



# 最大接续数的设定



# database的復旧モデル设置为単純

默认为完全，是否需要设置为（简单）単純？

## 设置为简单模式

优点

- 无日志备份。自动回收日志空间以减少空间需求，实际上不再需要管理事务日志空间。

缺点

- 只能恢复到备份的结尾，不能做时点还原
- 不能AlwaysOn 或数据库镜像
- 不能日志传送

## 设置为完整模式

优点

- 可以恢复到指定时点

缺点

- 需要日志备份
- 在进行完全数据备份后，需要进行日志备份，否则的话日志文件会一直增长下去到很大。



参考文档

[Microsoft SQL Server データベース：設定の最適なチューニング](https://experienceleague.adobe.com/docs/experience-manager-64/forms/administrator-help/maintain-aem-forms-database/microsoft-sql-server-database-fine.html?lang=ja)

[SQL Server の設計に関する考慮事項](https://docs.microsoft.com/ja-jp/system-center/scom/plan-sqlserver-design?view=sc-om-2019)

[構成オプションを使用して SQL Server のメモリ使用量を調整する方法](https://support.microsoft.com/ja-jp/topic/%E6%A7%8B%E6%88%90%E3%82%AA%E3%83%97%E3%82%B7%E3%83%A7%E3%83%B3%E3%82%92%E4%BD%BF%E7%94%A8%E3%81%97%E3%81%A6-sql-server-%E3%81%AE%E3%83%A1%E3%83%A2%E3%83%AA%E4%BD%BF%E7%94%A8%E9%87%8F%E3%82%92%E8%AA%BF%E6%95%B4%E3%81%99%E3%82%8B%E6%96%B9%E6%B3%95-d82cc37a-489d-b15f-7166-9a58044ebf5f)

