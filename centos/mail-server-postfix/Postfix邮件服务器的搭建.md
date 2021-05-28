#

##　mail 流程
postifx smtp协议 用于对外收发信件
dovecot pop3，imap协议，用户对内将信件传送到每个信箱客户端（Foxmail）
![](img\2021-04-19-18-31-03.png)

A主机某用户给B主机某用户发送邮件，主机A会通过SMTP联系B主机的SMTPD（通常也称作MTA，但由于它有客户端，所以分开写），先问它是否在线，对方给予回复，然后再告诉他邮件来自哪里，对方给予响应，再告诉它邮件发给谁的，对方给予响应，然后再发内容，最后以.结束。这是邮件开始发送，最后到达了SMTPD，SMTPD通过查询会知道这个邮件是发送给本机用户还是发送其它主机的邮件，如果是发送给本机的邮件它会调用本机的MDA把邮件投递到用户的mailbox中，如果不是本机的用户，就帮忙中继到其它主机，其它主机收到了就跟这个过程一样。

## 功能模块

操作系统（OS）CentOS 6.5 64位CentOS和RHEL是一样的，而且升级免费

数据库/目录服务MySQL 5.1.73CentOS 自带

邮件传输代理（MTA） postfix-2.6.6 使用最新版本2.6.6

邮件投递代理（MDA） maildrop 2.7.2 支持过滤和强大功能

Web帐户管理后台ExtMan 1.1 支持无限域名、无限用户

WebMail 系统ExtMail 1.2.0 支持多语言、全部模板化，功能基本齐全

日志分析及显示 mailgraph_ext 在ExtMan中已经包含了

其他数据认证库 Courier Authlb 0.66.1 负责courier-imap,maildrop的认证

SMTP认证库 Cyrus SASL 2.1.23 标准的SASL实现库，可以支持Courier authlib

内容过滤器 Amavisd-new 2.6.6 Content-Filter软件，支持与Camav/SA的挂接

内容级别的反垃圾邮件工具SpamAssassin-3.3.1 著名的SA，可以支持大量规则

防病毒软件（Anti-Virus）ClamAV 0.98.4 最热门的开源杀毒软件

SMTP阶段反垃圾邮件工具 Spam Locker 0.99 基于SMTP行为识别的Antispam软件，大量可选插件

高效的反垃圾邮件工具 Dspam-3.10.2 高精确度的、智能的过滤功能

## postfix结构图
![](img\2021-04-19-20-14-10.png)


## 1. mail域名设置

邮件服务器的DNS设置

DNS记录，需要你到你的域名托管商那里进行设置或者你自己管理DNS服务器。不少域名托管商不支持txt记录或者不支持DKIM记录，所以你就无法使用SPF和DKIM的功能。
DNS的修改，需要48小时以上才能生效。
国内的万网是不支持DKIM，目前新网是支持SPF和DKIM。

1.MX记录
邮件的MX记录最好是指向机器A记录，尽量不要直接指向IP地址（不符合规范）。
1.1 添加A记录
mail.example.com 192.168.1.100
1.2 添加MX记录
example.com mail.example.com
1.3 测试MX记录
host exmple.com
example.com mail is handled by 10 mail.example.com.
nslookup mail.example.com
Name:mail.example.com
Address:192.168.1.100

2.SPF记录
SPF是指Sender Policy Framework，是为了防范垃圾邮件而提出来的一种DNS记录类型，SPF是一种TXT类型的记录。SPF记录的本质，就是向收件人宣告：本域名的邮件从清单上所列IP发出的都是合法邮件，并非冒充的垃圾邮件。设置好SPF是正确设置邮件发送的域名记录和STMP的非常重要的一步。
例如：
SPF 记录指向A主机记录
example.com.           3600    IN      TXT     "v=spf1 mx mx:mail.example.com ~all"
SPF 记录指向IP地址
example.com.          3600    IN      TXT     "v=spf1 ip4:192.168.1.100 ~all"

2.1 如何查看SPF
Windows下进入DOS模式后用以下命令:
nslookup -type=txt 域名
Unix操作系统下用：
dig -t txt 域名

2.2 SPF的简单说明如下：
v=spf1 表示 spf1 的版本
IP4 代表IPv4进行验证(IP6代表用IPv6进行验证), 注意 “ip4:” 和“IP”之间是没有空格的
~all 代表结束

2.3 SPF记录例释
我们看这条SPF:
yourdomain.com "v=spf1 a mx mx:mail.jefflei.com ip4:202.96.88.88 ~all"
这条SPF记录具体的说明了允许发送 @yourdomain.com 的IP地址是：a （这个a是指 yourdomain.com 解析出来的IP地址，若没有配置应取消）
mx （yourdomain.com 对应的mx，即 mail.yourdomain.com的A记录所对应的ip）
mx:mail.jefflei.com （如果没有配置过mail.jefflei.com这条MX记录也应取消）
ip4:202.96.88.88 （直接就是 202.152.186.85 这个IP地址）
其他还有些语法如下：
- Fail, 表示没有其他任何匹配发生
~ 代表软失败，通常用于测试中
? 代表忽略

如果外发的ip不止一个，那么必须要包含多个
v=spf1 ip4:202.96.88.88 ip4:202.96.88.87 ~all

2.4 测试SPF设置结果
设置好 DNS中的SPF记录后，发送邮件给自己的gmail，然后查看邮件的源文件，应该能看到类似的邮件头，其中有pass表示设置成功。
Received-SPF: pass (google.com: domain of test@jefflei.com designates
    202.96.88.87 as permitted sender) client-ip=202.96.88.87;
需要注意的是，服务器的IP若有更改，需要同时修改SPF！！！

2.5 利用SPF记录防止垃圾邮件
在Unix下可以安装配置SpamAssassin之类的插件来防止垃圾邮件和钓鱼邮件(Phishing)


## 2. 送信设置

```
// 检查postfix是否支持cyrus dovecot功能
postconf -a

cyrus
dovecot
```

1. 编辑设置/etc/postfix/main.cf

**inet_interfaces** 是设置允许进行smtp连接的网段，只要inet_interfaces中的网段都可以不进行用户名密码验证，就通过smtp发信。

**mydestination** 是设置收信的域名。

```
vi /etc/postfix/main.cf

myhostname = mail.test.com                    //本机主机名

mydomain = test.com                          //服务器域名

myorigin = $mydomain                        //初始域名

inet_interfaces = 192.168.80.181, 127.0.0.1        //监听接口 可以允许连接的网段，因为smtp可以不进行用户名密码验证。

inet_protocols = ipv4                      //监听网络版本，可以不改

mydestination = $myhostname, $mydomain      //目标域

home_mailbox = Maildir/                    //邮件目录，在用户家目录下
```

postfix check            //检查配置文件是否有语法错误

systemctl start postfix      //启动postfix服务器



3. 通过 telnet 进行简单邮件发信测试


输入如下命令测试

```
telnet mail.test.com 25   //远程登录25端口

helo mail.test.com                           //声明本机的主机名

mail from:jack@aa.com                     //声明发件人地址

rcpt to:xxx@163.com                      //声明收件人地址

data                                  //写正文

i am jack!!

.                                   //以.结尾

quit                                 // 退出
```

![](img\2021-04-19-20-30-28.png)

ls /home/tom/Maildir/new/              //查看tom接收的邮件目录下的邮件

[root@mail named]# cat /home/tom/Maildir/new/1515869757.V805I600041M500758.mail.test.com   //查看tom接收的邮件内容

这是仅限于内网发邮件.

## 3.  收信设置

Dovecot的作用：
* dovecot当然就是一个MRA(IMAP与POP3服务器)，负责接收外界发送到本机的邮件。
* dovecot不仅仅负责邮件的接收和操作，而且负责素有的身份认证。

### 1. 安装dovecot提供收信服务

yum install -y dovecot               //安装dovecot服务

### 2. 编辑修改配置文件

1. vi /etc/dovecot/dovecot.conf

设置一下内容
```
protocols = imap pop3

listen = *

```

2. 命令vim /etc/dovecot/conf.d/10-ssl.conf

ssl = no

3. 命令vim /etc/dovecot/conf.d/10-auth.conf
disable_plaintext_auth=no
auth_mechanisms=plainlogin
!includeauth-system.conf.ext

4. vim/etc/dovecot/conf.d/10-logging.conf
添加
info_log_path=/var/log/dovecot_info.log
debug_log_path=/var/log/dovecot_debug.log

5. vim/etc/dovecot/conf.d/10-mail.conf取消以下注释
mail_location=mbox:~/mail:INBOX=/var/mail/%u#指定邮件的位置
然后启动服务
systemctl start dovecot
systemctl enable dovecot

6. 确认服务状态
netstat -anpt | grep dovecot         //查看110和143端口状态，处于监听状态则服务器开启成功

### 3 测试收信功能

1. 添加邮件用户

```
 groupadd mailusers          //添加邮件账号组mailusers

 useradd -g mailusers -s /sbin/nologin jack //添加邮件用户jack

 passwd jack

 useradd -g mailusers -s /sbin/nologin tom //添加邮件用户tom
 passwd tom                   
```

mkdir-p~/mail/.imap/INBOX

2. 
```
telnet mail.test.com 110         //登录连接110端口测试收信

user tom                //用户tom

pass 123                //密码123

list                   //列表邮件

1 381

.

retr 1

quit            //退出
```

dovecot 默认使用linux系统用户来进行认证的。

## 4. 发信认证配置--dovecot
发信认证配置网上说有两种方式，
1. 通过 dovecot 来认证

2. 通过 cyrus-sasl 来认证

下面给设置通过dovecot 来认证

设置 /etc/postfix/main.cf

使用Dovecot验证方式
smtpd_sasl_type = dovecot
地址一般都是这个
smtpd_sasl_path = private/auth
不管是Dovecot或者Cyrus都需要启动
smtpd_sasl_auth_enable = yes


## 4. 发信认证配置--cyrus-sasl

<font color=red>我觉得这个好像可以不设置</font>

1. 安装cyrus-sasl软件-------------
 yum install -y cyrus-sasl cyrus-sasl-scram //安装SMTP 认证服务，必须scram包

2. 修改SMTP配置文件---------------
vi /etc/sasl2/smtpd.conf
 
```
pwcheck_method:saslauthd
mech_list: plainlogin
log_level:3                 //修改日志等级为3便于排错
```

3. 修改saslauthd配置文件---------------
vi /etc/sysconfig/saslauthd
```
MECH=shadow                      //修改认证为shadow
``` 

4. 修改postfix主配置文件，在末尾添加以下内容---------------
vi /etc/sysconfig/saslauthd
```
smtpd_sasl_auth_enable= yes                       //开启认证
smtpd_sasl_security_options= noanonymous           //不允许匿名发信
mynetworks =127.0.0.0/8           //允许的网段,如果增加本机所在网段就会出现允许不验证也能向外域发信，192.168.80.0/24为本机网段，为了测试效果不予加入。
smtpd_recipient_restrictions= permit_mynetworks,permit_sasl_authenticated,reject_unauth_destination    //允许本地域以及认证成功的发信，拒绝认证失败的发信
``` 

5. 启动服务--------------
systemctl start saslauthd
systemctl enable saslauthd
systemctl restart postfix
systemctl restart dovecot

6. 字符终端测试认证发信
这里我们可以用base64生成密文通过SMTP认证发信

printf "jack" | openssl base64                   //生成jack用户名的密文
amFjaw==

[root@mail ~]#printf "123" | openssl base64                  //生成密码123的密文
MTIz

```
telnet mail.test.com 25

输入ehlo mail.test.com

显示以下内容则表示支持PLAIN LOGIN两种认证方式

继续输入

auth login                                      //认证登录

amFjaw==                                      //密文jack用户名

MTIz                                          // 密文jack密码

mailfrom:jack@aa.com

rcptto:1228308235@qq.com                   //用jack发送邮件给我个人的邮箱

data

this is test

.                                          //邮件内容

quit
```


## 5. 设置tls

### 通过Letsencrypt申请证书

### 修改配置文件

1. vim /etc/postfix/main.cf

```
# 添加到最后
// 收信时
smtpd_tls_security_level = may
smtpd_tls_cert_file = /etc/pki/tls/certs/server.crt
smtpd_tls_key_file = /etc/pki/tls/certs/server.key
smtpd_tls_loglevel = 0

// 发信时
smtp_tls_key_file = /home/xxx/ssl_keys/xxx.yyy.net/RSA_Key.ppk
smtp_tls_cert_file = /home/xxx/ssl_keys/xxx.yyy.net/cert.pem
smtp_tls_security_level = may
```
设置完这个后，重启邮件服务后，向其他邮件服务器发送邮件时，如果对方支持tls就用tls发送，如果对方不支持就用 25 发送邮件。
systemctl restart postfix


2. vim /etc/postfix/master.cf
   就会监听465（smtps)端口
```
# 17-18行: 取消注释
smtps       inet   n       -       n       -       -       smtpd
  -o smtpd_tls_wrappermode=yes
```

3. vim /etc/dovecot/conf.d/10-ssl.conf
```
# 6行: 取消注释
ssl = yes
# 12,13行: 指定证书
ssl_cert = </etc/pki/tls/certs/server.crt
ssl_key = </etc/pki/tls/certs/server.key
```

4. 开放SSL端口(端口的话，SMTP使用的是465, POP3使用995, IMAP使用993)
firewall-cmd --add-port={465/tcp,995/tcp,993/tcp} --permanent
firewall-cmd --reload 

5. 重启postfix
[root@mail ~]# systemctl restart postfix
Shutting down postfix: [ OK ]
Starting postfix: [ OK ]

6. 重启dovecot
[root@mail ~]# systemctl restart dovecot


## 6. 设置虚拟用户

检查postfix是否支持虚拟用户 postconf -m 
如果出现mysql表示支持

mysql安装
mysql安装我这边就不多说什么，附上命令：
```
yum install mariadb-server mariadb 
#启动
service mysqld start
#初始化密码
mysql_secure_installation
#然后一路  N 到底
#远程连接
grant all privileges  on *.* to root@'%' identified by "password";
flush privileges;
```

sql语句（后续使用）：

CREATE TABLE `admin` (
  `username` varchar(255) NOT NULL,
  `password` varchar(255) NOT NULL,
  `superadmin` tinyint(1) NOT NULL DEFAULT '0',
  `created` datetime NOT NULL DEFAULT '2000-01-01 00:00:00',
  `modified` datetime NOT NULL DEFAULT '2000-01-01 00:00:00',
  `active` tinyint(1) NOT NULL DEFAULT '1',
  `phone` varchar(30) CHARACTER SET utf8 NOT NULL DEFAULT '',
  `email_other` varchar(255) CHARACTER SET utf8 NOT NULL DEFAULT '',
  `token` varchar(255) CHARACTER SET utf8 NOT NULL DEFAULT '',
  `token_validity` datetime NOT NULL DEFAULT '2000-01-01 00:00:00',
  PRIMARY KEY (`username`)
) ENGINE=InnoDB DEFAULT CHARSET=latin1 COMMENT='Postfix Admin - Virtual Admins';

CREATE TABLE `alias` (
  `address` varchar(255) NOT NULL,
  `goto` text NOT NULL,
  `domain` varchar(255) NOT NULL,
  `created` datetime NOT NULL DEFAULT '2000-01-01 00:00:00',
  `modified` datetime NOT NULL DEFAULT '2000-01-01 00:00:00',
  `active` tinyint(1) NOT NULL DEFAULT '1',
  PRIMARY KEY (`address`),
  KEY `domain` (`domain`)
) ENGINE=InnoDB DEFAULT CHARSET=latin1 COMMENT='Postfix Admin - Virtual Aliases';

CREATE TABLE `alias_domain` (
  `alias_domain` varchar(255) NOT NULL DEFAULT '',
  `target_domain` varchar(255) NOT NULL DEFAULT '',
  `created` datetime NOT NULL DEFAULT '2000-01-01 00:00:00',
  `modified` datetime NOT NULL DEFAULT '2000-01-01 00:00:00',
  `active` tinyint(1) NOT NULL DEFAULT '1',
  PRIMARY KEY (`alias_domain`),
  KEY `active` (`active`),
  KEY `target_domain` (`target_domain`)
) ENGINE=InnoDB DEFAULT CHARSET=latin1 COMMENT='Postfix Admin - Domain Aliases';
CREATE TABLE `config` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `name` varchar(20) NOT NULL DEFAULT '',
  `value` varchar(20) NOT NULL DEFAULT '',
  PRIMARY KEY (`id`),
  UNIQUE KEY `name` (`name`)
) ENGINE=InnoDB AUTO_INCREMENT=2 DEFAULT CHARSET=latin1 COMMENT='PostfixAdmin settings';

CREATE TABLE `domain` (
  `domain` varchar(255) NOT NULL,
  `description` varchar(255) CHARACTER SET utf8 NOT NULL,
  `aliases` int(10) NOT NULL DEFAULT '0',
  `mailboxes` int(10) NOT NULL DEFAULT '0',
  `maxquota` bigint(20) NOT NULL DEFAULT '0',
  `quota` bigint(20) NOT NULL DEFAULT '0',
  `transport` varchar(255) NOT NULL,
  `backupmx` tinyint(1) NOT NULL DEFAULT '0',
  `created` datetime NOT NULL DEFAULT '2000-01-01 00:00:00',
  `modified` datetime NOT NULL DEFAULT '2000-01-01 00:00:00',
  `active` tinyint(1) NOT NULL DEFAULT '1',
  PRIMARY KEY (`domain`)
) ENGINE=InnoDB DEFAULT CHARSET=latin1 COMMENT='Postfix Admin - Virtual Domains';
CREATE TABLE `domain_admins` (
  `username` varchar(255) NOT NULL,
  `domain` varchar(255) NOT NULL,
  `created` datetime NOT NULL DEFAULT '2000-01-01 00:00:00',
  `active` tinyint(1) NOT NULL DEFAULT '1',
  KEY `username` (`username`)
) ENGINE=InnoDB DEFAULT CHARSET=latin1 COMMENT='Postfix Admin - Domain Admins';

CREATE TABLE `fetchmail` (
  `id` int(11) unsigned NOT NULL AUTO_INCREMENT,
  `domain` varchar(255) DEFAULT '',
  `mailbox` varchar(255) NOT NULL,
  `src_server` varchar(255) NOT NULL,
  `src_auth` enum('password','kerberos_v5','kerberos','kerberos_v4','gssapi','cram-md5','otp','ntlm','msn','ssh','any') DEFAULT NULL,
  `src_user` varchar(255) NOT NULL,
  `src_password` varchar(255) NOT NULL,
  `src_folder` varchar(255) NOT NULL,
  `poll_time` int(11) unsigned NOT NULL DEFAULT '10',
  `fetchall` tinyint(1) unsigned NOT NULL DEFAULT '0',
  `keep` tinyint(1) unsigned NOT NULL DEFAULT '0',
  `protocol` enum('POP3','IMAP','POP2','ETRN','AUTO') DEFAULT NULL,
  `usessl` tinyint(1) unsigned NOT NULL DEFAULT '0',
  `sslcertck` tinyint(1) NOT NULL DEFAULT '0',
  `sslcertpath` varchar(255) CHARACTER SET utf8 DEFAULT '',
  `sslfingerprint` varchar(255) DEFAULT '',
  `extra_options` text,
  `returned_text` text,
  `mda` varchar(255) NOT NULL,
  `date` timestamp NOT NULL DEFAULT '2000-01-01 00:00:00',
  `created` timestamp NOT NULL DEFAULT '2000-01-01 00:00:00',
  `modified` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP,
  `active` tinyint(1) NOT NULL DEFAULT '0',
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=latin1;

CREATE TABLE `log` (
  `timestamp` datetime NOT NULL DEFAULT '2000-01-01 00:00:00',
  `username` varchar(255) NOT NULL,
  `domain` varchar(255) NOT NULL,
  `action` varchar(255) NOT NULL,
  `data` text NOT NULL,
  `id` int(11) NOT NULL AUTO_INCREMENT,
  PRIMARY KEY (`id`),
  KEY `timestamp` (`timestamp`),
  KEY `domain_timestamp` (`domain`,`timestamp`)
) ENGINE=InnoDB AUTO_INCREMENT=9 DEFAULT CHARSET=latin1 COMMENT='Postfix Admin - Log';

CREATE TABLE `mailbox` (
  `username` varchar(255) NOT NULL,
  `password` varchar(255) NOT NULL,
  `name` varchar(255) CHARACTER SET utf8 NOT NULL,
  `maildir` varchar(255) NOT NULL,
  `quota` bigint(20) NOT NULL DEFAULT '0',
  `local_part` varchar(255) NOT NULL,
  `domain` varchar(255) NOT NULL,
  `created` datetime NOT NULL DEFAULT '2000-01-01 00:00:00',
  `modified` datetime NOT NULL DEFAULT '2000-01-01 00:00:00',
  `active` tinyint(1) NOT NULL DEFAULT '1',
  `phone` varchar(30) CHARACTER SET utf8 NOT NULL DEFAULT '',
  `email_other` varchar(255) CHARACTER SET utf8 NOT NULL DEFAULT '',
  `token` varchar(255) CHARACTER SET utf8 NOT NULL DEFAULT '',
  `token_validity` datetime NOT NULL DEFAULT '2000-01-01 00:00:00',
  PRIMARY KEY (`username`),
  KEY `domain` (`domain`)
) ENGINE=InnoDB DEFAULT CHARSET=latin1 COMMENT='Postfix Admin - Virtual Mailboxes';

CREATE TABLE `quota` (
  `username` varchar(255) NOT NULL,
  `path` varchar(100) NOT NULL,
  `current` bigint(20) NOT NULL DEFAULT '0',
  PRIMARY KEY (`username`,`path`)
) ENGINE=InnoDB DEFAULT CHARSET=latin1;

CREATE TABLE `quota2` (
  `username` varchar(100) NOT NULL,
  `bytes` bigint(20) NOT NULL DEFAULT '0',
  `messages` int(11) NOT NULL DEFAULT '0',
  PRIMARY KEY (`username`)
) ENGINE=InnoDB DEFAULT CHARSET=latin1;

CREATE TABLE `vacation` (
  `email` varchar(255) NOT NULL,
  `subject` varchar(255) NOT NULL,
  `body` text NOT NULL,
  `activefrom` timestamp NOT NULL DEFAULT '2000-01-01 00:00:00',
  `activeuntil` timestamp NOT NULL DEFAULT '2038-01-18 00:00:00',
  `cache` text NOT NULL,
  `domain` varchar(255) NOT NULL,
  `interval_time` int(11) NOT NULL DEFAULT '0',
  `created` datetime NOT NULL DEFAULT '2000-01-01 00:00:00',
  `modified` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP,
  `active` tinyint(1) NOT NULL DEFAULT '1',
  PRIMARY KEY (`email`),
  KEY `email` (`email`)
) ENGINE=InnoDB DEFAULT CHARSET=latin1 COMMENT='Postfix Admin - Virtual Vacation';

CREATE TABLE `vacation_notification` (
  `on_vacation` varchar(255) CHARACTER SET latin1 NOT NULL,
  `notified` varchar(255) CHARACTER SET latin1 NOT NULL DEFAULT '',
  `notified_at` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP,
  PRIMARY KEY (`on_vacation`,`notified`),
  CONSTRAINT `vacation_notification_pkey` FOREIGN KEY (`on_vacation`) REFERENCES `vacation` (`email`) ON DELETE CASCADE
) ENGINE=InnoDB DEFAULT CHARSET=utf8 COMMENT='Postfix Admin - Virtual Vacation Notifications';

创建虚拟用户
记住这里的5000 下面配置需要使用到 这里5000相当于uid 和gid
groupadd -g 5000 vmail
useradd -g vmail -u 5000 -s /sbin/nologin vmail

虚拟用户的作用：
邮箱用户的验证，一般有2种：

采用系统用户登录发送邮件。系统用户=邮件用户。（比如这里的vmail用户。我们可以直接用来发送邮件）
vmail 作为一个统一入口，任何人通过postfix发送或者接收邮件都认识是这个系统用户，但是具体的是谁需要通过后面的mysql中的帐号密码去验证。有个比喻很恰当。
虚拟用户不同于系统中已经存在的用户（/etc/passwd)中的用户，而是借助于一个普通的系统用户，实现多个用户共享的认证和使用。类似于普通系统用户就是一张电影票。你也可以拿这张票去看电影，我也可以那这张票去看电影。只要是获取了这张票，就可以坐在相应的位置上，享受电影服务，类比到系统中，就类似于获取了该实体用户的属性，如读写等权限，至于获取这张票的过程，牵扯到SASL认证

postfix配置
postfix 配置一共有个地方需要配置：一个是main.cf 文件 一个是master.cf 文件。

main.cf
打开 /etc/postfix/main.cf 配置文件：
```
#使用Dovecot验证方式
smtpd_sasl_type = dovecot
# 地址一般都是这个

smtpd_sasl_path = private/auth
#不管是Dovecot或者Cyrus都需要启动
smtpd_sasl_auth_enable = yes
```
```
#启用tls认证，不使用25端口
smtpd_tls_cert_file=/etc/pki/dovecot/certs/dovecot.pem
smtpd_tls_key_file=/etc/pki/dovecot/private/dovecot.pem
smtpd_use_tls=yes
#end
smtpd_sasl_local_domain = $myhostname
broken_sasl_auth_clients = yes
smtpd_recipient_restrictions = permit_mynetworks,permit_sasl_authenticated,reject_unauth_destination,reject_unknown_sender_domain
smtpd_client_restrictions = permit_sasl_authenticated
smtpd_sasl_security_options = noanonymous
proxy_read_maps = $local_recipient_maps $mydestination $virtual_alias_maps $virtual_alias_domains $virtual_mailbox_maps $virtual_mailbox_domains $relay_recipient_maps $relay_domains $canonical_maps $sender_canonical_maps $recipient_canonical_maps $relocated_maps $transport_maps $mynetwork
#虚拟用户文件所在地址
virtual_mailbox_base = /mnt/vmail/
virtual_mailbox_domains = proxy:mysql:/etc/postfix/sql/mysql_virtual_domains_maps.cf
virtual_alias_maps =
   proxy:mysql:/etc/postfix/sql/mysql_virtual_alias_maps.cf,
   proxy:mysql:/etc/postfix/sql/mysql_virtual_alias_domain_maps.cf,
   proxy:mysql:/etc/postfix/sql/mysql_virtual_alias_domain_catchall_maps.cf
virtual_mailbox_maps =
   proxy:mysql:/etc/postfix/sql/mysql_virtual_mailbox_maps.cf,
   proxy:mysql:/etc/postfix/sql/mysql_virtual_alias_domain_mailbox_maps.cf
#如果所有用户的邮箱都使用一个uid，那么指定一个静态的uid
virtual_uid_maps = static:5000
#虚拟用户gid
virtual_gid_maps = static:5000
#虚拟投递代理
#virtual_transport = dovecot
virtual_transport = lmtp:unix:private/dovecot-lmtp
# multiple email
dovecot_destination_recipient_limit = 1
```

virtual_alias_maps 和 virtual_mailbox_maps 配置需要注意下，这里是使用的配置文件去配置虚拟的别名和用户。具体内容入下：

mysql_virtual_domains_maps.cf
```
#数据库帐号
user = root
#数据库密码
password = root123
# 数据ip地址
hosts = 127.0.0.1:3306
#数据库名
dbname = postfix
#查询语句
query = SELECT domain FROM domain WHERE domain='%s' AND active = '1';
```

```
mysql_virtual_alias_domain_mailbox_maps.cf
user = root
password = root123
hosts = 127.0.0.1:3306
dbname = postfix
query = SELECT maildir FROM mailbox,alias_domain WHERE alias_domain.alias_domain = '%d' and mailbox.username = CONCAT('%u','@',alias_domain.target_domain) AND mailbox.active = 1 AND alias_domain.active='1';
```

```
mysql_virtual_alias_domain_maps.cf
user = root
password = root123
hosts = 127.0.0.1:3306
dbname = postfix
query = SELECT goto FROM alias,alias_domain WHERE alias_domain.alias_domain = '%d' and alias.address = CONCAT('%u', '@', alias_domain.target_domain) AND alias.active = 1 AND alias_domain.active='1';
```

```
mysql_virtual_alias_maps.cf
user = root
password = root123
hosts = 127.0.0.1:3306
dbname = postfix
query = SELECT goto FROM alias WHERE address='%s' AND active = '1';
```

```
mysql_virtual_domains_maps.cf
user = root
password = root123
hosts = 127.0.0.1:3306
dbname = postfix
query = SELECT domain FROM domain WHERE domain='%s' AND active = '1';
```
```
mysql_virtual_mailbox_limit_maps.cf
user = root
password = root123
hosts = 127.0.0.1:3306
dbname = postfix
query = SELECT quota FROM mailbox WHERE username='%s' AND active = '1';
```
```
mysql_virtual_mailbox_maps.cf

user = root
password = root123
hosts = 127.0.0.1:3306
dbname = postfix
query = SELECT maildir FROM mailbox WHERE username='%s' AND active = '1';
```

master.cf

master一共2个地方需要修改：

第一个是对应开启tls验证，大约在26行：
```
 smtps     inet  n       -       n       -       -       smtpd
 #  -o syslog_name=postfix/smtps
   -o smtpd_tls_wrappermode=yes
```

第二个是配置postfix使用Dovecot的LDA服务（Local Delivery Agent）本地传递服务，一脸懵逼，这是干嘛的。我真的不是清楚：Dovecot官网有一句话是这么说：
Dovecot LDA是本地传递代理，它从MTA接收邮件并将其传递到用户的邮箱

全是中文，然而我还是不懂。
master.cf 末尾加上：
```
dovecot   unix  -       n       n       -       -       pipe
     flags=DRhu user=vmail:vmail argv=/usr/libexec/dovecot/deliver -f ${sender} -d ${recipient}
```







## postfix目录

Postfix目录
/etc/postfix/

主配置目录，包含主配置文件，各类脚本，查询表等。

/etc/postfix/main.cf

主配置文件

/etc/postfix/master.cf

master程序配置文件

/usr/libexec/postfix/

服务器程序目录

/var/spool/postfix/

邮件队列相关目录

/var/spool/postfix/incoming/

传入队列

/var/spool/postfix/active/

活动队列

/var/spool/postfix/deferred/

推迟队列

/var/spool/postfix/hold/

约束队列

/var/spool/postfix/corrupt/

错误队列

/usr/sbin/

服务管理程序目录

/usr/sbin/postalais

用于构造，修改和查询别名表

/usr/sbin/postconf

用于显示和编辑主配置文件

/usr/sbin/postfix

用于启动和停止postfix服务

/usr/sbin/postmap

用于构造，修改或查询查询表

/usr/sbin/postqueue

用于一般用户管理邮件队列

/usr/sbin/postsuper

用于超级用户管理邮件对列

Postfix日志文件
Postfix系统的日志文件位于 /var/log/maillog 文件中，该文件记录了Postfix服务器运行状态信息

在安装调试Postfix邮件系统及日常维护过程中，经常会使用 tail 命令带 -f 选项实时观察日志内容变化。

tail -f /var/log/maillog

https://blog.csdn.net/weixin_33957648/article/details/92323879?utm_medium=distribute.pc_relevant.none-task-blog-BlogCommendFromMachineLearnPai2-17.control&dist_request_id=1328689.20011.16166472995119079&depth_1-utm_source=distribute.pc_relevant.none-task-blog-BlogCommendFromMachineLearnPai2-17.control


https://www.jianshu.com/p/ffe2182c12f3?utm_campaign=hugo

```

helo loaclhost
mail from:futari@futari-passport.ansyou.net
rcpt to:hhui258369@gmail.com
data
subject:こんにちは
from: futari@futari-passport.ansyou.net
to: huanghui_rock@outlook.jp

こんにちは
。
.
quit

```

$ /usr/sbin/postconf
オプション
- なし　⇒　現在の設定値を表示
- d　⇒　デフォルト値を表示
- n　⇒　デフォルトと異なる設定値のみ表示