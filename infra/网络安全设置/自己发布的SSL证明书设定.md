# CA証明書の作成

## CAの秘密鍵の作成
RSA秘密鍵を作成する。
des3アルゴリズムを利用

```sh
$ openssl genrsa -des3 2048 > ca.key

Generating RSA private key, 2048 bit long modulus (2 primes)
..............+++++
.....................+++++

```

## 公開鍵の作成
秘密鍵から公開鍵を作成する

```sh
$ openssl rsa -in ca.key -pubout -out ca-public.key
writing RSA key

```

## 証明書署名要求(CSR / Certificate Signing Request)の作成

証明書署名要求は、認証局にサーバの公開鍵に電子署名してもらうよう要求するメッセージです。 
(本稿では自己証明書を作成する為、認証局も自分自身ということになります) 
OpenSSL で証明書署名要求を作成するには openssl req コマンドを実行します。

**秘密鍵**から証明書署名要求（CSR）を作成する。

```sh

$ openssl req -new -key ca.key -sha256 > ca.csr

You are about to be asked to enter information that will be incorporated
into your certificate request.
What you are about to enter is what is called a Distinguished Name or a DN.
There are quite a few fields but you can leave some blank
For some fields there will be a default value,
If you enter '.', the field will be left blank.
-----
Country Name (2 letter code) [AU]:jp
State or Province Name (full name) [Some-State]:Tokyo
Locality Name (eg, city) []:shinjuku
Organization Name (eg, company) [Internet Widgits Pty Ltd]:huang ltd.
Organizational Unit Name (eg, section) []:dev part
Common Name (e.g. server FQDN or YOUR name) []:stg.huang.com
Email Address []:

Please enter the following 'extra' attributes
to be sent with your certificate request
A challenge password []:
An optional company name []:

```

CSRの内容を確認する。

```sh
openssl req -text -noout -in ca.csr

```

## 証明書の作成（認証局による公開鍵への自己署名）
証明書署名要求に秘密鍵で署名する（自己署名）。

```sh
$ openssl x509 -req -in ca.csr -signkey ca.key -days 10000 -out ca.crt
Signature ok
subject=C = jp, ST = Tokyo, L = shinjuku, O = huang ltd., OU = dev part, CN = stg.huang.com
Getting Private key
```

# Server証明書の作成

## 秘密鍵の作成

```sh
$ openssl genrsa 2048 > server.key
Generating RSA private key, 2048 bit long modulus (2 primes)
..............................+++++
............+++++
e is 65537 (0x010001)
```

内容を確認する。

```
openssl rsa -text -noout -in server.key

```
## 公開鍵の作成

秘密鍵から公開鍵を作成する

```sh
$ openssl rsa -in server.key -pubout -out server-public.key
writing RSA key
```

## 証明書署名要求の作成

```sh
openssl req -new -key server.key -subj "/CN=servername" > server.csr
```

## Server証明書の作成（認証局による公開鍵への署名）(成果物)
証明書を作成する（CAの秘密鍵で証明書署名要求に署名する）。


```sh
$ openssl x509 -req -in server.csr -CA ca.crt -CAkey ca.key -CAcreateserial -days 3650 -out server.crt
Signature ok
subject=CN = jp
Getting CA Private Key
```

## Server証明書の簡易作成
最後に本稿の成果物であるサーバ証明書を作成します。 普通であれば上で作成した証明書署名要求 (server.csr) を VeriSign などの機関に送付して認証局の秘密鍵で署名してもらいますが、本稿では自分で署名することでサーバ証明書を作成しますので上で作成した自分の秘密鍵 (server.key) で署名します。 サーバ証明書に署名するには openssl x509 コマンドを実行します。 今回は証明書の有効期限が10年(3650日)ある証明書を作成します。

```sh
openssl x509 -req -days 3650 -signkey server.key < server.csr > server.crt
```

### 証明書と秘密鍵をまとめてp12形式で保管する。

```sh
$ openssl pkcs12 -export -inkey server.key -in server.crt -out server.p12
Enter Export Password: Rock$1518
Verifying - Enter Export Password:
```

### 秘密鍵にパスフレーズを付与（暗号化）する。

```sh
$ openssl rsa -in server.key -aes256 -out server-enc.key
writing RSA key
Enter PEM pass phrase:
Verifying - Enter PEM pass phrase:
```

# WEBサーバへの設定

以下二つは必要となっています。

- server.key(秘密鍵)
- Server証明書
