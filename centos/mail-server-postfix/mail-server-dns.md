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
# host exmple.com
example.com mail is handled by 10 mail.example.com.
#nslookup mail.example.com
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
# dig -t txt 域名

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

# mail 認証
SPF、DKIM、DMARC 認証 - Gmail では、送受信されるメールが認証済みであることを検証することによってメールを保護しているため、認証されていないメールは迷惑メールに分類されることがあります。メールの認証は、SPF、DKIM、DMARC を使って行われます。

原文链接：https://blog.csdn.net/mal327/article/details/6700493