# HTTPS の接続確認

```sh
[root@STG ~]openssl s_client -connect www.google.com:443 -quiet

depth=3 C = BE, O = GlobalSign nv-sa, OU = Root CA, CN = GlobalSign Root CA
verify return:1
depth=2 C = US, O = Google Trust Services LLC, CN = GTS Root R1
verify return:1
depth=1 C = US, O = Google Trust Services LLC, CN = GTS CA 1C3
verify return:1
depth=0 CN = www.google.com
verify return:1
read:errno=0
```

# 特定プロトコルで通信させる

```sh
[root@STG ~]# openssl s_client -connect www.google.com:443 -tls1_1

...

SSL-Session:
    Protocol  : TLSv1.1
```

# 特定プロトコルで通信させるない
```sh
[root@STG ~]# openssl s_client -connect www.google.com:443 -no_tls1_2

...

SSL-Session:
    Protocol  : TLSv1.1
```

# 网站の証明書をすべて表示

```sh

[root@STG ~]# openssl s_client -connect www.google.com:443 -showcerts

CONNECTED(00000003)
depth=3 C = BE, O = GlobalSign nv-sa, OU = Root CA, CN = GlobalSign Root CA
verify return:1
depth=2 C = US, O = Google Trust Services LLC, CN = GTS Root R1
verify return:1
depth=1 C = US, O = Google Trust Services LLC, CN = GTS CA 1C3
verify return:1
depth=0 CN = www.google.com
verify return:1
---
Certificate chain
 0 s:/CN=www.google.com
   i:/C=US/O=Google Trust Services LLC/CN=GTS CA 1C3
-----BEGIN CERTIFICATE-----
MIIFVDCCBDygAwIBAgIRAL8wc3Bd3+wGCgAAAAEQN5MwDQYJKoZIhvcNAQELBQAw

...

```



# 网站証明書ファイルの内容を確認

网站証明書 = 网站的CSRファイル（网站Public key + 网站电子署名） + 证书的签发CA机关

```
$ openssl x509 -text -noout -in server-B4019921.cer
```

![](img\2021-11-18-11-24-44.png)
![](img\2021-11-18-11-25-59.png)

# CA机关証明書ファイルの内容を確認

CA机关証明書 = CA机关Public key + CA机关电子署名 + 上级CA签发机关（直到CA Root）

```
$ openssl x509 -text -noout -in ca-pfwsr3ca.cer
```

![](img\2021-11-18-11-29-51.png)
![](img\2021-11-18-11-31-10.png)

# RSA秘密鍵ファイルの内容を確認

用来生成网站的CSRファイル而所需要的 private key + public key

```
$ openssl rsa -text -noout -in server.key
```

![](img\2021-11-18-11-33-36.png)
![](img\2021-11-18-11-34-10.png)

# CSRファイルの内容を確認

为了让CA机关认证而生成的请求书。

```
openssl req -text -noout -in 2021futaripassport.csr
```

![](img\2021-11-18-11-38-27.png)

# サーバに設定されている証明書（full chain）を確認

full chain = 

```sh
openssl s_client -connect www.futari-passport.metro.tokyo.lg.jp:443 -showcerts

```

![](img\2021-11-18-11-45-45.png)
![](img\2021-11-18-11-48-14.png)
![](img\2021-11-18-11-49-36.png)
![](img\2021-11-18-11-53-39.png)

由此可以看出，其实服务器fullchain证明书 = 网站的证明书 + 上级签发机关的CA证明书（直到CA root 证明书）

每一个证明书 = public key + 电子署名 + 上级签发机关