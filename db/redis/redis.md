
重新启动redis服务
docker restart futari-redis

docker exec -it futari-redis bash

连接 redis
redis-cli

解除readonly 模式
CONFIG SET slaveof no one

测试写入
set test foo


设置readonly 模式
CONFIG SET slaveof 127.0.0.1 6379

查看设置情况
info replication

```
# Replication
role:slave
master_host:127.0.0.1
master_port:6379
master_link_status:down
master_last_io_seconds_ago:-1
master_sync_in_progress:0
slave_repl_offset:0
master_link_down_since_seconds:1616664481
slave_priority:100
slave_read_only:0
connected_slaves:0
master_replid:68f1b8c42043a9384fcda5a78affc4dc99ed9b59
master_replid2:0000000000000000000000000000000000000000
master_repl_offset:0
second_repl_offset:-1
repl_backlog_active:0
repl_backlog_size:1048576
repl_backlog_first_byte_offset:0
repl_backlog_histlen:0
```
