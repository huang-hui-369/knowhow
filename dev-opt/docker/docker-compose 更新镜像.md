# 利用docker命令更新

```shell
docker pull 新镜像名
docker stop 容器名
docker rm 容器名
docker rmi 业务镜像名
docker run -d --name 容器名 -p 对外端口:内部端口  新镜像名
```



# 利用docker-compose命令更新所有service

使用docker-compose方法：
配置好docker-compose.yml 文件，然后

```shell
docker-compose stop
docker-compose up -d --build
```

# 利用docker-compose命令更新指定service

docker-compose.yml 文件中有很多service，例如：mysql，nginx

配置好docker-compose.yml 文件（配置好image的最新版本），然后

```shell
# 停止服务
docker-compose stop nginx
# 删除容器
docker-compose rm -f nginx 
# 创建容器并启动服务
docker-compose up -d nginx 
```

