# Postfix+SASL进行用户发邮件认证

通过验证配置可以发现，如果想发送邮件给外部（中继邮件）基本配置只能在mynetwork规定的ip范围内使用。这个方式在现实中也是不可行的。互联网上常用的方式是通过账号的认证方式允许中继邮件。但Postfix本身没有认证功能所以只能借助于第三方认证组件SASL来实现。与Postfix配合较好的SASL有：dovecot-SASL和cyrus-SASL，以及courier-authlib这几款组件。各有千秋，使用哪个根据实际选择即可。

### 1. 确定cyrus-sasl已安装
```
[root@localhost ~]# rpm -qa | grep  cyrus-sasl
cyrus-sasl-gssapi-2.1.23-13.el6_3.1.x86_64
cyrus-sasl-plain-2.1.23-13.el6_3.1.x86_64
cyrus-sasl-lib-2.1.23-13.el6_3.1.x86_64
cyrus-sasl-2.1.23-13.el6_3.1.x86_64
cyrus-sasl-md5-2.1.23-13.el6_3.1.x86_64
cyrus-sasl-devel-2.1.23-13.el6_3.1.x86_64
```

### 2. 确定Postfix支持sasl认证
```
[root@localhost ~]# postconf -a
cyrus
dovecot
```
  #默认支持cyrus和dovecot这两种认证方式

### 3. Postfix主配置添加以下内容
参照 ./main.cf
```
[root@localhost ~]# vim /etc/postfix/main.cf
###################CYRUS-SASL################
broken_sasl_auth_clients = yes
  #定义是否支持像outlook、foxmail等非标准协议认证
smtpd_sasl_auth_enable = yes
  #开启sasl验证用户功能
smtpd_sasl_local_domain = $myhostname
  #用于识别本地主机
smtpd_sasl_security_options = noanonymous
  #不支持匿名用户
smtpd_sasl_path = smtpd
  #指定使用sasl的程序名
smtpd_banner = welcome to smtp.ywnds.com
  #定义telnet连接时显示信息
smtpd_client_restrictions = permit_sasl_authenticated
  #用于限制客户端连接服务器
smtpd_sasl_authenticated_header = yes
  #从头信息查找用户名
smtpd_sender_restrictions = permit_mynetworks,reject_sender_login_mismatch
  #定义发件人规则
smtpd_recipient_restrictions=permit_mynetworks,permit_sasl_authenticated, reject_invalid_hostname,reject_unauth_destination
  #定义收件人规则
  #permit_mynetworks：允许本地网络
  #permit_sasl_authenticated：允许sasl认证过的用户发送邮件
  #reject_unauth_destination：拒绝没有经过认证的目标地址（这个一定要放在最后）
  #reject_invalid_hostname：HELO命令中的主机名称无效时返回501
  #reject_non_fqdn_hostname：HELO命令中的主机名称不是FQDN形式则返回504
  #reject_non_fqdn_recipient：收件地址不是FQDN则返回504
  #reject_non_fqdn_sender：发件地址不是FQDN则返回504
  #reject_unauth_pipelining：拒绝不守规定的流水线操作
  #reject_unknown_client：DNS查不出客户端IP的PTR记录时拒绝
  #reject_unknown_hostname：HELO命令中的主机名称没有A和MX记录时拒绝
  #reject_unknown_recipient_domain：收件人地址的网域部分查不出有效的A或MX记录时拒绝
  #reject_unknown_sender_domain：发件人地址的网域部分查不出有效的A或MX记录时拒绝
```
### 4 查看SASL支持哪些认证机制

```
[root@localhost ~]# saslauthd -v
saslauthd 2.1.23
authentication mechanisms: getpwent kerberos5 pam rimap shadow ldap
```

### 5 Postfix开启基于SASL用户认证

这里介绍2种认证方式，saslauthd和auxprop，一个是使用系统的账号来做认证，一个使用外部的账户来做认证，对于安全性来说，当然是使用外部的账号更安全了，这里介绍的使用sasldb2数据库，mysql的方式暂不介绍。2种方式人选其一即可。

登录方式1
```
Saslauthd
[root@localhost ~]# vim /usr/lib64/sasl2/smtpd.conf
pwcheck_method: saslauthd
mech_list: PLAIN LOGIN
```

登录方式2
```
Auxprop
[root@localhost ~]# vim /usr/lib64/sasl2/smtpd.conf
pwcheck_method: auxprop
auxprop_plugin: sasldb
mech_list: PLAIN LOGIN CRAM-MD5 DIGEST-MD5 NTLM
```

### 6 SASL配置文件/etc/sysconfig/saslauthd

SASL只是个认证框架，实现认证的是认证模块，而pam是sasl默认使用的认证模块。如果使用shadow做认证的话直接修改就可以不需要做其他任何配置了。

```
Saslauthd
[root@localhost ~]# vim /etc/sysconfig/saslauthd
SOCKETDIR=/var/run/saslauthd
#MECK= pam
MECK = shadow

Auxprop
[root@localhost ~]# vi /etc/sysconfig/saslauthd
#MECH=
FLAGS=sasldb

[root@localhost ~]# saslpasswd2 -c -u 'ywnds.com' redis
  #执行之后输入2次密码就可以了
[root@localhost ~]# sasldblistusers2
  #查看添加的用户
[root@localhost ~]# saslpasswd2 -d redis@ywnds.com
  #删除用户
[root@localhost ~]# chown postfix:postfix /etc/sasldb2
[root@localhost ~]# chmod 640 /etc/sasldb2
  #数据库权限修改
⑦重启服务

[root@localhost ~]# /usr/sbin/postfix reload
[root@localhost ~]# service saslauthd restart
[root@localhost ~]# chkconfig saslauthd on
测试账号

[root@localhost ~]# testsaslauthd -u hadoop -p hadoop
0: OK “Success”
```

### 7.重启服务
```
[root@localhost ~]# /usr/sbin/postfix reload
[root@localhost ~]# service saslauthd restart
[root@localhost ~]# chkconfig saslauthd on
测试账号

[root@localhost ~]# testsaslauthd -u hadoop -p hadoop
0: OK “Success”
```

### 8. SMTP认证通过telenet确认

![](img\2020-11-20-17-36-45.png)


### 9. Postfix内部邮件过滤

除了在上面配置文件中使用的一些过滤指令外，管理员也可以使用访问表(access map)来自定义限制条件，自定义访问表的条件通常使用check_client_access, check_helo_access, check_sender_access, check_recipient_access进行，它们后面通常跟上type:mapname格式的访问表类型和名称。其中，check_sender_access和check_recipient_access用来检查客户端提供的邮件地址，因此，其访问表中可以使用完整的邮件地址，如admin@magedu.com；也可以只使用域名，如magedu.com；还可以只有用户名的部分，如marion@

9.1 案例1 禁止172.16.100.66通过postfix服务发送邮件

(1)首先编辑/etc/postfix/access文件，以之做为客户端检查的控制文件，在里面定义如下一行：

172.16.100.66          REJECT

(2)将此文件转换为hash格式产生一个access.db文件

postmap /etc/postfix/access

(3)配置postfix使用此文件对客户端进行检查编辑/etc/postfix/main.cf文件添加如下参数：

smtpd_client_restrictions = check_client_access hash:/etc/postfix/access

(4)让postfix重新载入配置文件即可进行发信控制的效果测试了

9.2 案例2.这里以禁止通过本服务器向microsoft.com域发送邮件
(1)首先建立/etc/postfix/denydstdomains文件(文件名任取)在里面定义如下一行：

microsoft.com          REJECT

(2)将此文件转换为hash格式

postmap /etc/postfix/denydstdomains

(3)配置postfix使用此文件对客户端进行检查编辑/etc/postfix/main.cf文件添加如下参数：

smtpd_recipient_restrictions = check_recipient_access hash:/etc/postfix/denydstdomains, permit_mynetworks, reject_unauth_destination

(4)让postfix重新载入配置文件即可进行发信控制的效果测试了

## maillog

-/var/log/maillog