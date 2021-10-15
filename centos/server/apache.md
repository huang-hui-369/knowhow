# apache install后的默认情报

## 默认的配置文件路径
/etc/httpd/conf
/etc/httpd/conf/httpd.conf

## 默认的log路径
/var/log/httpd/
sudo tail /var/log/httpd/error_log

## 默认的document路径
/var/www
/var/www/html

## apache 运行的用户
apache/apache
查看/etc/httpd/conf中的user，group

![](img\2021-10-14-13-23-50.png)

## 更改document路径
![](img\2021-10-14-13-27-19.png)

![](img\2021-10-14-13-27-41.png)

![](img\2021-10-14-13-28-06.png)

### 更改document路径后的权限问题

更改document路径后会提示 403 Permission denied

1. 查看error log提示没有权限查询/home/ec2-user/www
    ```
    [Thu Oct 14 12:46:29.661594 2021] [core:notice] [pid 15667] AH00094: Command line: '/usr/sbin/httpd -D FOREGROUND'
    [Thu Oct 14 12:46:37.061111 2021] [core:error] [pid 15668] (13)Permission denied: [client 203.141.134.232:62495] AH00035: access to /test.html denied (filesystem path '/home/ec2-user/www') because search permissions are missing on a component of the path
    ```
2. 将apache用户加入到ec2-user group
    ```sh
    usermod -a -G ec2-user apache
    # 确认apache 用户的组
    id apache
    uid=48(apache) gid=48(apache) groups=48(apache),1000(ec2-user)
    ```    
    ![](img\2021-10-14-13-32-08.png)

3. 还是提示没有权限，这个时候你要查看不仅是www目录，还有ec2-user有没有权限，要把ec2-user目录设置为755
   `chmod 755 /home/ec2-user`

4. 重启apache
   `sudo systemctl restart httpd`   
   现在再访问就好了
   **注意apache用户要有读和执行（用来查询目录）的权限**




