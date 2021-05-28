# DNS

域名系统（DNS，Domain Name System）是因特网的地址簿。DNS 通过映射不容易忘记的域名（例如example.com）到诸如192.0.2.8或0123:4567:89ab:cdef:0123:4567:89ab:cdef这样的 IP 地址。

## resource record组成
当你注册一个域名后，就可以设置多种 DNS 记录。每种记录都有一个 Type，一个 Host 和一个 Value
* Type 是提前定义好的
* Host 可以填根域名 (@) 或者子域名（www）
* Value 就是一个 IP 地址或者域名

## resource record类型
资源记录（resource record，简称 RR）组成了 DNS 数据库，常见的有以下几种：

* A 记录（Address record）：地址记录，把域名指向 IP 地址
* CNAME 记录（Canonical name record）：别名记录，把一个域名指向另一个域名，或其它 CNAME 记录、A 记录。 CNAME 缺点 一旦你为一个子域名定义了 CNAME 指向，就不能为这个子域名再定义其他类型的指向了，正由于此，你不能把 CNAME 定义在 根域名上。
* MX 记录（Mail exchange record）：邮件交换记录，指定用来接收消息的邮件服务器，如有多个邮件服务器，可指定各自的优先级
* TXT 记录（Text record）：文本记录，用于字符串
* AAAA 记录（IPv6 address record）：IPv6 地址记录，相当于支持 IPv6 的 A 记录
* CAA 记录（Certification Authority Authorization）：证书颁发机构授权记录，用来指定允许哪些 CA 机构为域名颁发证书,CAA 记录是一种证书安全机制，CA 机构颁发证书时会检查 CAA 记录，若未授权就拒绝为该域名颁发证书
* NAPTR 记录（Naming Authority Pointer）：允许通过正则表达式匹配并替换域名，比如网络电话系统把用户输入的电话号码转换成 SIP URI
* NS 记录（Name server record）：域名服务器记录，指定解析域名和子域名所使用的 DNS 服务器
* PTR 记录（PTR Resource Record）：PTR 记录，把 IP 地址指向域名，与 CNAME 的区别在于直接结束并返回结果，多用于反向解析（通过 IP 反查域名）
* SOA 记录（Start of authority record）：指定 DNS 权威信息，包括主域名服务器，域名管理员的邮箱，域名序列号等等
* SPF 记录（Sender Policy Framework record）：发件人策略框架记录，早期用于验证邮件发件人的身份，已废弃，建议使用 TXT 记录代替
* SRV 记录（Service locator record）：通用服务定位记录，指定服务所在的服务器（域名和端口号），多用于 SIP（Session Initiation Protocol，会话发起协议）

## dns 查询过程

以www.whitehouse.gov为例：

1. 客户端向本地 DNS 服务器发起递归查询（A 记录）
2. 本地 DNS 服务器向根 DNS 服务器发起迭代查询（A 记录）
3. 根 DNS 服务器返回.gov域名服务器的引用（A 记录）
4. 本地 DNS 服务器向.gov域名服务器发起迭代查询（A 记录）
5. .gov域名服务器返回whitehouse.gov域名服务器的引用（NS 记录）
6. 本地 DNS 服务器向whitehouse.gov域名服务器发起迭代查询（A 记录）
7. whitehouse.gov域名服务器回应迭代查询（www.whitehouse.gov的 IP 地址）
8. 本地 DNS 服务器回应最初的递归查询（www.whitehouse.gov的 IP 地址）

## 邮件相关的DNS设置

### 1. 邮件服务最常用到的DNS记录有A记录、MX记录、PTR记录、SPF记录。

* A记录
一个邮件服务器必须有一个A记录，A记录对应IPV4,AAAA对应IPV6。如QQ邮箱的其中的一个邮件服务器A记录：
mx1.qq.com      A       183.57.43.35

* MX记录
MX（Message Exchanger）记录是用来指定一个域的邮件服务器主机的记录。邮件服务器在发送邮件时，如找不到收件者域名的MX记录，会找该域的域名A记录，故一般都会将域名A记录与MX记录的主机名A记录设置相同的IP。MX记录值是一个主机名，而不是一个IP地址，这一点必须注意，实际中经常发现有些公司将MX记录的值设为IP地址，从而造成邮件无法正常接收。一个域可以有多个邮件服务器，即多个MX记录，不同的MX记录一般设置不同的优先级，优先级数值小的优先级高。


qq.com      MX preference = 10, mail exchanger = mx1.qq.com
qq.com      MX preference = 20, mail exchanger = mx2.qq.com
qq.com      MX preference = 30, mail exchanger = mx3.qq.com

* PTR记录
PTR（Pointer Record）记录,即反向解析记录，与MX记录相对，是用来将一个IP地址指向一个主机名，由ISP（Internet Service Provider）互联网服务提供商（如：中国电信）设置，可以多个IP指向同一个主机名。PTR记录是用来供邮件接收方验证发送方的邮件服务器主机。如果是租用的QQ、网易等企业邮箱，则不用设置PTR。如果邮件接收服务器与发送服务器不是同一台服务器，则PTR指向发送服务器，MX指向接收服务器。


```
nslookup
既定のサーバー:  UnKnown
Address:  192.168.1.1

> futari-passport.ansyou.net
サーバー:  UnKnown
Address:  192.168.1.1

権限のない回答:
名前:    futari-passport.ansyou.net
Address:  153.122.79.55

> set q=mx
 futari-passport.metro.tokyo.lg.jp
サーバー:  UnKnown
Address:  192.168.1.1

権限のない回答:
futari-passport.metro.tokyo.lg.jp       MX preference = 100, mail exchanger = futari-passport.metro.tokyo.lg.jp

> set q=ptr
> 153.122.79.55
サーバー:  UnKnown
Address:  192.168.1.1

権限のない回答:
55.79.122.153.in-addr.arpa      name = cc.ptr96.ptrcloud.net

```


* SPF记录、TXT记录
SPF（Sender Policy Framework 发件人策略框架）记录。SPF是为了防范垃圾邮件而提出来的一种DNS记录类型，它是一种TXT类型的记录，它用于登记某个域名拥有的用来外发邮件的所有IP地址。如果发送邮件服务器很多，则可能需要设置多个SPF记录。
SPF 记录包含在一个 TXT 记录之中，格式如下：
v=spf1 [[pre] type [ext] ] ... [mod]


1. unitoffice.com的设置
```
nslookup
既定のサーバー:  UnKnown
Address:  192.168.1.1
> set q=txt
> unitoffice.com
サーバー:  UnKnown
Address:  192.168.1.1

権限のない回答:
unitoffice.com  text =

        "v=spf1 include:_spf.heteml.jp ~all"
> _spf.heteml.jp
サーバー:  UnKnown
Address:  192.168.1.1

権限のない回答:
_spf.heteml.jp  text =

        "v=spf1 include:_spf01.heteml.jp include:_spf02.heteml.jp ~all"
> _spf01.heteml.jp
サーバー:  UnKnown
Address:  192.168.1.1

権限のない回答:
_spf01.heteml.jp        text =

        "v=spf1 +ip4:157.7.188.0/24 +ip4:157.7.189.0/24 ~all"

```

2. qq.com的设置
```
nslookup
既定のサーバー:  UnKnown
Address:  192.168.1.1
> set q=txt
> qq.com
サーバー:  UnKnown
Address:  192.168.1.1

権限のない回答:
qq.com  text =

        "v=spf1 include:spf.mail.qq.com -all"


> spf.mail.qq.com
サーバー:  UnKnown
Address:  192.168.1.1

権限のない回答:
spf.mail.qq.com text =

        "v=spf1 include:spf-a.mail.qq.com include:spf-b.mail.qq.com include:spf-c.mail.qq.com include:spf-d.mail.qq.com include:spf-e.mail.qq.com include:spf-f.mail.qq.com include:spf-g.mail.qq.com -all"

> spf-a.mail.qq.com
サーバー:  UnKnown
Address:  192.168.1.1

権限のない回答:
spf-a.mail.qq.com       text =

        "v=spf1 ip4:203.205.251.0/24 ip4:103.7.29.0/24 ip4:59.36.129.0/24 ip4:113.108.23.0/24 ip4:113.108.11.0/24 ip4:119.147.193.0/24 ip4:119.147.194.0/24 ip4:59.78.209.0/24 ip4:113.96.223.0/24 ip4:183.3.226.0/24 ip4:183.3.255.0/24 ip4:59.36.132.0/24 -all"
```

### 2. 一个完整的设置例子
unifoffice.com

* MX 记录
```
nslookup

> set q=mx
> unitoffice.com
サーバー:  UnKnown
Address:  192.168.1.1

権限のない回答:
unitoffice.com  MX preference = 10, mail exchanger = mx.hetemail.jp
```

* A 记录
```
> set q=a
> mx.hetemail.jp
サーバー:  UnKnown
Address:  192.168.1.1

権限のない回答:
名前:    mx.hetemail.jp
Address:  157.7.44.163
```

* PTR记录
```
> set q=ptr
> 157.7.44.163
サーバー:  UnKnown
Address:  192.168.1.1

権限のない回答:
163.44.7.157.in-addr.arpa       name = mx.heteml.jp
```

* SPF记录
```
> set q=txt
> unitoffice.com
サーバー:  UnKnown
Address:  192.168.1.1

権限のない回答:
unitoffice.com  text =

        "v=spf1 include:_spf.heteml.jp ~all"
> _spf.heteml.jp
サーバー:  UnKnown
Address:  192.168.1.1

権限のない回答:
_spf.heteml.jp  text =

        "v=spf1 include:_spf01.heteml.jp include:_spf02.heteml.jp ~all"
> _spf01.heteml.jp
サーバー:  UnKnown
Address:  192.168.1.1

権限のない回答:
_spf01.heteml.jp        text =

        "v=spf1 +ip4:157.7.188.0/24 +ip4:157.7.189.0/24 ~all"

```

### 3, DKIM记录

