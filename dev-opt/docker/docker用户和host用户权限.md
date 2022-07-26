# 默认root 用户执行

默认如果在容器中不創建和指定用戶的情况下，容器中的进程以 `root` 用户执行，并且这个 `root` 用户和宿主机中的 `root` 是同一个用户，這就是説

- 容器中运行的进程，在合适的机会下，有权限控制宿主机中的一切；
- 容器中运行的进程，以 `root` 用户执行，外界很难追溯到真实的用户；
- 容器进程生成的文件，是 `root`用户所有，普通用户没有权限读取修改。

## 容器中的root用户和host root用户的权限一样吗

虽然 Docker 容器内的 root 用户直接是宿主机的 root 用户，但是 Docker 可以保证两者在权限方面，拥有巨大的差异。此时，这种差异的存在完全是借助于Linux 内核的 Capabilities 机制。换言之，正是 Capabilities 在保障 Docker 容器的安全性。

### --privileged 参数

如果 Docker 容器需要足够的系统权限的话则直接将 `--privileged` 参数设为 true，这样Docker 容器进程就拥有了特权模式，所以一定要**慎用特权模式** 如：

```
docker run -it --privileged=true ubuntu:14.04 /bin/bash 
```

### Docker 容器的非特权模式

默认情况下，docker run 命令的 privileged 参数值为 false。因此，毫无疑问，Docker 容器内部的 root 用户将受到严格的权限限制，很多有系统相关的操作权限都将被剥夺，只具备超级用户的一些基本权限，不具有mount 操作、创建进程新命名空间等均无法实现。

### Docker 容器的特权模式

一旦将 docker run 命令的 privileged 参数设为 true，那么 Docker 容器的 root 权限将得到大幅度的提升。此时，Docker容器的 root 用户将获得 37 项 Capabilities 能力。`由于 Docker 容器与宿主机处于共享同一个内核操作系统的状态，因此，Docker 容器将完全拥有内核的管理权限` 。安全隐患，瞬间浮出水面。

## 通过nginx　docker证明默认root 用户执行

下面是docker-compose.yml

```sh
version: '3.7'                 # docker-composeのversionを指定
services:
  futari-nginx:
    image: nginx:1.20.2
    container_name: futari-nginx
    restart: always
    depends_on:
      - futari-spring
    volumes:
      - /home/futari/dist:/dist
      - /home/futari/doc:/doc
      - /home/futari/nginx/conf:/etc/nginx/conf.d
      - /home/futari/nginx/log:/var/log/nginx
    ports:
      - 80:80
      - 443:443
    networks:
      - futari-net
```

从上面的设置我们知道容器没事设置执行用户，也没有设置特权。

1. 进入nginx容器通过id查询当前用户

   ```sh
   # docker exec -it futari-nginx bash
   # id
   root@605040a69380:/# id
   uid=0(root) gid=0(root) groups=0(root)
   # whoami
   root@605040a69380:/# whoami
   root
   ```

2. 从映射到host的log文件来查看文件的所有者

   ```sh
   # cd /home/futari/nginx/log
   [root@STG-SRCMAKER log]# ll
   total 37916
   -rw-r--r-- 1 root root 33204619 Feb 21 17:12 access.log
   -rw-r--r-- 1 root root  5607726 Feb 21 15:18 error.log
   ```

3. 默认容器用root用户的好处

   只要映射了host的文件目录，容器用root用户的话，就不会有权限的问题。假如你用用户app1运行容器，那你必须将/home/futari/nginx/log的所有者设置为app1，否则就会因为没有权限，而不能写log，而报了错。

## polkitd用户

当我们查看mysql容器的data目录时，发现host目录的文件的所有者为polkitd，这是为什么呢？

1. 首先查看docker-compose.yml文件

   ```sh
   version: '3.7'                 # docker-composeのversionを指定
   services:
     db:
       image: mysql:8.0.21
       container_name: futari-mysql
       restart: always
       environment:
         MYSQL_DATABASE: futari
         MYSQL_USER: futari
         MYSQL_PASSWORD: Grandunit123@
         MYSQL_ROOT_PASSWORD: Grandunit123@
         TZ: 'Asia/Tokyo'
       ports:
         - "23306:3306"
       volumes:
         - /home/futari/mysql/initdb.d:/docker-entrypoint-initdb.d
         - /home/futari/conf:/etc/mysql/conf.d
         - /home/futari/mysql/logs:/var/log/mysql
         - /home/futari/mysql/data:/var/lib/mysql
         - /etc/localtime:/etc/localtime
       networks:
         - futari-net
   ```

2. 进入mysql容器查看/var/lib/mysql目录

   可以看到binlog文件的所有者为mysql

   ```sh
   # docker exec -it futari-mysql bash
   root@83e35f096047:/# cd /var/lib/mysql
   root@83e35f096047:/var/lib/mysql# ls -l
   total 191780
   -rw-r----- 1 mysql mysql   196608 Feb 24 11:53 '#ib_16384_0.dblwr'
   -rw-r----- 1 mysql mysql  8585216 Aug 17  2021 '#ib_16384_1.dblwr'
   drwxr-x--- 2 mysql mysql      187 Feb 16 17:37 '#innodb_temp'
   -rw-r----- 1 mysql mysql  3885188 Aug 17  2021  83e35f096047.log
   -rw-r----- 1 mysql mysql       56 May 12  2021  auto.cnf
   -rw-r----- 1 mysql mysql  1172353 Feb  9 14:33  binlog.000012
   -rw-r----- 1 mysql mysql      179 Feb  9 14:34  binlog.000013
   -rw-r----- 1 mysql mysql      179 Feb 16 17:36  binlog.000014
   -rw-r----- 1 mysql mysql      156 Feb 16 17:37  binlog.000015
   -rw-r----- 1 mysql mysql       64 Feb 16 17:37  binlog.index
   ```

3. 查看容器中mysql用户idmysql组id

   ```sh
   root@83e35f096047:/# cat /etc/passwd
   mysql:x:999:999::/home/mysql:/bin/sh
   ```

   

4. 查看host /home/futari/mysql/data 目录

   ```sh
   [root@STG-SRCMAKER futari]# cd /home/futari/mysql/data
   [root@STG-SRCMAKER data]# ll
   total 191780
   -rw-r----- 1 polkitd ssh_keys  3885188 Aug 17  2021 83e35f096047.log
   -rw-r----- 1 polkitd ssh_keys       56 May 12  2021 auto.cnf
   -rw-r----- 1 polkitd ssh_keys  1172353 Feb  9 14:33 binlog.000012
   -rw-r----- 1 polkitd ssh_keys      179 Feb  9 14:34 binlog.000013
   -rw-r----- 1 polkitd ssh_keys      179 Feb 16 17:36 binlog.000014
   -rw-r----- 1 polkitd ssh_keys      156 Feb 16 17:37 binlog.000015
   -rw-r----- 1 polkitd ssh_keys       64 Feb 16 17:37 binlog.index
   ```

5. 查看host中用户id999 mysql组id 999

   ```sh
   
   [root@STG-SRCMAKER data]# cat /etc/group
   ssh_keys:x:999:
   [root@STG-SRCMAKER data]# cat /etc/passwd
   polkitd:x:999:998:User for polkitd:/:/sbin/nologin
   ```

6. 查看容器中的entrypoint.sh

   ```
   cat /entrypoint.sh
   
   # If container is started as root user, restart as dedicated mysql user
   if [ "$(id -u)" = "0" ]; then
       mysql_note "Switching to dedicated user 'mysql'"
       exec gosu mysql "$BASH_SOURCE" "$@"
   fi
   ```

   这是因为mysql容器进程启动mysqld时用的是mysql用户，mysql的用户ID：组ID为999：999，而host中999的用户ID为polkitd，999的组ID为ssh_keys。

   如果host中没有999的用户ID，host会自动创建吗？容器还有权限写目录吗？ （我觉得host应该不会自动创建user，会出现没有权限写目录）

# host中没有的用户ID

加入容器中创建了一个1001：1001的用户，host中并没有这个用户，会怎样呢，从实验的结果来看，host中会直接显示用户ID1001：10001

相反，如果host中有的1002用户，容器中并没有这个用户，在容器中也会显示用户ID1002：10002

进入容器，创建用户php4 1004:1004

```sh
# 进入容器
sudo docker exec -it moj-php7.1 bash
useradd php4
su php4
mkdir phpdir
chown php4:php4
cd phpdir
touch b.txt
echo "test by php4" > b.txt
ls -l
-rw-r--r-- 1 php4 php4  18 Feb 28 05:26 b.txt
cd ..
ls -l 
drwxr-xr-x 2 php4 php4    32 Feb 28 05:26 phpdir
exit
exit
# host 中没有1004的用户
ll
drwxr-xr-x 2   1004   1004    32 Feb 28 14:26 phpdir
ll phpdir
-rw-r--r-- 1   1004   1004 18 Feb 28 14:26 b.txt
```









