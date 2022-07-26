# docker_network

docker容器之间如果想要通过网络访问的话，要么在同一个虚拟网络中，要么通过网络的gateway，也可以通过host ip 来访问另一个docker或者host机器的端口。

## 同一个网络中

这里的db，redis，tomcat，nginx都在同一个网络futari-net中，相互可以通过container_name访问。

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



  redis:
    image: redis:6.0.10
    container_name: futari-redis
    restart: always
    ports:
      - "6379:6379"
    networks:
      - futari-net

  futari-spring:
    build: /home/futari/spring
    container_name: futari-spring
    restart: always
    volumes:
      - /home/futari/spring/jar:/jar
      - /home/futari/spring/logs:/logs
      - /etc/localtime:/etc/localtime
      - /home/futari/doc:/doc
      - /home/futari/fonts:/usr/lib/jvm/java-8-openjdk-amd64/jre/lib/fonts
    tmpfs:
      - /tmp
    ports:
      - 8088:8088
    networks:
      - futari-net


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


networks:
  futari-net:
     driver: bridge
     ipam:
       driver: default
       config:
       - subnet: 172.24.0.0/16
```



## 通过gateway来访问

例如 docker nginx和docker apache-php不在同一个网络中，想要在nginx中访问apache服务器，可以通过访问nginx gateway 来访问。

如下 apache-php 将docker中的80端口映射到了host的9000端口中

```sh
CONTAINER ID   IMAGE                                               COMMAND                  CREATED        STATUS                          PORTS                                                                NAMES
605040a69380   nginx:1.20.2                                        "/docker-entrypoint.…"   46 hours ago   Up 46 minutes                   0.0.0.0:80->80/tcp, 0.0.0.0:443->443/tcp                             futari-nginx
f7cc8352b76b   php:7.1-apache                                      "docker-php-entrypoi…"   2 weeks ago    Up 47 hours                     0.0.0.0:9000->80/tcp                                                 moj-php7.1
```

查看 nginx的网络信息

Gateway为172.24.0.1，通过访问172.24.0.1的9000端口，就可以访问host的9000端口了，也就是 apache-php docker中的80端口

```
"Networks": {
                "futari_futari-net": {
                    "IPAMConfig": null,
                    "Links": null,
                    "Aliases": [
                        "605040a69380",
                        "futari-nginx"
                    ],
                    "NetworkID": "cf9695f33b4ce7e72e2eb1a147827cfb3e32877c34cbf704eabd4c0cf1ccff96",
                    "EndpointID": "8221deba38c19823b08cf5c3330be50f370de79038eb16b34f8f4f63e04833f3",
                    "Gateway": "172.24.0.1",
                    "IPAddress": "172.24.0.5",
                    "IPPrefixLen": 16,
                    "IPv6Gateway": "",
                    "GlobalIPv6Address": "",
                    "GlobalIPv6PrefixLen": 0,
                    "MacAddress": "02:42:ac:18:00:05",
                    "DriverOpts": null
                }
            }
```

查看 apache-php的网络信息

Gateway为172.18.0.1，nginx也可以直接访问172.18.0.1的端口9000，也就是 apache-php docker中的80端口

```
"Networks": {
                "docker-php_default": {
                    "IPAMConfig": null,
                    "Links": null,
                    "Aliases": [
                        "php",
                        "f7cc8352b76b"
                    ],
                    "NetworkID": "bec0e0d19e2b6954c785eff4ca1512e6b0b07ab0c268a2858250157d895209c5",
                    "EndpointID": "5eb3d14b144d1f4b0cbb66fc281948591626d30167e335b29d66b76aeb45d9f0",
                    "Gateway": "172.18.0.1",
                    "IPAddress": "172.18.0.2",
                    "IPPrefixLen": 16,
                    "IPv6Gateway": "",
                    "GlobalIPv6Address": "",
                    "GlobalIPv6PrefixLen": 0,
                    "MacAddress": "02:42:ac:12:00:02",
                    "DriverOpts": null
                }
```

ngnix docker的配置

 将 /mojphp/ 映射到了 http://172.18.0.1:9000/; 也就是apache-php docker中的80端口

如果出现502bad way 一定是网络地址，端口错了，网络不通。

```
server {
    listen       80;
    server_name futari-passport.srcmaker.net;
    client_max_body_size 20m;
    # html vue

    location / {
                add_header Cache-Control 'no-cache, must-revalidate, proxy-revalidate, max-age=0';
        root /dist/certbot/;
        index  index.html;
        expires modified +90d;
    }

    location /mojphp/ {
            proxy_pass http://172.18.0.1:9000/;
    }

    # proxy the PHP scripts to Apache listening on 127.0.0.1:80

    # location ~ \.php$ {
    #     proxy_pass   http://127.0.0.1:9000;
    # }


    #    return 301 https://futari-passport.srcmaker.net:443$request_uri;
}
```

## 通过host ip 来访问

输入ifconfig

```sh
ifconfig
# apache-php network
br-bec0e0d19e2b: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 172.18.0.1  netmask 255.255.0.0  broadcast 172.18.255.255
        ether 02:42:48:32:fc:4f  txqueuelen 0  (Ethernet)
        RX packets 0  bytes 0 (0.0 B)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 0  bytes 0 (0.0 B)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
# futari-net network
br-cf9695f33b4c: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 172.24.0.1  netmask 255.255.0.0  broadcast 172.24.255.255
        ether 02:42:da:65:4b:29  txqueuelen 0  (Ethernet)
        RX packets 0  bytes 0 (0.0 B)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 0  bytes 0 (0.0 B)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
# host eth0
eth0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 10.1.0.23  netmask 255.255.240.0  broadcast 10.1.15.255
        ether 02:00:34:49:00:01  txqueuelen 1000  (Ethernet)
        RX packets 3873886  bytes 2281668243 (2.1 GiB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 3609466  bytes 1231828627 (1.1 GiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
        
# host eth1
eth1: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 172.20.0.198  netmask 255.255.0.0  broadcast 172.20.255.255
        ether 1e:00:33:00:05:0c  txqueuelen 1000  (Ethernet)
        RX packets 49245516  bytes 3742310607 (3.4 GiB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 2307  bytes 98345 (96.0 KiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

lo: flags=73<UP,LOOPBACK,RUNNING>  mtu 65536
        inet 127.0.0.1  netmask 255.0.0.0
        loop  txqueuelen 1  (Local Loopback)
        RX packets 726267  bytes 46799939 (44.6 MiB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 726267  bytes 46799939 (44.6 MiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
```

ngnix docker的配置

```
 # backup service forward
    location ^~/web/ {
            proxy_pass http://futari-spring:8088/;
        }
        
    location ^~/app/ {
        proxy_pass http://172.20.0.198:8088/app/;
    }
```

这里就通过host eth1 172.20.0.198:8088端口来访问 docker futari-spring的8088端口（因为docker futari-spring 映射了 8088 --> host 的8088）

ngnix docker 本身和futari-spring docker 在同一个虚拟网络 futari-net中，所以

- 可以通过容器名 futari-spring:8088来访问

- 也可以通过host 的ip来访问172.20.0.198:8088

- 还可以通过gateway来访问 172.24.0.1:8088(通过gateway就相当于访问host)