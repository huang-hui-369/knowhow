# docker-compose ps什么也没有显示

## 原因

project name变不同了。 默认的project name为目录名,如果启动后，你改变了docker-compose.yml的目录的话，就有可能project name变了。

## 解决办法

- 设置project name

  ```sh
  docker-compose -f /opt/docker-compose/server1-compose.yml logs -p projectname
  ```

- 设置 `COMPOSE_PROJECT_NAME` 环境变量。

  ```
  -rw-rw-r--  1 auser auser 1692 Aug 22 20:34 docker-compose.yml
  -rw-rw-r--  1 auser auser   31 Aug 22 20:44 .env
  ```

  In the `.env` file, I can now set the `COMPOSE_PROJECT_NAME` variable:

  ```
  COMPOSE_PROJECT_NAME=myproject
  ```

  On running:

  ```sh
  docker-compose up -d
  ```

## 查看project name

```sh
docker inspect container_name

 "Labels": {
     "com.docker.compose.config-hash": "4384818af10022ae3d180b7b1268646cf13516589b4949ca4d69b1d7486c5a0a",
     "com.docker.compose.container-number": "1",
     "com.docker.compose.oneoff": "False",
     "com.docker.compose.project": "nginx-spring",
     "com.docker.compose.service": "futari-nginx",      # docker-compose project name
     "com.docker.compose.version": "1.24.1",
     "maintainer": "NGINX Docker Maintainers <docker-maint@nginx.com>"
 },
```

可以得到 project name 为 **nginx-spring**

## docker-compose -p project-name 就可以看到  process信息

```sh
docker-compose -p nginx-spring ps
futari-nginx    /docker-entrypoint.sh ngin ...   Up      0.0.0.0:443->443/tcp, 0.0.0.0:80->80/tcp
futari-spring   java -Djava.security.egd=f ...   Up      0.0.0.0:8088->8088/tcp

```

