# 1 install Certbot

```sh
sudo yum install epel-release
sudo yum install certbot 
# aapcheの場合、install　python-certbot-apache、nginxの場合、インストールしなくてもいいです。
sudo yum install python-certbot-apache
```

# ssl証明書の取得

```sh
#証明書インストールのテスト(example.com部分は適宜変更する)
certbot certonly --webroot -w /home/nginx/publichtml -d example.com --email demo@gmail.com --dry-run

#問題無ければ証明書インストール
# http://example.com/.well-known/acme-challenge/(取得したトークン)
certbot certonly --webroot -w /home/nginx/publichtml -d example.com --email demo@gmail.com
```

1. 引数
    
- -webroot: Apache/Nginxを起動したままの認証とする。
- -w: 認証時の Documentroot を指定。
- -d: ドメインを指定。
- –preferred-challenges: 認証方式を指定する。ここではHTTP認証とする。
- –agree-tos: 利用規約の同意し、確認画面を表示しない。
- -m: Let’s Encryptに登録するメールアドレス。登録したくなければ、代わりに –register-unsafely-without-email を指定すればよい。

2. 上記コマンドを実行したら、/home/nginx/publichtmlフォルダに.well-known/acme-challenge/tokenファイルを自動的に生成した。

3. /etc/letsencrypt/live/example.comのフォルダに下記ファイルをダウンロードした。

- 証明書: cert.pem
- 秘密鍵: privkey.pem
- 中間証明書: chain.pem
- 証明書＋中間証明書: fullchain.pem

4. 証明書の内容は、opensslコマンドで確認できます。
```
openssl x509 -text -noout -in /etc/letsencrypt/live/example.com/cert.pem

notBefore=Aug 11 04:04:31 2021 GMT
notAfter=Nov  9 04:04:29 2021 GMT
```
Not Before（発行日）、Not After（有効期限）

# ssl証明書の更新

1. 証明書更新のコマンド
    ```sh
    sudo /usr/bin/certbot renew --post-hook "systemctl restart nginx"
    ```
    オプションについて補足します。
    - –post-hook: 証明書の更新後に実行するコマンドを指定する。更新が必要なときのみ（＝デフォルトでは有効期限まで30日未満のときのみ）実行される。一般的には、更新後の証明書を反映するための、Apache/Nginxを再起動するコマンドを指定する。
    - –pre-hook: 証明書の更新前に実行するコマンドを指定する。更新が必要なときのみ（＝デフォルトでは有効期限まで30日未満のときのみ）実行される。
    - –dry-run: テスト実行する（いわゆるドライラン）。実際には証明書は更新しない。- - –pre-hook, –post-hookで指定したコマンドは実行される。このオプションはcertonlyサブコマンドでも使用できる。
    - –force-renewal: 有効期限までの日数に関わらず、強制的に証明書を更新する。ただし、一定期間内に更新できる回数には制限がある。
    - 
2. Config
   これは、certonlyサブコマンドで証明書を取得したときに、ドメインごとのConfigファイル /etc/letsencrypt/renewal/<ドメイン>.conf に、取得実行時のオプションが保存されており、renewサブコマンドによる更新実行時は、そのConfigを参照するからです。
    ```
    # view /etc/letsencrypt/renewal/example.com.conf
    
    --
    # renew_before_expiry = 30 days
    version = 1.13.0
    archive_dir = /etc/letsencrypt/archive/example.com
    cert = /etc/letsencrypt/live/example.com/cert.pem
    privkey = /etc/letsencrypt/live/example.com/privkey.pem
    chain = /etc/letsencrypt/live/example.com/chain.pem
    fullchain = /etc/letsencrypt/live/example.com/fullchain.pem
    
    # Options used in the renewal process
    [renewalparams]
    account = xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
    pref_challs = http-01,
    authenticator = webroot
    webroot_path = /var/www/certbot/example.com,
    server = https://acme-v02.api.letsencrypt.org/directory
    [[webroot_map]]
    example.com = /var/www/certbot/example.com--

    ```
3. 自動更新スクリプト
    ```sh
    #!/bin/bash
    #
    CERTBOT_CMD=/usr/bin/certbot
    WEBSERVER_RESTART_CMD="systemctl restart ngixn"
    
    MAILTO=<メール通知先アドレス>
    
    echo "===== Update SSL Certfile ====="
    echo "`date` Update SSL Certfile start"
    
    # 証明書の更新
    ${CERTBOT_CMD} renew \
    --post-hook "${WEBSERVER_RESTART_CMD}"
    
    LE_STATUS=$?
    
    # 証明書の取得に失敗したときはメールで通知
    if [ "$LE_STATUS" != 0 ]; then
        echo "Update SSL Certfile failed" |\
        mail -s "Update SSL Certfile in `hostname`" ${MAILTO}
    fi
    
    echo "`date` Update SSL Certfile end"

    ```
    実行権限を付与します。

    ```sh
    chmod 755 /root/bin/update_sslcert.sh
    ```

4. cronに登録する

    ```sh
    crontab -e
    # 下記一行を追加する
    #  3か月ごと、3日の朝7時半に実行しようとした場合、以下のようになります。
    00 04 08 */3 * /home/user1/batch/update_sslcert.sh 1>> /var/log/update_sslcert.log 2>&1
    ```

    ```sh
    crontab -e

    # 下記一行を追加する
    #  3か月ごと、3日の朝7時半に実行しようとした場合、以下のようになります。
    30 07 03 */3 * /usr/bin/certbot renew --post-hook "systemctl restart nginx" > /var/log/certbot_log 2>&1

    ```





# letsencrypt 証明書手で取得流れ
https://sslnow.ml/
