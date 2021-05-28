# ssh服务
ssh服务是一个守护进程(demon)，系统后台监听客户端的连接，ssh服务端的进程名为sshd,负责实时监听客户端的请求(IP 22端口)
ssh服务端由2部分组成： openssh(提供ssh服务)    openssl(提供加密的程序)


## 常用命令

### 登录
`ssh -p22 omd@192.168.25.137 `

### scp copy
`scp -P22 -r -p /home/omd/h.txt omd@192.168.25.137:/home/omd/`

### sftp
```
sftp -oPort=22 root@192.168.25.137                   
put /etc/hosts /tmp                   
get /etc/hosts /home/omd   
```
## 查看ssh的秘钥目录
`ll /root/.ssh/known_hosts  # 当前用户家目录的.ssh目录下`

## ssh的配置文件
`cat /etc/ssh/sshd_config`

### 查询sshd进程
`ps -ef | grep ssh`

### 查看ssh端口
`netstat -a | grep ssh`

## 如何防止SSH登录入侵
1. 更改端口
2. 禁止密码登录，改为密钥登录
3. 限制连接ip

