## 追加一块硬盘

通过vm界面可以追加一块硬盘名为[DATADISK] 然后挂载到DB-FUTARI的/data目录，
通过nfs共享DB-FUTARI的/data/futari/doc目录挂载到web server的/home/futari/doc目录

### 1. 通过vm界面可以 Attach  DATADISK to vm DB-FUTARI
  
### 2. Mount DATADISK to DB-FUTAR 

1.  通过fdisk 可以查到/dev/xvdb （新的硬盘）

2. 将新的硬盘format成xfs格式
   mkfs.xfs -f /dev/xvdb

3. 通过编辑/etc/fstab文件 自动mount/dev/xvdb 
    自動マウント /dev/xvdb --> /data
    
    vi /etc/fstab 下记追加
        /dev/xvdb               /data                   xfs     defaults        0 0

    要先有/data目录，没有的话
    mkdir -p /data
    
4. 使fstab文件生效
   mount -a

5. 确认硬盘mount
    执行命令 df
    ``` 
    Filesystem              1K-blocks    Used Available Use% Mounted on
    /dev/mapper/centos-root  18307072 3897936  14409136  22% /
    devtmpfs                  2008064       0   2008064   0% /dev
    tmpfs                     1891692       0   1891692   0% /dev/shm
    tmpfs                     1891692    8772   1882920   1% /run
    tmpfs                     1891692       0   1891692   0% /sys/fs/cgroup
    /dev/xvdb               104806400 1240964 103565436   2% /data
    ```

## 通过nfs共享硬盘

### 在DB server上安装 nfs-utils

1. 安装 nfs-utils
```
yum install -y rpcbind nfs-utils
```
2. 开放防火墙的nfs端口
   ```
   firewall-cmd --permanent --zone=public --add-service=nfs
   firewall-cmd --reload
   firewall-cmd --list-port
   ```
3. 启动nfs関連サービス
   ```
   systemctl start rpcbind
   systemctl start nfs
   ```

4. 设置nfs开机启动
   ```
   systemctl enable rpcbind.socket
   systemctl enable rpcbind
   systemctl enable nfs
   ```
5. 确定nfs状态
   ```
   systemctl status nfs
   ```
6. 设置共享权限
   编辑 /etc/exports 追加下行内容
   ```
   vi /etc/exports

   /data/futari/doc 172.20.0.168(rw)
   ```

### 在web server上安装 nfs-utils

方案1
```
yum install -y rpcbind nfs-utils
mount -v -t nfs 172.20.0.143:/data/futari/doc /home/futari/doc
```

方案2 永久挂载
```
vi /etc/fstab

# 追加下行内容
172.20.0.143:/data/futari/doc /home/futari/doc  nfs defaults 0 0

```
