[toc]

# docker 安装

## 1. Uninstall old versions
```
sudo yum remove docker \
                  docker-client \
                  docker-client-latest \
                  docker-common \
                  docker-latest \
                  docker-latest-logrotate \
                  docker-logrotate \
                  docker-engine
```

## 2. INSTALL DOCKER

### 1 安装yum-utils插件设置 stable repository

```
# 安装yum-utils插件
sudo yum install -y yum-utils

# 设置 stable repository
sudo yum-config-manager \
    --add-repo \
    https://download.docker.com/linux/centos/docker-ce.repo

```

### 2 INSTALL DOCKER ENGINE
1. To install a specific version of Docker Engine

```
# 安装最新版本的DOCKER ENGINE
$ sudo yum install docker-ce docker-ce-cli containerd.io
```

```
# 列出有效的版本的DOCKER ENGINE
$ yum list docker-ce --showduplicates | sort -r

docker-ce.x86_64  3:18.09.1-3.el7                     docker-ce-stable
docker-ce.x86_64  3:18.09.0-3.el7                     docker-ce-stable
docker-ce.x86_64  18.06.1.ce-3.el7                    docker-ce-stable
docker-ce.x86_64  18.06.0.ce-3.el7                    docker-ce-stable

# 安装指定版本的DOCKER ENGINE
$ sudo yum install docker-ce-<VERSION_STRING> docker-ce-cli-<VERSION_STRING> containerd.io
```
Docker is installed but not started. The docker group is created, but no users are added to the group.

2. Start Docker
   `$ sudo systemctl start docker`

3. 创建用户
   ```
   useradd futari
   passwd futari
   ```

4. 创建docker工作组（安装完docker ce后已经创建了docker group 可以不用）
   `$ sudo groupadd docker`
   
5. 将当前用户加入docker组
   `$ sudo usermod -aG docker futari`

6. 切换用户到
   `su futari`

7. Verify that Docker Engine is installed correctly 
   `$ docker run hello-world`

## 3. CentOS7中以非root身份运行Docker命令（可以忽略） 

1. 查询docker用户组
   CentOS7中使用yum安装完Docker后一般都会自动创建一个docker用户组
   ```
   cat /etc/group | grep docker
   docker:x:995:
   ```
   要想使用docker的命令就需要拥有对文件/var/run/docker.sock的操作权限。
   ```
   ll /var/run/docker.sock
   srw-rw---- 1 root docker 0 Jun 10 18:00 /var/run/docker.sock
   ```
   从中可见docker.sock文件对docker组是有读写权限的。因此只要普通用户属于docker组就也可以使用docker命令操作Docker。

2. 将docker用户组作为附加组添加给操作账号
    ```shell
    # 如果没有docker group 创建docker组
    sudo groupadd docker
    # 将docker用户组作为附加组添加到msp账号上
    sudo usermod -aG docker msp
    # run the following command to activate the changes to groups
    newgrp docker
    # Verify that you can run docker commands without sudo
    docker run hello-world
    ```

    如果出现以下错误
    ```
    WARNING: Error loading config file: /home/user/.docker/config.json - stat /home/user/.docker/config.json: permission denied
    ```
    方法1
    remove the ~/.docker/ directory (it is recreated automatically, but any custom settings are lost),
    `rm -rf  ~/.docker`

     方法2
     改变~/.docker/的组的所有者为msp
     ```
    $ sudo chown "$USER":"$USER" /home/"$USER"/.docker -R
    $ sudo chmod g+rwx "$HOME/.docker" -R
     ```
## 4. Configure Docker to start on boot

```
$ sudo systemctl enable docker.service
$ sudo systemctl enable containerd.service
```
## 5. 重启docker service
`sudo systemctl restart docker.service`

## 6. 确认docker监听端口，进程
```
$ sudo netstat -lntp | grep dockerd
tcp        0      0 127.0.0.1:2375          0.0.0.0:*               LISTEN      3758/dockerd
```
## 7. Configuring remote access with daemon.json
`vi /etc/docker/daemon.json`

```
{
  "hosts": ["unix:///var/run/docker.sock", "tcp://127.0.0.1:2375"]
}
```
## 8. Uninstall Docker Engine
```
$ sudo yum remove docker-ce docker-ce-cli containerd.io
$ sudo rm -rf /var/lib/docker
```

# 使用Docker镜像

## 1. 获取镜像
`docker pull [选项] [Docker Registry 地址[:端口号]/]仓库名[:标签]`

* Docker 镜像仓库地址：地址的格式一般是 <域名/IP>[:端口号]。默认地址是 Docker Hub。
* 仓库名：如之前所说，这里的仓库名是两段式名称，即 <用户名>/<软件名>。对于 Docker Hub，如果不给出用户名，则默认为 library，也就是官方镜像。

`$ docker pull ubuntu:16.04`

## 运行镜像
`$ docker run --name webserver -d -p 90:80 nginx`

参数介绍：

* -d：以demon的方式运行。
* -p 将80端口映射到host90端口

## 列出本地镜像
```
$ docker image ls
REPOSITORY    TAG    IMAGE ID        CREATED        SIZE
ubuntu        16.04  0458a4468cbc    3 days ago     112MB
hello-world   latest f2a91732366c    2 months ago   1.85kB
```

## 删除本地镜像
`$ docker image rm [选项] <镜像1> [<镜像2> ...]`

可以使用镜像ID或者镜像名来删除

`docker image rm f2a91732366c`

`docker image rm hello-world`

# 使用Dockerfile定制镜像

Dockerfile 是一个文本文件，其内包含了一条条的指令(Instruction)，每一条指令构建一层，因此每一条指令的内容，就是描述该层应当如何构建。

## 1 编写Dockerfile
例如：
```
// 内容
FROM nginx
RUN echo '<h1>Hello, Docker!</h1>' > /usr/share/nginx/html/index.html
```

## 2 构建镜像
在Dockerfile目录下，执行以下命令
`$ docker build -t nginx:v3 .`

docker是C/S结构，并非所有命令都是在本地执行，如docker build在Docker引擎（即服务器端）执行，因此我们在执行该命令时需要给出上下文。如Dockerfile中的命令COPY ./package.json /app/就是复制给出的上下文下的package.json。
  在给出上下文后，该目录下的内容会被发送到服务器，可以使用.dockerignore文件制定不需要发送的文件。

## 3 Dockerfile指令详解
```
// COPY <源路径>... <目标路径>，支持正则匹配
COPY hom* /mydir/
COPY hom?.txt /mydir/

// ADD和COPY类似，能够自动将gzip, bzip2 以及 xz压缩格式自动解压，
// 无需自动解压时，使用COPY
ADD ubuntu-xenial-core-cloudimg-amd64-root.tar.gz /

// CMD 容器启动命令，用于指定默认的容器主进程的启动命令，类似RUN
// 使用该格式'CMD ["可执行文件", "参数1", "参数2"...]'
// 注：CMD是注进程，退出后整个容器退出，因此不能后台执行
CMD service nginx start //后台执行nginx，执行完立刻退出
CMD ["nginx", "-g", "daemon off;"] //前台执行，正确

// ENTRYPOINT和CMD类似，不过可以继续加参数(如docker run myip -i)
// 或者执行脚本
ENTRYPOINT [ "curl", "-s", "http://ip.cn" ]
ENTRYPOINT ["docker-entrypoint.sh"]

// ENV <key> <value>
ENV NODE_VERSION 7.2.0
RUN curl -SLO https://nodejs.org/dist/v$NODE_VERSION/node-v$NODE_VERSION-linux-x64.tar.xz

// VOLUME 定义匿名卷
// '/data'目录就会在运行时自动挂载为匿名卷，任何向'/data'中写入的信息都不会记录进容器存储层，从而保证了容器存储层的无状态化。
VOLUME /data

// WORKDIR 指定工作目录
WORKDIR <工作目录路径>

// USER 指定当前用户
USER <用户名>

// ONBUILD 当内容与该项目相关时，前面加上ONBUILD即可重用
FROM node:slim
RUN mkdir /app
WORKDIR /app
ONBUILD COPY ./package.json /app
ONBUILD RUN [ "npm", "install" ]
ONBUILD COPY . /app/
CMD [ "npm", "start" ]
```

# 使用容器

## 新建并启动
```
$ docker run ubuntu:14.04 /bin/echo 'Hello world'
Hello world
```

当利用 docker run 来创建容器时，Docker 在后台运行的标准操作包括：

* 检查本地是否存在指定的镜像，不存在就从公有仓库下载
* 利用镜像创建并启动一个容器
* 分配一个文件系统，并在只读的镜像层外面挂载一层可读写层
* 从宿主主机配置的网桥接口中桥接一个虚拟接口到容器中去
* 从地址池配置一个 ip 地址给容器
* 执行用户指定的应用程序
* 执行完毕后容器被终止

## 查看容器
`docker ps`
`docker container ls -a`

## 启动已终止容器
`docker container start <id>`

## 终止容器
`docker container stop`

## 查看容器控制台输出
`$ docker container logs [container ID or NAMES]`

## 进入容器内部
$ docker exec -it webserver bash

参数
* -it：这是两个参数，一个是 -i：交互式操作，一个是 -t 终端。我们这里打算进入 bash 执行一些命令并查看返回结果，因此我们需要交互式终端。

## 导出容器
```
$ docker container ls -a
CONTAINER ID ……
7691a814370e ……

$ docker export 7691a814370e > ubuntu.tar
```

## 导入容器

将容器导入为镜像
```
$ cat ubuntu.tar | docker import - test/ubuntu:v1.0
$ docker image ls
REPOSITORY    TAG   IMAGE ID        CREATED     VIRTUAL SIZE
test/ubuntu   v1.0  9d37a6082e97    20s ago     171.3 MB

//也可以通过指定 URL 或者某个目录来导入
$ docker import http://ex.com/eximage.tgz ex/imagerepo
```

## 删除容器
docker container rm

# docker Compose

在实际生产环境中，一个应用往往由许多服务构成，而 docker 的最佳实践是一个容器只运行一个进程，因此运行多个微服务就要运行多个容器。多个容器协同工作需要一个有效的工具来管理他们，定义这些容器如何相互关联。compose 应运而生。

compose 是用来定义和运行一个或多个容器(通常都是多个)运行和应用的工具。使用 compose 可以简化容器镜像的构建以及容器的运行。

compose 使用 YAML 文件来定义多容器之间的关系。一个 docker-compose up 就可以把完整的应用跑起来。 本质上， compose 把 YAML 文件解析成 docker 命令的参数，然后调用相应的 docker 命令行接口，从而将应用以容器化的方式管理起来。它通过解析容器间的依赖关系顺序地启动容器。

**在 docker-compose.yml 文件中，每个定义的服务都至少包含 build或image 其中之一，其他命令都是可选的。 build 命令指定了包含 Dockerfile 的目录，可以是相对目录也可以是绝对目录。**

## docker-compose安装
1. ```shell
   # 安装docker-compose
   curl -L "https://github.com/docker/compose/releases/download/1.25.0/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
   # 给docker-compose执行权限
   chmod +x /usr/local/bin/docker-compose
   # check docker-compose
   docker-compose --version
   ```

## Compose 使用的三个步骤
1. 使用 Dockerfile 定义单个应用程序的环境。（镜像）
2. 使用 docker-compose.yml 定义构成应用程序的多个服务，这样它们可以在隔离环境中一起运行。
3. 执行 docker-compose up 命令来启动并运行整个应用程序。

## 一个简单例子
```
version: '3'
services:
  web:
    build: ./web/
    ports:
     - "5000:5000"
  redis:
    image: "redis:3.0.7"。
```

这个 YAML 文件定义了两个服务： Web 和 Redis， 服务的名称由用户自定义。提供 Web 服务的镜像从 Dockerfile 构建； Web 服务监听80端口，并和主机的8080端口建立映射；主机的当前目录挂载到容器里的 /code 目录上；Web 服务器通过链接 Redis 容器来访问后台 Redis 数据库。而 Redis 数据库服务是通过运行 Redis 镜像来提供的。

## 运行
在 docker-compose.yml 所在的目录下执行
`docker-compose up`

## 安装
1. af
   `sudo curl -L "https://github.com/docker/compose/releases/download/1.24.1/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose`

2. 赋予可执行权限
   `sudo chmod +x /usr/local/bin/docker-compose`

3. 创建链接
   `ln -s /usr/local/bin/docker-compose /usr/bin/docker-compose`

4. 查看版本
   `docker-compose --version`


## サービスを停止させる
`docker-compose stop`

## コンテナの停止、削除、さらにネットワーク、記憶領域を全て削除
`docker-compose down`

## 问题

### docker build 提示 No space left on device

docker stats 查看容器的内存占用情况

docker system df 查看磁盘状况

docker system prune 删除关闭的容器、无用的数据卷和网络，以及dangling镜像

删除 volume
docker volume prune